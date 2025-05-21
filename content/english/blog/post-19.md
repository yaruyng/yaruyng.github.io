---
title: "Retrieval Technique Series-2.How to Index Massive Disk Data Using B+ Trees"
meta_title: ""
description: ""
date: 2025-05-21T00:00:00Z
image: ""
categories: ["Database", "Retrieval"]
author: "yaruyng"
tags: ["Database", "Retrieval"]
draft: false
---
Database Retrieval: How to Index Massive Disk Data Using B+ Trees
<!--more-->

In today's data-driven world, the ability to efficiently store and retrieve massive amounts of information is crucial for any database system. When dealing with data that exceeds the capacity of main memory, specialized data structures become essential. Among these structures, the B+ tree stands out as one of the most powerful and widely implemented indexing mechanisms in modern database systems. This blog explores how B+ trees enable efficient retrieval of massive disk-based data.

## Key Features of B+ Trees

### Suitable for Range Queries

One of the most significant advantages of B+ trees is their exceptional performance for range queries. Unlike hash indexes that excel at point queries but struggle with ranges, B+ trees maintain data in sorted order, making them ideal for operations like:

```
SELECT * FROM customers WHERE age BETWEEN 25 AND 40;
```

The structure allows for efficient traversal through consecutive keys, as illustrated below:

```
┌─────────────────────────────────────────────────────┐
│                  Range Query Process                │
│                                                     │
│  1. Find the leaf containing the lower bound (25)   │
│  2. Scan sequentially through leaf nodes            │
│  3. Continue until reaching upper bound (40)        │
│                                                     │
└─────────────────────────────────────────────────────┘
```

## Structure of B+ Trees

A key design of the B+ tree is to make the size of a node equal to the size of a block. The data stored within a node is not an element, but an ordered array that can hold m elements.


### Internal Nodes

Internal nodes in a B+ tree serve as navigational aids, containing keys that guide the search process but don't store actual data records. Each internal node contains:

- Sorted keys that define the ranges
- Pointers to child nodes
- A minimum fill factor (typically 50%) to ensure efficient space utilization

```
┌───────── Internal Node ────────┐
│                                │
│  ┌─────┐┌─────┐┌─────┐         │                   
│  │ K₁  ││ K₂  ││ K₃  │         │
│  └─────┘└─────┘└─────┘         │
│  │ P₁  ││ P₂  ││ P₃  │  ...    │
│  └─────┘└─────┘└─────┘         │
│     │                          │
│     ▼                          │
│  Pointer to                    │
│  child node                    │
│                                │
└────────────────────────────────┘
```

### Leaf Nodes

Leaf nodes are where the actual data (or pointers to data) resides. When a B+ tree is used for database indexing, the leaf nodes typically store the key value (Key) and the location of the corresponding data record on disk (pointer). The key characteristics of leaf nodes include:

- They contain the indexed keys and associated data records (or pointers to records)
- All leaf nodes are at the same level, ensuring balanced tree height
- Leaf nodes are linked together in a doubly-linked list, facilitating efficient range queries

```
                       ┌──────── Leaf Node ────────┐                      ┌──────── Leaf Node ────────┐     
                       │                           │                      │                           │      
                       │  ┌─────┐┌─────┐┌─────┐    │                      │  ┌─────┐┌─────┐┌─────┐    │     
                       │  │ K₁  ││ K₂  ││ K₃  │    │                      │  │ K₁  ││ K₂  ││ K₃  │    │     
                       │  └─────┘└─────┘└─────┘    │                      │  └─────┘└─────┘└─────┘    │    
  ... ◄──────────────► │  │ P₁  ││ P₂  ││ P₃  │... │  ◄───────────────►   │  │ P₁  ││ P₂  ││ P₃  │... │      
        Previous Leaf  │  └─────┘└─────┘└─────┘    │     Next Leaf        │  └─────┘└─────┘└─────┘    │                  
                       │     │                     │                      │     │                     │ 
                       │     ▼                     │                      │     ▼                     │   
                       │  pointers to data         │                      │  pointers to data         │       
                       └───────────────────────────┘                      └───────────────────────────┘           

```

