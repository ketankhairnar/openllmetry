# Framework Instrumentation - Exploration Plan

> **Purpose**: Systematic recipe for analyzing AI framework instrumentation packages
> **Category**: Frameworks (6 packages: LangChain, LlamaIndex, Haystack, CrewAI, OpenAI Agents, MCP)
> **Reference**: See `/docs/instrumentation/EXPLORATION_GUIDE.md` for general methodology

---

## 📋 Framework Analysis Checklist

Use this checklist for each framework instrumentation package:

### 1. Instrumentation Approach

- [ ] **Instrumentation Pattern**
  - Callback-based (e.g., LangChain `BaseCallbackHandler`)
  - Method wrapping (e.g., `wrapt`)
  - Event hooks
  - Middleware/interceptors
  - Hybrid approach

- [ ] **Entry Point**
  - Where instrumentation is injected
  - How it's initialized
  - Auto-registration vs manual

- [ ] **Span Creation Strategy**
  - When spans are created
  - Span hierarchy management
  - Parent-child relationships
  - Context propagation mechanism

### 2. Component Coverage

- [ ] **Core Abstractions**
  - Workflows/chains/graphs
  - Tasks/steps/nodes
  - Agents
  - Tools/functions
  - LLM calls (direct integration)

- [ ] **Data Operations**
  - Retrievers/search
  - Document loaders
  - Data transformers
  - Embeddings

- [ ] **Orchestration**
  - Sequential execution
  - Parallel execution
  - Conditional branching
  - Loops/iterations

- [ ] **Integration Points**
  - LLM provider integrations
  - Vector DB integrations
  - External tool integrations

### 3. Data Capture Analysis

#### Span Hierarchy
- [ ] Document span structure for complex workflows
- [ ] Parent-child relationships
- [ ] Span naming conventions
- [ ] Span kinds (workflow, task, tool, etc.)

#### Span Attributes
- [ ] `traceloop.span.kind` - Entity type
- [ ] `traceloop.workflow.name` - Workflow name
- [ ] `traceloop.entity.name` - Component name
- [ ] `traceloop.entity.path` - Hierarchical path
- [ ] `gen_ai.system` - LLM provider (when applicable)
- [ ] Framework-specific attributes

#### Input/Output Capture
- [ ] `traceloop.entity.input` - Input data (JSON)
- [ ] `traceloop.entity.output` - Output data (JSON)
- [ ] Content tracing controls
- [ ] PII handling

#### Metrics
- [ ] Token usage propagation
- [ ] Operation duration
- [ ] Framework-specific metrics

### 4. Special Features

- [ ] **DSL/Expression Language Support**
  - LCEL (LangChain Expression Language)
  - Pipeline syntax
  - Custom DSL

- [ ] **Graph/Workflow Engines**
  - LangGraph
  - State machines
  - DAG execution
  - Node execution tracking

- [ ] **Streaming Support**
  - Sync streaming
  - Async streaming
  - Intermediate results
  - Stream buffering

- [ ] **Async/Concurrent Execution**
  - Async/await support
  - Concurrent task handling
  - Context propagation in async
  - Race conditions prevention

- [ ] **Multi-Agent Support**
  - Agent-to-agent communication
  - Handoffs
  - Hierarchical agents
  - Agent memory/state

### 5. Gaps & Limitations

- [ ] **Components NOT Instrumented**
  - List uninstrumented components
  - Why they're not instrumented
  - Workarounds

- [ ] **Known Issues**
  - Context detachment errors
  - Async edge cases
  - Memory leaks
  - Performance overhead

- [ ] **Missing Data**
  - Data not captured
  - Why it's not available
  - Manual alternatives

### 6. Manual Instrumentation Needs

- [ ] **When Manual Spans Needed**
  - Custom components
  - Advanced orchestration
  - Non-standard patterns

- [ ] **Integration Patterns**
  - How to add custom spans within framework
  - Context preservation
  - Best practices

- [ ] **Code Examples**
  - Custom component instrumentation
  - Workflow-level tracking
  - Tool execution tracking

### 7. Sample Applications

- [ ] List all framework samples
- [ ] Simple examples (basic usage)
- [ ] Complex examples (advanced patterns)
- [ ] Integration examples (with other tools)
- [ ] What each demonstrates

---

## 🤖 AI Analysis Prompt Template

Use this prompt to analyze a framework instrumentation package:

