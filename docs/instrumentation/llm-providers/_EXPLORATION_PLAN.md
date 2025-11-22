# LLM Provider Instrumentation - Exploration Plan

> **Purpose**: Systematic recipe for analyzing LLM provider instrumentation packages
> **Category**: LLM Providers (17 packages total)
> **Reference**: See `/docs/instrumentation/EXPLORATION_GUIDE.md` for general methodology

---

## 📋 LLM Provider Analysis Checklist

Use this checklist for each LLM provider package:

### 1. API Coverage

- [ ] **Chat/Completion API**
  - Synchronous methods
  - Asynchronous methods
  - Span names created

- [ ] **Streaming Support**
  - Sync streaming (`stream=True`)
  - Async streaming
  - Stream wrapper implementation
  - Chunk accumulation strategy

- [ ] **Embeddings API** (if applicable)
  - Synchronous embedding creation
  - Asynchronous embedding creation
  - Batch embedding support

- [ ] **Vision/Multimodal Support** (if applicable)
  - Image input handling (URLs, base64)
  - Base64 upload callback
  - Content capture strategy

- [ ] **Function/Tool Calling** (if applicable)
  - Legacy function calling
  - Modern tools API
  - Tool execution tracking
  - Tool result messages

- [ ] **Structured Outputs** (if applicable)
  - JSON mode
  - Schema-based outputs
  - Pydantic model support

- [ ] **Advanced APIs** (provider-specific)
  - Assistants/Agents API
  - Batch API
  - Fine-tuning API (if instrumented)
  - Provider-specific features

### 2. Data Capture Analysis

#### Request Attributes
- [ ] `gen_ai.system` - Provider name
- [ ] `gen_ai.request.model` - Model identifier
- [ ] `gen_ai.request.max_tokens` - Token limit
- [ ] `gen_ai.request.temperature` - Temperature setting
- [ ] `gen_ai.request.top_p` - Top-p sampling
- [ ] `llm.frequency_penalty` - Frequency penalty
- [ ] `llm.presence_penalty` - Presence penalty
- [ ] `llm.is_streaming` - Streaming flag
- [ ] Provider-specific parameters

#### Request Content (when `TRACELOOP_TRACE_CONTENT=true`)
- [ ] `gen_ai.prompt.{N}.role` - Message roles
- [ ] `gen_ai.prompt.{N}.content` - Message content
- [ ] `llm.request.functions` - Tool/function definitions
- [ ] `gen_ai.request.structured_output_schema` - Output schema

#### Response Attributes
- [ ] `gen_ai.response.model` - Actual model used
- [ ] `gen_ai.response.id` - Response/request ID
- [ ] `gen_ai.usage.input_tokens` - Input tokens
- [ ] `gen_ai.usage.output_tokens` - Output tokens
- [ ] `llm.usage.total_tokens` - Total tokens
- [ ] Provider-specific usage metrics

#### Response Content (when `TRACELOOP_TRACE_CONTENT=true`)
- [ ] `gen_ai.completion.{N}.role` - Completion role
- [ ] `gen_ai.completion.{N}.content` - Completion text
- [ ] `gen_ai.completion.{N}.finish_reason` - Stop reason
- [ ] `gen_ai.completion.{N}.tool_calls` - Tool calls
- [ ] Provider-specific response fields

#### Events (if using new semantic conventions)
- [ ] `gen_ai.user.message` - User messages
- [ ] `gen_ai.system.message` - System prompts
- [ ] `gen_ai.assistant.message` - Assistant responses
- [ ] `gen_ai.choice` - Completion choices
- [ ] Provider-specific events

#### Metrics
- [ ] `gen_ai.client.token.usage` (histogram) - Token usage
- [ ] `gen_ai.client.operation.duration` (histogram) - Request duration
- [ ] `gen_ai.client.generation.choices` (counter) - Number of choices
- [ ] Provider-specific metrics
- [ ] Exception counters

### 3. Special Features Documentation

- [ ] **Streaming Implementation**
  - How streams are wrapped
  - When span closes (stream completion)
  - Token counting in streaming
  - Error handling in streams

- [ ] **Async Support**
  - Context propagation in async
  - Async generator handling
  - Concurrent request handling

- [ ] **Provider-Specific Features**
  - Prompt caching (cache read/write tokens)
  - Extended thinking (Claude)
  - Reasoning tokens (OpenAI o1/o3)
  - Guardrails (Bedrock)
  - Content filtering (Azure)
  - Multi-region support

- [ ] **Privacy Controls**
  - Content tracing toggle
  - PII handling
  - Custom filtering callbacks

### 4. Implementation Analysis

- [ ] **Patching Mechanism**
  - Module paths wrapped
  - Methods wrapped (list all)
  - Wrapper function pattern
  - Use of `wrapt.wrap_function_wrapper`

- [ ] **Response Wrapping** (for streaming)
  - Proxy class implementation
  - Iterator/generator wrapping
  - Cleanup on completion
  - Memory leak prevention

