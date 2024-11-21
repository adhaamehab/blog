---
title: "Testing strategies for Data Systems with testcontainers"
date: 2023-12-21
draft: false
description: "An opininated rant about unit testing and e2e testing when building data systems."
summary: "Using testcontainers to gain confidence in any data system"
toc: true
readTime: true
autonumber: true
tags: ['testing', 'data-systems']
math: true
showTags: true
---
# Testing Data Systems: Beyond Unit Tests

## Introduction

After spending years building and maintaining data systems at scale, I feel like I've finally found the missing piece in testing data systems. While unit tests are great and provide a baseline of confidence, I never truly believed that they provide real confidence in the codebase.

Over the years, I've tried different testing strategies, and the trade-off is always between the time and effort it takes to build, maintain and run a good testing system versus release velocity. And mind you, I usually end up preferring speed over quality (aren't we all?)

Lately, we started using testcontainers as part of our CI. And I would never look back. 

### Problem

My biggest problem with having a real testing pipeline was the amount of effort it takes to set up. Some might argue that it's definitely worth it and that "any piece that's not tested is broken." But in the real world, we usually cut corners around that and resort to just unit testing, fully knowing it will miss a few bugs that can be catastrophic. 

But over the last year, I got the chance to use testcontainers in what I think is a very effective way. I think I wouldn't change this workflow for a while, and I would apply it to all my projects if I had the time.

## The Limits of Unit Testing

Don't get me wrong - I'm a strong advocate for unit tests. I aim for north of 60% coverage in all my projects, and I want them running on every PR. They're great at catching basic issues, testing edge cases, and providing quick feedback during development.

But here's the thing: data systems are complex beasts. They involve multiple moving parts, different storage systems, message queues, and complex state management. Unit tests simply can't capture all the ways these components interact in production.

Think about it - you can unit test your Kafka consumer logic perfectly, but what happens when message ordering gets weird? What about when your Postgres connection drops mid-transaction? These scenarios are impossible to simulate effectively with unit tests alone.

## The Power of Integration & E2E Testing

This is where integration and E2E testing with tools like testcontainers.com really shine. Instead of mocking everything, you get to test against real systems. It's as close as you can get to production without actually being in production and without having to build an entire infrastructure to run tests.

Let me walk you through how I typically structure this for a data pipeline. Let's take a common example: an ETL system moving data from Postgres through Kafka into S3.

### Real World Example

For this pipeline, I set up three main test suites:

First, I test the source system (Postgres):
```python
def test_postgres_extraction():
    with TestContainer('postgres:latest') as postgres:
        # Run latest migrations
        # Seed real production-like data
        # Execute extraction logic
        # Validate results match expectations
```

Then, the kafka pipeline:
```python
def test_kafka_transformation():
    with TestContainer('kafka:latest') as kafka:
        # Set up topics with production configs
        # Process sample dataset (ideally output of the last "passing" test)
        # Run transformations
        # Verify output schemas and content
```

Finally, the target system:
```python
def test_s3_loading():
    with MockS3Client() as s3:
        # Process transformed data (same data we verified in the last test)
        # Verify final state
        # Check the final bucket state
```

## The Case for Real Data

Here's something controversial: I strongly prefer using real production data in tests over synthetic data. Why? Because real data is messy. It has edge cases you'd never think to include in your synthetic datasets.

I've seen too many data pipelines fail in production because they were only tested against perfectly formatted synthetic data. Real world data is ugly - it has nulls where you don't expect them, weird Unicode characters, and values that push the boundaries of your assumptions. And I would rather have my testing pipeline catch those during development rather than get an alert at 2 AM.

## Validation Over Exact Matching

Another thing I would suggest: avoid exact matching in your tests. Instead, write validation rules that check for data properties and relationships. For example, instead of:

```python
assert result == expected_output
```

I prefer:

```python
assert all(validate_record(r) for r in results)
assert score > 0.0 and score < 1.0
assert result_count == expected_count
assert all(r.timestamp >= start_time for r in results)
```

This approach is more resilient to non-material changes and better reflects what you actually care about - the integrity and correctness of your data.

## CI/CD Integration

I run these tests in GitHub Actions, typically as a pre-merge check for critical paths and definitely before any release. Yes, they're slower than unit tests. Yes, they use more resources. But the confidence they provide is worth it.

The key is to be smart about when you run them. Not every commit needs to run the full suite. I usually have them trigger on:
- Pull requests to main/release branches
- Release to an env (dev/prod)
- Version tags

## The Payoff

Every project where I have this system set up, I've seen people (me included) very keen to contribute to it. Once you get the testing pipeline to a good place, it gives you a very clean development workflow. Specifically with GitHub Copilot being used heavily these days, it provides a great level of maturity and confidence.
