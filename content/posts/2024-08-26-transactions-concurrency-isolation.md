---
title: "Notes on Database ( Part 2 ) - Transactions, Concurrency and Isolation Levels"
date: 2024-08-25
draft: false
description: "Part two of the Postgres Database series. This post covers Transactions, ACID properties and Concurrency Control."
summary: "Summary of Postgres Transactions, ACID properties and Concurrency Control"
toc: true
readTime: true
autonumber: true
tags: ['databases', 'postgres', 'transactions', 'concurrency']
math: true
showTags: true
---

# Introduction

In the [previous post](/posts/2024-08-01-databases-1), we discussed how to study databases and the resources to use. In this post, we will dive into the core of databases: **Transactions**.

The reason I'm starting with transactions because they are (in my opinion) the most important concept in databases. When you interact with database, you don't need to understand the ins and outs of that database. Most people don't anyway. But in order to write a software that have a predictable behavior and is efficient, you need to understand what the database provides you and what you should expect from it.

This post will cover how transactions work (from the user perspective), What are the ACID properties and why they matter.

And then we will cover Isolation Levels and Concurrency Control, I think this has to be hand-to-hand with transactions in order to understand how they work and build a full example.

# The basics

## What's a Database?

Before I dive into transactions, I wanted to define what's a database on a very high-level. While an actual database system (like Postgres) is a very complex software. The very simple of view of a database would be a service that allows you to store, retrieve and iterate over data. For instance, if you store a bunch of CSV files on desk and use Python to read and write them, that's a database system.

But if you take this analogy and try to understand it in a more technical way, you will find that you have a few components:

1) Server (aka. Database Server): This will be the front-end component of our system. It's responsible for all user's interactions.
2) File System (aka. Storage Engine): This is the piece that handles how we read and write data on disk.
3) Query Processor: This is the piece that connects the server to the file system. It's responsible for translating user's query (with all of its type). And run the appropriate operation on data on disk.

This makes up a very simple yet complete database system. Right? Well, theoretically yes, but the actual implementation is far more complex than this. But we don't need to understand all of the implementation details in order to be a power user of a database system. This analogy I think is enough to get us started.

## What's a Query? 

So we established that we have data on disk, we have a front-end API to interact with the data and we have a way to translate that interactions to actual operations and algorithms. And this interactions are what we are focusing on. 

**Database Queries** defines how we should interact with the database. And it's where you define the database behavior in order to build a predictable software.

Queries varies in types, and there are lots of query languages out there. But since we are focusing on Postgres, anywhere in this series where we mention queries, we are talking about *SQL queries*.

### Queries Types

Since queries are used to describe what we need from the database and how the database should behave, we are specially interested in two types of queries:

1) **READ Queries**: These queries allow us to read specific data from the database. We can read data as is. Or we can do some operations and transformations on the data before we read it in order to get it in a specific shape.
2) **WRITE Queries**: These queries allow us to add new data to the database. We can add a new row to a table, or we can update an existing row.

The interesting (and sometimes tricky) part is when you have multiple queries happening at the same time. From different and similar types but touching the same row(s). Understanding these basic blocks of queries and transactions helps us grasp why transactions are so important. When you have multiple users or applications all trying to read, write, update, or delete data at the same time, you need a way to keep everything organized and consistent. And depending on your application's scope, this can have major implications.

## Transactions

From a very high-level, a **transaction** is a way of grouping together a set of queries so they could be executed together. This simple definition abstracts away all the details (which we will try to uncover) from the database user.

### A single block of work

Everytime you read about transactions, people explain the bank transfer problem. So I'm sticking with this!
Imagine you're building a banking app, and you need to transfer $100 from Ronaldo's account to Messi's. Sounds simple enough, right? But this is actually where things gets a bit tricky.

When you think about it, it's a simple process. Deduct $100 from Ronaldo's account and add $100 to Messi's. But then you realize, it's never that simple, right?

Computer goes down, network goes down, someone might unplug the internet cable while the transfer is in progress. There's infinite possibilities of things going wrong. This means, we need an efficient way to provide us with some sense of safety. 

Transactions is what does this. When you think about this operation as a transaction, the worst case scenario is partial failure. Meaning, we deduct $100 from Ronaldo but the money never reached Messi. With a simple transaction, the database guarantees us that both operations will be executed as a single block of work. So no matter what happens, both operations will be executed or none of them will.

