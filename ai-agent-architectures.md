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

#### Memory/Persistence
- Built-in conversation history
- Long-term context across sessions
- State management

#### Streaming
- Token-by-token output
- Real-time reasoning visibility
- Intermediate step streaming

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

## 6. Architecture Comparison

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

## 7. Common Anti-Patterns

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

## 8. Recommended Stack for Personal AI Assistant

Based on this research, optimal architecture for a personal AI like Clawd:

```
┌─────────────────────────────────────────────────────┐
│           Personal AI Assistant                      │
├─────────────────────────────────────────────────────┤
│  ┌─────────┐    ┌──────────┐    ┌──────────────┐  │
│  │ MCP     │───▶│ LangGraph│───▶│ Memory      │  │
│  │ Registry│    │ Workflow │    │ (Files +    │  │
│  │         │    │          │    │ RAG)        │  │
│  └─────────┘    └──────────┘    └──────────────┘  │
│       │               │               │            │
│       ▼               ▼               ▼            │
│  ┌─────────────────────────────────────────┐  │
│  │         Context Injection Layer          │  │
│  └─────────────────────────────────────────┘  │
│                      │                         │
│                      ▼                         │
│  ┌─────────────────────────────────────────┐  │
│  │         LLM (MiniMax, Claude, GPT)      │  │
│  └─────────────────────────────────────────┘  │
│                                              │
│  Channels: Telegram, Terminal, etc.            │
└─────────────────────────────────────────────────────┘
```

### Components

| Component | Purpose | Implementation |
|-----------|---------|---------------|
| **MCP** | Tool standardization | Custom or existing |
| **LangGraph** | Workflow orchestration | For complex tasks |
| **Memory Files** | Long-term memory | SOUL.md, USER.md, memory/ |
| **RAG** | Document retrieval | Semantic search |
| **Prompts** | Versioned behavior | Prompt specs |

---

## 9. Standards Timeline

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

## 10. Key Takeaways

1. **MCP is the future** for tool/data standardization
2. **LangGraph** dominates complex agent workflows
3. **Memory layers** are essential for persistent assistants
4. **Prompts > Assistants** for versionable behavior
5. **Human-in-the-loop** prevents agent runaway
6. **RAG + Memory** beats bigger context windows

---

## References

- MCP: modelcontextprotocol.io
- LangGraph: langchain.com/langgraph
- OpenAI Responses: platform.openai.com/docs/guides/responses-vs-chat-completions
- ReAct Pattern: arxiv.org/abs/2210.03629

---

*Research conducted: 2026-01-26*  
*Next review: 2026-02-26*  
*Status: Active*