```
Analyze the opentelemetry-instrumentation-{FRAMEWORK} package in comprehensive detail.

**Package Location**: `/home/user/openllmetry/packages/opentelemetry-instrumentation-{FRAMEWORK}/`

I need a **Level 3 (Comprehensive)** analysis covering:

## 1. Components Automatically Instrumented

List ALL framework components that are automatically instrumented:
- Core abstractions (workflows, chains, agents, tools, etc.)
- Data operations (retrievers, loaders, etc.)
- Orchestration patterns (sequential, parallel, conditional)
- Integration points (LLM providers, vector DBs)

For each component:
- How it's instrumented (callbacks, wrapping, hooks)
- What span kind is created
- Span naming pattern

## 2. Instrumentation Approach & Implementation

Explain:
- **Pattern used**: Callbacks vs wrapping vs hooks
- **Entry point**: Where/how instrumentation is injected
- **Span creation**: When spans are created, hierarchy management
- **Context propagation**: How context flows through framework
- **Module paths**: What's wrapped/hooked

## 3. Data Captured Automatically

### Span Hierarchy
- Document span structure for a typical workflow
- Parent-child relationships
- Example hierarchy with actual span names

### Span Attributes
List ALL attributes captured:
- Workflow identification (`traceloop.workflow.name`)
- Entity identification (`traceloop.entity.name`, `traceloop.entity.path`)
- Input/output data (`traceloop.entity.input/output`)
- LLM integration attributes (`gen_ai.*`)
- Framework-specific attributes

### Metrics
- What metrics are collected
- How they're aggregated
- Dimensions/attributes

## 4. Special Features Support

Analyze:
- **DSL/Expression Language**: How is it instrumented? (e.g., LCEL)
- **Graph/Workflow Engines**: State machines, DAGs (e.g., LangGraph)
- **Streaming**: Sync/async streaming support
- **Async/Concurrent**: Context propagation, concurrent execution
- **Multi-Agent**: Agent communication, handoffs

## 5. Gaps & Manual Instrumentation Needs

Identify:
- **Components NOT instrumented**: What's missing and why
- **Known limitations**: Edge cases, issues, workarounds
- **Manual scenarios**: When custom instrumentation is needed

Provide code examples for:
- Adding custom spans within framework
- Instrumenting custom components
- Workflow-level tracking

## 6. Span Hierarchy for Complex Workflows

Provide a detailed example of span hierarchy for a complex workflow:
- Visualize parent-child structure
- Show span names and attributes
- Explain how hierarchy is maintained

Example:
\`\`\`
Workflow.workflow
├── ChainStep.task
│   ├── LLM.chat
│   └── Tool.tool
└── AnotherStep.task
    └── LLM.chat
\`\`\`

## 7. Configuration Options

Document:
- Instrumentor initialization parameters
- Environment variables
- Content tracing controls
- Custom callbacks

## 8. Sample Applications

List all samples in `packages/sample-app/sample_app/{framework}*.py`:
- File names and paths
- What each demonstrates (basic, LCEL, graph, async, streaming)
- Key patterns shown

## Analysis Files

Please analyze:
1. **Main instrumentor**: `opentelemetry/instrumentation/{framework}/__init__.py`
2. **Callback handler**: Look for callback classes
3. **Handler files**: Span utilities, event emitters
4. **Test files**: `tests/` for edge cases and examples
5. **Sample apps**: `packages/sample-app/sample_app/`

Provide a comprehensive summary following the Framework Exploration Plan checklist.
```

---

## 📊 Documentation Template for Frameworks