This feature of transactions is what people refer to as **Atomicity**.

### A consistent and permanent state

And since we talked about things going wrong. We still need a predictable guarantee that if our transaction succeeds, the changes will reflect. Whether this happens in sub-millisecond or hours, we need that to be predictable and permanent.

This means that the database should provide us with a way to configure our transaction to ensure changes are permanent and consistent.

### Durable

This brings us to another important aspect of transactions - durability. When a transaction completes successfully, we need to be confident that those changes are permanent, even if our database crashes right after. It's like saving an important document - you want to be sure your work is actually saved, not just floating somewhere in temporary memory.

In our banking example, once we get confirmation that the money transfer went through, we need that change to stick. Both Ronaldo and Messi should see their new account balances even if the database server crashes and restarts right after the transfer.

# Understanding ACID Properties

You've probably heard the term "ACID" thrown around when people talk about database transactions. But the problem with this topic is that most people think they only need to memorize these properties as abstract concepts (mostly to answer questions in an interview).
But when we talk about database transactions, ACID properties are the backbone of what makes them reliable and predictable. And this was the core debate a few years ago between the SQL and NoSQL communities. Since if you don't have strong ACID properties, you don't really know what you will get when you query your database. (Though in the recent years, NoSQL databases improved in this area a lot.)

## Atomicity (All or Nothing)

Atomicity might sound like a complex term, but it's actually a very simple (and powerful) concept. Think of it as an "all or nothing" guarantee. When you mark a set of operations as a transaction, the database promises that _either all of them will succeed, or none of them will happen at all_.

Back to our banking example. Imagine you're building a financial app that handles investment portfolios. When a user wants to rebalance their portfolio, they might need to:

- Sell some stocks
- Wait for the sale to clear
- Buy different stocks with the proceeds
- Update their portfolio record

Without atomicity, you could end up in situations where:

