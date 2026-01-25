---
title: AI Agent Architectures - Standards & Frameworks Research
status: Active
lastVerified: 2026-01-26
nextReview: 2026-02-26
category: AI/Architecture
---

# AI Agent Architectures - Standards & Frameworks Research

**Status:** Active  
**Last Verified:** 2026-01-26  
**Next Review:** 2026-02-26  
**Category:** AI/Architecture  

---

## Executive Summary

This report explores the major standards, protocols, and frameworks shaping AI agent architecture in 2026. Key findings:

- **MCP (Model Context Protocol)** - Emerging standard for tool/data connection (Anthropic)
- **LangGraph** - Popular open-source framework for complex agent workflows
- **OpenAI Responses API** - Simplified mental model replacing Assistants API
- **ReAct Pattern** - Foundation for reasoning + acting agents
- **RAG + Memory** - Context augmentation strategies

---

## 1. Model Context Protocol (MCP)

**Source:** modelcontextprotocol.io  
**Type:** Open standard for AI-data/tool connection  
**Analogy:** "USB-C for AI applications"

### What It Does

MCP provides a standardized way for AI applications to connect to:
- **Data sources:** Local files, databases, calendars
- **Tools:** Search engines, calculators, custom APIs
- **Workflows:** Specialized prompts, automation

### Key Benefits

| Stakeholder | Benefit |
|-------------|---------|
| Developers | Reduces integration time and complexity |
| AI Applications | Access to ecosystem of data/tools |
| End Users | More capable agents that can access data/take actions |

### Use Cases

1. Agent accesses Google Calendar AND Notion simultaneously
2. Claude Code generates web app from Figma design
3. Enterprise chatbot queries multiple databases
4. AI model creates 3D designs and prints them

### Architecture

```
┌─────────────┐     MCP      ┌──────────────────┐
│ AI Agent    │────────────▶│ Data Sources    │
│ (Claude,    │             │ Files, DB, APIs │
│ ChatGPT)    │◀────────────│                 │
└─────────────┘             └──────────────────┘
       │                           ▲
       │                           │
       └───────────┬─────────────┘
                   │
              Tools/Actions
```

### Comparison

**Before MCP:** Custom integration for each data source  
**After MCP:** One standard, plug-and-play connections

---

## 2. LangGraph

**Source:** langchain.com/langgraph  
**Type:** Open-source agent framework (MIT licensed)  
**Focus:** Complex, stateful, multi-agent workflows

### Core Philosophy

> "Controllable cognitive architecture for any task"

### Key Features

#### Control Flows Supported
- Single agent
- Multi-agent
- Hierarchical
- Sequential

#### Human-in-the-Lop
- Drafts for review
- Approval checkpoints
- "Time-travel" to rollback/redo

#### Streaming
- Token-by-token output
- Real-time reasoning visibility
- Intermediate step streaming

### ⚠️ Important: What LangGraph Is NOT

**LangGraph is NOT for:**
- ❌ Long-term memory storage
- ❌ Document retrieval
- ❌ Persistent memories across sessions
- ❌ Versioned prompt management

**LangGraph IS for:**
- ✅ Orchestrating agent decisions/actions during a task
- ✅ Managing state between workflow steps
- ✅ Control flow (what step next?)
- ✅ Human checkpoints during execution

**Where memory lives:**
- LangGraph → ephemeral state during task
- MCP/RAG → external data retrieval
- Versioned Prompts → behavioral configuration
- Memory Layers → long-term persistence

### How It Differs From Other Frameworks

| Framework | Complexity | Flexibility | Best For |
|-----------|------------|-------------|----------|
| LangGraph | Medium | High | Complex, custom workflows |
| Simple Agents | Low | Low | Simple, generic tasks |
| Closed Systems | N/A | Low | Single-purpose agents |

### Key Concepts

1. **Nodes** - Agent actions or decisions
2. **Edges** - Transitions between nodes
3. **State** - Shared context across workflow
4. **Interrupts** - Human approval checkpoints

### Code Example Pattern

```python
from langgraph.graph import StateGraph

# Define state
class AgentState(TypedDict):
    messages: list
    context: dict
    next_action: str

# Build graph
graph = StateGraph(AgentState)
graph.add_node("analyze", analyze_step)
graph.add_node("reason", reason_step)
graph.add_node("act", act_step)

# Add edges
graph.add_edge("analyze", "reason")
graph.add_conditional_edges("reason", should_act)

app = graph.compile()
```

