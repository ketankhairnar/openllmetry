# Instrumentation Package Exploration Guide

> **Purpose**: This guide provides a systematic framework for analyzing and documenting instrumentation packages.
> **Audience**: Documentation contributors, future sessions, AI assistants
> **Last Updated**: 2025-11-22

---

## 📊 Depth & Breadth Framework

### Depth Levels

Choose the appropriate depth level based on package priority and available time:

#### **Level 1: Surface Analysis** (30-45 minutes)
Suitable for: Tier 3 packages, initial triage

**Activities**:
- Read package README and PyPI description
- List primary instrumented methods/APIs
- Check basic configuration options
- Run one sample application (if available)
- Document basic span names

**Deliverables**:
- Brief overview (2-3 paragraphs)
- List of auto-instrumented methods
- One code snippet example
- Link to sample app

---

#### **Level 2: Code Analysis** (2-3 hours)
Suitable for: Tier 2 packages, standard documentation

**Activities**:
- Read instrumentor `__init__.py` thoroughly
- Identify all wrapped methods and module paths
- Document span attributes and events
- Analyze test files for edge cases
- Run all related sample applications
- Check for streaming/async support
- Document configuration options

**Deliverables**:
- Complete list of instrumented APIs
- Span attributes table
- Configuration reference
- Multiple code examples
- Sample app references
- Known limitations

---

#### **Level 3: Comprehensive Analysis** (4-6 hours)
Suitable for: Tier 1 packages, critical documentation

**Activities**:
- Deep dive into patching mechanism
- Analyze all handler/utility files
- Document all span attributes, events, metrics
- Test streaming and async variants
- Explore edge cases and error handling
- Create custom test examples
- Document special features (vision, tools, etc.)
- Analyze metrics collection
- Test configuration options
- Document limitations and gaps
- Identify manual instrumentation scenarios

**Deliverables**:
- Comprehensive documentation with all sections
- Complete span/attribute/event/metric reference
- Multiple code examples with explanations
- Sample app references for all features
- Common patterns and best practices
- Troubleshooting section
- TODO list for future exploration

---

## 🎯 Breadth Coverage Checklist

For each instrumentation package, document the following:

### 1. **What's Instrumented** (Methods/APIs)
- [ ] List all wrapped methods with full module paths
- [ ] Organize by API category (e.g., chat, embeddings, tools)
- [ ] Note sync vs async variants
- [ ] Document streaming support
- [ ] Identify any beta/experimental APIs

### 2. **What's Captured** (Data/Telemetry)
- [ ] **Spans**: Names, span kind, hierarchy
- [ ] **Attributes**: Request parameters, response data, metadata
- [ ] **Events**: Event names, structure, when emitted
- [ ] **Metrics**: Metric names, types (histogram/counter), dimensions
- [ ] Note content tracing controls (privacy)

### 3. **Special Features**
- [ ] Streaming (sync/async)
- [ ] Async/await support
- [ ] Vision/multimodal (images, documents)
- [ ] Function/tool calling
- [ ] Structured outputs
- [ ] Provider-specific features (e.g., thinking, caching, guardrails)

### 4. **Gaps** (What's NOT Instrumented)
- [ ] APIs that exist but aren't wrapped
- [ ] Management/control plane operations
- [ ] Data that's not captured
- [ ] Known limitations

### 5. **Manual Work Needed** (When/How)
- [ ] Scenarios requiring custom spans
- [ ] Workflow orchestration patterns
- [ ] Tool execution tracking
- [ ] Custom metrics
- [ ] Example code snippets

### 6. **Configuration** (Options/Env Vars)
- [ ] Instrumentor initialization parameters
- [ ] Environment variables
- [ ] Privacy/content controls
- [ ] Custom callbacks
- [ ] Default values

### 7. **Examples** (Sample App References)
- [ ] List all relevant sample files
- [ ] Link to specific patterns
- [ ] Note which features each demonstrates

---

## 🔬 Exploration Methodology

### Step 1: Package Discovery
```bash
# Navigate to package directory
cd packages/opentelemetry-instrumentation-{NAME}/

# Check structure
ls -la

# Read README
cat README.md

# Check dependencies
cat pyproject.toml
```

### Step 2: Code Analysis
```bash
# Main instrumentor
cat opentelemetry/instrumentation/{NAME}/__init__.py

# Look for handler files
ls opentelemetry/instrumentation/{NAME}/*.py

# Check tests
ls tests/
```

### Step 3: Method Wrapping Analysis

Look for patterns like:
```python
from wrapt import wrap_function_wrapper

WRAPPED_METHODS = [...]

def _instrument(self, **kwargs):
    for method in WRAPPED_METHODS:
        wrap_function_wrapper(
            module, class.method, wrapper_function
        )
```

Document:
- Module paths
- Class and method names
- Wrapper function names
- Span names

### Step 4: Attribute Analysis

Look for patterns like:
```python
span.set_attribute("gen_ai.request.model", model)
span.set_attribute("gen_ai.usage.input_tokens", tokens)
```

Create a table of all attributes.

### Step 5: Sample App Testing

```bash
# Find related samples
cd packages/sample-app/sample_app/
ls *{provider}*.py

# Run sample
poetry run python {sample}.py
```

