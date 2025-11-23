# Framework Instrumentation Documentation

> **Category**: AI Frameworks
> **Total Packages**: 6
> **Completed**: 1/6 (17%)

---

## 📊 Documentation Status

| Framework | Package | Status | Session | Exploration | Priority |
|-----------|---------|--------|---------|-------------|----------|
| **LangChain** | `opentelemetry-instrumentation-langchain` | ✅ Complete | 3 | ✅ Complete | 🔴 MUST |
| **LlamaIndex** | `opentelemetry-instrumentation-llamaindex` | 📋 TODO | 4 | 📋 TODO | 🟡 Medium |
| **Haystack** | `opentelemetry-instrumentation-haystack` | 📋 TODO | 4 | 📋 TODO | 🟢 Lower |
| **CrewAI** | `opentelemetry-instrumentation-crewai` | 📋 TODO | 4 | 📋 TODO | 🟡 Medium |
| **OpenAI Agents** | `opentelemetry-instrumentation-openai-agents` | 📋 TODO | 4 | 📋 TODO | 🟡 Medium |
| **MCP** | `opentelemetry-instrumentation-mcp` | 📋 TODO | 4 | 📋 TODO | 🟢 Lower |

---

## 📋 Quick Reference

### Documentation Files

- **Tier 1 (Session 3)**:
  - `langchain.md` - LangChain, LCEL, LangGraph (MUST)

- **Tier 2** (Future Sessions):
  - `llamaindex.md` - LlamaIndex workflows
  - `crewai.md` - CrewAI multi-agent
  - `openai-agents.md` - OpenAI Agents SDK
  - `haystack.md` - Haystack pipelines
  - `mcp.md` - Model Context Protocol

### Exploration Resources

- **Exploration Plan**: [`_EXPLORATION_PLAN.md`](./_EXPLORATION_PLAN.md) - Analysis recipe for frameworks
- **Root Guide**: [`../EXPLORATION_GUIDE.md`](../EXPLORATION_GUIDE.md) - General exploration methodology

---

## 🎯 Next Session TODO (Session 3)

### Priority:
**langchain.md** - Most popular framework
- **Components Instrumented**:
  - Chains (all types, LCEL)
  - LLMs and Chat Models
  - Tools and Agents
  - LangGraph (state machines, workflows)
- **Special Features**:
  - LCEL (LangChain Expression Language) - full support
  - LangGraph - state machines, nodes, edges
  - Streaming (sync/async)
  - Async/concurrent execution
- **Gaps**:
  - Retrievers (only error tracking - document workaround)
- **Samples**:
  - `langchain_app.py` - Basic chain
  - `langchain_lcel.py` - LCEL pipelines
  - `langchain_agent.py` - Agents with tools
  - `langgraph_example.py` - LangGraph workflows
  - `langgraph_openai.py` - LangGraph + OpenAI

### Exploration Status
✅ Deep dive analysis complete for LangChain in Session 1
- Callback-based instrumentation
- Span hierarchy documented
- LCEL and LangGraph support analyzed
- All samples tested

---

## 📚 Template & Standards

### Use This Structure for Each Framework Doc:

```markdown
# {Framework} Instrumentation

## Table of Contents
- Overview
- Quick Start
- What's Auto-Instrumented (Components)
- Instrumentation Approach (Callbacks/Wrapping)
- Span Hierarchy (Visualization)
- What's Captured (Spans, Attributes, Metrics)
- Special Features (DSL, Graphs, Streaming, Async)
- Gaps & Limitations
- Manual Instrumentation Patterns
- Configuration Options
- Examples & Code Snippets
- Sample Applications Reference
- Common Patterns
- Troubleshooting
```

### Framework-Specific Focus:
- **Span Hierarchy**: Visualize workflow → task → tool structure
- **Component Coverage**: What's instrumented vs what needs manual work
- **Instrumentation Pattern**: How it's implemented (callbacks, wrapping, etc.)
- **Integration Examples**: With LLM providers, vector DBs

### Quality Checklist:
- [ ] All auto-instrumented components listed
- [ ] Instrumentation approach explained
- [ ] Span hierarchy visualized with examples
- [ ] Complete attribute reference
- [ ] Special features (DSL, graphs, streaming, async)
- [ ] Gaps clearly documented with workarounds
- [ ] Manual instrumentation patterns with code
- [ ] All sample apps referenced
- [ ] Integration patterns documented

---

## 🔍 Framework Comparison

| Feature | LangChain | LlamaIndex | CrewAI | Haystack |
|---------|-----------|------------|--------|----------|
| Chains/Workflows | ✅ Full | ✅ Full | ✅ Agents | ✅ Pipelines |
| Tools | ✅ Full | ✅ Full | ✅ Full | ✅ Tools |
| Retrievers | ⚠️ Errors only | ? | ? | ? |
| LCEL/DSL | ✅ LCEL | ✅ Workflows | - | ✅ Pipelines |
| Graph Engine | ✅ LangGraph | - | - | - |
| Streaming | ✅ Full | ? | ? | ? |
| Async | ✅ Full | ? | ? | ? |

*(? = Not yet explored)*

---

## 🔗 Related Resources

- **Main Progress Tracker**: [`../../PROGRESS.md`](../../PROGRESS.md)
- **Examples Index**: [`../../examples-index.md`](../../examples-index.md)
- **SDK Guide**: [`../../traceloop-sdk-guide.md`](../../traceloop-sdk-guide.md)
- **Sample Apps**:
  - LangChain: `langchain_*.py`, `langgraph_*.py`
  - LlamaIndex: `llama_index_*.py`, `llama_parse_*.py`
  - CrewAI: `crewai_*.py`
  - Haystack: `haystack_*.py`
  - OpenAI Agents: `openai_agents_*.py`
  - MCP: `mcp_*.py`

---

## 📝 Status Legend

- ✅ **Complete** - Documentation finished, reviewed
- 🚧 **In Progress** - Currently being written
- 📋 **TODO** - Not started yet

**Exploration Status:**
- ✅ **Complete** - Deep analysis finished
- 🚧 **In Progress** - Being analyzed
- 📋 **TODO** - Not yet explored

---

**Last Updated**: 2025-11-22 (Session 1)