## Search Process in B+ Trees

### Starting from the Root Node

Every search operation in a B+ tree begins at the root node. The process follows these steps:

1. Compare the search key with the keys in the root node
2. Determine which child pointer to follow
3. Traverse down the tree until reaching a leaf node

```
┌────────────────────────────────────────┐
│               Root Node                │
│           ┌─────┐   ┌─────┐            │
│           │ 30  │   │ 70  │            │
│           └─────┘   └─────┘            │
│              │         │               │
└──────────────┼─────────┼───────────────┘
               ▼         ▼
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│   < 30 Node     │ │  30-70 Node     │ │   > 70 Node     │
│ ┌────┐ ┌────┐   │ │ ┌────┐ ┌────┐   │ │ ┌────┐ ┌────┐   │
│ │ 10 │ │ 20 │   │ │ │ 40 │ │ 60 │   │ │ │ 80 │ │ 90 │   │
│ └────┘ └────┘   │ │ └────┘ └────┘   │ │ └────┘ └────┘   │
└─────────────────┘ └─────────────────┘ └─────────────────┘
```

### Traversing Internal Nodes Layer by Layer

As the search progresses through the tree:

1. At each internal node, the algorithm compares the search key with the node's keys
2. It follows the appropriate pointer based on the comparison
3. This process continues until reaching a leaf node

### Implementing Range Queries via Doubly-Linked Lists

The doubly-linked list connecting all leaf nodes is what makes B+ trees particularly efficient for range queries:

1. First, locate the leaf node containing the lower bound of the range
2. Scan sequentially through the linked list of leaf nodes
3. Continue until reaching the upper bound of the range

```
┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
│ Leaf 1   │◄─►│ Leaf 2   │◄─►│ Leaf 3   │◄─►│ Leaf 4   │
│ Keys:    │   │ Keys:    │   │ Keys:    │   │ Keys:    │
│ 10,15,20 │   │ 25,30,35 │   │ 40,45,50 │   │ 55,60,65 │
└──────────┘   └──────────┘   └──────────┘   └──────────┘
                    ▲               ▲
                    │               │
                    └───────────────┘
                    Range query from
                      30 to 45
```

## Dynamic Adjustments in B+ Trees(A node has 4 elements)

### Inserting Data

When inserting new data into a B+ tree:

1. Locate the appropriate leaf node
2. Insert the key in sorted order
3. If the node overflows (exceeds maximum capacity):
  - Split the node into two
  - Promote the middle key to the parent
  - Adjust pointers accordingly

```
Before insertion of 45:
┌───────────────┐
│    Node A     │
│ 30, 40, 50, 60│
└───────────────┘

After insertion and split:
        ┌─────┐
        │ 45  │  ← Promoted to parent
        └─────┘
         /   \
        /     \
┌───────────┐ ┌───────────┐
│  Node A   │ │  Node B   │
│ 30, 40    │ │ 45, 50, 60│
└───────────┘ └───────────┘
```

### Deleting Data

The deletion process involves:

1. Finding the key to be deleted
2. Removing it from the leaf node
3. If the node becomes underfilled:
  - Borrow keys from siblings if possible
  - Merge with siblings if necessary
  - Adjust parent nodes accordingly

```
Before deletion of 40:
┌───────────┐ ┌───────────┐
│  Node A   │ │  Node B   │
│ 30, 40    │ │ 45, 50, 60│
└───────────┘ └───────────┘
      Parent key: 45

After deletion and potential merge:
┌─────────────────┐
│     Node AB     │
│ 30, 45, 50, 60  │
└─────────────────┘
```

## Advantages of B+ Trees

### Suitable for Large-Scale Data

B+ trees are specifically designed to handle data that doesn't fit in memory:

- The tree structure minimizes disk I/O operations
- The height of the tree grows logarithmically with the number of records
- Even with millions or billions of records, a B+ tree typically has a height of 3-4 levels

