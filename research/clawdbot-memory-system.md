---
title: Clawdbot Memory System - How AI Assistants Remember
status: Active
lastVerified: 2026-01-26
nextReview: 2026-04-26
category: AI/Architecture
version: 1.0
---

# Clawdbot Memory System - How AI Assistants Remember

**Status:** Active  
**Last Verified:** 2026-01-26  
**Next Review:** 2026-04-26  
**Category:** AI/Architecture  
**Version:** 1.0

---

## Executive Summary

This research explores Clawdbot's memory architecture - a dual-layer system combining flat Markdown files for durability with vector search for semantic recall. The key innovation is automatic memory flush before context compaction, ensuring durable memories survive session resets. For personal AI assistants like Clawd, this architecture solves the fundamental tension between stateless agents and persistent relationships.

> **Core Insight:** Clawdbot's memory is "plain Markdown in the agent workspace" - the files are the source of truth, not the model's context window.

---

## 1. Introduction

### Purpose

This research documents how Clawdbot implements persistent memory for AI assistants. The goal is to understand the architecture so Clawd can leverage it effectively for remembering Ramon's preferences, project context, and long-term learnings.

### Scope

This document covers:
- Memory file structure (MEMORY.md and daily logs)
- Automatic memory flush before compaction
- Vector search with semantic recall
- Hybrid search (BM25 + vector)
- Session memory indexing (experimental)

This document does NOT cover:
- Channel-specific memory (WhatsApp, Telegram, etc.)
- External database integrations
- Memory encryption or security

### Methodology

Research was gathered from:
- Clawdbot documentation (docs.clawd.bot/concepts/memory)
- Clawdbot configuration examples
- Personal experimentation with Clawd's memory tools

### Audience

This document is intended for:
- Ramon (understanding how Clawd remembers)
- Clawd agents (how to use memory effectively)
- Anyone building persistent AI assistants

---

## 2. Memory Architecture

### Two-Layer Design

Clawdbot uses a simple but powerful dual-layer memory system:

| Layer | File | Purpose | Load Behavior |
|-------|------|---------|---------------|
| **Daily** | `memory/YYYY-MM-DD.md` | Running context, session logs | Today + yesterday at session start |
| **Long-term** | `MEMORY.md` | Curated durable memories | Main session only (never group chats) |

**Key Principle:** Both layers are plain Markdown files in the workspace. The model "remembers" only what gets written to disk.

### Why Markdown?

```
┌─────────────────────────────────────────────────────────────┐
│                    Workspace (~/clawd)                       │
├─────────────────────────────────────────────────────────────┤
│  📄 MEMORY.md              ← Curated long-term memory       │
│  📁 memory/                                                  │
│     ├── 2026-01-26.md       ← Today's session logs          │
│     └── 2026-01-25.md       ← Yesterday's context           │
└─────────────────────────────────────────────────────────────┘
```

Benefits:
- **Human-readable** - Can be edited directly
- **Version-controlled** - Git tracks changes
- **Portable** - Simple file copy
- **No lock-in** - No proprietary database required

### When to Write Memory

| Memory Type | File | Examples |
|-------------|------|----------|
| **Decisions** | MEMORY.md | "Ramon prefers concise responses" |
| **Preferences** | MEMORY.md | "No corporate fluff, casual tone" |
| **Durable facts** | MEMORY.md | "Ramon works in Europe/Amsterdam" |
| **Day-to-day notes** | memory/YYYY-MM-DD.md | "Working on accumulator-betting skill" |
| **Running context** | memory/YYYY-MM-DD.md | "Today's tasks: X, Y, Z" |

**Rule:** If someone says "remember this," write it to disk. Don't keep it in RAM.

---

## 3. Automatic Memory Flush

### The Compaction Problem

AI assistants have limited context windows. When sessions approach their limit, Clawdbot triggers **auto-compaction** - summarizing and truncating the conversation history.

**Problem:** If the model hasn't written important memories to disk, they're lost during compaction.

### Solution: Memory Flush

Clawdbot solves this with automatic memory flush:

```json
{
  "agents": {
    "defaults": {
      "compaction": {
        "reserveTokensFloor": 20000,
        "memoryFlush": {
          "enabled": true,
          "softThresholdTokens": 4000,
          "systemPrompt": "Session nearing compaction. Store durable memories now.",
          "prompt": "Write any lasting notes to memory/YYYY-MM-DD.md; reply with NO_REPLY if nothing to store."
        }
      }
    }
  }
}
```

