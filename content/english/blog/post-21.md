---
title: "Retrieval Technique Series-4.How Search Engines Generate Indexes for Trillions of Websites?"
meta_title: ""
description: ""
date: 2025-06-06T00:00:00Z
image: ""
categories: ["Database", "Retrieval"]
author: "yaruyng"
tags: ["Database", "Retrieval"]
draft: false
---
Index Construction
<!--more-->

# How to Generate Inverted Indexes Larger Than Memory Capacity

## Review of In-Memory Inverted Index Generation

For small document collections that fit in memory, we generate hash-based inverted indexes as follows:

1. Assign unique, sequential IDs to each document
2. Scan each document sequentially, generating <keyword, document ID, keyword position> tuples
3. Store these tuples in an inverted table with the keyword as the key (position information can be omitted if unnecessary)
4. Repeat until all documents are processed

This creates an in-memory inverted index.
```text
[Document Analysis Process]

+------------+                +---------------------------------------------+
| word 1     |                | Keywords | DocID  | Position | Posting List |
| word 2     |    Analyze     +----------+--------+----------+--------------+
| word 1     | ------------>  | Word 1   | Doc 2  | [1,3]    |              |
| doc 2      |                | Word 2   | Doc 2  | [2]      |              |
+------------+                +---------------------------------------------+
                                    |
                                    |
                                    v
If key exists in posting list, directly insert node:
word 1 -------> doc 1 -------> doc 2

If key doesn't exist in index, insert key and create posting list:
word 2 -------> doc 2

```

## Handling Large-Scale Document Collections

For large-scale document collections, we need a different approach. Can we split them into smaller collections to build inverted indexes in memory? How do we combine these smaller indexes into a complete large-scale inverted index stored on disk?

Industrial-grade inverted indexes are more complex than what we've studied. For example, if a document contains "Geek Time" , not only might these four characters be added to the dictionary as keywords, but "Geek" , "Time" , and "Geek Time"  might also be added. This results in a very large dictionary, potentially containing millions or tens of millions of terms.

Since words have different lengths and storage requirements, we assign a number to each word in the dictionary and store the corresponding string. In the posting list, we record not just document IDs but also position information, frequency, and other details. Each node in the posting list is a complex structure identified by document ID.
```text
Dictionary                          Posting List
+--------------+      +-----------------------------------------------+
| word ID 1    | ---> | [doc 1, pos, tf,...] -> [doc 2, pos, tf,...]   -> ... -> [doc 19, pos, tf,...] |
| string       |      +-----------------------------------------------+
+--------------+
| word ID 2    |      +-----------------------------------------------+
| string       | ---> | [doc 19, pos, tf,...] -> [doc 21, pos, tf,...]  -> ... -> [doc 38, pos, tf,...] |
+--------------+      +-----------------------------------------------+
       :
       :
| word ID n    |      +-----------------------------------------------+
| string       | ---> | [doc 7, pos, tf,...] -> [doc 11, pos, tf,...]  -> ... -> [doc 43, pos, tf,...] |
+--------------+      +-----------------------------------------------+

```

## Generating Industrial-Grade Inverted Indexes

Here's how we generate a large-scale inverted index:

1. Divide large document collections into multiple smaller sets
2. Generate in-memory inverted indexes for each small document set
3. Store these in-memory indexes on disk as temporary inverted files:
  - Sort the document lists by keyword string size
  - Write each keyword and its corresponding document list as a record to the temporary file
  - Records in the temporary file are ordered, and we don't need to store keyword IDs (as they're only locally unique)
```text
Index Table (In Memory)                                                                                  Temporary Files (Disk)
                                             
+----------------+                                                                                    +---------------------------+
| word ID 1      |                                                                                    | string 1 | posting list 1 |
| string         |---> doc 1 --> doc 2 --> ... --> doc 19                                             +---------------------------+
+----------------+                                                                                    | string 2 | posting list 2 |
                                                                                                      +---------------------------+
+----------------+                                                                                    | string 3 | posting list 3 |
| word ID 2      |                                                  Write to temporary files          +---------------------------+
| string         |---> doc 19 --> doc 20 --> ... --> doc 34         by key value order                |            ...            |
+----------------+                                                ----------------->                  +---------------------------+
                                                                                                      | string n | posting list n |
        ...                                                                                           +---------------------------+                                            
+----------------+                                             
| word ID n      |                                             
| string         |---> doc 1 --> doc 20 --> ... --> doc 53     
+----------------+

```
4. Process each batch of small document collections to generate corresponding temporary files
5. Merge multiple temporary files using multi-way merge:
  - Extract the current record's keyword from each temporary file
  - For identical keywords, read out and merge the corresponding posting lists
  - If the posting list fits in memory, merge it there and write the result to the final inverted file
  - If the posting list is too large, process it in segments
  - Assign a globally unique ID to each keyword after merging
