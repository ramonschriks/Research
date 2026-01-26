# Hybrid Memory Search Patterns for User Profiles

**Source:** Discussion with Ramon (2026-01-26)  
**Category:** AI/Architecture/Retrieval

---

## Context

Ramon is building a user profile system for his work project. Key requirements:
- Extract facts/nodes from conversation history
- Store in structured format (graph)
- Allow multiple agents/prompts to query user context
- Scale beyond token limits

---

## Key Learning: Two Hybrid Patterns

### Pattern 1: Vector + FTS5/BM25 (Flat Documents)
**Used by:** Clawdbot memory system, unstructured text search

| Component | Purpose |
|-----------|---------|
| **Vector Search** | Semantic matching ("what does user prefer") |
| **FTS5/BM25** | Exact keyword matching ("email", "2026-01-26", IDs) |

```python
# Clawdbot's approach
def memory_search(query):
    vector_results = vector_search(query)  # Semantic
    fts_results = fts5_search(query)       # Keywords
    return merge_results(vector_results, fts_results)
```

**Best for:** Searching across MEMORY.md, daily logs, text files

### Pattern 2: Vector + Graph (Structured Data)
**For:** User profiles, node-based data with relationships

| Component | Purpose |
|-----------|---------|
| **Vector Search** | Semantic matching ("response preferences") |
| **Graph Traversal** | Get related nodes, understand connections |

```python
# Ramon's work project
def user_profile(query):
    # Step 1: Semantic search in nodes
    nodes = vector_search(query, k=5)
    
    # Step 2: Traverse relationships from found nodes
    related = graph.traverse(start_nodes=nodes, depth=2)
    
    return combine(nodes, related)
```

**Best for:** User profiles, product catalogs, entity graphs

---

## Core Insight

> **Hybrid search is not one-size-fits-all.** The "hybrid" part (combining two retrieval methods) is the pattern. The backends (FTS5 vs Graph) depend on data structure.

| Data Structure | Recommended Hybrid |
|----------------|-------------------|
| Flat text files (Markdown) | Vector + FTS5/BM25 |
| Structured nodes + relationships | Vector + Graph |
| Key-value stores | Vector + Exact Match |

---

## Ramon's Real-World Problem

### Problem Statement
> "We have many agents/prompts. Currently we inject the full user profile graph in context. This doesn't scale as the profile grows."

### Solution: On-Demand Retrieval Tool

```
┌─────────────────────────────────────────────────┐
│ AGENT PROMPT (no full profile)                  │
│ "You are a helpful assistant."                  │
└─────────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────┐
│ AGENT REASONING                                 │
│ "I need user preferences. Call user_profile."   │
└─────────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────┐
│ TOOL: user_profile(query: str) → str            │
│                                                 │
│ 1. Vector search → semantic match              │
│ 2. Graph traversal → related nodes             │
│ 3. Format for agent                            │
└─────────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────┐
│ AGENT GENERATES RESPONSE                        │
│ Uses retrieved context only                     │
└─────────────────────────────────────────────────┘
```

### Benefits

| Before (Profile Injection) | After (On-Demand Tool) |
|---------------------------|------------------------|
| Full graph in every prompt | Only relevant facts |
| Token cost: O(n) per agent | Token cost: O(k) per query |
| Fails at 200+ nodes | Scales to 10K+ nodes |
| All agents see all data | Agents see only queried data |

---

## Implementation Roadmap

### Phase 1: Current State (50 nodes)
```python
# Pass full profile - works fine
def agent_prompt():
    return f"""
    You are helpful. User profile:
    {json.dumps(user_graph)}
    """
```

### Phase 2: Migration Trigger (>200 nodes)
- Profile doesn't fit in context
- Token costs rising
- Adding new agents becomes expensive

### Phase 3: Hybrid Tool Implementation
```python
def user_profile(query: str) -> str:
    """Tool for agents to query user context."""
    # Vector search
    nodes = vector_db.search(query, k=5)
    
    # Graph traversal
    related = graph.traverse(nodes, depth=2)
    
    return format_for_agent(nodes, related)
```

### Phase 4: GraphRAG (10K+ nodes)
```python
def user_profile(query: str) -> str:
    """Use pre-computed community summaries."""
    # Find relevant communities
    communities = vector_db.search(query, k=3)
    
    # Get pre-computed summaries (not raw nodes)
    summaries = [c.summary for c in communities]
    
    # Fallback: detailed lookup if needed
    if need_details(query):
        details = graph.lookup(query)
        return combine(summaries, details)
    
    return summaries
```

---

## When to Use Each Pattern

| Scenario | Pattern | Why |
|----------|---------|-----|
| 50 nodes, 5 agents | Direct profile injection | Simple, works |
| 500 nodes, 20 agents | Vector + Graph tool | Selective retrieval |
| 10K nodes, 50 agents | GraphRAG | Pre-computed summaries |
| Unstructured text search | Vector + FTS5 | Documents, not graphs |

---

## Key Takeaways

1. ✅ **Hybrid search = two retrieval methods combined**
2. ✅ **Backend depends on data structure** (FTS5 for text, Graph for nodes)
3. ✅ **On-demand retrieval scales better** than profile injection
4. ✅ **Tool pattern** (agent calls tool) is the implementation
5. ✅ **Plan migration** before hitting token limits

---

## Related Research

- [research/ai-agent-architectures.md](../research/ai-agent-architectures.md) - MCP, LangGraph patterns
- [research/clawdbot-memory-system.md](../research/clawdbot-memory-system.md) - Clawdbot's implementation

---

## Tags

#hybrid-search #vector #graph #user-profiles #retrieval #RAG