### How It Works

```
Session starts → Context fills up → Nears compaction threshold
                    ↓
           Memory flush triggers
                    ↓
    System prompts: "Store durable memories now"
                    ↓
    Agent writes to memory/YYYY-MM-DD.md
                    ↓
    Agent replies NO_REPLY (user never sees this)
                    ↓
    Compaction proceeds → Memories are safe on disk
```

**Silent by default:** The flush uses `NO_REPLY` so the user never sees this background turn.

### Implications for Clawd

1. **Write proactively** - When making decisions or learning preferences, write to memory immediately
2. **Use the daily file** - For session-specific context, use `memory/YYYY-MM-DD.md`
3. **Curate for MEMORY.md** - Only durable, cross-session memories belong in MEMORY.md

---

## 4. Vector Memory Search

### Semantic Recall

Beyond flat files, Clawdbot can build a **vector index** over memory files, enabling semantic search:

```
Query: "What does Ramon prefer for responses?"
           ↓
    Vector search finds related memory chunks
           ↓
    Returns snippets with file + line ranges
```

### Configuration Options

| Provider | Setup | Pros | Cons |
|----------|-------|------|------|
| **OpenAI** | `memorySearch.provider: "openai"` | Fast, batch support | API cost |
| **Gemini** | `memorySearch.provider: "gemini"` | Native support | API cost |
| **Local** | `memorySearch.provider: "local"` | Free, privacy | Requires GGUF model (~0.6GB) |

### Default Behavior

If no provider is configured, Clawdbot auto-selects:
1. Local (if `memorySearch.local.modelPath` is configured)
2. OpenAI (if API key available)
3. Gemini (if API key available)
4. Disabled (fallback)

### Tools Available