```markdown
# {Framework Name} Instrumentation

> **Exploration Status**: ✅ Complete | 🚧 In Progress | 📋 TODO
> **Last Updated**: {Date}
> **Package**: `opentelemetry-instrumentation-{framework}`
> **Instrumented Library**: `{framework}` Python package
> **Package Path**: `packages/opentelemetry-instrumentation-{framework}/`

## Table of Contents
- [Overview](#overview)
- [Quick Start](#quick-start)
- [What's Auto-Instrumented](#whats-auto-instrumented)
- [Span Hierarchy](#span-hierarchy)
- [What's Captured](#whats-captured)
- [Special Features](#special-features)
- [Gaps & Limitations](#gaps--limitations)
- [Manual Instrumentation](#manual-instrumentation)
- [Configuration](#configuration)
- [Examples](#examples)
- [Common Patterns](#common-patterns)
- [Troubleshooting](#troubleshooting)

## Overview

Brief description of the framework and what this instrumentation provides.

## Quick Start

\`\`\`python
from traceloop.sdk import Traceloop
from {framework} import ...

# Initialize with auto-instrumentation
Traceloop.init(app_name="my-app")

# All {Framework} operations are automatically traced
chain = ... # Build your chain/workflow
result = chain.invoke(...)
# ✅ Full workflow traced automatically
# ✅ Proper span hierarchy maintained
\`\`\`

**Sample Reference**: [`packages/sample-app/sample_app/{framework}_app.py`](../../packages/sample-app/sample_app/{framework}_app.py)

## What's Auto-Instrumented

### Core Components

| Component | Span Kind | Auto-Instrumented | Notes |
|-----------|-----------|-------------------|-------|
| Workflows/Chains | `workflow` | ✅ Yes | Full tracing |
| Tasks/Steps | `task` | ✅ Yes | All steps |
| Agents | `agent` | ✅ Yes | Agent execution |
| Tools | `tool` | ✅ Yes | Tool calls |
| LLM Calls | `llm` | ✅ Yes | Via callbacks |
| Retrievers | `task` | ⚠️ Partial | See limitations |

### {Framework-Specific Features}

{List DSL, graph engine, etc.}

## Span Hierarchy

### Example: Complex Chain Execution

\`\`\`
MyWorkflow.workflow
├── PromptTemplate.task
├── ChatModel.chat (OpenAI)
├── OutputParser.task
└── Tool.tool
    └── NestedLLM.chat
\`\`\`

### Hierarchy Attributes

| Attribute | Description | Example |
|-----------|-------------|---------|
| `traceloop.span.kind` | Entity type | `"workflow"`, `"task"`, `"tool"` |
| `traceloop.workflow.name` | Top-level workflow | `"MyWorkflow"` |
| `traceloop.entity.name` | Component name | `"ChatModel"` |
| `traceloop.entity.path` | Hierarchical path | `"workflow.chain.tool"` |

## What's Captured

### Span Attributes

#### Workflow/Task Identification

| Attribute | Description | Example |
|-----------|-------------|---------|
| `traceloop.workflow.name` | Workflow name | `"RAG_Pipeline"` |
| ... | ... | ... |

#### Input/Output Data (when `TRACELOOP_TRACE_CONTENT=true`)

| Attribute | Description | Format |
|-----------|-------------|--------|
| `traceloop.entity.input` | Input data | JSON string |
| `traceloop.entity.output` | Output data | JSON string |

### Metrics

{Framework-specific metrics if any}

## Special Features

### {DSL Support (e.g., LCEL)}

{Explanation with code examples}

### {Graph/Workflow Engine (e.g., LangGraph)}

{Explanation with code examples}

### Streaming Support

{Explanation with code examples}

### Async/Concurrent Execution

{Explanation with code examples}

## Gaps & Limitations

### Components NOT Instrumented

- **{Component}**: {Why not instrumented}
  - **Workaround**: {Manual instrumentation example}

### Known Issues

- **{Issue}**: {Description and workaround}

## Manual Instrumentation

### Custom Components

\`\`\`python
from opentelemetry import trace

tracer = trace.get_tracer(__name__)

# Custom component that needs manual span
with tracer.start_as_current_span("my_custom_component") as span:
    # Your custom logic
    result = custom_operation()
    span.set_attribute("custom.attribute", value)
\`\`\`

### Integration with Framework

{How to add custom spans within framework patterns}

## Configuration

\`\`\`python
from opentelemetry.instrumentation.{framework} import {Framework}Instrumentor

{Framework}Instrumentor(
    use_legacy_attributes=True,
    # ... other options
).instrument()
\`\`\`

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `TRACELOOP_TRACE_CONTENT` | Enable/disable content logging | `true` |

## Examples

### Basic Usage

{Code example}

### {Advanced Pattern}

{Code example}

## Sample Applications

| Sample File | Demonstrates | Complexity |
|-------------|--------------|------------|
| `{framework}_app.py` | Basic chain | Simple |
| `{framework}_agent.py` | Agent with tools | Medium |
| `{framework}_graph.py` | Graph workflow | Advanced |

## Common Patterns

### Pattern 1: {Name}

{Best practice with code}

## Troubleshooting

### Issue: {Common Problem}

**Symptoms**: {What you see}
**Cause**: {Why it happens}
**Solution**: {How to fix}
```

---

## 🎯 Priority Matrix for Frameworks

| Framework | Tier | Priority | Reason |
|-----------|------|----------|--------|
| LangChain | 1 | 🔴 MUST | Most popular, LCEL, LangGraph |
| LlamaIndex | 1 | 🟡 Medium | Popular for RAG |
| Haystack | 2 | 🟢 Lower | NLP pipelines |
| CrewAI | 2 | 🟡 Medium | Multi-agent workflows |
| OpenAI Agents | 1 | 🟡 Medium | New official agents SDK |
| MCP | 2 | 🟢 Lower | Model Context Protocol |

---

## ✅ Completion Criteria

A framework documentation is complete when:

- [ ] All auto-instrumented components documented
- [ ] Instrumentation approach explained (callbacks/wrapping)
- [ ] Span hierarchy visualized with examples
- [ ] Complete attribute reference
- [ ] Special features analyzed (DSL, graphs, streaming, async)
- [ ] Gaps and limitations identified
- [ ] Manual instrumentation patterns documented with examples
- [ ] Configuration options documented
- [ ] All sample apps referenced with complexity levels
- [ ] Common patterns documented
- [ ] Known issues with workarounds listed

---

**End of Framework Exploration Plan**