---

## 3. OpenAI Responses API (2026)

**Source:** platform.openai.com/docs/guides/responses-vs-chat-completions  
**Type:** API paradigm shift (Assistants → Responses)

### What Changed

| Old (Assistants) | New (Responses) |
|-----------------|-----------------|
| Assistants (persistent objects) | Prompts (versioned configs) |
| Threads (message storage) | Conversations (item streams) |
| Runs (async execution) | Responses (direct execution) |
| Messages only | Messages + Tool calls + Outputs |

### Mental Model Shift

```
OLD (Assistant-based):
Assistant → Thread → Run → Steps → Messages

NEW (Response-based):
Prompt (versioned) → Conversation → Response → Items
```

### Why The Change

1. **Portability** - Prompts can be versioned, reviewed, rolled back
2. **Separation of Concerns** - Code handles orchestration, prompts handle behavior
3. **Consistency** - Same prompt works across chat, streaming, realtime
4. **Tool Consistency** - Schemas embedded in prompts

### Migration Path

```
1. Identify Assistants
2. Create Prompts in dashboard
3. Store prompt IDs in source control
4. Use Conversations + Responses API
```

---

## 4. ReAct Pattern

**Source:** General agent pattern  
**Type:** Reasoning + Acting foundation

### What It Is

ReAct = **Re**asoning + **Ac**tion

Agents that:
1. **Think** about what to do
2. **Act** by using tools
3. **Observe** the result
4. **Repeat** until done

### Pattern Flow

```
Thought: "I need to check the weather"
Action: call_weather_tool(lat, lon)
Observation: "Sunny, 22°C"
Thought: "Good weather for outdoor activities"
Final Answer: "It's sunny and 22°C today!"
```

---

## 5. RAG + Memory Architecture

**Type:** Context augmentation strategy

### Layered Memory

| Layer | Purpose | Update Frequency |
|-------|---------|-----------------|
| **Working Memory** | Current context | Per message |
| **Short-Term** | Session history | Per session |
| **Long-Term** | Curated memories | Manual distillation |
| **External** | Documents/files | On retrieval |

### RAG Flow

```
Query → Embed → Vector Search → Relevant Docs → Context → Response
```

### Memory Consolidation

```
Session Memory (daily logs)
     ↓ (weekly distillation)
Long-Term Memory (curated)
     ↓ (semantic search)
Agent Context
```

---

## 6. Long-Term Memory Management with RAG

**Goal:** Efficiently retrieve relevant memories without wasting tokens on irrelevant data.

### The Token Problem

| Approach | Tokens Used | Problem |
|----------|-------------|---------|
| Dump everything | All memories | Context overflow, high cost |
| No memory | Zero | Agent forgets everything |
| RAG (smart) | Only relevant | ✅ Optimal |

### How RAG Solves This

```
Query: "What are my financial principles?"
         │
         ▼
┌─────────────────────────────────────┐
│  1. Embed Query                     │
│     → Vector representation         │
└─────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────┐
│  2. Vector Search                   │
│     → Find similar memories         │
│     → Top 3-5 most relevant         │
└─────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────┐
│  3. Inject Only Relevant Memories   │
│     → Few KB instead of MB          │
└─────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────┐
│  4. Response                        │
│     → Informed by context           │
└─────────────────────────────────────┘
```

### Efficiency Techniques for Massive Datasets

#### 1. Semantic Chunking

**Don't:** Store entire documents  
**Do:** Split into meaningful chunks (paragraphs, sections)

```text
BAD:  "Here's my entire USER.md (10KB)"
      ↑ Wastes tokens on irrelevant sections

GOOD: "USER.md → Financial Principles section (200 bytes)"
      ↑ Only relevant memory
```

#### 2. Hierarchy & Tags

```
memory/
├── 2026-01-26.md      (session logs)
├── 2026-01-25.md
├── USER.md            (preferences)
├── MEMORY.md          (curated)
└── knowledge/         (patterns)
    ├── patterns/
    ├── mistakes/
    └── glossary/
```

**Search strategy:**
1. Query → Check memory file dates
2. Query → Check MEMORY.md (curated)
3. Query → Check knowledge/ (patterns)
4. Query → Session logs (if needed)

#### 3. Semantic Search (Vector DB)

