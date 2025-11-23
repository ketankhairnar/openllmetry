# Anthropic Instrumentation

> **Package**: `opentelemetry-instrumentation-anthropic`
> **Supported Versions**: Anthropic SDK >= 0.3.0
> **Auto-Instrumentation**: ✅ Yes (via Traceloop SDK)
> **Provider**: Anthropic Claude, AWS Bedrock (Anthropic models)

---

## Table of Contents

- [Overview](#overview)
- [Quick Start](#quick-start)
- [What's Auto-Instrumented](#whats-auto-instrumented)
- [What's Captured](#whats-captured)
- [Advanced Features](#advanced-features)
- [AWS Bedrock Integration](#aws-bedrock-integration)
- [Configuration Options](#configuration-options)
- [Manual Instrumentation Scenarios](#manual-instrumentation-scenarios)
- [Sample Applications](#sample-applications)
- [Common Patterns](#common-patterns)
- [Troubleshooting](#troubleshooting)

---

## Overview

The Anthropic instrumentation package automatically traces all Anthropic SDK API calls, capturing request parameters, response data, token usage (including prompt caching), and performance metrics. It provides comprehensive support for Claude models including advanced features like extended thinking, tool use, and multimodal inputs.

**Key Features**:
- ✅ Complete API coverage (messages, completions, streaming)
- ✅ Extended thinking support (Claude 3.7 Sonnet)
- ✅ Prompt caching with detailed metrics
- ✅ Tool use / function calling
- ✅ Vision and multimodal inputs
- ✅ Streaming with context managers
- ✅ AWS Bedrock integration
- ✅ Beta API support

---

## Quick Start

### With Traceloop SDK (Recommended)

```python
import anthropic
from traceloop.sdk import Traceloop

# Initialize Traceloop - automatically instruments Anthropic
Traceloop.init(app_name="my-claude-app")

# Use Anthropic as normal - all calls are automatically traced
client = anthropic.Anthropic(api_key="your-api-key")
message = client.messages.create(
    model="claude-3-5-sonnet-20241022",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello, Claude!"}]
)
print(message.content[0].text)
```

### Manual Instrumentation

```python
import anthropic
from opentelemetry.instrumentation.anthropic import AnthropicInstrumentor

# Initialize instrumentation
AnthropicInstrumentor().instrument()

# Use Anthropic - all calls are traced
client = anthropic.Anthropic(api_key="your-api-key")
message = client.messages.create(
    model="claude-3-5-sonnet-20241022",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello, Claude!"}]
)
```

---

## What's Auto-Instrumented

### Messages API (Current)

**Span Name**: `"anthropic.chat"`

| Method | Streaming | Async | Description |
|--------|-----------|-------|-------------|
| `client.messages.create()` | ❌ | ✅ | Create message |
| `client.messages.stream()` | ✅ | ✅ | Streaming message |
| `client.beta.messages.create()` | ❌ | ✅ | Beta messages API |
| `client.beta.messages.stream()` | ✅ | ✅ | Beta streaming |

**Supports**:
- Multi-turn conversations
- System prompts
- Tool use / function calling
- Vision and multimodal inputs
- Extended thinking (Claude 3.7+)
- Prompt caching
- Streaming with helpers (`text_stream`, `get_final_message()`, `until_done()`)

### Completions API (Legacy)

**Span Name**: `"anthropic.completion"`

| Method | Streaming | Async | Description |
|--------|-----------|-------|-------------|
| `client.completions.create()` | ✅ | ✅ | Legacy completions |

**Note**: Legacy API, prefer Messages API for modern applications.

### AWS Bedrock Integration

| Method | Streaming | Async | Description |
|--------|-----------|-------|-------------|
| `bedrock_client.messages.create()` | ❌ | ✅ | Bedrock messages |
| `bedrock_client.messages.stream()` | ✅ | ✅ | Bedrock streaming |

**Note**: Uses `anthropic.AnthropicBedrock` client. See [AWS Bedrock Integration](#aws-bedrock-integration) section.

---

## What's Captured

### Spans

Each API call creates a span with:
- **Span Name**: `"anthropic.chat"` (Messages) or `"anthropic.completion"` (Legacy)
- **Span Kind**: `CLIENT`
- **Duration**: Total request time
- **Status**: `OK` or `ERROR` with exception details

### Request Attributes (GenAI Semantic Conventions)

| Attribute | Description | Example |
|-----------|-------------|---------|
| `gen_ai.system` | Provider name | `"Anthropic"` |
| `gen_ai.request.model` | Model identifier | `"claude-3-5-sonnet-20241022"` |
| `gen_ai.request.max_tokens` | Max tokens limit | `1024` |
| `gen_ai.request.temperature` | Temperature setting | `1.0` |
| `gen_ai.request.top_p` | Top-p sampling | `0.9` |

### Prompt Messages (When `TRACELOOP_TRACE_CONTENT=true`)

| Attribute | Description | Example |
|-----------|-------------|---------|
| `gen_ai.prompt.{i}.role` | Message role | `"user"`, `"assistant"`, `"system"` |
| `gen_ai.prompt.{i}.content` | Message text/content | `"Hello, Claude!"` |
| `gen_ai.prompt.{i}.tool_calls.{j}.id` | Tool call ID | `"toolu_01A..."` |
| `gen_ai.prompt.{i}.tool_calls.{j}.name` | Tool name | `"get_weather"` |
| `gen_ai.prompt.{i}.tool_calls.{j}.arguments` | Tool arguments (JSON) | `"{\"location\":\"SF\"}"` |

### Response Attributes

| Attribute | Description | Example |
|-----------|-------------|---------|
| `gen_ai.response.model` | Actual model used | `"claude-3-5-sonnet-20241022"` |
| `gen_ai.response.id` | Message ID | `"msg_01..."` |
| `gen_ai.usage.input_tokens` | Input tokens | `150` |
| `gen_ai.usage.output_tokens` | Output tokens | `300` |
| `gen_ai.usage.cache_read_input_tokens` | Cached prompt tokens | `100` |
| `gen_ai.usage.cache_creation_input_tokens` | Cache creation tokens | `500` |

### Completion Content (When `TRACELOOP_TRACE_CONTENT=true`)

| Attribute | Description | Example |
|-----------|-------------|---------|
| `gen_ai.completion.{i}.role` | Response role | `"assistant"`, `"thinking"` |
| `gen_ai.completion.{i}.content` | Response text | `"Hello! How can I help?"` |
| `gen_ai.completion.{i}.finish_reason` | Stop reason | `"end_turn"`, `"max_tokens"`, `"tool_use"` |
| `gen_ai.completion.{i}.tool_calls.{j}.id` | Tool call ID | `"toolu_01A..."` |
| `gen_ai.completion.{i}.tool_calls.{j}.name` | Function name | `"calculate"` |
| `gen_ai.completion.{i}.tool_calls.{j}.arguments` | Arguments (JSON) | `"{\"x\":5,\"y\":3}"` |

### Anthropic-Specific Attributes

| Attribute | Description | Example |
|-----------|-------------|---------|
| `llm.request.type` | Request type | `"completion"` |
| `llm.is_streaming` | Streaming flag | `true` |
| `llm.frequency_penalty` | Frequency penalty | `0.0` |
| `llm.presence_penalty` | Presence penalty | `0.0` |
| `llm.usage.total_tokens` | Total tokens | `450` |

### Tool Definitions

| Attribute | Description | Example |
|-----------|-------------|---------|
| `llm.request.functions.{i}.name` | Tool name | `"get_weather"` |
| `llm.request.functions.{i}.description` | Tool description | `"Get current weather"` |
| `llm.request.functions.{i}.input_schema` | JSON schema | `"{\"type\":\"object\"...}"` |

### Events (When `use_legacy_attributes=False`)

Instead of span attributes, events are emitted as OpenTelemetry LogRecords:

| Event Type | Description | Fields |
|------------|-------------|--------|
| `gen_ai.system.message` | System prompts | `content`, `role` |
| `gen_ai.user.message` | User messages | `content`, `role` |
| `gen_ai.assistant.message` | Assistant messages with tools | `content`, `role`, `tool_calls` |
| `gen_ai.choice` | Completion choices | `content`, `role`, `finish_reason`, `tool_calls` |

### Metrics

**Histograms**:
- `gen_ai.token_usage` - Token counts (input/output) with type label
- `gen_ai.operation_duration_milliseconds` - Request duration

**Counters**:
- `gen_ai.generation_choices` - Number of choices with finish_reason
- `anthropic.completion.exceptions` - Exception count by type

---

## Advanced Features

### Extended Thinking (Claude 3.7 Sonnet)

Claude 3.7 Sonnet and later support extended thinking, where the model "thinks" before responding:

```python
message = client.messages.create(
    model="claude-3-7-sonnet-20250219",
    max_tokens=4096,
    thinking={
        "type": "enabled",
        "budget_tokens": 1024  # Tokens allocated for thinking
    },
    messages=[{
        "role": "user",
        "content": "Solve this complex problem step by step..."
    }]
)

# Response contains thinking + assistant content
for block in message.content:
    if block.type == "thinking":
        print(f"Thinking: {block.thinking}")
    elif block.type == "text":
        print(f"Response: {block.text}")
```

**What's Captured**:
- Thinking content as separate completion block with `role="thinking"`
- Assistant response as separate completion block with `role="assistant"`
- Both captured in `gen_ai.completion.{i}.content` with different indices

**Sample**: See tests in `test_thinking.py`

### Prompt Caching

Anthropic's prompt caching reduces costs and latency for repeated content:

```python
# System message caching
message = client.messages.create(
    model="claude-3-5-sonnet-20241022",
    max_tokens=1024,
    system=[
        {
            "type": "text",
            "text": "Very long system prompt...",  # Eligible for caching
            "cache_control": {"type": "ephemeral"}
        }
    ],
    messages=[{"role": "user", "content": "User query"}]
)

# Check cache metrics
usage = message.usage
print(f"Cache read: {usage.cache_read_input_tokens}")  # Tokens from cache
print(f"Cache creation: {usage.cache_creation_input_tokens}")  # New cache
```

**What's Captured**:
- `gen_ai.usage.cache_read_input_tokens` - Tokens served from cache
- `gen_ai.usage.cache_creation_input_tokens` - Tokens written to cache
- Both included in total token calculations

**Supported**:
- System message caching
- User message caching (last message in conversation)
- Tool definition caching

**Sample**: See tests in `test_prompt_caching.py`

### Tool Use / Function Calling

Full tool use instrumentation with definitions and executions:

```python
tools = [
    {
        "name": "get_weather",
        "description": "Get the current weather in a location",
        "input_schema": {
            "type": "object",
            "properties": {
                "location": {
                    "type": "string",
                    "description": "City and state, e.g., San Francisco, CA"
                }
            },
            "required": ["location"]
        }
    }
]

message = client.messages.create(
    model="claude-3-5-sonnet-20241022",
    max_tokens=1024,
    tools=tools,
    messages=[{"role": "user", "content": "What's the weather in SF?"}]
)

# Check for tool use
for block in message.content:
    if block.type == "tool_use":
        print(f"Tool: {block.name}")
        print(f"Arguments: {block.input}")
```

**Captured**:
- **Tool Definitions**: `llm.request.functions.{i}.name/description/input_schema`
- **Tool Calls**: `gen_ai.completion.{i}.tool_calls.{j}.id/name/arguments`

**Works with streaming**: Tool calls captured from final accumulated response.

**Sample**: See tests in `test_messages.py` (search for `test_anthropic_tools`)

### Vision / Multimodal

Vision inputs with images are fully instrumented:

```python
import base64

# Read and encode image
with open("image.jpg", "rb") as f:
    image_data = base64.b64encode(f.read()).decode("utf-8")

message = client.messages.create(
    model="claude-3-5-sonnet-20241022",
    max_tokens=1024,
    messages=[{
        "role": "user",
        "content": [
            {"type": "text", "text": "What's in this image?"},
            {
                "type": "image",
                "source": {
                    "type": "base64",
                    "media_type": "image/jpeg",
                    "data": image_data
                }
            }
        ]
    }]
)
```

**Base64 Image Handling**:

```python
from opentelemetry.instrumentation.anthropic import AnthropicInstrumentor

# Configure callback to upload base64 images externally
async def upload_image(trace_id, span_id, image_name, base64_data):
    # Upload to S3, Cloud Storage, etc.
    url = await upload_to_storage(base64_data)
    return url

AnthropicInstrumentor().instrument(
    upload_base64_image=upload_image
)
```

**Why?** Large base64 images can bloat traces. The callback uploads them externally and replaces base64 with URLs.

**Supported Formats**: JPEG, PNG, GIF, WebP

**Sample**: See `packages/sample-app/sample_app/anthropic_vision_base64_example.py`

### Streaming

Streaming responses are fully instrumented with support for helper methods:

```python
# Basic streaming
with client.messages.stream(
    model="claude-3-5-sonnet-20241022",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Tell me a story"}]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)

# Access final message
message = stream.get_final_message()
print(f"\nTokens used: {message.usage.input_tokens}")
```

**Async Streaming**:

```python
async with client.messages.stream(
    model="claude-3-5-sonnet-20241022",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Tell me a story"}]
) as stream:
    async for text in stream.text_stream:
        print(text, end="", flush=True)
```

**What's Tracked**:
- ✅ Complete accumulated response
- ✅ Final token usage (input/output/cache)
- ✅ Tool calls (if any)
- ✅ Finish reason
- ✅ All response metadata

**Helper Methods Supported**:
- `text_stream` - Text-only iteration
- `get_final_message()` - Complete message object
- `until_done()` - Wait for completion

**Samples**:
- `packages/sample-app/sample_app/anthropic_joke_streaming_example.py` (sync)
- `packages/sample-app/sample_app/async_anthropic_joke_streaming.py` (async)

---

## AWS Bedrock Integration

Anthropic SDK includes native Bedrock support via `AnthropicBedrock` client:

### Setup

```python
from anthropic import AnthropicBedrock
from traceloop.sdk import Traceloop

Traceloop.init()

# Create Bedrock client
client = AnthropicBedrock(
    aws_access_key="your-access-key",
    aws_secret_key="your-secret-key",
    aws_region="us-east-1"
)

# Use exactly like standard Anthropic client - automatically traced
message = client.messages.create(
    model="anthropic.claude-3-5-sonnet-20241022-v2:0",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello!"}]
)
```

### What's Different

**Model IDs**: Use Bedrock format:
- `anthropic.claude-3-5-sonnet-20241022-v2:0`
- `anthropic.claude-3-haiku-20240307-v1:0`
- `anthropic.claude-3-opus-20240229-v1:0`

**Authentication**: AWS credentials (access key, secret key, region)

**All Features Supported**:
- ✅ Messages API
- ✅ Streaming
- ✅ Tool use
- ✅ Vision
- ✅ Prompt caching
- ✅ Extended thinking (Claude 3.7+)

**Special Pattern**: `with_raw_response()` for raw HTTP responses:

```python
raw_response = client.messages.with_raw_response().create(
    model="anthropic.claude-3-5-sonnet-20241022-v2:0",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello!"}]
)

# Automatically calls .parse() to extract message
message = raw_response.parse()
```

**Samples**: See tests in `test_bedrock_with_raw_response.py`

**Note**: For more advanced Bedrock features (multi-model support, guardrails, cross-region inference), see the [Bedrock instrumentation documentation](./bedrock.md).

---

## Configuration Options

### Instrumentor Parameters

```python
from opentelemetry.instrumentation.anthropic import AnthropicInstrumentor

AnthropicInstrumentor().instrument(
    # Token enrichment - use count_tokens API if response lacks usage
    enrich_token_usage=False,

    # Error handling - custom exception logger
    exception_logger=None,

    # Legacy vs events mode - use span attributes (True) vs events (False)
    use_legacy_attributes=True,

    # Custom metric attributes - add custom dimensions to metrics
    get_common_metrics_attributes=lambda: {"env": "production"},

    # Image handling - upload base64 images externally
    upload_base64_image=my_upload_function
)
```

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `TRACELOOP_TRACE_CONTENT` | `"true"` | Capture prompts and completions |
| `TRACELOOP_METRICS_ENABLED` | `"true"` | Enable metrics collection |

**Privacy Control**:

```bash
# Disable content capture for privacy
export TRACELOOP_TRACE_CONTENT=false
```

With content tracing disabled:
- ✅ Model names, token counts, durations still captured
- ❌ Actual prompt/completion text NOT captured
- ❌ Tool call arguments NOT captured

### Token Usage Enrichment

```python
# Enable token enrichment for responses without usage data
AnthropicInstrumentor().instrument(enrich_token_usage=True)
```

**When useful**: Some API responses may lack `usage` field. With `enrich_token_usage=True`, the instrumentor calls the `count_tokens` API to estimate tokens.

**Cost**: Extra API call per request without usage data.

---

## Manual Instrumentation Scenarios

### When Auto-Instrumentation Isn't Enough

1. **Custom Span Attributes**: Add application-specific context

```python
from opentelemetry import trace

tracer = trace.get_tracer(__name__)

with tracer.start_as_current_span("conversation") as span:
    span.set_attribute("user.id", user_id)
    span.set_attribute("conversation.topic", "support")

    message = client.messages.create(...)  # Auto-instrumented
```

2. **Workflow Tracking**: Use Traceloop decorators for multi-step workflows

```python
from traceloop.sdk.decorators import workflow, task

@task(name="generate_response")
def generate_response(user_input):
    return client.messages.create(
        model="claude-3-5-sonnet-20241022",
        max_tokens=1024,
        messages=[{"role": "user", "content": user_input}]
    )

@workflow(name="customer_support")
def handle_ticket(ticket):
    summary = generate_response(f"Summarize: {ticket}")
    response = generate_response(f"Respond to: {summary.content[0].text}")
    return response
```

3. **Tool Execution Tracking**: Trace tool execution separately

```python
from traceloop.sdk.decorators import task

@task(name="execute_tool")
def execute_tool(tool_name, tool_input):
    # Your tool execution logic
    if tool_name == "get_weather":
        return get_weather_data(tool_input["location"])
    # ...

# Use in conversation loop
message = client.messages.create(model="...", tools=tools, messages=messages)

for block in message.content:
    if block.type == "tool_use":
        result = execute_tool(block.name, block.input)  # Traced
        # Add result to messages and continue
```

---

## Sample Applications

All samples located in `packages/sample-app/sample_app/`

| File | Features Demonstrated |
|------|----------------------|
| `anthropic_joke_example.py` | Basic message creation, sync API |
| `anthropic_joke_streaming_example.py` | Streaming responses, event iteration |
| `anthropic_vision_base64_example.py` | Vision/multimodal, base64 images, async |
| `async_anthropic_example.py` | Async messages, agent decorator pattern |
| `async_anthropic_joke_streaming.py` | Async streaming, multiple agents |

### Running Samples

```bash
# Install dependencies
cd packages/sample-app
poetry install

# Set API key
export ANTHROPIC_API_KEY=your-key-here

# Run a sample
poetry run python sample_app/anthropic_joke_example.py
```

---

## Common Patterns

### 1. Multi-Turn Conversations

```python
from traceloop.sdk.decorators import workflow

@workflow(name="conversation")
def chat_conversation(messages):
    """Entire conversation tracked as one workflow"""
    message = client.messages.create(
        model="claude-3-5-sonnet-20241022",
        max_tokens=1024,
        messages=messages
    )
    return message

# Use
messages = [
    {"role": "user", "content": "Hello!"},
]

response1 = chat_conversation(messages)
messages.append({"role": "assistant", "content": response1.content})
messages.append({"role": "user", "content": "Tell me more"})
response2 = chat_conversation(messages)
```

### 2. Tool Use Loop

```python
def conversation_with_tools(user_input):
    messages = [{"role": "user", "content": user_input}]

    while True:
        message = client.messages.create(
            model="claude-3-5-sonnet-20241022",
            max_tokens=1024,
            tools=tools,
            messages=messages
        )

        if message.stop_reason != "tool_use":
            return message.content[0].text

        # Execute tools
        messages.append({"role": "assistant", "content": message.content})

        tool_results = []
        for block in message.content:
            if block.type == "tool_use":
                result = execute_tool(block.name, block.input)
                tool_results.append({
                    "type": "tool_result",
                    "tool_use_id": block.id,
                    "content": result
                })

        messages.append({"role": "user", "content": tool_results})
```

### 3. Streaming with Accumulation

```python
def stream_and_save(prompt):
    """Stream response and save complete text"""
    complete_text = ""

    with client.messages.stream(
        model="claude-3-5-sonnet-20241022",
        max_tokens=1024,
        messages=[{"role": "user", "content": prompt}]
    ) as stream:
        for text in stream.text_stream:
            print(text, end="", flush=True)
            complete_text += text

    return complete_text
```

### 4. Prompt Caching for Long Context

```python
def cached_system_conversation(system_prompt, user_queries):
    """Use cached system prompt for multiple queries"""

    for query in user_queries:
        message = client.messages.create(
            model="claude-3-5-sonnet-20241022",
            max_tokens=1024,
            system=[{
                "type": "text",
                "text": system_prompt,  # Cached after first call
                "cache_control": {"type": "ephemeral"}
            }],
            messages=[{"role": "user", "content": query}]
        )

        print(f"Cache read tokens: {message.usage.cache_read_input_tokens}")
        yield message.content[0].text
```

---

## Troubleshooting

### Issue: No spans appearing

**Symptoms**: Anthropic calls work but no traces visible

**Solutions**:
1. Verify instrumentation is initialized:
   ```python
   from traceloop.sdk import Traceloop
   Traceloop.init()  # Must be called before Anthropic import
   ```

2. Check if instrumentation is suppressed:
   ```python
   from opentelemetry import context
   from opentelemetry.instrumentation.anthropic.shared import SUPPRESS_LANGUAGE_MODEL_INSTRUMENTATION_KEY

   # Ensure not suppressed
   assert context.get_value(SUPPRESS_LANGUAGE_MODEL_INSTRUMENTATION_KEY) != True
   ```

3. Verify exporter is configured:
   ```python
   from opentelemetry.sdk.trace.export import ConsoleSpanExporter
   Traceloop.init(exporter=ConsoleSpanExporter())  # Debug with console
   ```

### Issue: Content not captured

**Symptoms**: Spans exist but prompt/completion content is empty

**Solution**: Enable content tracing:
```bash
export TRACELOOP_TRACE_CONTENT=true
```

### Issue: Streaming spans incomplete

**Symptoms**: Streaming responses show partial data

**Cause**: Stream must be fully consumed (or context manager exited) for span to complete.

**Solution**:
```python
# Bad - stream not consumed
stream = client.messages.stream(...)
# Span still open!

# Good - use context manager
with client.messages.stream(...) as stream:
    for text in stream.text_stream:
        process(text)
# Span now complete
```

### Issue: Missing cache metrics

**Symptoms**: Prompt caching enabled but no cache token attributes

**Cause**: Cache control must be explicitly set in requests.

**Solution**:
```python
# Ensure cache_control is set
system=[{
    "type": "text",
    "text": "Your long system prompt...",
    "cache_control": {"type": "ephemeral"}  # Required!
}]
```

### Issue: Tool calls not captured in streaming

**Symptoms**: Tool use works but not visible in traces during streaming

**Cause**: Tool calls are only available in final message, not individual chunks.

**Solution**: Use `stream.get_final_message()`:
```python
with client.messages.stream(...) as stream:
    for text in stream.text_stream:
        print(text, end="")

    # Access complete message with tool calls
    final_message = stream.get_final_message()
    for block in final_message.content:
        if block.type == "tool_use":
            # Now available
            print(f"Tool: {block.name}")
```

### Issue: Bedrock authentication errors

**Symptoms**: AWS credentials errors when using `AnthropicBedrock`

**Solutions**:
1. Verify credentials:
   ```python
   import boto3
   # Test AWS credentials
   boto3.client('bedrock-runtime', region_name='us-east-1')
   ```

2. Check IAM permissions:
   - Required: `bedrock:InvokeModel`
   - Optional (streaming): `bedrock:InvokeModelWithResponseStream`

3. Verify region supports Bedrock:
   - Available regions: `us-east-1`, `us-west-2`, `ap-southeast-1`, etc.

### Issue: High cardinality metrics

**Symptoms**: Too many unique metric combinations

**Cause**: User IDs or other high-cardinality data in metric attributes.

**Solution**: Filter attributes:
```python
AnthropicInstrumentor().instrument(
    get_common_metrics_attributes=lambda: {
        "env": "production",
        # Don't include user_id, session_id, etc.
    }
)
```

---

## Additional Resources

- **Anthropic API Documentation**: https://docs.anthropic.com/
- **Prompt Caching Guide**: https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching
- **Tool Use Guide**: https://docs.anthropic.com/en/docs/build-with-claude/tool-use
- **AWS Bedrock Documentation**: https://docs.aws.amazon.com/bedrock/
- **OpenTelemetry GenAI Semantic Conventions**: https://opentelemetry.io/docs/specs/semconv/gen-ai/
- **Traceloop SDK Guide**: [../../traceloop-sdk-guide.md](../../traceloop-sdk-guide.md)
- **Sample Applications**: [../../examples-index.md](../../examples-index.md)

---

**Last Updated**: 2025-11-22 (Session 2)