```
┌────────────────────────────────────────────────┐
│        B+ Tree Performance Characteristics     │
├────────────────────────┬───────────────────────┤
│ Number of Records      │ Typical Tree Height   │
├────────────────────────┼───────────────────────┤
│ 1,000                  │ 2                     │
│ 1,000,000              │ 3                     │
│ 1,000,000,000          │ 4                     │
└────────────────────────┴───────────────────────┘
```

### Index Data Stored on Disk

B+ trees are optimized for disk-based storage:

- Node size is aligned with disk block size (typically 4KB to 16KB)
- This alignment minimizes the number of disk I/O operations
- Internal nodes are often cached in memory for faster access

```
┌───────────────────────────────────────────────────┐
│              Disk-Based B+ Tree                   │
│                                                   │
│  ┌─────────┐                                      │
│  │ Memory  │  Root and frequently accessed nodes  │
│  │ Cache   │  kept in memory                      │
│  └─────────┘                                      │
│        │                                          │
│        ▼                                          │
│  ┌─────────┐                                      │
│  │  Disk   │  Most nodes stored on disk,          │
│  │ Storage │  loaded as needed                    │
│  └─────────┘                                      │
│                                                   │
└───────────────────────────────────────────────────┘
```

## Design Philosophy

### Separation of Index and Data

A key design principle of B+ trees is the separation of indexing structure from the actual data:

- Leaf nodes contain either the full data records or pointers to them
- This separation allows for more efficient use of cache memory
- It also enables different organizations of the data itself

```
┌───────────────────────────────────────────────────┐
│           Index and Data Separation               │
│                                                   │
│  ┌─────────────────┐                              │
│  │   B+ Tree Index │                              │
│  │   Structure     │                              │
│  └────────┬────────┘                              │
│           │                                       │
│           │ References                            │
│           ▼                                       │
│  ┌─────────────────┐                              │
│  │   Actual Data   │                              │
│  │   Records       │                              │
│  └─────────────────┘                              │
│                                                   │
└───────────────────────────────────────────────────┘
```

## Practical Implementation

To better understand how B+ trees work in practice, let's consider simplified implementations in Java and Go:

### Java Implementation

```java
import java.util.ArrayList;
import java.util.List;
import java.util.AbstractMap.SimpleEntry;

public class BPlusTree<K extends Comparable<K>, V> {
    private Node<K, V> root;
    private final int maxKeys;
    
    public BPlusTree(int maxKeys) {
        this.root = new LeafNode<>(maxKeys);
        this.maxKeys = maxKeys;
    }
    
    public V search(K key) {
        return root.search(key);
    }
    
    public List<SimpleEntry<K, V>> rangeQuery(K startKey, K endKey) {
        List<SimpleEntry<K, V>> result = new ArrayList<>();
        
        // Find leaf node containing the start key
        LeafNode<K, V> node = findLeafNode(root, startKey);
        
        // Collect all keys in range by following leaf pointers
        while (node != null) {
            for (int i = 0; i < node.keys.size(); i++) {
                K key = node.keys.get(i);
                if (key.compareTo(startKey) >= 0 && key.compareTo(endKey) <= 0) {
                    result.add(new SimpleEntry<>(key, node.values.get(i)));
                }
                if (key.compareTo(endKey) > 0) {
                    return result;
                }
            }
            node = node.next;
        }
        
        return result;
    }
    
    private LeafNode<K, V> findLeafNode(Node<K, V> node, K key) {
        if (node instanceof LeafNode) {
            return (LeafNode<K, V>) node;
        }
        
        InternalNode<K, V> internalNode = (InternalNode<K, V>) node;
        int i = 0;
        while (i < internalNode.keys.size() && key.compareTo(internalNode.keys.get(i)) >= 0) {
            i++;
        }
        return findLeafNode(internalNode.children.get(i), key);
    }
    
    private abstract static class Node<K extends Comparable<K>, V> {
        List<K> keys;
        int maxKeys;
        
        Node(int maxKeys) {
            this.keys = new ArrayList<>();
            this.maxKeys = maxKeys;
        }
        
        abstract V search(K key);
    }
    
    private static class InternalNode<K extends Comparable<K>, V> extends Node<K, V> {
        List<Node<K, V>> children;
        
        InternalNode(int maxKeys) {
            super(maxKeys);
            this.children = new ArrayList<>();
        }
        
        @Override
        V search(K key) {
            int i = 0;
            while (i < keys.size() && key.compareTo(keys.get(i)) >= 0) {
                i++;
            }
            return children.get(i).search(key);
        }
    }
    
    private static class LeafNode<K extends Comparable<K>, V> extends Node<K, V> {
        List<V> values;
        LeafNode<K, V> next;
        LeafNode<K, V> prev;
        
        LeafNode(int maxKeys) {
            super(maxKeys);
            this.values = new ArrayList<>();
        }
        
        @Override
        V search(K key) {
            for (int i = 0; i < keys.size(); i++) {
                if (keys.get(i).equals(key)) {
                    return values.get(i);
                }
            }
            return null;
        }
    }
}
```

