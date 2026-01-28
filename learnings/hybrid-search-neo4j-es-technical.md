# Hybrid Search Technical Implementation: Neo4j + Elasticsearch

**Date:** 2026-01-28  
**Purpose:** Exact code examples for setting up hybrid search between Neo4j and Elasticsearch

---

## Why This Matters

### The Problem with Unstructured Profile Data

Your profile data is **unstructured text** stored as facts:
```
(User)-[:HAS_FACT]->(Fact {text: "Loves Italian food"})
(User)-[:HAS_FACT]->(Fact {text: "Going to Japan in March"})
(User)-[:HAS_FACT]->(Fact {text: "Vegetarian since 2020"})
```

**You cannot query this by category** because there are no fixed properties like `preference_category` or `fact_type`. You only have `Fact.text`.

### The Solution: Search, Don't Traverse

Instead of graph traversal, use **full-text search** on the `text` property:

| Query Type | What It Does | Example |
|------------|--------------|---------|
| **BM25 (keyword)** | Finds exact words | "food" → "Loves Italian food" |
| **Vector (semantic)** | Finds concepts | "what do they eat?" → "Vegetarian since 2020" |

### Why Two Databases?

| Database | Strength | Use For |
|----------|----------|---------|
| **Neo4j** | Graph relationships, traversals | Profile structure, user segments, preferences graph |
| **Elasticsearch** | Full-text search, BM25, scaling | Message history, search across all user data |

**Key insight:** Neo4j is bad at text search. Elasticsearch is bad at graphs. Use each for what it's best at.

---

## Your Use Case: Conversational AI Profile

### Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    YOUR ARCHITECTURE                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  NEO4J (Graph - Profile Structure)                      │    │
│  │  ─────────────────────────────────                       │    │
│  │  (User)-[:HAS_FACT]->(Fact {text: "..."})              │    │
│  │  (User)-[:HAS_PREFERENCE]->(Preference {...})          │    │
│  │  (User)-[:BELONGS_TO]->(Segment {...})                 │    │
│  │                                                          │    │
│  │  ✓ Fast: Traverse user → facts, segments               │    │
│  │  ✗ Slow: Search "food preferences" across all facts    │    │
│  └─────────────────────────────────────────────────────────┘    │
│                          │                                       │
│                          │ Sync (CDC or periodic)                │
│                          ▼                                       │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  ELASTICSEARCH (Search - Messages + Profile Facts)      │    │
│  │  ─────────────────────────────────────────               │    │
│  │  profile_facts index: Fact text searchable              │    │
│  │  messages index: Message history searchable             │    │
│  │                                                          │    │
│  │  ✓ Fast: BM25 + vector hybrid search                    │    │
│  │  ✗ No graph relationships                               │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Workflow: User Asks "What are my food preferences?"

```
1. User Query: "What are my food preferences?"

2. Agent (or preprocessing) extracts search terms:
   - keywords: ["food", "diet", "vegetarian", "cuisine"]
   - intent: profile_search

3. Query Elasticsearch (BM25):
   POST /profile_facts/_search
   {
     "query": {
       "bool": {
         "must": [
           {"term": {"user_id": "user_123"}},
           {"match": {"fact_text": "food diet vegetarian"}}
         ]
       }
     }
   }

4. Returns:
   - "Loves Italian food" (score: 2.1)
   - "Vegetarian since 2020" (score: 1.8)
   - "Allergic to peanuts" (score: 1.5)

5. Inject into agent context:
   "User food preferences:
    - Loves Italian food
    - Vegetarian since 2020
    - Allergic to peanuts"

6. Agent responds with personalized answer
```

### Implementation Steps

| Step | Action | Technology |
|------|--------|------------|
| 1 | Index profile facts in Neo4j | `CREATE FULLTEXT INDEX ... FOR (f:Fact) ON EACH [f.text]` |
| 2 | Index messages in Elasticsearch | `PUT /messages` with BM25 mapping |
| 3 | Sync new facts to Elasticsearch | Kafka CDC or direct query |
| 4 | Query at runtime | BM25 for keywords, vector for semantic |
| 5 | Inject results into agent | Pre-inject via system prompt |

### Minimal V1 Setup

```cypher
-- Step 1: Create full-text index in Neo4j
CREATE FULLTEXT INDEX profileFactsIndex 
FOR (f:Fact) ON EACH [f.text];

-- Step 2: Search for food preferences
CALL db.index.fulltext.queryNodes("profileFactsIndex", "food diet")
YIELD node, score
MATCH (u:User)-[:HAS_FACT]->(node)
WHERE u.id = $user_id
RETURN node.text AS preference, score;
```

