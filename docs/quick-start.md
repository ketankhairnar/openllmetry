# Quick Start Guide

> **Status**: 🚧 Work in Progress
> **Last Updated**: 2025-11-22 (Session 1)
> **Completion Target**: Session 3

---

## 🚀 5-Minute Setup

*This guide will be completed in Session 3*

### Installation

```bash
pip install traceloop-sdk
```

### Basic Usage

```python
from traceloop.sdk import Traceloop

# Initialize
Traceloop.init(app_name="my-app")

# Use AI libraries normally
from openai import OpenAI

client = OpenAI()
response = client.chat.completions.create(
    model="gpt-4",
    messages=[{"role": "user", "content": "Hello!"}]
)
# ✅ Automatically traced!
```

---

## 📋 Planned Content (Session 3)

This guide will include:

- ✅ Installation steps
- ✅ First trace in 5 minutes
- ✅ Viewing traces (Console, Traceloop Cloud, Custom backends)
- ✅ Adding manual instrumentation (`@workflow`, `@task`)
- ✅ Common patterns (RAG, agents, streaming)
- ✅ Troubleshooting quick fixes
- ✅ Next steps

---

## 🔗 For Now, See:

- **[Traceloop SDK Guide](traceloop-sdk-guide.md)** - Complete documentation
- **[Examples Index](examples-index.md)** - 64 sample applications
- **[README](README.md)** - Documentation overview

---

**Status**: 🚧 WIP - Will be completed in Session 3
