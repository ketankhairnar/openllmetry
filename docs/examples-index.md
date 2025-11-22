# Examples Index - Sample Applications Catalog

> **Status**: ✅ Complete (Continuously Improving)
> **Last Updated**: 2025-11-22
> **Total Samples**: 64 Python files
> **Location**: `packages/sample-app/sample_app/`

---

## Table of Contents

1. [Quick Reference](#quick-reference)
2. [LLM Provider Examples](#llm-provider-examples)
3. [Framework & Orchestration Examples](#framework--orchestration-examples)
4. [Vector Database & RAG Examples](#vector-database--rag-examples)
5. [Traceloop SDK Feature Examples](#traceloop-sdk-feature-examples)
6. [Model Context Protocol Examples](#model-context-protocol-examples)
7. [By Use Case](#by-use-case)
8. [By Complexity](#by-complexity)

---

## Quick Reference

### Run Any Sample

```bash
cd packages/sample-app

# Install dependencies
poetry install

# Set API keys (as needed)
export OPENAI_API_KEY=your-key
export ANTHROPIC_API_KEY=your-key
# ... etc

# Run a sample
poetry run python sample_app/openai_streaming.py
```

### VCR Cassettes

Samples use VCR cassettes for API calls. To re-record:

```bash
# Re-record all cassettes (requires API keys)
poetry run pytest tests/ --record-mode=all

# Re-record specific test
poetry run pytest tests/test_specific.py --record-mode=once
```

---

## LLM Provider Examples

### OpenAI (10 samples)

| Sample File | Demonstrates | Complexity | Key Features |
|-------------|--------------|------------|--------------|
| [`openai_streaming.py`](../packages/sample-app/sample_app/openai_streaming.py) | Basic streaming completions | ⭐ Simple | `@workflow` decorator, streaming |
| [`openai_functions.py`](../packages/sample-app/sample_app/openai_functions.py) | Function calling | ⭐⭐ Medium | Tool definitions, function calls |
| [`openai_structured_outputs.py`](../packages/sample-app/sample_app/openai_structured_outputs.py) | Structured output parsing | ⭐⭐ Medium | Pydantic models, schema validation |
| [`openai_assistant.py`](../packages/sample-app/sample_app/openai_assistant.py) | OpenAI Assistants API | ⭐⭐ Medium | Assistants, threads, runs |
| [`openai_streaming_assistant.py`](../packages/sample-app/sample_app/openai_streaming_assistant.py) | Streaming with Assistants | ⭐⭐ Medium | Assistants streaming |
| [`openai_vision_base64_example.py`](../packages/sample-app/sample_app/openai_vision_base64_example.py) | Vision model with base64 | ⭐⭐ Medium | Vision, base64 images |
| [`openai_agents_example.py`](../packages/sample-app/sample_app/openai_agents_example.py) | Full OpenAI Agents SDK | ⭐⭐⭐ Complex | Agents, handoffs, tools (666 lines) |
| [`openai_agents_using_litellm.py`](../packages/sample-app/sample_app/openai_agents_using_litellm.py) | Agents with LiteLLM proxy | ⭐⭐ Medium | LiteLLM integration |
| [`azure_openai.py`](../packages/sample-app/sample_app/azure_openai.py) | Azure OpenAI integration | ⭐ Simple | Azure endpoints, authentication |
| [`litellm_example.py`](../packages/sample-app/sample_app/litellm_example.py) | LiteLLM proxy | ⭐ Simple | Multi-provider proxy |

**Code Snippet** (openai_streaming.py):
```python
from traceloop.sdk import Traceloop
from traceloop.sdk.decorators import workflow
from openai import OpenAI

Traceloop.init(app_name="joke_generation_streaming")

@workflow(name="joke_creation")
def create_joke():
    client = OpenAI()
    response = client.chat.completions.create(
        model="gpt-4",
        messages=[{"role": "user", "content": "Tell me a joke about Python"}],
        stream=True
    )

    for chunk in response:
        if chunk.choices[0].delta.content:
            print(chunk.choices[0].delta.content, end="")
    print()
```

---

### Anthropic (5 samples)

| Sample File | Demonstrates | Complexity | Key Features |
|-------------|--------------|------------|--------------|
| [`anthropic_joke_example.py`](../packages/sample-app/sample_app/anthropic_joke_example.py) | Basic Claude messages | ⭐ Simple | Messages API, `@workflow` |
| [`anthropic_joke_streaming_example.py`](../packages/sample-app/sample_app/anthropic_joke_streaming_example.py) | Streaming responses | ⭐ Simple | Streaming |
| [`async_anthropic_example.py`](../packages/sample-app/sample_app/async_anthropic_example.py) | Async Claude calls | ⭐⭐ Medium | async/await |
| [`async_anthropic_joke_streaming.py`](../packages/sample-app/sample_app/async_anthropic_joke_streaming.py) | Async streaming | ⭐⭐ Medium | Async streaming |
| [`anthropic_vision_base64_example.py`](../packages/sample-app/sample_app/anthropic_vision_base64_example.py) | Vision with base64 | ⭐⭐ Medium | Vision, base64 images |

**Code Snippet** (anthropic_joke_example.py):
```python
from traceloop.sdk import Traceloop
from traceloop.sdk.decorators import workflow
from anthropic import Anthropic

Traceloop.init(app_name="joke_generation")

@workflow(name="joke_creation")
def create_joke():
    client = Anthropic()
    message = client.messages.create(
        model="claude-3-5-sonnet-20241022",
        max_tokens=1024,
        messages=[{"role": "user", "content": "Tell me a joke about Python"}]
    )
    return message.content[0].text
```

---

### Other LLM Providers (14 samples)

| Provider | Sample Files | Key Features |
|----------|--------------|--------------|
| **Cohere** | [`cohere_example.py`](../packages/sample-app/sample_app/cohere_example.py) | Chat and rerank APIs |
| **Groq** | [`groq_example.py`](../packages/sample-app/sample_app/groq_example.py) | Fast inference |
| **Ollama** | [`ollama_streaming.py`](../packages/sample-app/sample_app/ollama_streaming.py) | Local models, streaming |
| **Replicate** | [`replicate_functions.py`](../packages/sample-app/sample_app/replicate_functions.py), [`replicate_streaming.py`](../packages/sample-app/sample_app/replicate_streaming.py) | Replicate API, streaming |
| **Google Gemini** | [`gemini.py`](../packages/sample-app/sample_app/gemini.py) | Gemini models |
| **Google GenAI** | [`google_genai_image_example.py`](../packages/sample-app/sample_app/google_genai_image_example.py) | Images with GenAI |
| **Vertex AI** | [`vertex_gemini_vision_example.py`](../packages/sample-app/sample_app/vertex_gemini_vision_example.py), [`vertexai_streaming.py`](../packages/sample-app/sample_app/vertexai_streaming.py) | Vertex Gemini, streaming |
| **AWS Bedrock** | [`bedrock_example_app.py`](../packages/sample-app/sample_app/bedrock_example_app.py) | Bedrock with Cohere |
| **IBM Watsonx** | [`watsonx_generate.py`](../packages/sample-app/sample_app/watsonx_generate.py), [`watsonx_flow.py`](../packages/sample-app/sample_app/watsonx_flow.py), [`langchain_watsonx.py`](../packages/sample-app/sample_app/langchain_watsonx.py) | Watsonx generation, flows |
| **Writer** | [`writer_example.py`](../packages/sample-app/sample_app/writer_example.py) | Writer.ai integration |

---

## Framework & Orchestration Examples

### LangChain (7 samples)

| Sample File | Demonstrates | Complexity | Key Features |
|-------------|--------------|------------|--------------|
| [`langchain_app.py`](../packages/sample-app/sample_app/langchain_app.py) | Basic chain with prompts | ⭐ Simple | Chain, PromptTemplate |
| [`langchain_lcel.py`](../packages/sample-app/sample_app/langchain_lcel.py) | LCEL pipelines | ⭐⭐ Medium | Pipe operator, LCEL |
| [`langchain_agent.py`](../packages/sample-app/sample_app/langchain_agent.py) | Agents with tools | ⭐⭐⭐ Complex | AgentExecutor, tools |
| [`langchain_watsonx.py`](../packages/sample-app/sample_app/langchain_watsonx.py) | LangChain + Watsonx | ⭐⭐ Medium | Watsonx integration |
| [`langgraph_example.py`](../packages/sample-app/sample_app/langgraph_example.py) | LangGraph workflows | ⭐⭐⭐ Complex | State machines, tools |
| [`langgraph_openai.py`](../packages/sample-app/sample_app/langgraph_openai.py) | LangGraph + OpenAI | ⭐⭐⭐ Complex | LangGraph with OpenAI |

**Code Snippet** (langchain_lcel.py):
```python
from traceloop.sdk import Traceloop
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

Traceloop.init(app_name="langchain_lcel_app")

# LCEL chain with pipe operator
prompt = ChatPromptTemplate.from_template("Tell me a joke about {topic}")
model = ChatOpenAI(model="gpt-4")
output_parser = StrOutputParser()

# Chain components with | operator
chain = prompt | model | output_parser

# Invoke chain
result = chain.invoke({"topic": "Python"})
# ✅ Full chain traced with proper span hierarchy
```

---

### LlamaIndex (4 samples)

| Sample File | Demonstrates | Complexity | Key Features |
|-------------|--------------|------------|--------------|
| [`llama_index_workflow_app.py`](../packages/sample-app/sample_app/llama_index_workflow_app.py) | LlamaIndex workflows | ⭐⭐ Medium | Workflows, async steps |
| [`llama_index_chroma_app.py`](../packages/sample-app/sample_app/llama_index_chroma_app.py) | LlamaIndex + Chroma | ⭐⭐ Medium | Vector store integration |
| [`llama_index_chroma_huggingface_app.py`](../packages/sample-app/sample_app/llama_index_chroma_huggingface_app.py) | LlamaIndex + Chroma + HF | ⭐⭐⭐ Complex | HuggingFace embeddings |
| [`llama_parse_app.py`](../packages/sample-app/sample_app/llama_parse_app.py) | Document parsing | ⭐⭐ Medium | LlamaParse |

---

### Other Frameworks (3 samples)

| Sample File | Demonstrates | Complexity | Key Features |
|-------------|--------------|------------|--------------|
| [`crewai_example.py`](../packages/sample-app/sample_app/crewai_example.py) | CrewAI multi-agent | ⭐⭐⭐ Complex | Multi-agent workflows |
| [`haystack_app.py`](../packages/sample-app/sample_app/haystack_app.py) | Haystack RAG | ⭐⭐ Medium | RAG pipelines, embeddings |
| [`writer_example.py`](../packages/sample-app/sample_app/writer_example.py) | Writer.ai | ⭐ Simple | Writer integration |

---

## Vector Database & RAG Examples

### Pinecone (3 samples)

| Sample File | Demonstrates | Complexity | Key Features |
|-------------|--------------|------------|--------------|
| [`pinecone_app.py`](../packages/sample-app/sample_app/pinecone_app.py) | Pinecone RAG | ⭐⭐ Medium | Query, upsert, RAG pipeline |
| [`pinecone_app_sentence_transformers.py`](../packages/sample-app/sample_app/pinecone_app_sentence_transformers.py) | Pinecone + Sentence Transformers | ⭐⭐ Medium | SentenceTransformers embeddings |
| [`thread_pool_example.py`](../packages/sample-app/sample_app/thread_pool_example.py) | ThreadPoolExecutor | ⭐⭐⭐ Complex | Context preservation in threads |

**Code Snippet** (pinecone_app.py):
```python
from traceloop.sdk import Traceloop
from traceloop.sdk.decorators import workflow, task
from pinecone import Pinecone
from openai import OpenAI

Traceloop.init(app_name="pinecone_rag")

@workflow(name="semantic_search")
def search_documents(query: str):
    # Generate embedding (auto-instrumented)
    embedding = generate_embedding(query)

    # Query Pinecone (auto-instrumented)
    results = query_pinecone(embedding)

    return results

@task(name="generate_embedding")
def generate_embedding(text: str):
    client = OpenAI()
    response = client.embeddings.create(
        model="text-embedding-3-small",
        input=text
    )
    return response.data[0].embedding

@task(name="query_pinecone")
def query_pinecone(embedding):
    pc = Pinecone()
    index = pc.Index("my-index")
    return index.query(vector=embedding, top_k=5)
```

---

### Chroma (2 samples)

| Sample File | Demonstrates | Complexity | Key Features |
|-------------|--------------|------------|--------------|
| [`chroma_app.py`](../packages/sample-app/sample_app/chroma_app.py) | Scientific fact-checking | ⭐⭐ Medium | Chroma, OpenAI embeddings |
| [`chroma_sentence_transformer_app.py`](../packages/sample-app/sample_app/chroma_sentence_transformer_app.py) | Chroma + Sentence Transformers | ⭐⭐ Medium | SentenceTransformers |

---

### Weaviate (2 samples)

| Sample File | Demonstrates | Complexity | Key Features |
|-------------|--------------|------------|--------------|
| [`weaviate_v3.py`](../packages/sample-app/sample_app/weaviate_v3.py) | Weaviate v3 API | ⭐⭐ Medium | Weaviate v3 |
| [`weaviate_v4.py`](../packages/sample-app/sample_app/weaviate_v4.py) | Weaviate v4 API | ⭐⭐ Medium | Weaviate v4 |

---

### Redis (1 sample)

| Sample File | Demonstrates | Complexity | Key Features |
|-------------|--------------|------------|--------------|
| [`redis_rag_app.py`](../packages/sample-app/sample_app/redis_rag_app.py) | Redis vector search | ⭐⭐ Medium | Redis, RAG |

---

## Traceloop SDK Feature Examples

### Decorators (3 samples)

| Sample File | Demonstrates | Complexity | Key Features |
|-------------|--------------|------------|--------------|
| [`methods_decorated_app.py`](../packages/sample-app/sample_app/methods_decorated_app.py) | All decorators | ⭐⭐ Medium | `@workflow`, `@task`, `@agent`, `@tool` |
| [`classes_decorated_app.py`](../packages/sample-app/sample_app/classes_decorated_app.py) | Class-based agents | ⭐⭐ Medium | Class methods, agents |
| [`async_methods_decorated_app.py`](../packages/sample-app/sample_app/async_methods_decorated_app.py) | Async class agents | ⭐⭐⭐ Complex | Async class methods |

**Code Snippet** (methods_decorated_app.py):
```python
from traceloop.sdk import Traceloop
from traceloop.sdk.decorators import workflow, task, agent, tool
from openai import OpenAI

Traceloop.init(app_name="decorated_app")

@workflow(name="create_joke")
def create_joke_workflow():
    return create_joke()

@task(name="joke_creation", version=1)
def create_joke():
    return llm_call("Tell me a joke")

@agent(name="llm_agent")
def llm_call(prompt: str):
    return execute_openai_call(prompt)

@tool(name="openai_call")
def execute_openai_call(prompt: str):
    client = OpenAI()
    response = client.chat.completions.create(
        model="gpt-4",
        messages=[{"role": "user", "content": prompt}]
    )
    return response.choices[0].message.content

# Span hierarchy:
# create_joke.workflow
# └── joke_creation.task
#     └── llm_agent.agent
#         └── openai_call.tool
#             └── openai.chat (auto-instrumented)
```

---

### Advanced Features (5 samples)

| Sample File | Demonstrates | Complexity | Key Features |
|-------------|--------------|------------|--------------|
| [`dataset_example.py`](../packages/sample-app/sample_app/dataset_example.py) | Datasets & experiments | ⭐⭐⭐ Complex | Dataset creation, experiments (512 lines) |
| [`experiment/experiment_example.py`](../packages/sample-app/sample_app/experiment/experiment_example.py) | Experiment runner | ⭐⭐⭐ Complex | Evaluators, datasets |
| [`experiment/medical_prompts.py`](../packages/sample-app/sample_app/experiment/medical_prompts.py) | Prompt templates | ⭐⭐ Medium | Medical prompts |
| [`prompt_registry_example_app.py`](../packages/sample-app/sample_app/prompt_registry_example_app.py) | Prompt registry | ⭐⭐ Medium | Versioned prompts, variables |
| [`prompt_registry_vision.py`](../packages/sample-app/sample_app/prompt_registry_vision.py) | Prompt registry (vision) | ⭐⭐ Medium | Vision prompts |
| [`multiple_span_processors.py`](../packages/sample-app/sample_app/multiple_span_processors.py) | Multiple processors | ⭐⭐ Medium | Console + Traceloop exporters |
| [`manual_logging_example.py`](../packages/sample-app/sample_app/manual_logging_example.py) | Manual span reporting | ⭐⭐⭐ Complex | `track_llm_call()` context manager |

---

### Agent Handoffs (2 samples)

| Sample File | Demonstrates | Complexity | Key Features |
|-------------|--------------|------------|--------------|
| [`sample_handoff_app.py`](../packages/sample-app/sample_app/sample_handoff_app.py) | OpenAI Agents handoffs | ⭐⭐⭐ Complex | Handoff hierarchy (223 lines) |
| [`simple_handoff_demo.py`](../packages/sample-app/sample_app/simple_handoff_demo.py) | Simplified handoff | ⭐⭐ Medium | Basic handoff pattern |

---

## Model Context Protocol Examples

| Sample File | Demonstrates | Complexity | Key Features |
|-------------|--------------|------------|--------------|
| [`mcp_sonnet_example.py`](../packages/sample-app/sample_app/mcp_sonnet_example.py) | MCP with Claude | ⭐⭐⭐ Complex | MCP client, tool calling |
| [`mcp_dev_assistant_demo.py`](../packages/sample-app/sample_app/mcp_dev_assistant_demo.py) | Dev assistant demo | ⭐⭐⭐ Complex | MCP demo |
| [`mcp_dev_assistant_server.py`](../packages/sample-app/sample_app/mcp_dev_assistant_server.py) | MCP server | ⭐⭐⭐ Complex | Server implementation |

---

## By Use Case

### 🎯 RAG (Retrieval-Augmented Generation)

| Sample | Vector DB | Embeddings | LLM |
|--------|-----------|------------|-----|
| [`pinecone_app.py`](../packages/sample-app/sample_app/pinecone_app.py) | Pinecone | OpenAI | OpenAI |
| [`chroma_app.py`](../packages/sample-app/sample_app/chroma_app.py) | Chroma | OpenAI | OpenAI |
| [`redis_rag_app.py`](../packages/sample-app/sample_app/redis_rag_app.py) | Redis | OpenAI | OpenAI |
| [`llama_index_chroma_app.py`](../packages/sample-app/sample_app/llama_index_chroma_app.py) | Chroma | LlamaIndex | LlamaIndex |
| [`haystack_app.py`](../packages/sample-app/sample_app/haystack_app.py) | In-memory | Haystack | Haystack |

---

### 🤖 Multi-Agent Systems

| Sample | Framework | Complexity |
|--------|-----------|------------|
| [`crewai_example.py`](../packages/sample-app/sample_app/crewai_example.py) | CrewAI | ⭐⭐⭐ |
| [`openai_agents_example.py`](../packages/sample-app/sample_app/openai_agents_example.py) | OpenAI Agents | ⭐⭐⭐ |
| [`langgraph_example.py`](../packages/sample-app/sample_app/langgraph_example.py) | LangGraph | ⭐⭐⭐ |
| [`langchain_agent.py`](../packages/sample-app/sample_app/langchain_agent.py) | LangChain | ⭐⭐⭐ |

---

### 🔧 Tool Use & Function Calling

| Sample | Provider | Complexity |
|--------|----------|------------|
| [`openai_functions.py`](../packages/sample-app/sample_app/openai_functions.py) | OpenAI | ⭐⭐ |
| [`langchain_agent.py`](../packages/sample-app/sample_app/langchain_agent.py) | LangChain | ⭐⭐⭐ |
| [`langgraph_example.py`](../packages/sample-app/sample_app/langgraph_example.py) | LangGraph | ⭐⭐⭐ |
| [`mcp_sonnet_example.py`](../packages/sample-app/sample_app/mcp_sonnet_example.py) | MCP + Anthropic | ⭐⭐⭐ |

---

### 🖼️ Vision & Multimodal

| Sample | Provider | Input Type |
|--------|----------|------------|
| [`openai_vision_base64_example.py`](../packages/sample-app/sample_app/openai_vision_base64_example.py) | OpenAI | Base64 images |
| [`anthropic_vision_base64_example.py`](../packages/sample-app/sample_app/anthropic_vision_base64_example.py) | Anthropic | Base64 images |
| [`google_genai_image_example.py`](../packages/sample-app/sample_app/google_genai_image_example.py) | Google GenAI | Images |
| [`vertex_gemini_vision_example.py`](../packages/sample-app/sample_app/vertex_gemini_vision_example.py) | Vertex AI | Images |

---

### 📊 Structured Outputs

| Sample | Provider | Schema Type |
|--------|----------|-------------|
| [`openai_structured_outputs.py`](../packages/sample-app/sample_app/openai_structured_outputs.py) | OpenAI | Pydantic models |

---

### 🌊 Streaming

| Sample | Provider | Type |
|--------|----------|------|
| [`openai_streaming.py`](../packages/sample-app/sample_app/openai_streaming.py) | OpenAI | Sync streaming |
| [`anthropic_joke_streaming_example.py`](../packages/sample-app/sample_app/anthropic_joke_streaming_example.py) | Anthropic | Sync streaming |
| [`async_anthropic_joke_streaming.py`](../packages/sample-app/sample_app/async_anthropic_joke_streaming.py) | Anthropic | Async streaming |
| [`ollama_streaming.py`](../packages/sample-app/sample_app/ollama_streaming.py) | Ollama | Sync streaming |
| [`replicate_streaming.py`](../packages/sample-app/sample_app/replicate_streaming.py) | Replicate | Sync streaming |
| [`vertexai_streaming.py`](../packages/sample-app/sample_app/vertexai_streaming.py) | Vertex AI | Sync streaming |

---

### ⚡ Async/Await

| Sample | Provider/Framework | Features |
|--------|--------------------|----------|
| [`async_anthropic_example.py`](../packages/sample-app/sample_app/async_anthropic_example.py) | Anthropic | Async calls |
| [`async_anthropic_joke_streaming.py`](../packages/sample-app/sample_app/async_anthropic_joke_streaming.py) | Anthropic | Async streaming |
| [`async_methods_decorated_app.py`](../packages/sample-app/sample_app/async_methods_decorated_app.py) | SDK | Async class methods |
| [`llama_index_workflow_app.py`](../packages/sample-app/sample_app/llama_index_workflow_app.py) | LlamaIndex | Async workflows |

---

## By Complexity

### ⭐ Simple (Getting Started)

Perfect for learning the basics:

- [`openai_streaming.py`](../packages/sample-app/sample_app/openai_streaming.py) - OpenAI streaming
- [`anthropic_joke_example.py`](../packages/sample-app/sample_app/anthropic_joke_example.py) - Anthropic basic
- [`azure_openai.py`](../packages/sample-app/sample_app/azure_openai.py) - Azure OpenAI
- [`langchain_app.py`](../packages/sample-app/sample_app/langchain_app.py) - LangChain chain
- [`groq_example.py`](../packages/sample-app/sample_app/groq_example.py) - Groq inference

### ⭐⭐ Medium (Common Patterns)

Real-world use cases:

- [`openai_functions.py`](../packages/sample-app/sample_app/openai_functions.py) - Function calling
- [`openai_structured_outputs.py`](../packages/sample-app/sample_app/openai_structured_outputs.py) - Structured outputs
- [`pinecone_app.py`](../packages/sample-app/sample_app/pinecone_app.py) - RAG with Pinecone
- [`chroma_app.py`](../packages/sample-app/sample_app/chroma_app.py) - RAG with Chroma
- [`langchain_lcel.py`](../packages/sample-app/sample_app/langchain_lcel.py) - LCEL pipelines
- [`methods_decorated_app.py`](../packages/sample-app/sample_app/methods_decorated_app.py) - SDK decorators
- [`prompt_registry_example_app.py`](../packages/sample-app/sample_app/prompt_registry_example_app.py) - Prompt management

### ⭐⭐⭐ Complex (Advanced Patterns)

Production-grade examples:

- [`openai_agents_example.py`](../packages/sample-app/sample_app/openai_agents_example.py) - Full agents system (666 lines)
- [`langchain_agent.py`](../packages/sample-app/sample_app/langchain_agent.py) - LangChain agents
- [`langgraph_example.py`](../packages/sample-app/sample_app/langgraph_example.py) - LangGraph workflows
- [`crewai_example.py`](../packages/sample-app/sample_app/crewai_example.py) - Multi-agent CrewAI
- [`dataset_example.py`](../packages/sample-app/sample_app/dataset_example.py) - Datasets & experiments (512 lines)
- [`thread_pool_example.py`](../packages/sample-app/sample_app/thread_pool_example.py) - Thread pool context
- [`mcp_sonnet_example.py`](../packages/sample-app/sample_app/mcp_sonnet_example.py) - MCP with Claude

---

## Quick Start Examples by Goal

### "I want to get started with OpenLLMetry"
→ [`openai_streaming.py`](../packages/sample-app/sample_app/openai_streaming.py)

### "I need RAG with vector databases"
→ [`pinecone_app.py`](../packages/sample-app/sample_app/pinecone_app.py) or [`chroma_app.py`](../packages/sample-app/sample_app/chroma_app.py)

### "I'm building agents with LangChain"
→ [`langchain_agent.py`](../packages/sample-app/sample_app/langchain_agent.py)

### "I need multi-step workflows"
→ [`langgraph_example.py`](../packages/sample-app/sample_app/langgraph_example.py)

### "I want to use SDK decorators"
→ [`methods_decorated_app.py`](../packages/sample-app/sample_app/methods_decorated_app.py)

### "I'm using Anthropic Claude"
→ [`anthropic_joke_example.py`](../packages/sample-app/sample_app/anthropic_joke_example.py)

### "I need function calling"
→ [`openai_functions.py`](../packages/sample-app/sample_app/openai_functions.py)

### "I'm building multi-agent systems"
→ [`crewai_example.py`](../packages/sample-app/sample_app/crewai_example.py)

---

## Running Samples

### Setup

```bash
cd packages/sample-app
poetry install
```

### Set API Keys

```bash
# Required for most samples
export OPENAI_API_KEY=your-key

# Required for specific providers
export ANTHROPIC_API_KEY=your-key
export COHERE_API_KEY=your-key
export PINECONE_API_KEY=your-key
# ... etc
```

### Run

```bash
# Basic example
poetry run python sample_app/openai_streaming.py

# RAG example
poetry run python sample_app/pinecone_app.py

# Agent example
poetry run python sample_app/langchain_agent.py
```

### Testing with VCR

```bash
# Run tests (uses recorded cassettes)
poetry run pytest tests/

# Re-record cassettes (requires API keys)
poetry run pytest tests/ --record-mode=all

# Record only new tests
poetry run pytest tests/ --record-mode=new_episodes
```

---

## Sample Application Template

Use this template for your own applications:

```python
from traceloop.sdk import Traceloop
from traceloop.sdk.decorators import workflow, task
from openai import OpenAI

# 1. Initialize Traceloop SDK
Traceloop.init(app_name="your-app-name")

# 2. Define your workflow
@workflow(name="your_workflow")
def your_workflow(input_data):
    # Process input
    result = process_step(input_data)

    # Call LLM (auto-instrumented)
    response = llm_call(result)

    return response

@task(name="process_step")
def process_step(data):
    # Your business logic
    return processed_data

@task(name="llm_call")
def llm_call(prompt):
    client = OpenAI()
    response = client.chat.completions.create(
        model="gpt-4",
        messages=[{"role": "user", "content": prompt}]
    )
    return response.choices[0].message.content

# 3. Run your workflow
if __name__ == "__main__":
    result = your_workflow("your input")
    print(result)
```

---

## Contributing New Samples

To add a new sample application:

1. Create file in `packages/sample-app/sample_app/`
2. Follow naming convention: `{provider/framework}_{feature}.py`
3. Include:
   - `Traceloop.init()` call
   - Clear comments explaining what's demonstrated
   - Example output in docstring
4. Add VCR test in `packages/sample-app/tests/`
5. Update this index

---

## Resources

- **SDK Guide**: [`traceloop-sdk-guide.md`](traceloop-sdk-guide.md)
- **Instrumentation Docs**: [`instrumentation/`](instrumentation/)
- **Pattern Guides**: [`patterns/`](patterns/)
- **Sample App Directory**: `packages/sample-app/sample_app/`
- **Tests**: `packages/sample-app/tests/`

---

**Documentation Status**: ✅ Complete (Session 1)
**Last Updated**: 2025-11-22
**Total Samples Cataloged**: 64