| Tool | Purpose |
|------|---------|
| `memory_search` | Semantic query across MEMORY.md + memory/*.md |
| `memory_get` | Read specific memory file by path |

**Security:** Both tools only access MEMORY.md and memory/ directories.

---

## 5. Hybrid Search (BM25 + Vector)

### Why Hybrid?

Vector search is great at "this means the same thing" but weak at exact tokens. BM25 (full-text search) is the opposite.

**Example:**
| Query | Vector Search | BM25 Search |
|-------|---------------|-------------|
| "Mac Studio gateway host" | ✅ Good | ❌ Poor |
| "a828e60" (ID) | ❌ Poor | ✅ Good |

### How Hybrid Works

Clawdbot combines both signals:

```
Query → Vector Search (top K by similarity)
     → BM25 Search (top K by keyword match)
     → Merge with weighted scores
     → Final ranked results
```

### Default Weights

```json
{
  "memorySearch": {
    "query": {
      "hybrid": {
        "enabled": true,
        "vectorWeight": 0.7,
        "textWeight": 0.3,
        "candidateMultiplier": 4
      }
    }
  }
}
```

**Result:** 70% semantic, 30% keyword - pragmatic for real-world notes.

---

## 6. Session Memory (Experimental)

### What It Is

Clawdbot can optionally index session transcripts, surfacing past conversations via `memory_search`.

```json
{
  "agents": {
    "defaults": {
      "memorySearch": {
        "experimental": { "sessionMemory": true },
        "sources": ["memory", "sessions"]
      }
    }
  }
}
```

### Tradeoffs

| Aspect | Benefit | Concern |
|--------|---------|---------|
| Recall | Can reference past conversations | Privacy (session logs on disk) |
| Context | Richer context across sessions | Storage and indexing overhead |
| Discovery | Find relevant past discussions | Potential for over-reliance |

### Security Note

Session logs live at `~/.clawdbot/agents/<agentId>/sessions/*.jsonl`. Any process with filesystem access can read them.

---

## 7. Recommended Implementation for Clawd

### For Ramon's Personal Assistant

| Component | Recommendation | Reason |
|-----------|----------------|--------|
| **Memory layer 1** | `memory/YYYY-MM-DD.md` | Daily logs for session context |
| **Memory layer 2** | `MEMORY.md` | Durable preferences and facts |
| **Search provider** | Local (gguf embedding) | Free, privacy, no API costs |
| **Session memory** | Disabled | Privacy-conscious default |
| **Memory flush** | Enabled (default) | Automatic backup before compaction |

### Workflow for Clawd

```
Session Start
    ↓
Read: MEMORY.md (if main session)
Read: memory/YYYY-MM-DD.md
Read: memory/YYYY-MM-DD (yesterday)
    ↓
During Session
    - Write decisions to memory/YYYY-MM-DD.md
    - Write durable facts to MEMORY.md
    - Use memory_search when unsure ("What does Ramon prefer?")
    ↓
Near Compaction
    - Automatic memory flush triggers
    - Write any pending notes
    - Reply NO_REPLY
    ↓
Session End
    - Durable memories preserved on disk
```

### Best Practices

1. **Write early, write often** - Don't wait for memory flush
2. **Be specific** - "Ramon prefers bullet points" > "Ramon has preferences"
3. **Use semantic labels** - Keywords help both human and vector search
4. **Curate MEMORY.md** - Only durable, verified information
5. **Trust the system** - Memory flush will catch what you miss

---

## Key Takeaways

1. ✅ **Dual-layer memory** - Daily logs (session) + curated (long-term)
2. ✅ **Markdown files** - Human-readable, version-controlled, portable
3. ✅ **Automatic flush** - Memories saved before context compaction
4. ✅ **Vector search** - Semantic recall across memory files
5. ✅ **Hybrid search** - Combines semantic + keyword for better results
6. ✅ **Silent operation** - Memory flush uses NO_REPLY (user never sees it)
7. ✅ **Write proactively** - Don't rely on automatic flush alone

---

## Common Mistakes

- ❌ **Keeping memories in context** - If it's not on disk, it's lost on compaction
- ❌ **Writing everything to MEMORY.md** - Reserve for durable, curated memories
- ❌ **Ignoring memory_search** - Use it when unsure about preferences
- ❌ **Vague memory entries** - Specific, actionable memories are more useful

---

## References

1. [Clawdbot Memory Documentation](https://docs.clawd.bot/concepts/memory) - Primary source
2. [Clawdbot GitHub](https://github.com/clawdbot/clawdbot) - Implementation details
3. [Manthan Gupta Tweet](https://x.com/manthanguptaa/status/2015780646770323543) - Original article inspiration

---

## Appendix A: Glossary

| Term | Definition |
|------|------------|
| **Auto-compaction** | Automatic summarization and truncation of session history when context fills |
| **BM25** | Full-text search algorithm (keyword matching) |
| **Memory flush** | Automatic backup of memories before compaction |
| **Vector search** | Semantic similarity matching using embeddings |
| **Hybrid search** | Combination of vector + BM25 for better recall |

---

## Audio Summary (NotebookLM Ready)

**Intro:** This document covers Clawdbot's memory system for AI assistants, including the dual-layer architecture with daily logs and curated long-term memory, automatic memory flush before context compaction, and hybrid search combining semantic and keyword matching.

**Sections:**
1. Introduction - Purpose and scope of documenting Clawdbot's memory
2. Memory Architecture - Dual-layer design (MEMORY.md + daily logs)
3. Automatic Memory Flush - How memories are saved before compaction
4. Vector Memory Search - Semantic recall across memory files
5. Hybrid Search - Combining BM25 and vector for better results
6. Session Memory - Experimental feature for indexing conversations
7. Implementation - Recommended setup for Clawd

**Key Takeaways:**
- Files are the source of truth, not the model's context
- Write memories proactively, don't rely on automatic flush
- Use memory_search for semantic recall across sessions
- Hybrid search balances semantic understanding with exact matching

**Conclusion:** Clawdbot's memory architecture solves the fundamental problem of stateless AI agents by combining simple Markdown files with sophisticated vector search. For personal assistants like Clawd, this enables persistent relationships without complex database infrastructure.

---

## Standards & Guidelines Applied

| Standard | Organization | Application |
|----------|--------------|-------------|
| **ANSI/NISO Z39.18** | NISO | Research report structure |
| **GLISC Guidelines** | Grey Literature | Clear section organization |
| **Google Technical Writing** | Google | Concise, actionable prose |

---

*Research document created: 2026-01-26*  
*Based on: Clawdbot documentation and Manthan Gupta's article*  
*Template: research-template.md (ANSI/NISO Z39.18 compliant)*
