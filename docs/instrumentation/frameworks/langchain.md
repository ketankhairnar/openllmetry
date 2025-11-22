# LangChain Instrumentation

> **Package**: `opentelemetry-instrumentation-langchain`
> **Supported Versions**: langchain-core > 0.1.0
> **Auto-Instrumentation**: ✅ Yes (via Traceloop SDK)
> **Framework**: LangChain, LangChain Expression Language (LCEL), LangGraph

---

## Table of Contents

- [Overview](#overview)
- [Quick Start](#quick-start)
- [What's Auto-Instrumented](#whats-auto-instrumented)
- [What's Captured](#whats-captured)
- [Advanced Features](#advanced-features)
- [Configuration Options](#configuration-options)
- [Manual Instrumentation Scenarios](#manual-instrumentation-scenarios)
- [Sample Applications](#sample-applications)
- [Common Patterns](#common-patterns)
- [Troubleshooting](#troubleshooting)

---

## Overview

The LangChain instrumentation package automatically traces all LangChain operations through its callback system. It provides comprehensive support for LCEL (LangChain Expression Language), agents, tools, chains, and works with 12+ LLM providers.

**Key Features**:
- ✅ Complete LCEL support (Runnables, pipes, parallel execution)
- ✅ Agent and tool execution tracking
- ✅ LangGraph workflow instrumentation
- ✅ Multi-provider LLM support (OpenAI, Anthropic, Bedrock, etc.)
- ✅ Streaming and async operations
- ✅ Tool calling and structured outputs
- ✅ Automatic workflow/task hierarchy
- ✅ Trace context propagation to LLM backends

---

## Quick Start

### With Traceloop SDK (Recommended)

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from traceloop.sdk import Traceloop

# Initialize Traceloop - automatically instruments LangChain
Traceloop.init(app_name="my-langchain-app")

# Build LCEL chain - all operations automatically traced
prompt = ChatPromptTemplate.from_template("Tell me a joke about {topic}")
model = ChatOpenAI(model="gpt-4o-mini")
output_parser = StrOutputParser()

chain = prompt | model | output_parser

# Invoke chain - creates workflow span with task hierarchy
result = chain.invoke({"topic": "AI"})
print(result)
```

### Manual Instrumentation

```python
from opentelemetry.instrumentation.langchain import LangchainInstrumentor

# Initialize instrumentation
LangchainInstrumentor().instrument()

# Use LangChain - all operations are traced
chain = prompt | model | output_parser
result = chain.invoke({"topic": "AI"})
```

---

## What's Auto-Instrumented

### LangChain Expression Language (LCEL)

**All Runnable types are automatically instrumented**:

| Component | Span Type | Description |
|-----------|-----------|-------------|
| `RunnableSequence` | workflow | Piped operations (`a \| b \| c`) |
| `RunnableLambda` | task | Custom functions |
| `RunnableParallel` | workflow | Parallel execution |
| `ChatPromptTemplate` | task | Prompt templates |
| `Output Parsers` | task | JSON, String, Pydantic parsers |
| `Custom Runnables` | task | User-defined runnables |

**Example Hierarchy**:
```python
chain = prompt | model | parser
# Creates:
# └─ RunnableSequence.workflow
#    ├─ ChatPromptTemplate.task
#    ├─ ChatOpenAI.chat (LLM span)
#    └─ StrOutputParser.task
```

### LLM Providers

**Supported via vendor detection** (12+ providers):

| Provider | Detected As | Span Name Pattern |
|----------|-------------|-------------------|
| OpenAI / Azure OpenAI | `openai` / `Azure` | `ChatOpenAI.chat` |
| Anthropic | `Anthropic` | `ChatAnthropic.chat` |
| AWS Bedrock | `bedrock` | `ChatBedrock.chat` |
| Google (Vertex AI, Gemini) | `Google` | `ChatVertexAI.chat` |
| Cohere | `Cohere` | `ChatCohere.chat` |
| HuggingFace | `HuggingFace` | `HuggingFaceHub.completion` |
| Ollama | `Ollama` | `ChatOllama.chat` |
| Together | `Together` | `ChatTogether.chat` |
| Replicate | `Replicate` | `ChatReplicate.chat` |
| Fireworks | `Fireworks` | `ChatFireworks.chat` |
| Groq | `Groq` | `ChatGroq.chat` |
| Mistral AI | `Mistral` | `ChatMistralAI.chat` |
| IBM Watsonx | `IBM` | `WatsonxLLM.completion` |

**Automatic detection**: Vendor identified from LLM class name, no configuration needed.

### Agents & Tools

| Component | Span Type | Description |
|-----------|-----------|-------------|
| `AgentExecutor` | workflow | Agent orchestration |
| `create_react_agent` | workflow | ReAct pattern agents |
| `create_openai_tools_agent` | workflow | Tool-calling agents |
| Custom tools | tool | Functions decorated with `@tool` |
| Built-in tools | tool | Search, calculator, etc. |

**Agent Loop Tracking**:
- Each agent iteration creates task spans
- Tool executions become child tool spans
- Multi-turn conversations properly hierarchical

### LangGraph

**Full support via callback mechanism**:
- State graphs instrumented as workflows
- Individual nodes become tasks
- LLM calls within nodes properly traced
- Parent-child relationships maintained

### Legacy Chains

| Chain Type | Support | Span Type |
|------------|---------|-----------|
| `LLMChain` | ✅ | workflow |
| `SequentialChain` | ✅ | workflow |
| `ConversationChain` | ✅ | workflow |
| Document chains | ✅ | workflow |
| Custom chains | ✅ | workflow |

**Note**: LCEL is preferred, but legacy chains fully supported.

---

## What's Captured

### Spans

Each component creates a span with:
- **Span Name**: `{ComponentName}.{SpanKind}` (e.g., `ChatOpenAI.chat`, `RunnableSequence.workflow`)
- **Span Kind**: Based on component type (workflow, task, tool)
- **Duration**: Component execution time
- **Hierarchy**: Parent-child relationships reflecting execution flow

### Workflow/Task Attributes

| Attribute | Description | Example |
|-----------|-------------|---------|
| `traceloop.span.kind` | Span classification | `"workflow"`, `"task"`, `"tool"` |
| `traceloop.workflow.name` | Top-level workflow name | `"RunnableSequence"` |
| `traceloop.entity.name` | Component name | `"ChatOpenAI"` |
| `traceloop.entity.path` | Hierarchical path | `"AgentExecutor.agent_loop"` |
| `traceloop.entity.input` | Component input (JSON) | `{"topic": "AI"}` |
| `traceloop.entity.output` | Component output (JSON) | `"Here's a joke..."` |

### LLM Request Attributes (GenAI Semantic Conventions)

| Attribute | Description | Example |
|-----------|-------------|---------|
| `gen_ai.system` | Provider name | `"openai"`, `"Anthropic"` |
| `gen_ai.request.model` | Model identifier | `"gpt-4o-mini"` |
| `gen_ai.response.model` | Actual model used | `"gpt-4o-mini-2024-07-18"` |
| `gen_ai.request.max_tokens` | Token limit | `1024` |
| `gen_ai.request.temperature` | Temperature setting | `0.7` |
| `gen_ai.request.top_p` | Top-p sampling | `1.0` |
| `llm.request.type` | Request type | `"chat"`, `"completion"` |

### Prompt & Completion Content (When `TRACELOOP_TRACE_CONTENT=true`)

| Attribute | Description | Example |
|-----------|-------------|---------|
| `gen_ai.prompt.{i}.role` | Message role | `"user"`, `"system"`, `"assistant"` |
| `gen_ai.prompt.{i}.content` | Prompt text | `"Tell me a joke"` |
| `gen_ai.completion.{i}.role` | Response role | `"assistant"` |
| `gen_ai.completion.{i}.content` | Response text | `"Why did..."` |
| `gen_ai.completion.{i}.finish_reason` | Stop reason | `"stop"`, `"tool_calls"`, `"length"` |

### Tool Calling

| Attribute | Description | Example |
|-----------|-------------|---------|
| `llm.request.functions.{i}.name` | Function name | `"get_weather"` |
| `llm.request.functions.{i}.description` | Function description | `"Get current weather"` |
| `llm.request.functions.{i}.parameters` | JSON schema | `"{\"type\":\"object\"...}"` |
| `gen_ai.completion.{i}.tool_calls.{j}.id` | Tool call ID | `"call_abc123"` |
| `gen_ai.completion.{i}.tool_calls.{j}.name` | Called function | `"get_weather"` |
| `gen_ai.completion.{i}.tool_calls.{j}.arguments` | Call arguments (JSON) | `"{\"location\":\"SF\"}"` |

### Token Usage

| Attribute | Description | Example |
|-----------|-------------|---------|
| `gen_ai.usage.input_tokens` | Prompt tokens | `150` |
| `gen_ai.usage.output_tokens` | Completion tokens | `300` |
| `llm.usage.total_tokens` | Total tokens | `450` |
| `gen_ai.usage.cache_read_input_tokens` | Cached tokens | `100` |

### Association Properties (Metadata)

Custom metadata passed via `config` parameter:

```python
chain.invoke(
    {"topic": "AI"},
    config={"metadata": {"user_id": "123", "session_id": "abc"}}
)
```

Captured as:
- `traceloop.association_properties.user_id` = `"123"`
- `traceloop.association_properties.session_id` = `"abc"`

### Metrics

**Token Usage Histogram**:
- Name: `gen_ai.token.usage`
- Unit: tokens
- Labels: `gen_ai.system`, `gen_ai.response.model`, `gen_ai.token.type` (input/output)

**Operation Duration Histogram**:
- Name: `gen_ai.operation.duration`
- Unit: seconds
- Labels: `gen_ai.system`, `gen_ai.response.model`

---

## Advanced Features

### LCEL Pipes and Parallelism

**Piped operations** (sequential):
```python
chain = prompt | model | parser
# Creates linear span hierarchy
```

**Parallel execution**:
```python
from langchain_core.runnables import RunnableParallel

chain = RunnableParallel({
    "joke": prompt | model1,
    "poem": prompt | model2
})
# Creates parallel child spans under RunnableParallel.workflow
```

**What's Tracked**:
- ✅ Execution order
- ✅ Parallel vs sequential execution
- ✅ Individual component timings
- ✅ Complete input/output data flow

### Agents and Tool Execution

Full agent loop instrumentation:

```python
from langchain.agents import AgentExecutor, create_openai_tools_agent
from langchain.tools import tool

@tool
def get_weather(location: str) -> str:
    """Get the current weather."""
    return f"Weather in {location}: Sunny"

agent = create_openai_tools_agent(llm, tools=[get_weather], prompt=prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools)

result = agent_executor.invoke({"input": "What's the weather in SF?"})
```

**Span Hierarchy**:
```
AgentExecutor.workflow
├─ ChatOpenAI.chat (agent reasoning)
├─ get_weather.tool (tool execution)
└─ ChatOpenAI.chat (final response)
```

**What's Tracked**:
- ✅ Agent iterations and loops
- ✅ Tool selection and execution
- ✅ Tool results
- ✅ Final agent output

### Streaming

Streaming is fully supported with complete data capture:

```python
for chunk in chain.stream({"topic": "AI"}):
    print(chunk, end="", flush=True)

# Span created with:
# - Full accumulated response
# - Complete token usage
# - Finish reason
```

**Async Streaming**:
```python
async for chunk in chain.astream({"topic": "AI"}):
    print(chunk, end="", flush=True)
```

**What's Tracked**:
- ✅ Complete streamed response
- ✅ Token counts (all chunks)
- ✅ Streaming duration
- ✅ Component hierarchy maintained

### Structured Outputs

Tool calling and structured output parsing are fully instrumented:

```python
from pydantic import BaseModel

class Joke(BaseModel):
    setup: str
    punchline: str

# Structured output via tool calling
structured_llm = model.with_structured_output(Joke)
result = structured_llm.invoke("Tell me a joke")

# Tool definitions captured in span
# Pydantic model schema recorded
# Output validation transparent to tracing
```

**What's Captured**:
- Tool definitions as `llm.request.functions`
- Tool call invocations
- Structured output in entity.output

### LangGraph Workflows

LangGraph state graphs are instrumented via callbacks:

```python
from langgraph.graph import StateGraph

workflow = StateGraph(AgentState)
workflow.add_node("agent", agent_node)
workflow.add_node("action", action_node)
workflow.add_edge("agent", "action")

app = workflow.compile()
result = app.invoke({"input": "query"})
```

**What's Tracked**:
- ✅ Graph structure (via callback events)
- ✅ Node executions as tasks
- ✅ LLM calls within nodes
- ✅ State transitions

### Trace Context Propagation

W3C Trace Context automatically injected into OpenAI API calls:

```python
# Traceloop → LangChain → OpenAI
# Trace context flows through all layers

from langchain_openai import ChatOpenAI

model = ChatOpenAI()
result = model.invoke("Hello")

# OpenAI receives `traceparent` header
# Enables distributed tracing across services
```

**Supported**:
- ✅ OpenAI (via `langchain_openai`)
- ✅ Azure OpenAI
- ✅ OpenAI-compatible endpoints

**Disable if needed**:
```python
LangchainInstrumentor().instrument(disable_trace_context_propagation=True)
```

### Async Operations

Full async support with safe context handling:

```python
import asyncio

async def async_chain():
    result = await chain.ainvoke({"topic": "AI"})
    return result

asyncio.run(async_chain())
```

**Safety Features**:
- ✅ Context detachment guards
- ✅ Race condition prevention
- ✅ Async callback support
- ✅ Proper span parent-child relationships

---

## Configuration Options

### Instrumentor Parameters

```python
from opentelemetry.instrumentation.langchain import LangchainInstrumentor

LangchainInstrumentor().instrument(
    # Error handling - custom exception logger
    exception_logger=None,

    # Trace context propagation - disable OpenAI header injection
    disable_trace_context_propagation=False,

    # Legacy vs events mode - use span attributes (True) vs log records (False)
    use_legacy_attributes=True
)
```

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `TRACELOOP_TRACE_CONTENT` | `"true"` | Capture prompts, completions, inputs, outputs |

**Privacy Control**:

```bash
# Disable content capture for privacy
export TRACELOOP_TRACE_CONTENT=false
```

With content tracing disabled:
- ✅ Component names, models, durations still captured
- ✅ Token counts still recorded
- ❌ Actual prompts/completions NOT captured
- ❌ Input/output data NOT captured
- ❌ Tool call arguments NOT captured

### Metadata Passing

Add custom metadata to traces:

```python
result = chain.invoke(
    {"topic": "AI"},
    config={
        "metadata": {
            "user_id": "user123",
            "session_id": "session456",
            "environment": "production"
        }
    }
)

# Appears as:
# traceloop.association_properties.user_id = "user123"
# traceloop.association_properties.session_id = "session456"
# traceloop.association_properties.environment = "production"
```

**Supported Types**: Strings, numbers, booleans (sanitized for OpenTelemetry)

---

## Manual Instrumentation Scenarios

### When Auto-Instrumentation Isn't Enough

1. **Custom Workflow Names**: Use Traceloop decorators for better names

```python
from traceloop.sdk.decorators import workflow, task

@workflow(name="customer_support_flow")
def handle_support_ticket(ticket):
    summary = summarize_chain.invoke({"ticket": ticket})
    response = response_chain.invoke({"summary": summary})
    return response

# Creates: customer_support_flow.workflow
# Instead of: generic function name
```

2. **Manual Task Boundaries**: Add explicit task spans

```python
from traceloop.sdk.decorators import task

@task(name="validate_input")
def validate_input(user_input):
    # Validation logic
    return cleaned_input

@task(name="post_process")
def post_process(llm_output):
    # Post-processing logic
    return final_output

# Use in chain
chain = RunnableLambda(validate_input) | model | RunnableLambda(post_process)
```

3. **Custom Attributes**: Add application-specific context

```python
from opentelemetry import trace

@workflow(name="document_qa")
def answer_question(question, document_id):
    tracer = trace.get_tracer(__name__)
    span = trace.get_current_span()

    # Add custom attributes
    span.set_attribute("document.id", document_id)
    span.set_attribute("question.length", len(question))

    result = qa_chain.invoke({"question": question, "context": load_doc(document_id)})
    return result
```

4. **Error Context**: Add error details

```python
@workflow(name="safe_invoke")
def safe_invoke_chain(input_data):
    try:
        return chain.invoke(input_data)
    except Exception as e:
        span = trace.get_current_span()
        span.set_attribute("error.type", type(e).__name__)
        span.set_attribute("error.handled", True)
        return {"error": str(e)}
```

---

## Sample Applications

All samples located in `packages/sample-app/sample_app/`

### 1. Basic LCEL Chain

**File**: `langchain_app.py`

**Features**:
- Simple LCEL pipeline
- ChatPromptTemplate with variables
- ChatOpenAI integration
- String output parsing
- Multi-step workflow

**Code**:
```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

prompt = ChatPromptTemplate.from_template("Tell me a joke about {topic}")
model = ChatOpenAI(model="gpt-4o-mini")
output_parser = StrOutputParser()

chain = prompt | model | output_parser
result = chain.invoke({"topic": "AI"})
```

### 2. Agent with Tools

**File**: `langchain_agent.py`

**Features**:
- AgentExecutor pattern
- Custom tool definitions
- Tool-calling loop
- Multi-turn LLM interactions

**What's Demonstrated**:
- Agent iteration tracking
- Tool execution spans
- Agent decision-making visibility

### 3. Async LCEL with Functions

**File**: `langchain_lcel.py`

**Features**:
- Async invocation (`ainvoke`)
- Function calling (tool definitions)
- Metadata propagation
- Pydantic output parsing

**Code**:
```python
chain = prompt | model.bind(functions=tools) | parser
result = await chain.ainvoke(
    {"input": "query"},
    {"metadata": {"user_id": "123"}}
)
```

### 4. IBM Watsonx Integration

**File**: `langchain_watsonx.py`, `watsonx-langchain.py`

**Features**:
- Non-OpenAI provider (Watsonx)
- Custom LLM parameters
- Legacy LLMChain pattern
- Workflow decorators

**What's Demonstrated**:
- Vendor-agnostic instrumentation
- Custom provider support
- Legacy vs LCEL patterns

### Running Samples

```bash
# Install dependencies
cd packages/sample-app
poetry install

# Set API keys
export OPENAI_API_KEY=your-key-here

# Run a sample
poetry run python sample_app/langchain_app.py
```

---

## Common Patterns

### 1. Multi-Step LCEL Pipeline

```python
from langchain_core.runnables import RunnablePassthrough

chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | prompt
    | model
    | parser
)

# Creates hierarchical spans:
# RunnableSequence.workflow
# ├─ RunnableParallel.workflow (context + question)
# │  └─ Retriever.task
# ├─ ChatPromptTemplate.task
# ├─ ChatOpenAI.chat
# └─ StrOutputParser.task
```

### 2. Agent Loop with Multiple Tools

```python
from langchain.agents import create_openai_tools_agent, AgentExecutor
from langchain.tools import tool

@tool
def search(query: str) -> str:
    """Search the web."""
    return search_api(query)

@tool
def calculate(expression: str) -> float:
    """Calculate math expression."""
    return eval(expression)

agent = create_openai_tools_agent(llm, tools=[search, calculate], prompt=prompt)
executor = AgentExecutor(agent=agent, tools=tools, verbose=True)

result = executor.invoke({"input": "What's 25% of the population of France?"})

# Traces show:
# - Agent reasoning steps
# - Tool selections
# - Tool executions
# - Final synthesis
```

### 3. Streaming with Progress Tracking

```python
@workflow(name="streaming_response")
def stream_response(query):
    full_response = ""

    for chunk in chain.stream({"query": query}):
        print(chunk, end="", flush=True)
        full_response += chunk

    # Span includes complete response and timing
    return full_response
```

### 4. Batch Processing with Metadata

```python
inputs = [
    {"topic": "AI", "metadata": {"batch_id": "1", "item": "1"}},
    {"topic": "ML", "metadata": {"batch_id": "1", "item": "2"}},
    {"topic": "DL", "metadata": {"batch_id": "1", "item": "3"}}
]

results = chain.batch(
    [{"topic": item["topic"]} for item in inputs],
    config=[{"metadata": item["metadata"]} for item in inputs]
)

# Each invocation gets its own span with metadata
```

### 5. Conditional Routing

```python
from langchain_core.runnables import RunnableBranch

branch = RunnableBranch(
    (lambda x: "code" in x["query"], code_chain),
    (lambda x: "math" in x["query"], math_chain),
    default_chain
)

chain = prompt | branch | parser

# Traces show which branch was taken
```

---

## Troubleshooting

### Issue: No spans appearing

**Symptoms**: LangChain works but no traces visible

**Solutions**:
1. Verify Traceloop initialization before LangChain imports:
   ```python
   from traceloop.sdk import Traceloop
   Traceloop.init()  # Must be called first

   from langchain_openai import ChatOpenAI  # Then import LangChain
   ```

2. Check callback handler injection:
   ```python
   from langchain_core.callbacks import get_callback_manager

   # Should include TraceloopCallbackHandler
   print(get_callback_manager().handlers)
   ```

3. Verify exporter configured:
   ```python
   from opentelemetry.sdk.trace.export import ConsoleSpanExporter
   Traceloop.init(exporter=ConsoleSpanExporter())
   ```

### Issue: Incomplete span hierarchy

**Symptoms**: Spans appear but parent-child relationships broken

**Cause**: Context propagation issues in async code.

**Solution**:
```python
# Use proper async patterns
async def async_workflow():
    # Correct
    result = await chain.ainvoke(input)

    # Incorrect - loses context
    # result = asyncio.create_task(chain.ainvoke(input))
```

### Issue: Content not captured

**Symptoms**: Spans exist but prompts/completions empty

**Solution**: Enable content tracing:
```bash
export TRACELOOP_TRACE_CONTENT=true
```

### Issue: Tool calls not visible

**Symptoms**: Agent works but tool executions not traced

**Causes**:
1. Tools not properly decorated
2. Content tracing disabled

**Solutions**:
```python
# Ensure tools are decorated
from langchain.tools import tool

@tool  # Must have decorator
def my_tool(input: str) -> str:
    """Tool description."""
    return result

# Enable content tracing
export TRACELOOP_TRACE_CONTENT=true
```

### Issue: Token counts missing

**Symptoms**: LLM spans exist but no token usage

**Causes**:
1. LLM provider doesn't return usage data
2. Streaming without proper callback handling

**Solutions**:
```python
# Use providers that return usage
# OpenAI, Anthropic, Bedrock all supported

# For streaming, ensure callbacks fire
stream = chain.stream(input)
list(stream)  # Consume stream to trigger on_llm_end callback
```

### Issue: High cardinality on association properties

**Symptoms**: Too many unique span attribute combinations

**Cause**: Including high-cardinality data in metadata (user IDs, session IDs).

**Solution**: Filter what goes into association properties:
```python
# Don't include high-cardinality fields in config metadata
# Instead use span attributes directly

from opentelemetry import trace

span = trace.get_current_span()
span.set_attribute("user.id", user_id)  # Not in metadata
```

### Issue: OpenAI trace propagation errors

**Symptoms**: Errors related to traceparent headers

**Solution**: Disable trace propagation:
```python
LangchainInstrumentor().instrument(disable_trace_context_propagation=True)
```

### Issue: Memory leaks in long-running applications

**Symptoms**: Memory usage grows over time

**Cause**: Span contexts not properly detached.

**Solution**: Use workflow decorators for clean boundaries:
```python
@workflow(name="request_handler")
def handle_request(data):
    # Workflow scope ensures proper cleanup
    return chain.invoke(data)
```

---

## Additional Resources

- **LangChain Documentation**: https://python.langchain.com/docs/
- **LCEL Guide**: https://python.langchain.com/docs/expression_language/
- **LangGraph**: https://langchain-ai.github.io/langgraph/
- **LangChain Callbacks**: https://python.langchain.com/docs/modules/callbacks/
- **OpenTelemetry GenAI Semantic Conventions**: https://opentelemetry.io/docs/specs/semconv/gen-ai/
- **Traceloop SDK Guide**: [../../traceloop-sdk-guide.md](../../traceloop-sdk-guide.md)
- **Sample Applications**: [../../examples-index.md](../../examples-index.md)

---

**Last Updated**: 2025-11-22 (Session 3)
