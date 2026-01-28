# Conversational AI Profile Architecture

**Date:** 2026-01-28  
**Source:** research/hybrid-search-architecture-conversational-ai.md

## Recap: Hybrid Search Architecture

| Data | Storage | Search Type | Why |
|------|---------|-------------|-----|
| **User Profile (unstructured facts)** | Neo4j | **Vector** (semantic) | Conceptual queries on free-text facts |
| **Message History** | Elasticsearch | **BM25** (keyword) | Exact term matching at scale |

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      YOUR ARCHITECTURE                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  NEO4J (Profile)          ELASTICSEARCH (Messages)              │
│  ────────────────         ───────────────────────               │
│  (User)-[:HAS_FACT]->     messages_index:                       │
│    (Fact {text: "..."})     "I had sushi for lunch"             │
│                                                                  │
│  Search:                   Search:                              │
│  Vector Index              BM25 (keyword)                       │
│  "What do I like to eat?"  "sushi" → exact match                │
│                                                                  │
│  ✅ Native: Neo4j 5.x      ✅ Native: ES BM25 built-in          │
│  ✅ Semantic: concepts     ✅ Exact: keyword matching           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Key Insight

**Profile facts are unstructured text** — stored as `(User)-[:HAS_FACT]->(Fact {text: "Loves Italian food"})`. You can't query by category. You must **search** the text.

- **Vector search** for conceptual queries: "What do I like to eat?"
- **BM25** for keyword queries: "sushi", "Italian restaurant"

## Neo4j Vector Support

Neo4j 5.x+ has native vector indexes:

```cypher
CREATE VECTOR INDEX profileFactsVector 
FOR (f:Fact) ON (f.embedding);

CALL db.index.vector.queryNodes("profileFactsVector", 10, $embedding)
YIELD node, similarity;
```

## When to Use What

| Query Type | Example | Search |
|------------|---------|--------|
| Conceptual | "What do I like to eat?" | **Vector** (Neo4j) |
| Keyword | "sushi restaurant" | **BM25** (Elasticsearch) |
| Hybrid | "Italian food near beach" | Both + rerank |

## TL;DR

- **Profile** → Neo4j + Vector search (semantic)
- **Messages** → Elasticsearch + BM25 (keyword)
- **Unstructured data** → Search, don't traverse