- [ ] **Error Handling**
  - Exception capture
  - Error metrics
  - `@dont_throw` decorator usage
  - Instrumentation suppression support

- [ ] **Configuration Options**
  - Instrumentor init parameters
  - Custom callbacks
  - Metric attribute callbacks
  - Image upload callbacks

### 5. Sample Applications

- [ ] List all related samples in `packages/sample-app/sample_app/`
- [ ] Note which features each demonstrates
- [ ] Verify samples work
- [ ] Extract key patterns

### 6. Manual Instrumentation Scenarios

- [ ] Workflow orchestration (multi-step agents)
- [ ] Tool execution tracking
- [ ] Custom retry logic
- [ ] Batch processing
- [ ] Custom metrics
- [ ] Non-SDK API calls

---

## 🤖 AI Analysis Prompt Template

Use this prompt to analyze a new LLM provider package:

```
Analyze the opentelemetry-instrumentation-{PROVIDER} package in comprehensive detail.

**Package Location**: `/home/user/openllmetry/packages/opentelemetry-instrumentation-{PROVIDER}/`

I need a **Level 3 (Comprehensive)** analysis covering:

## 1. API Methods/Operations Automatically Instrumented

List ALL wrapped methods with:
- Full module paths
- Synchronous vs asynchronous variants
- Span names created
- API categories (chat, embeddings, vision, etc.)

Example format:
- `{provider}.resources.chat.completions.Completions.create` → Span: `{provider}.chat`
- `{provider}.resources.chat.completions.AsyncCompletions.create` → Span: `{provider}.chat`

## 2. Data Captured Automatically

### Spans
- Span names and hierarchy
- Span kind (CLIENT, etc.)

### Attributes - Request
List ALL request attributes captured:
- Model configuration (model, temperature, max_tokens, etc.)
- Prompts/messages (when TRACELOOP_TRACE_CONTENT=true)
- Tool/function definitions
- Provider-specific parameters

### Attributes - Response
List ALL response attributes:
- Model and response metadata
- Token usage (input, output, total, cached, reasoning)
- Completions/responses
- Tool calls
- Provider-specific fields

### Events (if applicable)
- Event types emitted
- Event structure and attributes
- When events are emitted vs span attributes

### Metrics
- Metric names and types (histogram/counter)
- Metric dimensions/attributes
- What each metric measures

## 3. Special Features Support

Analyze support for:
- **Streaming** (sync/async) - How implemented? When does span close?
- **Async/await** - Full support? Context propagation?
- **Vision/Multimodal** - Image handling? Base64 upload?
- **Function/Tool Calling** - Legacy and modern APIs?
- **Structured Outputs** - JSON mode? Schema support?
- **Provider-Specific Features** - Caching, thinking, guardrails, etc.

## 4. How It Patches the SDK

Explain:
- Patching mechanism (wrapt pattern)
- All module paths and methods wrapped
- Response wrapping strategy (for streaming)
- Error handling approach
- Cleanup and memory management

## 5. Configuration Options

Document:
- Instrumentor initialization parameters
- Environment variables supported
- Custom callbacks (exception_logger, upload_base64_image, etc.)
- Default values

## 6. Scenarios Requiring Manual Instrumentation

Identify when automatic instrumentation is insufficient:
- Multi-step workflows
- Tool execution
- Custom retry logic
- Batch processing
- Custom metrics
- Other scenarios

Provide code examples for manual instrumentation.

## 7. Sample Applications

List all samples in `packages/sample-app/sample_app/{provider}*.py`:
- File names and paths
- What each demonstrates
- Key patterns shown

## Analysis Approach

Please analyze:
1. **Main instrumentor**: `opentelemetry/instrumentation/{provider}/__init__.py`
2. **Handler files**: Look for span_utils, streaming, event_emitter, etc.
3. **Test files**: `tests/` directory for edge cases
4. **Sample apps**: `packages/sample-app/sample_app/{provider}*.py`

Provide a comprehensive summary following the LLM Provider Exploration Plan checklist.
```

---

## 📊 Documentation Template for LLM Providers