```bash
# Step 3: Create Elasticsearch index
PUT /profile_facts
{
  "mappings": {
    "properties": {
      "user_id": {"type": "keyword"},
      "fact_text": {"type": "text"},
      "extracted_at": {"type": "date"}
    }
  }
}

# Step 4: Search
POST /profile_facts/_search
{
  "query": {
    "bool": {
      "must": [
        {"term": {"user_id": "user_123"}},
        {"match": {"fact_text": "food diet"}}
      ]
    }
  }
}
```

### When to Add Vector Search

| Scenario | Use BM25 | Add Vector |
|----------|----------|------------|
| User asks: "What food do I like?" | ✅ "food" matches "Loves Italian food" | Optional |
| User asks: "What do I usually eat?" | ❌ No "food" in message | ✅ "eat" matches "Vegetarian" via semantics |
| User asks: "Remember that Italian place?" | ✅ "Italian" matches | Optional |

**Start with BM25 only. Add vector search if users ask conceptual questions that don't use exact keywords.**

---

## 1. Neo4j: Full-Text Index (BM25-ish)

### Create Full-Text Index on Profile Facts

```cypher
// Index for unstructured profile facts
CREATE FULLTEXT INDEX profileFactsIndex 
FOR (f:Fact) 
ON EACH [f.text];

// Index for messages
CREATE FULLTEXT INDEX messageTextIndex 
FOR (m:Message) 
ON EACH [m.text];
```

### Search Profile Facts (BM25)

```cypher
// Basic search
CALL db.index.fulltext.queryNodes("profileFactsIndex", "food vegetarian")
YIELD node, score
RETURN node.id AS fact_id, node.text AS fact_text, score
ORDER BY score DESC
LIMIT 10;

// With user filter
CALL db.index.fulltext.queryNodes("profileFactsIndex", "food diet")
YIELD node, score
MATCH (u:User)-[:HAS_FACT]->(node)
WHERE u.id = $user_id
RETURN node.text AS fact_text, score
ORDER BY score DESC;
```

### Filter by User + Get Related Context

```cypher
// Get profile facts for user, ordered by relevance
CALL db.index.fulltext.queryNodes("profileFactsIndex", "restaurant Italian cuisine")
YIELD node, score
MATCH (u:User {id: $user_id})-[:HAS_FACT]->(node)
RETURN 
  node.text AS fact,
  score,
  node.created_at AS extracted_at,
  node.source AS source_conversation
ORDER BY score DESC
LIMIT 5;
```

---

## 2. Neo4j: Vector Index (Semantic Search)

### Create Vector Index (Neo4j 5.x+)

```cypher
// Requires: Neo4j 5.x with GDS or Vector index support
// Store embeddings as node property first

// Generate embeddings (via OpenAI, or store from conversation AI)
MATCH (f:Fact)
WHERE f.embedding IS NULL
WITH f LIMIT 100
CALL genai.vector.encode(f.text, "OpenAI", {apiKey: $api_key})
YIELD vector
SET f.embedding = vector
SET f.embedding_model = "text-embedding-3-small"
SET f.embedding_dims = 1536;
```

```cypher
// Create vector index
CREATE VECTOR INDEX profileFactsVector 
FOR (f:Fact) ON (f.embedding)
OPTIONS {
  indexConfig: {
    `vector.dimensions`: 1536,
    `vector.similarity_function`: 'cosine',
    `vector.hnsw.space`: 'cosine'
  }
};
```

### Semantic Search

```cypher
// Query with embedding (convert query to vector first)
WITH $query_text AS query_text
CALL genai.vector.encode(query_text, "OpenAI", {apiKey: $api_key})
YIELD query_vector
CALL db.index.vector.queryNodes("profileFactsVector", 10, query_vector)
YIELD node, similarity
MATCH (u:User)-[:HAS_FACT]->(node)
WHERE u.id = $user_id
RETURN node.text AS fact, similarity
ORDER BY similarity DESC
LIMIT 5;
```

---

## 3. Elasticsearch: Message Index

### Create Index with BM25 Mapping

