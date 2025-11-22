# Pattern Guides & Best Practices

> **Purpose**: Common patterns, best practices, and troubleshooting for OpenLLMetry instrumentation
> **Status**: WIP - To be developed in Sessions 3-4

---

## 📋 Pattern Guides

| Guide | Status | Description |
|-------|--------|-------------|
| `auto-vs-manual.md` | 📋 WIP | Decision guide: When to use auto vs manual instrumentation |
| `streaming.md` | 📋 WIP | Patterns for streaming LLM responses |
| `async-patterns.md` | 📋 WIP | Async/await and concurrent execution patterns |
| `troubleshooting.md` | 📋 WIP | Common issues and solutions |

---

## 🎯 Planned Content

### auto-vs-manual.md

**Decision Framework**:
- When automatic instrumentation is sufficient
- When manual spans are needed
- How to combine both approaches
- Best practices for complex workflows

**Topics**:
- Workflow orchestration
- Tool execution tracking
- Multi-step agent patterns
- RAG pipelines
- Custom business logic

### streaming.md

**Streaming Patterns**:
- LLM streaming (OpenAI, Anthropic, etc.)
- Framework streaming (LangChain, LlamaIndex)
- Span lifecycle in streaming
- Token counting in streams
- Error handling

**Topics**:
- Sync streaming
- Async streaming
- Buffering strategies
- Intermediate results
- Stream completion tracking

### async-patterns.md

**Async Patterns**:
- Context propagation in async code
- Concurrent LLM calls
- Parallel workflow execution
- Race conditions prevention
- Async generators

**Topics**:
- async/await basics
- Concurrent execution
- Context preservation
- Error handling
- Performance considerations

### troubleshooting.md

**Common Issues**:
- Missing spans
- Incomplete data capture
- Context detachment errors
- Memory leaks
- Performance overhead
- Privacy/PII concerns

**For Each Issue**:
- Symptoms
- Root cause
- Solution
- Prevention

---

## 🔗 Related Resources

- **Main Progress Tracker**: [`../PROGRESS.md`](../PROGRESS.md)
- **SDK Guide**: [`../traceloop-sdk-guide.md`](../traceloop-sdk-guide.md)
- **Examples Index**: [`../examples-index.md`](../examples-index.md)

---

## 📝 Status Legend

- ✅ **Complete** - Documentation finished, reviewed
- 🚧 **In Progress** - Currently being written
- 📋 **WIP** - Stub created, to be developed
- 📋 **TODO** - Not started yet

---

**Last Updated**: 2025-11-22 (Session 1)
