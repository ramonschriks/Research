# Clawdbot Memory System Learnings

**Source:** research/clawdbot-memory-system.md  
**Extracted:** 2026-01-26

---

## Key Learnings

### 1. Files are Source of Truth
- Clawdbot memory = plain Markdown files in workspace
- Model only "remembers" what gets written to disk
- Don't rely on context window for durable memories

### 2. Dual-Layer Memory Design
| Layer | File | Purpose |
|-------|------|---------|
| Daily | `memory/YYYY-MM-DD.md` | Running session context |
| Long-term | `MEMORY.md` | Curated preferences/facts |

### 3. Automatic Memory Flush
- Triggers before context compaction
- Silent operation (NO_REPLY) so user never sees it
- Saves durable memories to disk automatically

### 4. Vector Search for Semantic Recall
- Semantic search across memory files
- Hybrid search (BM25 + vector) for best results
- Can use OpenAI, Gemini, or local embeddings

### 5. Memory Search Tools
- `memory_search` - Semantic query
- `memory_get` - Read specific file

---

## Actionable Takeaways

1. ✅ Write decisions to memory proactively
2. ✅ Curate MEMORY.md with durable preferences
3. ✅ Trust automatic flush but don't rely on it alone
4. ✅ Use memory_search when unsure about preferences

---

## Tags
#memory #clawdbot #persistence #ai