```bash
PUT /messages
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1,
    "analysis": {
      "analyzer": {
        "message_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "english_stemmer"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "message_id": { "type": "keyword" },
      "user_id": { "type": "keyword" },
      "conversation_id": { "type": "keyword" },
      "message_text": { 
        "type": "text",
        "analyzer": "message_analyzer",
        "fields": {
          "keyword": { "type": "keyword" },
          "exact": { "type": "text", "analyzer": "standard" }
        }
      },
      "timestamp": { "type": "date" },
      "embedding": { "type": "dense_vector", "dims": 1536 }
    }
  }
}
```

### BM25 Search (Keyword)

```bash
# Basic BM25 search
POST /messages/_search
{
  "query": {
    "bool": {
      "must": [
        { "term": { "user_id": "user_123" }},
        { 
          "match": { 
            "message_text": {
              "query": "Italian restaurant beach",
              "boost": 2
            }
          }
        }
      ]
    }
  },
  "highlight": {
    "fields": {
      "message_text": {}
    }
  },
  "size": 10
}
```

### Hybrid Search (BM25 + Vector) - Elasticsearch 8.x+

```bash
POST /messages/_search
{
  "query": {
    "bool": {
      "must": [
        { "term": { "user_id": "user_123" }}
      ],
      "should": [
        {
          "match": {
            "message_text": {
              "query": "Italian restaurant beach",
              "boost": 1
            }
          }
        },
        {
          "knn": {
            "field": "embedding",
            "query_vector": $query_embedding,
            "k": 10,
            "boost": 0.7
          }
        }
      ]
    }
  }
}
```

### Hybrid with RRF (Reciprocal Rank Fusion)

```bash
POST /messages/_search
{
  "query": {
    "bool": {
      "must": [
        { "term": { "user_id": "user_123" }}
      ]
    }
  },
  "rank": {
    "rrf": {
      "window_size": 100,
      "rank_constant": 60
    }
  }
}
```

---

## 4. Elasticsearch: Profile Facts Index

```bash
PUT /profile_facts
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  },
  "mappings": {
    "properties": {
      "user_id": { "type": "keyword" },
      "fact_text": { 
        "type": "text",
        "analyzer": "standard"
      },
      "fact_type": { "type": "keyword" },
      "confidence": { "type": "float" },
      "source": { "type": "keyword" },
      "extracted_at": { "type": "date" },
      "embedding": { "type": "dense_vector", "dims": 1536 }
    }
  }
}
```

```bash
# Search profile facts
POST /profile_facts/_search
{
  "query": {
    "bool": {
      "must": [
        { "term": { "user_id": "user_123" }},
        { 
          "match": { 
            "fact_text": {
              "query": "food diet vegetarian restaurant",
              "minimum_should_match": "50%"
            }
          }
        }
      ]
    }
  },
  "size": 10
}
```

---

## 5. Python: Hybrid Query Client

