# OpenLLMetry Documentation

> **Status**: 🚧 Work in Progress
> **Last Updated**: 2025-11-22 (Session 1)
> **Version**: Foundation established

---

## 🚀 Quick Navigation

### Getting Started

- **[Traceloop SDK Guide](traceloop-sdk-guide.md)** ✅ - Complete guide to the SDK
- **[Quick Start Guide](quick-start.md)** 🚧 - 5-minute getting started (Coming in Session 3)
- **[Examples Index](examples-index.md)** ✅ - Catalog of 64 sample applications

### Instrumentation Documentation

#### LLM Providers
- **[OpenAI](instrumentation/llm-providers/openai.md)** 📋 (Session 2) - Including Azure OpenAI
- **[Anthropic](instrumentation/llm-providers/anthropic.md)** 📋 (Session 2) - Claude models
- **[AWS Bedrock](instrumentation/llm-providers/bedrock.md)** 📋 (Session 2) - Multi-model platform
- [All Providers →](instrumentation/llm-providers/README.md) - 17 total providers

#### Frameworks
- **[LangChain](instrumentation/frameworks/langchain.md)** 📋 (Session 3) - Chains, LCEL, LangGraph
- [All Frameworks →](instrumentation/frameworks/README.md) - 6 frameworks

#### Vector Databases
- [All Vector DBs →](instrumentation/vector-databases/README.md) - 7 databases

### Resources

- **[Exploration Guide](instrumentation/EXPLORATION_GUIDE.md)** - How to analyze packages
- **[Progress Tracker](PROGRESS.md)** - Session tracking and handover
- **[Patterns](patterns/README.md)** 🚧 - Best practices (Coming in Session 3)

---

## 📊 Documentation Progress

**Session 1 Complete** (2025-11-22):
- ✅ Infrastructure and templates
- ✅ Traceloop SDK Guide (comprehensive)
- ✅ Examples Index (64 samples cataloged)
- ✅ Exploration plans for all categories

**Next Sessions**:
- 📋 Session 2: Tier 1 LLM providers (OpenAI, Anthropic, Bedrock)
- 📋 Session 3: LangChain + polish WIP docs
- 📋 Session 4+: Tier 2/3 expansion

See **[PROGRESS.md](PROGRESS.md)** for detailed tracking.

---

## 🎯 What is OpenLLMetry?

**OpenLLMetry** is a comprehensive observability solution for LLM applications, providing:

- ✅ **Auto-instrumentation** for 29+ AI libraries and frameworks
- ✅ **OpenTelemetry-based** - works with any OTLP backend
- ✅ **Semantic conventions** following OpenTelemetry GenAI spec
- ✅ **Privacy controls** for sensitive data
- ✅ **Production-ready** with minimal overhead

### Supported Technologies

**LLM Providers** (17):
OpenAI, Anthropic, AWS Bedrock, Cohere, Mistral, Ollama, Groq, Vertex AI, Google GenAI, IBM Watsonx, and more

**AI Frameworks** (6):
LangChain, LlamaIndex, Haystack, CrewAI, OpenAI Agents, MCP

**Vector Databases** (7):
Pinecone, Chroma, Qdrant, Weaviate, Milvus, LanceDB, Marqo

---

## 🏃 Quick Start

### Installation

```bash
pip install traceloop-sdk
```

### Basic Usage

```python
from traceloop.sdk import Traceloop

# Initialize once at startup
Traceloop.init(app_name="my-app")

# Use your AI libraries normally - they're auto-instrumented!
from openai import OpenAI

client = OpenAI()
response = client.chat.completions.create(
    model="gpt-4",
    messages=[{"role": "user", "content": "Hello!"}]
)
# ✅ Automatically traced with full observability
```

**See [Traceloop SDK Guide](traceloop-sdk-guide.md) for complete documentation.**

---

## 📚 Documentation Structure

