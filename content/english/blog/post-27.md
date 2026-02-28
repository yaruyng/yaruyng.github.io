---
title: "Retrieval Strategy Design: Vector, Keyword, and Hybrid Search"
meta_title: ""
description: ""
date: 2026-02-28T00:00:00Z
image: ""
categories: ["RAG", "LLM"]
author: "yaruyng"
tags: ["RAG", "LLM"]
draft: false
---

This article explains how to design a **modern retrieval strategy** for AI systems, especially Retrieval-Augmented Generation (RAG). The focus is not only on definitions, but on **engineering trade-offs, system architecture, and practical defaults**.

The target audience is backend engineers who can already *use* embeddings, but want to **design reliable and controllable search systems**.

---

## 1. Where Retrieval Strategy Fits in the System

A typical modern retrieval pipeline looks like this:

```
User Query
  ↓
Query Rewrite / Intent Analysis
  ↓
Multi-Channel Retrieval
  (Vector / Keyword / Metadata)
  ↓
Hybrid Merge
  ↓
Top-K Limiting
  ↓
Score Threshold Filtering
  ↓
(Optional) Reranking
  ↓
LLM Generation
```

Concepts like **vector search**, **hybrid search**, **Top-K**, and **threshold filtering** are not isolated features. They work together inside the *recall and filtering* stages of this pipeline.

---

## 2. Vector Search: The Semantic Recall Layer

### 2.1 What Vector Search Solves

Vector search addresses the problem of **semantic mismatch**:

* The user and the document use different words
* The meaning is similar, but lexical overlap is low

Example:

```
Query: How to reduce dopamine addiction
Document: Attention control and dopamine regulation
```

Keyword search fails here, but embeddings succeed.

---

### 2.2 Core Parameters Engineers Must Understand

#### Similarity Metric

The most common similarity metrics are:

* Cosine Similarity (industry default)
* Dot Product
* L2 Distance

Most embedding models are trained assuming **cosine similarity**, so databases typically follow that convention.

---

#### Index Type (Performance-Critical)

| Index Type | Use Case                            |
| ---------- | ----------------------------------- |
| Flat       | Small datasets, maximum accuracy    |
| HNSW       | General-purpose, production default |
| IVF        | Very large-scale datasets           |

For most knowledge-base and RAG systems, **HNSW** is the best trade-off.

---

### 2.3 The Fundamental Weakness of Vector Search

Vector search is strong at recall, but weak at precision:

* It retrieves *related* content
* It may retrieve *irrelevant but semantically nearby* content

This is why vector search **must be combined** with:

* Top-K limits
* Score thresholds
* Reranking

---

## 3. Keyword Search (BM25): The Precision Layer

Keyword search is not obsolete. Its role is **deterministic precision**.

It excels at:

* Code and stack traces
* API names
* Error messages
* Proper nouns
* Numbers and IDs

In many technical queries, keyword search outperforms embeddings.

Another key benefit is **controllability**: keyword matching acts as a deterministic filter that reduces hallucinations.

---

## 4. Hybrid Search: The Industry Standard

Hybrid search combines the strengths of both approaches:

* Vector search for semantic recall
* Keyword search for lexical precision

This is no longer optional in production systems.

---

### 4.1 Parallel Hybrid (Most Common)

```
Vector Search Top-K = 20
Keyword Search Top-K = 20
↓
Merge Results
↓
Rerank
```

Advantages:

* Simple to implement
* Stable behavior
* Widely used in production

---

### 4.2 Score Fusion Hybrid

A weighted scoring approach:

```
Final Score = α × Vector Score + β × BM25 Score
```

This method is suitable for search-engine-like systems that require strong global ranking.

---

## 5. Top-K: A Recall Boundary, Not a Quality Guarantee

A common misconception is:

> Higher Top-K means better results

In reality:

* Top-K defines the *maximum recall scope*
* Large Top-K increases noise
* Token usage and latency increase rapidly

### Practical Defaults

| Scenario       | Recommended Top-K |
| -------------- | ----------------- |
| FAQ            | 3–5               |
| Technical Docs | 5–10              |
| Code Search    | 10–20             |

For most RAG systems:

* Vector Top-K: 8–10
* Keyword Top-K: 8–10

---

## 6. Score Threshold Filtering: The Missing Safeguard

Top-K always returns results — even when nothing is relevant.

Threshold filtering solves this:

```
Only keep results where score > threshold
```

Without thresholds, systems produce classic failures:

```
Query: Apple phone
Result: Apple fruit
```

---

### Threshold Guidelines (Cosine Similarity)

| Similarity | Interpretation    |
| ---------- | ----------------- |
| > 0.85     | Strongly relevant |
| 0.75–0.85  | Acceptable        |
| < 0.70     | Noise             |

Many production systems use:

```
threshold ≈ 0.78
```

---

## 7. A Practical, Production-Ready Retrieval Strategy

A robust default pipeline:

```
1. Optional Query Rewrite
2. Vector Search (Top-K = 10)
3. Keyword Search (Top-K = 10)
4. Merge Results
5. Filter: score > 0.78
6. Rerank Top 5
7. Send Top 3 to LLM
```

This structure balances recall, precision, cost, and stability.

---

## 8. What Engineers Should Actually Focus On

### 8.1 Recall vs Precision Trade-off

```
Vector Search → Recall
Keyword Search → Precision
Reranker → Final Quality
```

Understanding this triangle is more important than tuning any single parameter.

---

### 8.2 Chunk Design Matters More Than Algorithms

Poor chunking breaks all retrieval strategies:

* Chunks too long → embedding dilution
* Chunks too short → context fragmentation

Good retrieval starts with good chunk boundaries.

---

### 8.3 Top-K Is Not the Final Output Size

Typical production flow:

```
Retrieve 20
Filter to 12
Rerank to 5
LLM consumes 3
```

---

## Conclusion

Modern retrieval systems are not built on vector search alone.

**Hybrid retrieval + threshold filtering + reranking** is the real foundation of stable, production-grade RAG systems.

If you design retrieval with a system mindset instead of a single-algorithm mindset, quality improves dramatically.