```python
from neo4j import GraphDatabase
from elasticsearch import Elasticsearch
import openai
import numpy as np

class HybridSearchClient:
    def __init__(self, neo4j_uri, neo4j_auth, es_url, openai_key):
        self.neo4j = GraphDatabase.driver(neo4j_uri, auth=neo4j_auth)
        self.es = Elasticsearch([es_url])
        self.openai = openai.OpenAI(api_key=openai_key)
    
    def get_embedding(self, text: str) -> list:
        """Convert text to embedding vector."""
        response = self.openai.embeddings.create(
            model="text-embedding-3-small",
            input=text
        )
        return response.data[0].embedding
    
    # === NEO4J FULL-TEXT SEARCH ===
    def search_neo4j_facts(self, user_id: str, query: str, limit: int = 5):
        """BM25-style search on profile facts in Neo4j."""
        with self.neo4j.session() as session:
            result = session.run("""
                CALL db.index.fulltext.queryNodes("profileFactsIndex", $query)
                YIELD node, score
                MATCH (u:User {id: $user_id})-[:HAS_FACT]->(node)
                RETURN node.text AS fact, score, node.id AS fact_id
                ORDER BY score DESC
                LIMIT $limit
            """, query=query, user_id=user_id, limit=limit)
            return [dict(record) for record in result]
    
    # === NEO4J VECTOR SEARCH ===
    def search_neo4j_vector(self, user_id: str, query: str, limit: int = 5):
        """Semantic search on profile facts in Neo4j."""
        embedding = self.get_embedding(query)
        with self.neo4j.session() as session:
            result = session.run("""
                CALL db.index.vector.queryNodes("profileFactsVector", $limit, $embedding)
                YIELD node, similarity
                MATCH (u:User {id: $user_id})-[:HAS_FACT]->(node)
                RETURN node.text AS fact, similarity
                ORDER BY similarity DESC
            """, embedding=embedding, user_id=user_id, limit=limit)
            return [dict(record) for record in result]
    
    # === ELASTICSEARCH BM25 SEARCH ===
    def search_es_messages(self, user_id: str, query: str, limit: int = 10):
        """BM25 search on messages in Elasticsearch."""
        response = self.es.search(
            index="messages",
            query={
                "bool": {
                    "must": [
                        {"term": {"user_id": user_id}},
                        {"match": {"message_text": query}}
                    ]
                }
            },
            size=limit,
            highlight={"fields": {"message_text": {}}}
        )
        return [{
            "text": hit["_source"]["message_text"],
            "score": hit["_score"],
            "timestamp": hit["_source"]["timestamp"],
            "highlight": hit.get("highlight", {})
        } for hit in response["hits"]["hits"]]
    
    # === ELASTICSEARCH HYBRID SEARCH ===
    def search_es_hybrid(self, user_id: str, query: str, limit: int = 10):
        """BM25 + vector hybrid search in Elasticsearch."""
        embedding = self.get_embedding(query)
        response = self.es.search(
            index="messages",
            query={
                "bool": {
                    "must": [{"term": {"user_id": user_id}}],
                    "should": [
                        {"match": {"message_text": {"query": query, "boost": 1}}},
                        {"knn": {"field": "embedding", "query_vector": embedding, "k": limit, "boost": 0.7}}
                    ]
                }
            },
            size=limit
        )
        return [{
            "text": hit["_source"]["message_text"],
            "score": hit["_score"],
            "type": "hybrid"
        } for hit in response["hits"]["hits"]]
    
    # === UNIFIED HYBRID SEARCH ===
    def search_all(self, user_id: str, query: str, limit: int = 5):
        """Search profile facts + messages with BM25 + vector."""
        # Get embedding for vector queries
        embedding = self.get_embedding(query)
        
        # Parallel searches
        results = {
            "profile_facts_bm25": self.search_neo4j_facts(user_id, query, limit),
            "profile_facts_vector": self.search_neo4j_vector(user_id, query, limit),
            "messages_bm25": self.search_es_messages(user_id, query, limit),
            "messages_hybrid": self.search_es_hybrid(user_id, query, limit)
        }
        
        # Combine and normalize scores
        combined = []
        for source, items in results.items():
            for item in items:
                combined.append({
                    "text": item["text"],
                    "source": source,
                    "score": item.get("score", 0.5) if isinstance(item.get("score"), (int, float)) else 0.5,
                    "type": "profile_fact" if "profile" in source else "message"
                })
        
        # Normalize scores to 0-1 range
        max_score = max([r["score"] for r in combined]) if combined else 1
        for item in combined:
            item["normalized_score"] = item["score"] / max_score if max_score > 0 else 0
        
        # Sort by normalized score
        combined.sort(key=lambda x: x["normalized_score"], reverse=True)
        
        return {
            "all_results": combined[:limit * 4],
            "by_type": {
                "profile_facts": [r for r in combined if r["type"] == "profile_fact"][:limit],
                "messages": [r for r in combined if r["type"] == "message"][:limit]
            }
        }


# Usage
client = HybridSearchClient(
    neo4j_uri="bolt://localhost:7687",
    neo4j_auth=("neo4j", "password"),
    es_url="http://localhost:9200",
    openai_key="sk-..."
)

# User asks: "What restaurants near the beach do I like?"
results = client.search_all(
    user_id="user_123",
    query="restaurant beach Italian food preferences"
)

# Agent uses results in context
print(results["by_type"]["profile_facts"])
print(results["by_type"]["messages"])
```

---

## 6. Sync: Neo4j → Elasticsearch

