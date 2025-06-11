---
title: "Retrieval Technique Series-5.How Large-Scale Search Systems Accelerate Retrieval with Distributed Technology"
meta_title: ""
description: ""
date: 2025-06-11T00:00:00Z
image: ""
categories: ["Database", "Retrieval"]
author: "yaruyng"
tags: ["Database", "Retrieval"]
draft: false
---
Index Construction
<!--more-->

In the era of big data, search systems must handle massive volumes of information and user queries efficiently. Traditional single-server architectures quickly become bottlenecks as data and traffic grow. To address this, large-scale search systems leverage distributed technology and index sharding to accelerate retrieval and ensure scalability. In this post, we’ll explore how these techniques work, their advantages, and the challenges they bring.

## What are the advantages of distributed technology?
Distributed technology is the decomposition of large tasks into multiple subtasks, using multiple servers to jointly undertake tasks, which greatly improves the overall system's service capabilities compared to single machine systems.

## What does a simple distributed structure look like? 
A complete distributed system will have complex service management mechanisms, including service registration, service discovery, load balancing, traffic control, remote calling, and redundant backup. Here, let's first set aside the implementation details of distributed systems and return to their essence, which is to start with "letting multiple servers share the task" and see how a simple distributed retrieval system works. Firstly, we need a server that receives requests, but it does not perform specific query tasks. It is only responsible for task distribution, which we call a distribution server. Multiple index servers are the ones that actually perform the retrieval task, each of which stores a complete inverted index and is capable of completing the retrieval task. When the distribution server receives a request, it will send the current query request to a relatively idle index server for querying based on the load balancing mechanism. The specific retrieval work is independently completed by the indexing server and the results are returned.
```text
                +----------------------+
                |   Dispatcher Server  |        The dispatcher server receives the request and forwards it to a specific index server, 
                +----------------------+          according to the load balancing mechanism.
                        |
             -------------------------------------------------------
             |                      |                              |
+---------------------+  +---------------------+   ...   +---------------------+
|  Full Index Data    |  |  Full Index Data    |         |  Full Index Data    |
|  Index Server 1     |  |  Index Server 2     |         |  Index Server n     |
+---------------------+  +---------------------+         +---------------------+

The index server processes the request and returns the search result.

```

## What is Index Sharding?
Index sharding is the process of splitting a large search index into smaller, manageable pieces (shards) that can be distributed across multiple servers. Each shard contains a subset of the data, allowing the system to process queries in parallel and balance the load.

## How Index Sharding Works
There are two main strategies for sharding:

### 1. Document-Based Sharding
Each shard contains a subset of documents. 
```text
+---------------------------------------------------------------+
|                        All Documents                          |
|  +-------------------+      ...      +-------------------+    |
|  | Document Set 1    |               | Document Set n    |    |
|  +-------------------+               +-------------------+    |
+---------------------------------------------------------------+
           |                                      |
           v                                      v
   Generate Index Shard 1                 Generate Index Shard n
           |                                      |
           v                                      v
+--------------------------------+   +--------------------------------+
| word 1 ->doc 1->doc 2...doc 19 |   |word 1 ->doc 23->doc 25...doc 41|
| word 2 ->doc 3->doc 4...doc 14 |   |word 2 ->doc 12->doc 16...doc 30|
|   ...                          |   |    ...                         |
| word n ->doc 1->doc 3...doc 15 |   |word n ->doc 21->doc 24...doc 35|
+--------------------------------+   +--------------------------------+

Note: The posting list in a single shard is incomplete.

```
When a query arrives:
```text
                        +----------------------+
                        |  Dispatcher Server   |
                        +----------------------+
                                 |
        -------------------------------------------------
        |                       |                      |
+-------------------+  +-------------------+   ...   +-------------------+
|  Index Shard 1    |  |  Index Shard 2    |         |  Index Shard 3    |
|  Index Server 1   |  |  Index Server 2   |         |  Index Server n   |
+-------------------+  +-------------------+         +-------------------+

1. The dispatcher server receives the query request and sends the request to all index servers with different index shards.

2. Each index server searches its own loaded index shard and returns the search results to the dispatcher server.

3. The dispatcher server merges all returned results and returns the final result.

```
**Advantages:**

- Accelerates search efficiency.
- Evenly distributes queries and balances server load.
- Makes it easier to update the index by adding or modifying documents.
**Management Challenges:**

- Requires careful balancing of query loads across shards.

### 2. Keyword-Based Sharding
Each shard is responsible for a subset of keywords. 
```text
+-------------------+
|   All Documents   |
+-------------------+
          |
          v
   Generate Complete Inverted Index
          |
          v

 Incomplete |
 dictionary | Complete posting list in shard
 in shard   |                 
+--------------------------------------------+
| word 1  -----> doc 1 -> doc 2 ... doc 41   |
| word 2  -----> doc 3 -> doc 4 ... doc 30   |
|   ...                                      |
| word 20 -----> doc 1 -> doc 3 ... doc 35   |
+--------------------------------------------+
+--------------------------------------------+
| word 41 -----> doc 1 -> doc 5 ... doc 41   |
| word 42 -----> doc 2 -> doc 6 ... doc 30   |
|   ...                                      |
| word n  -----> doc 3 -> doc 4 ... doc 35   |
+--------------------------------------------+

```
When a query arrives:
```text
Query: key1 + key2

                        +----------------------+
                        |  Dispatcher Server   |
                        +----------------------+
                          /               \
                         /                 \
        +---------------------+   +---------------------+   ...   +---------------------+
        |  Full Index Data    |   |  Full Index Data    |         |  Full Index Data    |
        |  Index Server 1     |   |  Index Server 2     |         |  Index Server n     |
        +---------------------+   +---------------------+         +---------------------+

 (Return posting list for key1)   (Return posting list for key2)

1. The dispatcher server receives the request and, based on the query, dispatches it to one or more index servers.
2. Index servers process the request and return the complete search results.
3. The dispatcher server merges the results and returns the final result.

```
**Advantages:**

- Reduces duplicate computation.
**Management Challenges:**

- If queries contain many keywords not evenly distributed, performance may drop.
- High-frequency keywords can overload specific shards.


## Conclusion
Index sharding and distributed technology are essential for building scalable, high-performance search systems. By splitting the index and distributing the workload, these systems can handle massive data volumes and high query rates efficiently. However, careful planning and management are required to avoid bottlenecks and ensure balanced performance.
