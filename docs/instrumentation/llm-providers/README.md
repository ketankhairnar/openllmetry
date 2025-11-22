# LLM Provider Instrumentation Documentation

> **Category**: LLM Providers
> **Total Packages**: 17
> **Completed**: 0/17 (0%)

---

## 📊 Documentation Status

### Tier 1 - Priority (Must Document First)

| Provider | Package | Status | Session | Exploration |
|----------|---------|--------|---------|-------------|
| **OpenAI** (incl. Azure) | `opentelemetry-instrumentation-openai` | ✅ Complete | 2 | ✅ Complete |
| **Anthropic** | `opentelemetry-instrumentation-anthropic` | ✅ Complete | 2 | ✅ Complete |
| **AWS Bedrock** | `opentelemetry-instrumentation-bedrock` | ✅ Complete | 2 | ✅ Complete |

### Tier 2 - Medium Priority

| Provider | Package | Status | Session | Exploration |
|----------|---------|--------|---------|-------------|
| **Cohere** | `opentelemetry-instrumentation-cohere` | 📋 TODO | 4 | 📋 TODO |
| **Groq** | `opentelemetry-instrumentation-groq` | 📋 TODO | 4 | 📋 TODO |
| **Mistral AI** | `opentelemetry-instrumentation-mistralai` | 📋 TODO | 4 | 📋 TODO |
| **Ollama** | `opentelemetry-instrumentation-ollama` | 📋 TODO | 4 | 📋 TODO |
| **Replicate** | `opentelemetry-instrumentation-replicate` | 📋 TODO | 4 | 📋 TODO |
| **Together** | `opentelemetry-instrumentation-together` | 📋 TODO | 4 | 📋 TODO |

### Tier 3 - Lower Priority

| Provider | Package | Status | Session | Exploration |
|----------|---------|--------|---------|-------------|
| **Vertex AI** | `opentelemetry-instrumentation-vertexai` | 📋 TODO | 5 | 📋 TODO |
| **Google GenAI** | `opentelemetry-instrumentation-google-generativeai` | 📋 TODO | 5 | 📋 TODO |
| **IBM Watsonx** | `opentelemetry-instrumentation-watsonx` | 📋 TODO | 5 | 📋 TODO |
| **Aleph Alpha** | `opentelemetry-instrumentation-alephalpha` | 📋 TODO | 5 | 📋 TODO |
| **Writer** | `opentelemetry-instrumentation-writer` | 📋 TODO | 5 | 📋 TODO |
| **HuggingFace Transformers** | `opentelemetry-instrumentation-transformers` | 📋 TODO | 5 | 📋 TODO |
| **AWS SageMaker** | `opentelemetry-instrumentation-sagemaker` | 📋 TODO | 5 | 📋 TODO |
| **OpenAI Agents** | `opentelemetry-instrumentation-openai-agents` | 📋 TODO | 4 | 📋 TODO |

---

## 📋 Quick Reference

### Documentation Files

- **Tier 1 (Current)**:
  - `openai.md` - OpenAI and Azure OpenAI (MUST)
  - `anthropic.md` - Anthropic Claude (MUST)
  - `bedrock.md` - AWS Bedrock (MUST)

- **Tier 2** (Future Sessions):
  - Individual docs for each provider

- **Tier 3** (Future Sessions):
  - Individual docs for each provider

### Exploration Resources

- **Exploration Plan**: [`_EXPLORATION_PLAN.md`](./_EXPLORATION_PLAN.md) - Analysis recipe for LLM providers
- **Root Guide**: [`../EXPLORATION_GUIDE.md`](../EXPLORATION_GUIDE.md) - General exploration methodology

---

## 🎯 Next Session TODO (Session 2)

### Priority Order:
1. **openai.md** - Most critical
   - Include Azure OpenAI integration
   - Chat, completions, embeddings, assistants, responses API
   - Streaming, async, vision, tools, structured outputs
   - All samples: `openai_*.py`, `azure_openai.py`

2. **anthropic.md** - Enterprise critical
   - Messages API, completions, streaming
   - Tool use, extended thinking, prompt caching
   - Bedrock integration notes
   - All samples: `anthropic_*.py`

3. **bedrock.md** - AWS enterprise
   - Multi-model support (Anthropic, Cohere, AI21, Meta, Amazon)
   - invoke_model, converse APIs, streaming
   - Guardrails, prompt caching, cross-region
   - All samples: `bedrock_*.py`

### Exploration Already Complete
✅ Deep dive analysis finished for OpenAI, Anthropic, Bedrock in Session 1
- All APIs documented
- Sample files identified
- Code patterns extracted
- Ready for documentation

---

## 📚 Template & Standards

### Use This Structure for Each Provider Doc:

```markdown
# {Provider} Instrumentation

## Table of Contents
- Overview
- Quick Start
- What's Auto-Instrumented
- What's Captured (Spans, Attributes, Events, Metrics)
- Advanced Features (Streaming, Async, Vision, Tools, etc.)
- Manual Instrumentation Scenarios
- Configuration Options
- Examples & Code Snippets
- Sample Applications Reference
- Common Patterns
- Troubleshooting
```

### Quality Checklist:
- [ ] All auto-instrumented APIs listed
- [ ] Complete attribute reference table
- [ ] Code snippets with inline explanations
- [ ] Sample file references with paths
- [ ] Special features documented (streaming, async, vision, tools)
- [ ] Manual instrumentation scenarios with examples
- [ ] Configuration options with defaults
- [ ] Common patterns and best practices

---

## 🔗 Related Resources

- **Main Progress Tracker**: [`../../PROGRESS.md`](../../PROGRESS.md)
- **Examples Index**: [`../../examples-index.md`](../../examples-index.md)
- **SDK Guide**: [`../../traceloop-sdk-guide.md`](../../traceloop-sdk-guide.md)
- **Sample Apps**: `packages/sample-app/sample_app/`

---

## 📝 Status Legend

- ✅ **Complete** - Documentation finished, reviewed
- 🚧 **In Progress** - Currently being written
- 📋 **TODO** - Not started yet
- ⏸️ **Postponed** - Deprioritized

**Exploration Status:**
- ✅ **Complete** - Deep analysis finished
- 🚧 **In Progress** - Being analyzed
- 📋 **TODO** - Not yet explored

---

**Last Updated**: 2025-11-22 (Session 1)