```markdown
# {Provider Name} Instrumentation

> **Exploration Status**: ✅ Complete | 🚧 In Progress | 📋 TODO
> **Last Updated**: {Date}
> **Package**: `opentelemetry-instrumentation-{provider}`
> **Instrumented Library**: `{provider}` Python SDK
> **Package Path**: `packages/opentelemetry-instrumentation-{provider}/`

## Table of Contents
- [Overview](#overview)
- [Quick Start](#quick-start)
- [What's Auto-Instrumented](#whats-auto-instrumented)
- [What's Captured](#whats-captured)
- [Advanced Features](#advanced-features)
- [Manual Instrumentation](#manual-instrumentation)
- [Configuration](#configuration)
- [Examples](#examples)
- [Common Patterns](#common-patterns)
- [Troubleshooting](#troubleshooting)

## Overview

Brief description of what this instrumentation does and key features.

## Quick Start

\`\`\`python
from traceloop.sdk import Traceloop
from {provider} import {Client}

# Initialize with auto-instrumentation
Traceloop.init(app_name="my-app")

# All {Provider} calls are automatically traced
client = {Client}()
response = client.chat.completions.create(
    model="...",
    messages=[{"role": "user", "content": "Hello!"}]
)
# ✅ Span created automatically
# ✅ Prompts, responses, and tokens captured
\`\`\`

**Sample Reference**: [`packages/sample-app/sample_app/{provider}_example.py`](../../packages/sample-app/sample_app/{provider}_example.py)

## What's Auto-Instrumented

### Chat/Completion API

| Method | Span Name | Support |
|--------|-----------|---------|
| `{provider}.chat.completions.create()` | `{provider}.chat` | ✅ Sync |
| `{provider}.chat.completions.acreate()` | `{provider}.chat` | ✅ Async |
| `{provider}.chat.completions.create(stream=True)` | `{provider}.chat` | ✅ Streaming |

### {Other APIs as applicable}

## What's Captured

### Span Attributes

#### Request Attributes (Always Captured)

| Attribute | Description | Example |
|-----------|-------------|---------|
| `gen_ai.system` | Provider name | `"{provider}"` |
| `gen_ai.request.model` | Model identifier | `"model-name"` |
| ... | ... | ... |

#### Request Content (When `TRACELOOP_TRACE_CONTENT=true`)

| Attribute | Description | Example |
|-----------|-------------|---------|
| `gen_ai.prompt.0.role` | Message role | `"user"` |
| ... | ... | ... |

#### Response Attributes

| Attribute | Description | Example |
|-----------|-------------|---------|
| `gen_ai.usage.input_tokens` | Input tokens | `150` |
| ... | ... | ... |

### Metrics

| Metric Name | Type | Description | Dimensions |
|-------------|------|-------------|------------|
| `gen_ai.client.token.usage` | Histogram | Token usage | `gen_ai.token.type`, `gen_ai.response.model` |
| ... | ... | ... | ... |

## Advanced Features

### Streaming

{Explanation with code examples}

### Async Support

{Explanation with code examples}

### {Provider-Specific Features}

{Explanation with code examples}

## Manual Instrumentation

### When Manual Spans Are Needed

{Scenarios with code examples}

## Configuration

\`\`\`python
from opentelemetry.instrumentation.{provider} import {Provider}Instrumentor

instrumentor = {Provider}Instrumentor(
    param1=value1,
    param2=value2
)
instrumentor.instrument()
\`\`\`

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `TRACELOOP_TRACE_CONTENT` | Enable/disable content logging | `true` |
| ... | ... | ... |

## Examples

{Multiple code snippets with explanations}

## Sample Applications

| Sample File | Demonstrates |
|-------------|--------------|
| `{provider}_example.py` | Basic usage |
| ... | ... |

## Common Patterns

{Best practices}

## Troubleshooting

{Known issues and solutions}
```

---

## 🎯 Priority Matrix for LLM Providers

| Provider | Tier | Priority | Reason |
|----------|------|----------|--------|
| OpenAI | 1 | 🔴 MUST | Most popular, includes Azure |
| Anthropic | 1 | 🔴 MUST | Enterprise critical, Claude family |
| Bedrock | 1 | 🔴 MUST | AWS enterprise, multi-model |
| Cohere | 2 | 🟡 Medium | RAG popular |
| Groq | 2 | 🟡 Medium | Fast inference |
| Mistral AI | 2 | 🟡 Medium | European AI leader |
| Ollama | 2 | 🟡 Medium | Local models popular |
| Replicate | 2 | 🟡 Medium | Open source models |
| Together | 2 | 🟡 Medium | Inference platform |
| Vertex AI | 3 | 🟢 Lower | Google Cloud |
| Google GenAI | 3 | 🟢 Lower | Gemini direct |
| Watsonx | 3 | 🟢 Lower | IBM enterprise |
| Aleph Alpha | 3 | 🟢 Lower | European specialized |
| Writer | 3 | 🟢 Lower | Content generation |
| Transformers | 3 | 🟢 Lower | HuggingFace local |
| SageMaker | 3 | 🟢 Lower | AWS ML platform |
| OpenAI Agents | 1 | 🟡 Medium | New agents SDK |

---

## ✅ Completion Criteria

A provider documentation is complete when:

- [ ] All auto-instrumented APIs documented
- [ ] Complete attribute reference table created
- [ ] Special features analyzed (streaming, async, vision, tools)
- [ ] Provider-specific features documented
- [ ] Manual instrumentation scenarios identified with examples
- [ ] Configuration options documented
- [ ] All sample apps referenced
- [ ] Code snippets tested
- [ ] Common patterns documented
- [ ] Known issues listed

---

**End of LLM Provider Exploration Plan**
