# AI Agent Architecture Learnings

**Source:** research/ai-agent-architectures.md  
**Extracted:** 2026-01-26

---

## Key Learnings

### 1. MCP is the New Standard for Tool Integration
- Model Context Protocol (MCP) = "USB-C for AI applications"
- Provides standardized protocol for connecting AI to external data/tools
- Single implementation works across multiple data sources

### 2. Orchestration vs Memory Separation
- LangGraph handles complex workflow orchestration
- RAG + Files handle memory management
- Keep concerns separate for better maintainability

### 3. OpenAI's Shift to Responses API
- Moving away from Assistants API to Responses API with versioned prompts
- Versioned prompts allow A/B testing and rollback

### 4. ReAct Pattern for Tool Use
- Think → Act → Observe loop
- Standard pattern for agents that use tools

### 5. Clawdbot's Architecture Model
- Pi agent (RPC mode) with tool streaming
- Multi-agent routing per sender
- Session-based context management

---

## Actionable Takeaways

1. ✅ Use MCP for future tool integrations
2. ✅ Keep orchestration (LangGraph) separate from memory
3. ✅ Consider versioned prompts for production
4. ✅ Implement ReAct pattern for tool-using agents

---

## Tags
#ai #agents #mcp #langgraph #architecture