```python
from neo4j import GraphDatabase
from elasticsearch import Elasticsearch
from kafka import KafkaProducer, KafkaConsumer
import json
import datetime

class Neo4jToElasticsearchSync:
    def __init__(self, neo4j_uri, neo4j_auth, es_url, kafka_url):
        self.neo4j = GraphDatabase.driver(neo4j_uri, auth=neo4j_auth)
        self.es = Elasticsearch([es_url])
        self.producer = KafkaProducer(bootstrap_servers=kafka_url)
        self.consumer = KafkaConsumer(
            'neo4j.profile_facts',
            bootstrap_servers=kafka_url,
            value_deserializer=lambda x: json.loads(x.decode('utf-8'))
        )
    
    # === DIRECT SYNC ===
    def sync_user_facts(self, user_id: str):
        """Sync all profile facts for a user to Elasticsearch."""
        with self.neo4j.session() as session:
            result = session.run("""
                MATCH (u:User {id: $user_id})-[:HAS_FACT]->(f:Fact)
                RETURN f.id AS fact_id, f.text AS fact_text, 
                       f.type AS fact_type, f.confidence AS confidence,
                       f.created_at AS created_at, f.source AS source
            """, user_id=user_id)
            
            for record in result:
                self.es.index(
                    index="profile_facts",
                    id=record["fact_id"],
                    document={
                        "user_id": user_id,
                        "fact_text": record["fact_text"],
                        "fact_type": record["fact_type"],
                        "confidence": record["confidence"],
                        "extracted_at": record["created_at"].isoformat() if record["created_at"] else None,
                        "source": record["source"]
                    }
                )
    
    def sync_messages(self, user_id: str, hours: int = 24):
        """Sync recent messages to Elasticsearch."""
        with self.neo4j.session() as session:
            result = session.run("""
                MATCH (u:User {id: $user_id})-[:SENT]->(m:Message)
                WHERE m.timestamp >= datetime() - duration({hours: $hours})
                RETURN m.id AS message_id, m.text AS message_text,
                       m.timestamp AS timestamp, m.conversation_id AS conversation_id
            """, user_id=user_id, hours=hours)
            
            for record in result:
                self.es.index(
                    index="messages",
                    id=record["message_id"],
                    document={
                        "message_id": record["message_id"],
                        "user_id": user_id,
                        "message_text": record["message_text"],
                        "timestamp": record["timestamp"].isoformat() if record["timestamp"] else None,
                        "conversation_id": record["conversation_id"]
                    }
                )
    
    # === CDC-STYLE SYNC (Kafka) ===
    def on_profile_fact_created(self, fact_data: dict):
        """Called when new profile fact is created in Neo4j."""
        self.es.index(
            index="profile_facts",
            id=fact_data["fact_id"],
            document={
                "user_id": fact_data["user_id"],
                "fact_text": fact_data["text"],
                "fact_type": fact_data.get("type"),
                "confidence": fact_data.get("confidence", 0.8),
                "extracted_at": datetime.utcnow().isoformat(),
                "source": fact_data.get("source")
            }
        )
    
    def consume_kafka_events(self):
        """Consume events from Kafka and sync to Elasticsearch."""
        for message in self.consumer:
            event = message.value
            if event["type"] == "fact_created":
                self.on_profile_fact_created(event["data"])


# Run sync
sync = Neo4jToElasticsearchSync(
    neo4j_uri="bolt://localhost:7687",
    neo4j_auth=("neo4j", "password"),
    es_url="http://localhost:9200",
    kafka_url="localhost:9092"
)

# Sync all data for user
sync.sync_user_facts("user_123")
sync.sync_messages("user_123", hours=48)
```

---

## 7. Quick Reference

| Task | Tool | Command |
|------|------|---------|
| Index profile facts (BM25) | Neo4j | `CREATE FULLTEXT INDEX profileFactsIndex FOR (f:Fact) ON EACH [f.text]` |
| Search profile facts | Neo4j | `CALL db.index.fulltext.queryNodes("profileFactsIndex", "query")` |
| Index embeddings | Neo4j | `CREATE VECTOR INDEX profileFactsVector ON (f.embedding)` |
| Semantic search | Neo4j | `CALL db.index.vector.queryNodes("profileFactsVector", k, embedding)` |
| Index messages | Elasticsearch | `PUT /messages` with BM25 mapping |
| Search messages | Elasticsearch | `POST /messages/_search` with `match` query |
| Hybrid search | Elasticsearch | `POST /messages/_search` with `knn` + `match` |
| Sync to ES | Python | `es.index(index="messages", ...)` |

---

## 8. Summary

| Component | Technology | Use Case |
|-----------|------------|----------|
| Profile Facts (BM25) | Neo4j Full-Text Index | Exact keyword search on unstructured facts |
| Profile Facts (Vector) | Neo4j Vector Index | Semantic search on unstructured facts |
| Messages (BM25) | Elasticsearch | Full-text search with scoring |
| Messages (Vector) | Elasticsearch (knn) | Semantic search on messages |
| Sync | Direct Query or Kafka CDC | Keep ES in sync with Neo4j |

Start with Neo4j Full-Text + Elasticsearch BM25. Add vector search if you need semantic matching. 💎