### Go Implementation

```go
package bplustree

import (
    "fmt"
)

type KeyValuePair struct {
    Key   interface{}
    Value interface{}
}

type BPlusTreeNode struct {
    IsLeaf   bool
    Keys     []interface{}
    Children []*BPlusTreeNode
    Next     *BPlusTreeNode
    Prev     *BPlusTreeNode
    Values   []interface{}
    MaxKeys  int
}

type BPlusTree struct {
    Root    *BPlusTreeNode
    MaxKeys int
}

func NewBPlusTree(maxKeys int) *BPlusTree {
    return &BPlusTree{
        Root: &BPlusTreeNode{
            IsLeaf:  true,
            Keys:    make([]interface{}, 0),
            Values:  make([]interface{}, 0),
            MaxKeys: maxKeys,
        },
        MaxKeys: maxKeys,
    }
}

func (t *BPlusTree) Search(key interface{}) interface{} {
    return t.search(t.Root, key)
}

func (t *BPlusTree) search(node *BPlusTreeNode, key interface{}) interface{} {
    if node.IsLeaf {
        for i, k := range node.Keys {
            if compare(k, key) == 0 {
                return node.Values[i]
            }
        }
        return nil
    }

    // If not leaf, find the appropriate child
    i := 0
    for i < len(node.Keys) && compare(key, node.Keys[i]) >= 0 {
        i++
    }
    return t.search(node.Children[i], key)
}

func (t *BPlusTree) RangeQuery(startKey, endKey interface{}) []KeyValuePair {
    // Find leaf node containing the start key
    node := t.Root
    for !node.IsLeaf {
        i := 0
        for i < len(node.Keys) && compare(startKey, node.Keys[i]) >= 0 {
            i++
        }
        node = node.Children[i]
    }

    // Collect all keys in range by following leaf pointers
    result := make([]KeyValuePair, 0)
    for node != nil {
        for i, key := range node.Keys {
            if compare(key, startKey) >= 0 && compare(key, endKey) <= 0 {
                result = append(result, KeyValuePair{Key: key, Value: node.Values[i]})
            }
            if compare(key, endKey) > 0 {
                return result
            }
        }
        node = node.Next
    }

    return result
}

// Helper function to compare keys
func compare(a, b interface{}) int {
    switch a := a.(type) {
    case int:
        return a - b.(int)
    case string:
        if a < b.(string) {
            return -1
        } else if a > b.(string) {
            return 1
        }
        return 0
    default:
        panic("Unsupported type for comparison")
    }
}
```

## Conclusion

B+ trees have become the backbone of indexing in disk-based database systems for good reasons. Their balanced structure ensures consistent performance regardless of data size, while their leaf-level linked list enables efficient range queries. The design specifically addresses the challenges of disk-based storage by minimizing I/O operations and maximizing cache utilization.

When designing database systems that must handle large volumes of data, understanding B+ trees is essential. They represent one of the most elegant solutions to the problem of efficiently retrieving data from disk-based storage systems, balancing the trade-offs between memory usage, disk access patterns, and query performance.

Whether you're developing a database system in Java, Go, or any other language, the B+ tree is a fundamental concept that demonstrates how clever algorithm design can overcome the physical limitations of storage hardware. The implementations provided above, while simplified, illustrate the core concepts that make B+ trees so valuable in database systems.
