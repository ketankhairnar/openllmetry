# AWS Bedrock Instrumentation

> **Package**: `opentelemetry-instrumentation-bedrock`
> **Supported Versions**: boto3 >= 1.28.57
> **Auto-Instrumentation**: ✅ Yes (via Traceloop SDK)
> **Provider**: AWS Bedrock (multi-model support)

---

## Table of Contents

- [Overview](#overview)
- [Quick Start](#quick-start)
- [What's Auto-Instrumented](#whats-auto-instrumented)
- [Supported Models](#supported-models)
- [What's Captured](#whats-captured)
- [Advanced Features](#advanced-features)
- [Configuration Options](#configuration-options)
- [Manual Instrumentation Scenarios](#manual-instrumentation-scenarios)
- [Sample Applications](#sample-applications)
- [Common Patterns](#common-patterns)
- [Troubleshooting](#troubleshooting)

---

## Overview

The AWS Bedrock instrumentation package automatically traces all Bedrock runtime API calls across multiple model providers. It provides comprehensive support for streaming, guardrails, prompt caching, and the new unified Converse API.

**Key Features**:
- ✅ Multi-model support (Anthropic, Cohere, AI21, Meta, Amazon, Imported)
- ✅ Complete API coverage (invoke_model, converse, streaming variants)
- ✅ Guardrails monitoring with detailed metrics
- ✅ Prompt caching detection and tracking
- ✅ Streaming with sophisticated event handling
- ✅ Cross-region inference profiles
- ✅ Tool use support (Converse API)

---

## Quick Start

### With Traceloop SDK (Recommended)

```python
import boto3
import json
from traceloop.sdk import Traceloop

# Initialize Traceloop - automatically instruments Bedrock
Traceloop.init(app_name="my-bedrock-app")

# Create Bedrock client - automatically traced
bedrock = boto3.client(
    service_name='bedrock-runtime',
    region_name='us-east-1'
)

# Use Bedrock - all calls are traced
body = json.dumps({
    "anthropic_version": "bedrock-2023-05-31",
    "max_tokens": 1024,
    "messages": [{
        "role": "user",
        "content": "Hello!"
    }]
})

response = bedrock.invoke_model(
    modelId='anthropic.claude-3-5-sonnet-20241022-v2:0',
    body=body
)

result = json.loads(response['body'].read())
print(result['content'][0]['text'])
```

### Manual Instrumentation

```python
import boto3
from opentelemetry.instrumentation.bedrock import BedrockInstrumentor

# Initialize instrumentation
BedrockInstrumentor().instrument()

# Use Bedrock - all calls are traced
bedrock = boto3.client('bedrock-runtime', region_name='us-east-1')
response = bedrock.invoke_model(...)
```

---

## What's Auto-Instrumented

### Invoke Model API

**Span Name**: `"bedrock.completion"`

| Method | Streaming | Description |
|--------|-----------|-------------|
| `invoke_model()` | ❌ | Synchronous completion |
| `invoke_model_with_response_stream()` | ✅ | Streaming completion |

**Use For**:
- Model-specific completion requests
- Raw model APIs (provider-specific formats)
- Maximum flexibility with model parameters

### Converse API (Unified)

**Span Name**: `"bedrock.converse"`

| Method | Streaming | Description |
|--------|-----------|-------------|
| `converse()` | ❌ | Synchronous conversation |
| `converse_stream()` | ✅ | Streaming conversation |

**Use For**:
- Unified chat interface across all models
- Built-in tool use support
- Simplified message format
- Guardrail integration

**Advantages**:
- ✅ Consistent API across all providers
- ✅ Native tool calling support
- ✅ Simplified message structure
- ✅ Better error handling

---

## Supported Models

The instrumentation automatically detects and supports all Bedrock model providers:

### Anthropic (Claude)

**Models**: Claude 3.5 Sonnet, Claude 3 Opus, Claude 3 Haiku, Claude 2.1, Claude 2.0, Claude Instant

**Model IDs**:
- `anthropic.claude-3-5-sonnet-20241022-v2:0`
- `anthropic.claude-3-opus-20240229-v1:0`
- `anthropic.claude-3-haiku-20240307-v1:0`
- `anthropic.claude-v2:1`
- `anthropic.claude-instant-v1`

**Features Supported**:
- ✅ Messages API and legacy completions
- ✅ Streaming
- ✅ Prompt caching
- ✅ Vision (multimodal)
- ✅ Tool use
- ✅ Thinking blocks (Claude 3.7+, when available)

### Cohere

**Models**: Command, Command Light, Command R, Command R+

**Model IDs**:
- `cohere.command-text-v14`
- `cohere.command-light-text-v14`
- `cohere.command-r-v1:0`
- `cohere.command-r-plus-v1:0`

**Features Supported**:
- ✅ Text generation
- ✅ Streaming
- ✅ Token counting

### AI21 Labs

**Models**: Jurassic-2 Mid, Jurassic-2 Ultra

**Model IDs**:
- `ai21.j2-mid-v1`
- `ai21.j2-ultra-v1`

**Features Supported**:
- ✅ Text completion
- ✅ Token counting

### Meta

**Models**: Llama 2, Llama 3, Llama 3.1, Llama 3.2

**Model IDs**:
- `meta.llama2-13b-chat-v1`
- `meta.llama2-70b-chat-v1`
- `meta.llama3-8b-instruct-v1:0`
- `meta.llama3-70b-instruct-v1:0`
- `meta.llama3-1-405b-instruct-v1:0`

**Features Supported**:
- ✅ Chat and instruction formats
- ✅ Streaming
- ✅ Token counting

### Amazon

**Models**: Titan Text, Titan Embeddings, Nova

**Model IDs**:
- `amazon.titan-text-express-v1`
- `amazon.titan-text-lite-v1`
- `amazon.nova-micro-v1:0`
- `amazon.nova-lite-v1:0`
- `amazon.nova-pro-v1:0`

**Features Supported**:
- ✅ Text generation
- ✅ System messages (Nova)
- ✅ Streaming
- ✅ Converse API (Nova)

### Cross-Region Inference Profiles

**ARN Format**:
```
arn:aws:bedrock:region:account-id:inference-profile/us.anthropic.claude-3-5-sonnet-20241022-v2:0
```

**Supported Regions**: `us`, `us-gov`, `eu`, `apac`

**Detection**: Automatically extracts model ID from ARN and regional prefixes.

---

## What's Captured

### Spans

Each API call creates a span with:
- **Span Name**: `"bedrock.completion"` (invoke_model) or `"bedrock.converse"` (converse)
- **Span Kind**: `CLIENT`
- **Duration**: Total request time
- **Status**: `OK` or `ERROR` with exception details

### Request Attributes (GenAI Semantic Conventions)

| Attribute | Description | Example |
|-----------|-------------|---------|
| `gen_ai.system` | Provider name | `"AWS"`, `"bedrock"` |
| `gen_ai.request.model` | Model identifier | `"claude-3-5-sonnet-20241022-v2"` |
| `gen_ai.request.max_tokens` | Max tokens limit | `1024` |
| `gen_ai.request.temperature` | Temperature setting | `0.7` |
| `gen_ai.request.top_p` | Top-p sampling | `0.9` |
| `llm.request.type` | Request type | `"completion"`, `"chat"` |

### Prompt Messages (When `TRACELOOP_TRACE_CONTENT=true`)

**Format varies by provider and API**:

| Attribute | Description | Example |
|-----------|-------------|---------|
| `gen_ai.prompt.{i}.role` | Message role | `"user"`, `"assistant"`, `"system"` |
| `gen_ai.prompt.{i}.content` | Message content | `"Hello!"` or JSON string |
| `gen_ai.prompt.0.user` | User prompt (simple format) | `"Tell me a joke"` |

### Response Attributes

| Attribute | Description | Example |
|-----------|-------------|---------|
| `gen_ai.response.model` | Model used | `"claude-3-5-sonnet-20241022-v2"` |
| `gen_ai.response.id` | Response/request ID | `"msg_..."` |
| `gen_ai.usage.input_tokens` | Input tokens | `150` |
| `gen_ai.usage.output_tokens` | Output tokens | `300` |
| `llm.usage.total_tokens` | Total tokens | `450` |

### Completion Content (When `TRACELOOP_TRACE_CONTENT=true`)

| Attribute | Description | Example |
|-----------|-------------|---------|
| `gen_ai.completion.{i}.content` | Response text | `"Here's the answer..."` |
| `gen_ai.completion.{i}.role` | Response role | `"assistant"` |
| `gen_ai.completion.{i}.finish_reason` | Stop reason | `"end_turn"`, `"max_tokens"` |

### Guardrails Attributes

When guardrails are configured and activated:

| Attribute | Description | Example |
|-----------|-------------|---------|
| `aws.bedrock.guardrail.id` | Guardrail identifier | `"guardrail-abc:1"` |
| `llm.prompts.prompt_filter_results` | Violation details (JSON) | See below |

**Prompt Filter Results** (JSON structure):
```json
{
  "sensitive": {
    "pii": ["EMAIL", "PHONE_NUMBER"],
    "regex": ["Account Number"]
  },
  "topic": ["Dangerous Topics"],
  "content": ["Violence"],
  "words": ["custom_word_match"]
}
```

### Prompt Caching Attributes (Anthropic Models)

| Attribute | Description | Example |
|-----------|-------------|---------|
| `gen_ai.prompt_caching` | Cache operation type | `"read"`, `"write"` |

### Events (When `use_legacy_attributes=False`)

OpenTelemetry log records emitted instead of span attributes:

| Event Type | Description | Fields |
|------------|-------------|--------|
| `gen_ai.user.message` | User messages | `content` |
| `gen_ai.assistant.message` | Assistant messages | `content`, `tool_calls` |
| `gen_ai.choice` | Completion choices | `content`, `role`, `finish_reason`, `tool_calls` |

### Metrics

**Histograms**:
- `gen_ai.model.op.duration` - Operation duration (seconds)
- `gen_ai.model.usage` - Token counts with `gen_ai.token.type` label
- `gen_ai.bedrock.guardrail.latency` - Guardrail processing time (ms)

**Counters**:
- `gen_ai.model.op.choice` - Number of choices generated
- `llm.bedrock.completions.exceptions` - Exception count by type
- `gen_ai.bedrock.guardrail.activation` - Guardrail trigger count
- `gen_ai.bedrock.guardrail.sensitive_info` - PII/regex violations
- `gen_ai.bedrock.guardrail.topics` - Topic policy violations
- `gen_ai.bedrock.guardrail.content` - Content filter hits
- `gen_ai.bedrock.guardrail.words` - Word filter matches
- `gen_ai.bedrock.guardrail.coverage` - Characters protected
- `gen_ai.prompt.caching` - Cache read/write tokens

---

## Advanced Features

### Guardrails

Bedrock Guardrails provide content filtering and safety controls with detailed monitoring:

```python
# Enable guardrails in invoke_model
body = json.dumps({
    "anthropic_version": "bedrock-2023-05-31",
    "max_tokens": 1024,
    "messages": [{
        "role": "user",
        "content": "User input..."
    }]
})

response = bedrock.invoke_model(
    modelId='anthropic.claude-3-5-sonnet-20241022-v2:0',
    body=body,
    guardrailIdentifier='your-guardrail-id',
    guardrailVersion='1'
)

# Check if guardrail intervened
response_body = json.loads(response['body'].read())
if response_body.get('stop_reason') == 'guardrail_intervened':
    print("Guardrail blocked the response")
```

**Converse API**:
```python
response = bedrock.converse(
    modelId='anthropic.claude-3-5-sonnet-20241022-v2:0',
    messages=[{"role": "user", "content": [{"text": "User input..."}]}],
    guardrailConfig={
        'guardrailIdentifier': 'your-guardrail-id',
        'guardrailVersion': '1'
    }
)
```

**What's Captured**:

1. **Guardrail Activation**: Span attribute `aws.bedrock.guardrail.id`
2. **Violation Details**: Structured JSON in `llm.prompts.prompt_filter_results`
3. **Metrics**:
   - Activation counter
   - Processing latency (histogram)
   - Coverage (characters protected)
   - Violation counts by type (sensitive info, topics, content, words)

**Violation Types**:
- **Sensitive Information**: PII (email, phone, SSN) and custom regex patterns
- **Topic Filters**: Blocked conversation topics
- **Content Filters**: Violence, sexual content, hate speech, etc.
- **Word Filters**: Custom and managed word lists

### Prompt Caching (Anthropic Models)

Bedrock automatically caches prompts for Anthropic Claude models:

```python
# First call - creates cache
response1 = bedrock.invoke_model(
    modelId='anthropic.claude-3-5-sonnet-20241022-v2:0',
    body=json.dumps({
        "anthropic_version": "bedrock-2023-05-31",
        "max_tokens": 1024,
        "system": "Very long system prompt...",  # Eligible for caching
        "messages": [{"role": "user", "content": "Query 1"}]
    })
)

# Second call - reads from cache
response2 = bedrock.invoke_model(
    modelId='anthropic.claude-3-5-sonnet-20241022-v2:0',
    body=json.dumps({
        "anthropic_version": "bedrock-2023-05-31",
        "max_tokens": 1024,
        "system": "Very long system prompt...",  # Cache hit!
        "messages": [{"role": "user", "content": "Query 2"}]
    })
)
```

**What's Captured**:
- **Span Attribute**: `gen_ai.prompt_caching` = `"read"` or `"write"`
- **Metrics**: `gen_ai.prompt.caching` counter with `gen_ai.cache.type` label

**Detection**: From HTTP response headers:
- `x-amzn-bedrock-cache-read-input-token-count`
- `x-amzn-bedrock-cache-write-input-token-count`

### Streaming

Both streaming APIs are fully instrumented with event accumulation:

**invoke_model_with_response_stream**:
```python
response = bedrock.invoke_model_with_response_stream(
    modelId='anthropic.claude-3-5-sonnet-20241022-v2:0',
    body=json.dumps({
        "anthropic_version": "bedrock-2023-05-31",
        "max_tokens": 1024,
        "messages": [{"role": "user", "content": "Tell me a story"}]
    })
)

# Stream events
stream = response['body']
for event in stream:
    chunk = event.get('chunk')
    if chunk:
        data = json.loads(chunk['bytes'].decode())
        if data['type'] == 'content_block_delta':
            print(data['delta'].get('text', ''), end='', flush=True)
```

**converse_stream**:
```python
response = bedrock.converse_stream(
    modelId='amazon.nova-pro-v1:0',
    messages=[{
        "role": "user",
        "content": [{"text": "Tell me a story"}]
    }]
)

# Stream events
stream = response['stream']
for event in stream:
    if 'contentBlockDelta' in event:
        delta = event['contentBlockDelta']['delta']
        if 'text' in delta:
            print(delta['text'], end='', flush=True)
```

**What's Tracked**:
- ✅ Complete accumulated response
- ✅ Final token usage
- ✅ Finish reason
- ✅ Guardrail metrics (if applicable)
- ✅ Prompt caching metrics (if applicable)

**Implementation**: Stream wrapper accumulates all events before finalizing the span.

### Tool Use (Converse API)

The Converse API provides built-in tool use support:

```python
tools = [{
    "toolSpec": {
        "name": "get_weather",
        "description": "Get current weather",
        "inputSchema": {
            "json": {
                "type": "object",
                "properties": {
                    "location": {"type": "string"}
                },
                "required": ["location"]
            }
        }
    }
}]

response = bedrock.converse(
    modelId='anthropic.claude-3-5-sonnet-20241022-v2:0',
    messages=[{
        "role": "user",
        "content": [{"text": "What's the weather in SF?"}]
    }],
    toolConfig={"tools": tools}
)

# Check for tool use
for block in response['output']['message']['content']:
    if 'toolUse' in block:
        print(f"Tool: {block['toolUse']['name']}")
        print(f"Input: {block['toolUse']['input']}")
```

**What's Captured**:
- Tool definitions (when content tracing enabled)
- Tool calls with ID, name, and arguments
- Emitted as `gen_ai.choice` event with `tool_calls` array

### Cross-Region Inference

Use inference profiles for cross-region load balancing:

```python
response = bedrock.invoke_model(
    # ARN format for cross-region
    modelId='arn:aws:bedrock:us-east-1:123456789:inference-profile/us.anthropic.claude-3-5-sonnet-20241022-v2:0',
    body=json.dumps({...})
)

# Or use region prefix format
response = bedrock.invoke_model(
    modelId='us.anthropic.claude-3-5-sonnet-20241022-v2:0',
    body=json.dumps({...})
)
```

**Supported Regions**: `us`, `us-gov`, `eu`, `apac`

**Model Detection**: Automatically extracts base model ID from ARN or regional prefix.

---

## Configuration Options

### Instrumentor Parameters

```python
from opentelemetry.instrumentation.bedrock import BedrockInstrumentor

BedrockInstrumentor().instrument(
    # Token enrichment - use Anthropic SDK for token counting
    enrich_token_usage=False,

    # Error handling - custom exception logger
    exception_logger=None,

    # Legacy vs events mode - use span attributes (True) vs events (False)
    use_legacy_attributes=True
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
- ✅ Guardrail metrics still collected
- ❌ Actual prompt/completion text NOT captured
- ❌ Tool call arguments NOT captured

### Token Usage Enrichment

```python
# Enable token enrichment for models without usage data
BedrockInstrumentor().instrument(enrich_token_usage=True)
```

**When useful**: Some older models or APIs may not return token counts. With `enrich_token_usage=True`, the instrumentor uses the Anthropic SDK to estimate tokens for Claude models.

**Requirement**: Anthropic SDK must be installed.

---

## Manual Instrumentation Scenarios

### When Auto-Instrumentation Isn't Enough

1. **Custom Span Attributes**: Add application-specific context

```python
from opentelemetry import trace

tracer = trace.get_tracer(__name__)

with tracer.start_as_current_span("user-query") as span:
    span.set_attribute("user.id", user_id)
    span.set_attribute("query.type", "support")

    response = bedrock.invoke_model(...)  # Auto-instrumented
```

2. **Workflow Tracking**: Use Traceloop decorators for multi-step workflows

```python
from traceloop.sdk.decorators import workflow, task

@task(name="generate_response")
def generate_response(prompt, model_id):
    body = json.dumps({
        "anthropic_version": "bedrock-2023-05-31",
        "max_tokens": 1024,
        "messages": [{"role": "user", "content": prompt}]
    })
    response = bedrock.invoke_model(modelId=model_id, body=body)
    return json.loads(response['body'].read())

@workflow(name="multi_model_comparison")
def compare_models(prompt):
    claude = generate_response(prompt, "anthropic.claude-3-5-sonnet-20241022-v2:0")
    llama = generate_response(prompt, "meta.llama3-70b-instruct-v1:0")
    return {"claude": claude, "llama": llama}
```

3. **Tool Execution Tracking**: Trace tool execution separately

```python
from traceloop.sdk.decorators import task

@task(name="execute_tool")
def execute_tool(tool_name, tool_input):
    if tool_name == "get_weather":
        return get_weather_data(tool_input["location"])
    # ...

# Use in conversation loop
while True:
    response = bedrock.converse(...)
    if response['stopReason'] != 'tool_use':
        break

    for block in response['output']['message']['content']:
        if 'toolUse' in block:
            result = execute_tool(block['toolUse']['name'], block['toolUse']['input'])
            # Add result to messages and continue
```

---

## Sample Applications

### Main Sample

**File**: `packages/sample-app/sample_app/bedrock_example_app.py`

**Features**:
- Cohere Command Text model
- invoke_model API
- Traceloop workflow and task decorators
- Basic instrumentation setup

**Code Snippet**:
```python
from traceloop.sdk import Traceloop
from traceloop.sdk.decorators import task, workflow
import boto3
import json

Traceloop.init(app_name="joke_generation_service")
bedrock = boto3.client('bedrock-runtime', region_name='us-east-1')

@task(name="joke_creation")
def create_joke():
    body = json.dumps({
        "prompt": "Tell me a joke about opentelemetry",
        "max_tokens": 200,
        "temperature": 0.5,
        "p": 0.5,
    })
    response = bedrock.invoke_model(
        body=body,
        modelId='cohere.command-text-v14',
        accept='application/json',
        contentType='application/json'
    )
    response_body = json.loads(response['body'].read())
    return response_body['generations'][0]['text']

@workflow(name="pirate_joke_generator")
def joke_workflow():
    print(create_joke())

joke_workflow()
```

### Running the Sample

```bash
# Install dependencies
cd packages/sample-app
poetry install

# Configure AWS credentials
export AWS_ACCESS_KEY_ID=your-key
export AWS_SECRET_ACCESS_KEY=your-secret
export AWS_REGION=us-east-1

# Run sample
poetry run python sample_app/bedrock_example_app.py
```

---

## Common Patterns

### 1. Multi-Model Comparison

```python
from traceloop.sdk.decorators import workflow, task

@task(name="invoke_model")
def invoke_model(model_id, prompt):
    # Unified function for any Bedrock model
    response = bedrock.converse(
        modelId=model_id,
        messages=[{"role": "user", "content": [{"text": prompt}]}]
    )
    return response['output']['message']['content'][0]['text']

@workflow(name="model_comparison")
def compare_models(prompt):
    results = {}
    models = [
        "anthropic.claude-3-5-sonnet-20241022-v2:0",
        "meta.llama3-70b-instruct-v1:0",
        "amazon.nova-pro-v1:0"
    ]

    for model in models:
        results[model] = invoke_model(model, prompt)

    return results
```

### 2. Guardrails with Fallback

```python
def safe_invoke_with_fallback(prompt, guardrail_id):
    """Try with guardrails, fall back to unguarded on intervention"""
    try:
        response = bedrock.converse(
            modelId='anthropic.claude-3-5-sonnet-20241022-v2:0',
            messages=[{"role": "user", "content": [{"text": prompt}]}],
            guardrailConfig={
                'guardrailIdentifier': guardrail_id,
                'guardrailVersion': '1'
            }
        )

        if response.get('stopReason') == 'guardrail_intervened':
            # Log intervention and return safe response
            print("Guardrail intervened")
            return "I cannot respond to that request."

        return response['output']['message']['content'][0]['text']

    except Exception as e:
        print(f"Guardrail error: {e}")
        # Fallback without guardrail
        response = bedrock.converse(
            modelId='anthropic.claude-3-5-sonnet-20241022-v2:0',
            messages=[{"role": "user", "content": [{"text": prompt}]}]
        )
        return response['output']['message']['content'][0]['text']
```

### 3. Streaming with Progress Tracking

```python
def stream_with_progress(prompt):
    """Stream response with progress indicators"""
    response = bedrock.converse_stream(
        modelId='anthropic.claude-3-5-sonnet-20241022-v2:0',
        messages=[{"role": "user", "content": [{"text": prompt}]}]
    )

    complete_text = ""
    token_count = 0

    for event in response['stream']:
        if 'contentBlockDelta' in event:
            delta = event['contentBlockDelta']['delta']
            if 'text' in delta:
                text = delta['text']
                complete_text += text
                token_count += len(text.split())
                print(text, end='', flush=True)

                # Progress indicator every 50 tokens
                if token_count % 50 == 0:
                    print(f"\n[{token_count} tokens]", flush=True)

    return complete_text
```

### 4. Tool Use Loop with Converse API

```python
def conversation_with_tools(user_input, tools):
    """Multi-turn conversation with tool execution"""
    messages = [{
        "role": "user",
        "content": [{"text": user_input}]
    }]

    while True:
        response = bedrock.converse(
            modelId='anthropic.claude-3-5-sonnet-20241022-v2:0',
            messages=messages,
            toolConfig={"tools": tools}
        )

        message = response['output']['message']
        messages.append(message)

        if response['stopReason'] != 'tool_use':
            # Final response
            return message['content'][0]['text']

        # Execute tools
        tool_results = []
        for block in message['content']:
            if 'toolUse' in block:
                tool_use = block['toolUse']
                result = execute_tool(tool_use['name'], tool_use['input'])
                tool_results.append({
                    "toolResult": {
                        "toolUseId": tool_use['toolUseId'],
                        "content": [{"text": result}]
                    }
                })

        # Add tool results to conversation
        messages.append({
            "role": "user",
            "content": tool_results
        })
```

---

## Troubleshooting

### Issue: No spans appearing

**Symptoms**: Bedrock calls work but no traces visible

**Solutions**:
1. Verify Bedrock client is created after instrumentation:
   ```python
   from traceloop.sdk import Traceloop
   Traceloop.init()  # Must be called first
   bedrock = boto3.client('bedrock-runtime')  # Then create client
   ```

2. Check service name:
   ```python
   # Correct
   bedrock = boto3.client('bedrock-runtime')

   # Wrong - won't be instrumented
   bedrock = boto3.client('bedrock')  # This is for model management, not runtime
   ```

3. Verify exporter:
   ```python
   from opentelemetry.sdk.trace.export import ConsoleSpanExporter
   Traceloop.init(exporter=ConsoleSpanExporter())
   ```

### Issue: Guardrail metrics not appearing

**Symptoms**: Guardrails work but no metrics visible

**Solutions**:
1. Verify guardrail is actually activated:
   ```python
   response = bedrock.invoke_model(...)
   body = json.loads(response['body'].read())
   print(f"Stop reason: {body.get('stop_reason')}")
   # Should be 'guardrail_intervened' when activated
   ```

2. Check metrics are enabled:
   ```bash
   export TRACELOOP_METRICS_ENABLED=true
   ```

3. Ensure meter provider is configured:
   ```python
   from opentelemetry.sdk.metrics import MeterProvider
   from opentelemetry import metrics
   metrics.set_meter_provider(MeterProvider())
   ```

### Issue: Token counts missing

**Symptoms**: Spans exist but token usage attributes are zero or missing

**Solutions**:
1. Check model support - not all models return token counts:
   ```python
   # Models with reliable token counts:
   # - Anthropic Claude (all versions)
   # - Cohere Command
   # - Meta Llama
   # - Amazon Nova

   # Models that may lack counts:
   # - Some older Titan models
   ```

2. Enable token enrichment for Anthropic models:
   ```python
   BedrockInstrumentor().instrument(enrich_token_usage=True)
   ```

3. Verify response structure:
   ```python
   response = bedrock.invoke_model(...)
   body = json.loads(response['body'].read())
   print(f"Usage: {body.get('usage')}")  # Should contain input_tokens/output_tokens
   ```

### Issue: Streaming spans incomplete

**Symptoms**: Streaming responses show partial data

**Cause**: Stream must be fully consumed for span to complete.

**Solution**:
```python
# Bad - stream not consumed
response = bedrock.invoke_model_with_response_stream(...)
stream = response['body']
# Span still open!

# Good - consume stream
response = bedrock.invoke_model_with_response_stream(...)
stream = response['body']
for event in stream:
    process(event)
# Span now complete
```

### Issue: Prompt caching not detected

**Symptoms**: Caching enabled but no metrics

**Cause**: Prompt caching only works with Anthropic Claude models on Bedrock.

**Solution**:
1. Verify model supports caching:
   ```python
   # Supported
   modelId='anthropic.claude-3-5-sonnet-20241022-v2:0'
   modelId='anthropic.claude-3-opus-20240229-v1:0'
   modelId='anthropic.claude-3-haiku-20240307-v1:0'

   # Not supported
   modelId='cohere.command-r-v1:0'  # No caching
   modelId='meta.llama3-70b-instruct-v1:0'  # No caching
   ```

2. Check response headers:
   ```python
   response = bedrock.invoke_model(...)
   print(response['ResponseMetadata']['HTTPHeaders'])
   # Look for 'x-amzn-bedrock-cache-read-input-token-count'
   ```

### Issue: Cross-region models not working

**Symptoms**: ARN model IDs fail or show incorrect model names

**Solution**:
1. Verify ARN format:
   ```python
   # Correct
   modelId='arn:aws:bedrock:us-east-1:123456789:inference-profile/us.anthropic.claude-3-5-sonnet-20241022-v2:0'

   # Incorrect
   modelId='arn:aws:bedrock:us-east-1:123456789:model/anthropic.claude-3-5-sonnet-20241022-v2:0'
   ```

2. Use simple regional prefix instead:
   ```python
   modelId='us.anthropic.claude-3-5-sonnet-20241022-v2:0'
   ```

### Issue: Converse API tool calls not captured

**Symptoms**: Tool use works but not visible in traces

**Solution**: Ensure content tracing is enabled:
```bash
export TRACELOOP_TRACE_CONTENT=true
```

Tool calls are only captured when content tracing is enabled for privacy reasons.

---

## Additional Resources

- **AWS Bedrock Documentation**: https://docs.aws.amazon.com/bedrock/
- **Bedrock Runtime API Reference**: https://docs.aws.amazon.com/bedrock/latest/APIReference/API_Operations_Amazon_Bedrock_Runtime.html
- **Bedrock Guardrails**: https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails.html
- **Bedrock Prompt Caching**: https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-caching.html
- **Converse API Guide**: https://docs.aws.amazon.com/bedrock/latest/userguide/conversation-inference.html
- **OpenTelemetry GenAI Semantic Conventions**: https://opentelemetry.io/docs/specs/semconv/gen-ai/
- **Traceloop SDK Guide**: [../../traceloop-sdk-guide.md](../../traceloop-sdk-guide.md)
- **Sample Applications**: [../../examples-index.md](../../examples-index.md)

---

**Last Updated**: 2025-11-22 (Session 2)
