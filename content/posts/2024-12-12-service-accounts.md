---
title: "Cloud Native Service Accounts"
date: 2024-12-14
draft: false
description: "Understanding how what service accounts are and how they work in cloud native environments"
summary: "Understanding how what service accounts are and how they work in cloud native environments"
toc: true
readTime: true
autonumber: true
tags: ['kubernetes', 'cloud']
math: true
showTags: true
---

Last week, I had a talk with one of my mentees and we touched on some authentication concepts. And I realized that while the idea of service accounts might be straightforward to some, it usually takes some time before you grasp how they actually work.

## Intro
You are starting your day and you just typed in your username and password for your cloud console (or maybe through your company SSO). And now based on your permission, you can do certain actions. So you go for the buckets service, and you can view some bucket and you try to delete a bucket and...
shCopyError deleting bucket...Permission Denied!
The reason you're seeing this is that your cloud admin gave you a specific permission that allows you to view and edit buckets, but never delete it. Obvious, right?
Service accounts aren't much different. You create "accounts" for each service with certain permissions.

## Identity and Access Management (IAM)
Before diving deeper into service accounts, let's understand IAM - the system that makes all of this possible. IAM is like a security guard at a building who checks IDs and knows exactly what areas each person can access.
In cloud environments, IAM serves two main purposes:

- **Authentication**: Verifying who you are (or which service is making the request)
- **Authorization**: Determining what you're allowed to do

When you logged into your cloud console, IAM verified your identity and checked what permissions you have. These permissions are usually organized into roles - collections of permissions that make sense together. For example, a "Storage Viewer" role might include permissions to list and read buckets, but not modify them.

<img width="667" alt="iam workflow" src="https://github.com/user-attachments/assets/d9857075-caa5-45ff-88e6-0423cfaa16c5" />


---
## Service Accounts

### Cloud Service Accounts

Now you have to write up a new script that will read from the bucket and write it to your local disk. But you realize that you needed to login with your credentials in order to view the bucket.
Here's where service accounts are needed. You need to grant a specific service certain permissions in order for the service to execute the task.
So when you use the service account to authenticate your service, it will go through a very similar workflow like the one you do when you login. It will use the service account provided to talk first to the IAM Auth services to pull credentials and permissions. Then it will use those to talk to the cloud APIs (in our example, it will read and download data files from the bucket services).

The beauty of service accounts in cloud environments is that they're designed for programmatic access. Instead of username/password combinations, they use keys and tokens that can be easily rotated and managed programmatically. This allows you to have a fine-grained control over what your service can do. And you can easily rotate the credentials when they expire.

<img width="667" alt="cloud-sa" src="https://github.com/user-attachments/assets/5903f7de-040c-4045-aed1-8c6594b2e4c4" />

And all cloud providers have a predefined set of roles that you can use for almost all scenarios. From experience, the less "custom" roles you have, the better and cleaner access you have.
Most likely, you will need to assign multiple roles to a service account. But this is better than having a single role with a lot of permissions.

### Kubernetes Service Accounts
Inside Kubernetes, service accounts work similarly but with some key differences in implementation. When a pod starts up in your cluster, it automatically gets assigned a default service account unless you specify one. This service account determines what API operations that pod can perform against the Kubernetes API server.
For example, if you have a monitoring pod that needs to list all pods across namespaces to collect metrics, it would need a service account with permissions to perform those list operations. The workflow looks like this:

Pod starts up and receives the service account token through an automatically mounted volume
When the pod needs to talk to the API server, it uses this token for authentication
The API server validates the token and checks the service account's permissions
Based on the permissions, the request either succeeds or gets denied

What makes Kubernetes service accounts special is how tightly they're integrated with the cluster's role-based access control (RBAC) system. You define what a service account can do by binding it to roles or cluster roles.

<img width="1047" alt="image" src="https://github.com/user-attachments/assets/9d14b85e-9614-48f6-9e7f-f4ec846d82f0" />


## The Bridge: Workload Identity
Now here's where things get interesting - what happens when your pod running in Kubernetes needs to talk to cloud services? You have two different authentication systems that need to work together:

- **Kubernetes service accounts**: for in-cluster authentication
- **Cloud service accounts**: for cloud API authentication

This is where workload identity comes in. It creates a trust relationship between your Kubernetes service accounts and cloud service accounts. Instead of storing cloud credentials in your cluster (which can be a security risk), workload identity allows pods to acquire cloud credentials dynamically based on their Kubernetes service account.
Here's how it works:

- You configure a mapping between a Kubernetes service account and a cloud service account
- When a pod starts up with that Kubernetes service account, it receives a special token
- The pod can exchange this token for temporary cloud credentials
- The pod can now make authenticated calls to cloud APIs using these credentials

<img width="1295" alt="image" src="https://github.com/user-attachments/assets/042d75e4-de91-4441-b72b-9d4194534ec6" />


This approach has several benefits:

- No need to manage cloud credentials manually (which is a very big risk)
- Fine-grained control over which pods can access what cloud resources
- Clear audit logs of which workloads are accessing cloud services

## Best Practices
Now, to the most common question I always get. What's the best practice for structuring the service account when you have a microserives system that does a lot of cloud interactions?
-- The answer is to use the **Principle of least privilege**: Give service accounts only the permissions they absolutely need to function. And use Policies and RBAC to abstract and assign required access for service accounts.

## Final Thoughts
Service accounts are a fundamental building block of cloud native security. Whether you're working with cloud providers directly or through Kubernetes, understanding how service accounts work helps you build more efficient systems.
With cloud platforms, it's now much more straightforward to build secure systems. Using tools like IAM, Service Accounts and Workload Identity, you can build a zero-trust system with very little effort or security knowledge.