Observe:
- What spans are created
- What data is captured
- Any errors or warnings

### Step 6: Documentation

Use the standard template structure:
1. Quick Start
2. What's Auto-Instrumented
3. What's Captured
4. Manual Instrumentation Scenarios
5. Configuration
6. Examples
7. Common Patterns
8. Troubleshooting

---

## 🤖 AI Assistant Prompts

Use these prompts to analyze packages with AI assistance:

### General Analysis Prompt

```
Analyze the opentelemetry-instrumentation-{PACKAGE} package in detail.

Package location: /home/user/openllmetry/packages/opentelemetry-instrumentation-{PACKAGE}/

I need to understand:

1. **What's Auto-Instrumented**
   - List all API methods/operations that are automatically wrapped
   - Include full module paths and class names
   - Note sync vs async variants
   - Document streaming support

2. **What's Captured**
   - Span names and hierarchy
   - All span attributes (request and response)
   - Events emitted (if any)
   - Metrics collected (if any)

3. **Special Features**
   - Streaming support (sync/async)
   - Async/await support
   - Provider-specific features
   - Advanced capabilities

4. **Implementation Details**
   - How the SDK/library is patched (wrapt mechanism)
   - Module paths and methods wrapped
   - Response wrapping strategy (for streaming)
   - Error handling approach

5. **Manual Instrumentation Scenarios**
   - What scenarios require custom spans
   - When automatic instrumentation isn't sufficient
   - Code examples for manual instrumentation

6. **Sample Applications**
   - List all related samples in packages/sample-app/sample_app/
   - What each sample demonstrates

Please analyze:
- Main instrumentor code
- Handler/utility files
- Test files
- Related sample applications

Provide a comprehensive summary following the exploration guide template.
```

### Specific Feature Analysis Prompt

```
For opentelemetry-instrumentation-{PACKAGE}, I need detailed analysis of:

**{FEATURE}** (e.g., streaming, async, vision, tool calling)

Please analyze:
1. How {FEATURE} is implemented
2. What data is captured
3. Code examples
4. Related sample applications
5. Known limitations

Look at:
- Instrumentor code
- Handler files
- Test files specifically for {FEATURE}
- Sample apps demonstrating {FEATURE}
```

### Comparison Prompt

```
Compare instrumentation approaches between:
- opentelemetry-instrumentation-{PACKAGE_A}
- opentelemetry-instrumentation-{PACKAGE_B}

Focus on:
1. Patching mechanism differences
2. Data captured (attributes, events, metrics)
3. Special feature support
4. Configuration options
5. Manual instrumentation needs

Identify common patterns and unique approaches.
```

---

## 📝 Documentation Template

Use this template for each instrumentation package:

```markdown
# {Package Name} Instrumentation

> **Exploration Status**: ✅ Complete | 🚧 In Progress | 📋 TODO
> **Last Updated**: {Date}
> **Package**: `opentelemetry-instrumentation-{name}`
> **Package Path**: `packages/opentelemetry-instrumentation-{name}/`

## Table of Contents
- [Quick Start](#quick-start)
- [What's Auto-Instrumented](#whats-auto-instrumented)
- [What's Captured](#whats-captured)
- [Manual Instrumentation Scenarios](#manual-instrumentation-scenarios)
- [Configuration Options](#configuration-options)
- [Examples & Code Snippets](#examples--code-snippets)
- [Sample Applications](#sample-applications)
- [Common Patterns](#common-patterns)
- [Troubleshooting](#troubleshooting)

## Quick Start

{30-second setup code snippet}

## What's Auto-Instrumented

{Detailed list of wrapped methods}

## What's Captured

### Spans
{Span names, hierarchy}

### Attributes
{Complete attribute reference table}

### Events
{Event types and structure}

### Metrics
{Metric names, types, dimensions}

## Manual Instrumentation Scenarios

{When/how to add custom spans}

## Configuration Options

{Instrumentor parameters + environment variables}

## Examples & Code Snippets

{Inline code with explanations + sample file references}

## Sample Applications

{Links to sample-app files}

## Common Patterns

{Best practices and common use cases}

## Troubleshooting

{Known issues and solutions}

---

## 📋 TODO: Future Exploration
- [ ] {Specific feature to analyze}
- [ ] {API to test}
- [ ] {Edge case to document}
```

---

## ✅ Quality Checklist

Before marking documentation as complete:

- [ ] All auto-instrumented methods documented
- [ ] Complete attribute reference table
- [ ] Code snippets tested and working
- [ ] Sample app references verified
- [ ] Configuration options documented with defaults
- [ ] Manual instrumentation scenarios with examples
- [ ] Common patterns identified
- [ ] Known limitations documented
- [ ] Internal links working
- [ ] Follows template structure
- [ ] Proofread for clarity and accuracy

---

## 🔗 Related Resources

- **PROGRESS.md**: Session tracking and handover
- **examples-index.md**: Complete sample app catalog
- **traceloop-sdk-guide.md**: SDK foundation documentation
- **Category _EXPLORATION_PLAN.md**: Category-specific analysis recipes

---

**End of Exploration Guide** - Use this as reference for all instrumentation analysis