- The stocks are sold but the new purchase fails
- The purchase succeeds but the portfolio record isn't updated
- Some operations succeed while others fail, leaving the system in an inconsistent state


  ![img](https://github.com/user-attachments/assets/206a34da-302f-425c-8c87-14a5e524add6)


Atomicity prevents these _partial failures_. The entire rebalancing operation either completes successfully or rolls back entirely. If anything goes wrong at any step, all changes are undone automatically. This is why we often say transactions are "rolled back" - it's like rewinding a tape to the beginning if something goes wrong. (Think a time machine)

## Consistency (Following the Rules)

Consistency ensures that a transaction can only bring the database from one valid state to another valid state. It's like having a set of rules that must always be followed.

While consistency is a very large topic on its own, and some people argue that it's only part of the *ACID* transaction to complete the acronym. But Generally speaking, Consistency is set of rules that ensure that every transaction must take your database from one valid state to another valid state. 

Consistency ensures that no transaction can break the DB rules, even if multiple transactions are running at the same time. If any rule would be violated, the entire transaction fails and rolls back (because of the atomicity property)

For example, if we have a rule that account balances can't go negative, a transaction that would make a user's balance negative should fail completely. The database won't let us break our own rules.

![img](https://github.com/user-attachments/assets/eaa01cb2-7572-4a43-ad3e-04e8f89a2e89)


## Isolation (Working Independently)

Isolation is probably the trickiest ACID property to understand. It determines how transactions deal with each other when they're running at the same time.

Imagine you and your friend are both trying to book the last seat on a flight at exactly the same time. Isolation ensures that only one of you can get that seat - the other person's transaction will have to fail or wait.

Different isolation levels provide different guarantees about what concurrent transactions can see. We will touch more on this below.

![img](https://github.com/user-attachments/assets/4fe6542a-80ee-488c-b5d9-0f921d47716f)


## Durability (Changes that Stick)

We covered this briefly earlier - durability means that once a transaction is committed, it stays committed. Your changes are permanent and survive any subsequent system or power failures.

It doesn't just mean the data is saved - it means it's saved in a way that can survive various types of failures and can be recovered correctly when the system restarts (We will cover disaster recovery in a later post).

# Isolation & Concurrency Control 

Now that we understand what transactions are and what guarantees they provide, let's talk about how databases handle multiple transactions happening at the same time.

## Why Concurrency Control Matters
Assume you're building a realtime data pipeline that handles user activity data from multiple sources. Your system needs to:

- Ingest events from various services.
- Process and aggregate metrics.
- Generate reports.

Without concurrency control, you will most likely hit a lot of data race problems, this can cause:

- Inconsistent aggregations because metrics are being updated while being read
- Lost updates when multiple processes try to modify the same user profile
- Incorrect reports because they're running on data being modified

This is why databases implement sophisticated concurrency control mechanisms. They ensure that concurrent operations work together in a predictable way that allows us to maintain data consistency.


## The Building Blocks: Locks
At the heart of concurrency control are locks. Think of locks like access controls in a secure facility - they manage who can access what resources and when. In databases, we have different types of locks:

### 1. Shared Locks (Read Locks)
These are like multiple analysts reading from the same dataset. Multiple transactions can hold shared locks on the same data simultaneously. For example, when generating different reports from the same dataset, multiple processes can read simultaneously without interference.

### 2. Exclusive Locks (Write Locks)
These are like a data cleanup job that needs exclusive access to fix inconsistencies. Nobody else should be modifying or even reading the data until the cleanup is complete. When a process needs to modify data, it requires an exclusive lock.

## Understanding Isolation Levels

As mentioned before, Isolation is probably the most subtle and complex of the ACID properties. It determines how changes made by one transaction affect other concurrent transactions.

### 1. Read Committed
This is Postgres's default isolation level, providing a practical balance between consistency and performance.
Under Read Committed:
```sql
-- Process 1: Calculating hourly metrics
BEGIN;
SELECT SUM(events) FROM metrics WHERE hour = '2024-02-01 14:00';  -- Sees 1000 events
-- Meanwhile, Process 2 backfills some missing events
SELECT SUM(events) FROM metrics WHERE hour = '2024-02-01 14:00';  -- Now sees 1200 events
COMMIT;
```
This is usually acceptable for metrics and analytics where small inconsistencies are tolerable.

### 2. Repeatable Read
Repeatable Read is particularly useful for generating consistent reports or doing complex data analysis. It ensures that if you read something once, you'll see the same data again within the same transaction.
```sql
-- Process 1: Generating daily report
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SELECT COUNT(*) FROM user_profiles;  -- Sees 10000 users
-- Meanwhile, new users are added
SELECT COUNT(*) FROM user_profiles;  -- Still sees 10000 users
COMMIT;
```
This is valuable when you need consistency across multiple reads, like:

Generating end-of-day reports
Running complex analytical queries
Performing data validation checks

### 3. Serializable
This strongest isolation level ensures that concurrent transactions behave as if they ran sequentially. It's crucial for:

Critical data updates where consistency is paramount
Complex workflows with interdependent data
Data migration or transformation processes

For example, when updating user hierarchies or permission structures, you might need serializable isolation to ensure the integrity of your access control system.

### Practical Applications
Consider these scenarios in a data system:

#### Event Processing

```sql
-- Process raw events into aggregated metrics
BEGIN;
INSERT INTO raw_events SELECT * FROM event_queue;
UPDATE metrics 
SET count = count + (SELECT COUNT(*) FROM raw_events WHERE processed = false);
UPDATE raw_events SET processed = true;
COMMIT;
```

#### User Profile Updates
```sql
-- Update user preferences while maintaining consistency
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SELECT * FROM user_profiles WHERE user_id = 123 FOR UPDATE;
UPDATE user_preferences SET settings = new_settings WHERE user_id = 123;
UPDATE user_audit_log SET last_modified = CURRENT_TIMESTAMP WHERE user_id = 123;
COMMIT;
```

Data Cleanup Jobs

```sql
-- Remove stale data while maintaining referential integrity
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
DELETE FROM expired_sessions WHERE last_access < NOW() - INTERVAL '30 days';
UPDATE user_metrics SET active_sessions = 
  (SELECT COUNT(*) FROM sessions WHERE user_id = user_metrics.user_id);
COMMIT;
```



## Summary

Understanding transactions is crucial for building reliable applications that use databases. They give us important guarantees through ACID properties:
- Atomicity ensures our operations succeed or fail as a unit
- Consistency keeps our data valid
- Isolation helps manage concurrent access
- Durability makes our changes permanent

The key is choosing the right isolation level for your needs - balancing between consistency and performance. For most applications, the default Read Committed level works well, but some situations might require stronger guarantees.

In the next post, we'll dive deeper into MVCC in Postgres.