| Tool | Cost | Complexity |
|------|------|------------|
| Local embeddings | Free | Medium |
| Pinecone | Paid | Low |
| Weaviate | Free | Medium |
| Chroma | Free | Low |

**Flow:**
```python
# Indexing (once)
for doc in memory_files:
    embed(doc) → store in vector_db

# Querying (each session)
query = "financial preferences"
results = vector_search(query, top_k=3)
inject(results)
```

#### 4. Caching Frequently Used Memories

**Hot memories** (used every session):
- USER.md
- SOUL.md
- AGENTS.md

**Cold memories** (rarely used):
- Old session logs
- Archived patterns

**Strategy:**
```python
if memory in ["USER.md", "SOUL.md"]:
    # Always inject
    inject(memory)
else:
    # RAG search
    results = semantic_search(query)
    inject(results)
```

#### 5. Summarization for Old Memories

**Don't:** Keep raw logs forever  
**Do:** Distill old logs into condensed memories

```
Week 1: memory/2026-01-20.md (5KB raw)
Week 4: Extract key insights → MEMORY.md (500 bytes)
        Archive raw log
Month 12: MEMORY.md summary only
```

### Recommended Memory Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    MEMORY LAYERS                         │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌─────────────────────────────────────────────────┐    │
│  │ LAYER 1: Always Loaded (Hot)                    │    │
│  │ - USER.md (preferences)                         │    │
│  │ - SOUL.md (identity)                            │    │
│  │ - AGENTS.md (conventions)                       │    │
│  │ Size: ~5KB total                                │    │
│  └─────────────────────────────────────────────────┘    │
│                        │                                 │
│                        ▼                                 │
│  ┌─────────────────────────────────────────────────┐    │
│  │ LAYER 2: RAG Retrieval (Conditional)            │    │
│  │ - MEMORY.md (curated long-term)                 │    │
│  │ - knowledge/ patterns                           │    │
│  │ - research/                                     │    │
│  │ Size: Variable (inject ~2-5KB)                  │    │
│  └─────────────────────────────────────────────────┘    │
│                        │                                 │
│                        ▼                                 │
│  ┌─────────────────────────────────────────────────┐    │
│  │ LAYER 3: Session Logs (Archive)                 │    │
│  │ - memory/YYYY-MM-DD.md                          │    │
│  │ - Rarely queried directly                       │    │
│  │ - Distilled to LAYER 2 over time                │    │
│  │ Size: Growing (archived)                        │    │
│  └─────────────────────────────────────────────────┘    │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### Token Budget Example

| Component | Tokens (approx) | Frequency |
|-----------|-----------------|-----------|
| USER.md | 500 | Every session |
| SOUL.md | 300 | Every session |
| AGENTS.md | 400 | Every session |
| RAG results | 1000 | Per query type |
| Working memory | 2000 | Per message |
| **Total** | **~4,200** | Well under limits |

### Summary: RAG for Memory

| Principle | Implementation |
|-----------|---------------|
| **Chunk, don't dump** | Split memories into meaningful pieces |
| **Hierarchy matters** | Hot/Cold/Layered approach |
| **Embed for search** | Vector similarity, not keyword |
| **Cache hot memories** | USER.md always loaded |
| **Distill old logs** | Raw → Curated over time |
| **Limit context** | Only relevant 3-5 memories |

---

## 7. Architecture Comparison

### By Use Case

| Use Case | Recommended Architecture |
|----------|----------------------|
| Simple chatbot | Direct API (no framework) |
| Tool-using agent | MCP + simple orchestration |
| Multi-step workflow | LangGraph |
| Multi-agent system | LangGraph + MCP |
| Enterprise integration | MCP + RAG + Memory |

### By Complexity

```
Simple          Medium          Complex
  │               │               │
  ▼               ▼               ▼
Chat API    LangGraph       Multi-agent
   │           │               │
   │       Prompts       MCP + LangGraph
   │           │               │
   └───────┴───────┴───────────┘
             │
             ▼
       RAG + Memory (all levels)
```

---

## 8. Common Anti-Patterns

### What NOT To Do

1. **No memory persistence** - Agent forgets everything each session
2. **Everything in system prompt** - Context window overflow
3. **Hardcoded tools** - No extensibility
4. **No human checkpoints** - Agents run wild
5. **Single context window** - No long-term learning

### What TO Do