```text
Temporary File 1               Temporary File 2           Temporary File 3
+-----------------------+  +-----------------------+  +-----------------------+
| string 1|posting list |  | string 1|posting list |  | string 3|posting list |
+-----------------------+  +-----------------------+  +-----------------------+
| string 2|posting list |  | string 3|posting list |  | string 4|posting list |
+-----------------------+  +-----------------------+  +-----------------------+
| string 3|posting list |  | string 4|posting list |  | string 5|posting list |
+-----------------------+  +-----------------------+  +-----------------------+
|          ...          |  |           ...         |  |           ...         |
+-----------------------+  +-----------------------+  +-----------------------+
| string n|posting list |  | string i|posting list |  | string j|posting list |
+-----------------------+  +-----------------------+  +-----------------------+
        |                         |                         |
        +-------------------------+-------------------------+
                                  |
                                  v
                     +-------------------------------------+
                     | word ID 1 | string 1 | posting list |
                     +-------------------------------------+
                     | word ID 2 | string 2 | posting list |
                     +-------------------------------------+
                     | word ID 3 | string 3 | posting list |
                     +-------------------------------------+
                     |                  ...                |
                     +-------------------------------------+
                     | word ID n | string n | posting list |
                     +-------------------------------------+
                            Complete Sorted File

```

This approach is similar to the Map-Reduce distributed computing paradigm, making it easy to implement on multiple machines to significantly improve efficiency.

## Using Disk-Based Inverted Files for Retrieval

When using large-scale inverted files for retrieval, the core principle is to load as much data as possible into memory since memory retrieval is much faster than disk access.

An inverted index consists of two parts: the dictionary (key collection) and the document lists. In many applications, the dictionary is small enough to load into memory using a hash table.
```text
Hash Table (In Memory)                 |        Inverted Index File (Disk)
                                       |
+----------------+                     |        +----------------------------+
| word ID 1      |                     |        | word ID 1 | posting list 1 |
| string         |---> pos ------------|------> +----------------------------+
+----------------+                     |        | word ID 2 | posting list 2 |
                                       |        +----------------------------+
+----------------+                     |        | word ID 3 | posting list 3 |
| word ID 2      |                     |        +----------------------------+
| string         |---> pos ------------|------> |              ...           |
+----------------+                     |        +----------------------------+
        :                              |        | word ID n | posting list n |
        :                              |        +----------------------------+
+----------------+                     |
| word ID n      |                     |
| string         |---> pos ------------|------->
+----------------+                     |
                                       |

```
When a query occurs:
1. Search the in-memory hash table to find the corresponding key
2. Read the posting list associated with that key from disk into memory for processing

If the dictionary is too large for memory, we can use a B+ tree to search it, treating it as an ordered sequence of keys.

The retrieval process can be summarized in two steps:
1. Use a B+ tree or similar technology to query the keyword in the dictionary
2. Read out the posting list for that keyword and process it in memory
```text
B-tree/B+ tree (In Memory)      Dictionary File (Disk)           Inverted Index File (Disk)
                                                              
       •                    +----------------------------+     +----------------------------+
      / \                   | word ID 1 | string 1 | pos |---->| word ID 1 | posting list 1 |
     •   •--------------->  +----------------------------+     +----------------------------+
    /                       | word ID 2 | string 2 | pos |---->| word ID 2 | posting list 2 |
   •                        +----------------------------+     +----------------------------+
  / \                       | word ID 3 | string 3 | pos |---->| word ID 3 | posting list 3 |
 •   •------------------->  +----------------------------+     +----------------------------+
                            |              ...           |     |             ...            |
                            +----------------------------+     +----------------------------+
                            | word ID n | string n | pos |---->| word ID n | posting list n |
                            +----------------------------+     +----------------------------+

```
## Handling Very Large Posting Lists

For extremely popular keywords that might appear in hundreds of millions of pages, the posting lists may be too large to load into memory. In such cases:

1. Create B+ tree-like indexes for oversized posting lists
2. Load only useful data blocks into memory to reduce disk access
3. For shorter posting lists, load them directly into memory
4. Use caching techniques like LRU to keep frequently used posting lists in memory

## Key Takeaways

1. We can generate trillion-level inverted indexes using multi-file merging and implement retrieval through dictionary and document list queries.

2. Two fundamental design principles:
  - Load as much data as possible into memory (index compression is crucial here)
  - Break large data collections into smaller sets (the core idea of distributed systems)