```
docs/
├── README.md (this file)          - Overview and navigation
├── PROGRESS.md                    - Session tracking
├── traceloop-sdk-guide.md         - ✅ Complete SDK documentation
├── examples-index.md              - ✅ 64 sample applications
├── quick-start.md                 - 🚧 Getting started guide
│
├── instrumentation/
│   ├── EXPLORATION_GUIDE.md       - Analysis methodology
│   │
│   ├── llm-providers/
│   │   ├── README.md              - Provider docs navigation
│   │   ├── _EXPLORATION_PLAN.md   - Analysis recipe
│   │   ├── openai.md              - 📋 Session 2
│   │   ├── anthropic.md           - 📋 Session 2
│   │   ├── bedrock.md             - 📋 Session 2
│   │   └── ...                    - 📋 Future sessions
│   │
│   ├── frameworks/
│   │   ├── README.md              - Framework docs navigation
│   │   ├── _EXPLORATION_PLAN.md   - Analysis recipe
│   │   ├── langchain.md           - 📋 Session 3
│   │   └── ...                    - 📋 Future sessions
│   │
│   └── vector-databases/
│       ├── README.md              - Vector DB navigation
│       ├── _EXPLORATION_PLAN.md   - Analysis recipe
│       └── ...                    - 📋 Future sessions
│
└── patterns/
    ├── README.md                  - Pattern guides overview
    ├── auto-vs-manual.md          - 🚧 Session 3
    ├── streaming.md               - 🚧 Session 3
    ├── async-patterns.md          - 🚧 Session 3
    └── troubleshooting.md         - 🚧 Session 3
```

---

## 🔍 Finding Documentation

### By Technology

- **Using OpenAI?** → See [openai.md](instrumentation/llm-providers/openai.md) (Session 2)
- **Using Anthropic Claude?** → See [anthropic.md](instrumentation/llm-providers/anthropic.md) (Session 2)
- **Using LangChain?** → See [langchain.md](instrumentation/frameworks/langchain.md) (Session 3)
- **Using Pinecone/Chroma?** → See [vector-databases/](instrumentation/vector-databases/)

### By Goal

- **Getting started** → [traceloop-sdk-guide.md](traceloop-sdk-guide.md)
- **See examples** → [examples-index.md](examples-index.md)
- **Learn patterns** → [patterns/](patterns/) (Coming in Session 3)
- **Troubleshoot** → [traceloop-sdk-guide.md#troubleshooting](traceloop-sdk-guide.md#troubleshooting)

### By Task

- **Auto vs Manual** → [patterns/auto-vs-manual.md](patterns/auto-vs-manual.md) (Session 3)
- **Streaming** → [patterns/streaming.md](patterns/streaming.md) (Session 3)
- **Async/Await** → [patterns/async-patterns.md](patterns/async-patterns.md) (Session 3)

---

## 🤝 Contributing to Documentation

### For Future Sessions

1. Review **[PROGRESS.md](PROGRESS.md)** for current status
2. Check category READMEs for TODO lists
3. Use **[EXPLORATION_GUIDE.md](instrumentation/EXPLORATION_GUIDE.md)** for analysis
4. Follow exploration plans in each category

### Exploration Workflow

1. **Choose a package** from TODO lists
2. **Run exploration** using category-specific recipe
3. **Document findings** using template
4. **Update progress tracker**
5. **Commit and push**

See **[instrumentation/EXPLORATION_GUIDE.md](instrumentation/EXPLORATION_GUIDE.md)** for detailed methodology.

---

## 📞 Support & Resources

- **GitHub Repository**: https://github.com/traceloop/openllmetry
- **Issues**: https://github.com/traceloop/openllmetry/issues
- **Traceloop Docs**: https://traceloop.com/docs
- **OpenTelemetry**: https://opentelemetry.io/

---

## Status Legend

- ✅ **Complete** - Documentation finished, reviewed
- 🚧 **In Progress** - Currently being written
- 📋 **TODO** - Planned for future sessions

---

**Last Updated**: 2025-11-22 (Session 1)
**Next Session**: Session 2 - LLM Providers Tier 1 (OpenAI, Anthropic, Bedrock)