1. ✅ Use MCP for tool standardization
2. ✅ Version prompts/configs
3. ✅ Implement memory layers
4. ✅ Add human-in-the-loop for critical actions
5. ✅ Monitor agent behavior (LangSmith, similar)

---

## 9. Recommended Stack for Personal AI Assistant

Based on this research, optimal architecture for a personal AI like Clawd:

```
┌─────────────────────────────────────────────────────────────┐
│           Personal AI Assistant                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────────┐ │
│  │ MCP         │───▶│ LangGraph   │───▶│ RAG + Memory    │ │
│  │ Registry    │    │ Workflow    │    │ (Files + Search)│ │
│  └─────────────┘    └─────────────┘    └─────────────────┘ │
│         │                  │                    │           │
│         ▼                  ▼                    ▼           │
│  ┌─────────────────────────────────────────────────────┐   │
│  │         Versioned Prompts (Behavior Config)         │   │
│  │         - System instructions                       │   │
│  │         - Tool definitions                          │   │
│  │         - Temperature, schema                       │   │
│  └─────────────────────────────────────────────────────┘   │
│                            │                               │
│                            ▼                               │
│  ┌─────────────────────────────────────────────────────┐   │
│  │         Context Injection Layer                      │   │
│  │         - Hot memories (USER.md, SOUL.md)           │   │
│  │         - RAG results (curated memories)            │   │
│  │         - Working memory (conversation)             │   │
│  └─────────────────────────────────────────────────────┘   │
│                            │                               │
│                            ▼                               │
│  ┌─────────────────────────────────────────────────────┐   │
│  │         LLM (MiniMax, Claude, GPT)                  │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                              │
│  Channels: Telegram, Terminal, etc.                          │
└─────────────────────────────────────────────────────────────┘
```

### Components (Clarified)

| Component | Purpose | What It Is NOT |
|-----------|---------|----------------|
| **MCP** | Tool standardization | Not workflow orchestration |
| **LangGraph** | Workflow orchestration | NOT memory management |
| **Memory Files** | Long-term persistence | Raw files, needs RAG layer |
| **RAG** | Smart retrieval | Needs vector DB or search |
| **Prompts** | Versioned behavior | NOT memory storage |

### Separation of Concerns

```
┌──────────────────────────────────────────────────────────┐
│  ORCHESTRATION (LangGraph)                               │
│  "What step next? What's the state?"                     │
└──────────────────────────────────────────────────────────┘
                           │
                           ▼ (Uses these)
┌──────────────────────────────────────────────────────────┐
│  BEHAVIOR (Prompts)                                      │
│  "Who am I? How do I behave?"                            │
└──────────────────────────────────────────────────────────┘
                           │
                           ▼ (Retrieves from)
┌──────────────────────────────────────────────────────────┐
│  MEMORY (RAG + Files)                                    │
│  "What do I remember? What's relevant?"                  │
└──────────────────────────────────────────────────────────┘
                           │
                           ▼ (Consumes)
┌──────────────────────────────────────────────────────────┐
│  LLM (Model)                                             │
│  "Generate response"                                     │
└──────────────────────────────────────────────────────────┘
```

---

## 10. Standards Timeline

```
2024          2025          2026
  │             │             │
  │             │     ┌──────┴──────┐
  │             │     │             │
Assistants    LangGraph   MCP + A2A
  API         matures    standardization
  │             │             │
  └─────┬───────┴──────┬──────┘
        │               │
        ▼               ▼
   Responses API   Open standards
   (simplified)   (MCP, A2A, etc.)
```

---

## 11. Key Takeaways

1. **MCP is the future** for tool/data standardization
2. **LangGraph** dominates complex agent workflows (orchestration, NOT memory)
3. **Memory layers** are essential for persistent assistants
4. **Prompts > Assistants** for versionable behavior
5. **Human-in-the-loop** prevents agent runaway
6. **RAG + Memory** beats bigger context windows
7. **LangGraph ≠ Memory** - separation of concerns critical

---

## 12. References

- MCP: modelcontextprotocol.io
- LangGraph: langchain.com/langgraph
- OpenAI Responses: platform.openai.com/docs/guides/responses-vs-chat-completions
- ReAct Pattern: arxiv.org/abs/2210.03629

---

*Research conducted: 2026-01-26*  
*Next review: 2026-02-26*  
*Status: Active*
