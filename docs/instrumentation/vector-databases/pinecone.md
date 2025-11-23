# Pinecone Instrumentation

> **Package**: `opentelemetry-instrumentation-pinecone`
> **Supported Versions**: pinecone-client >= 2.2.2, < 6
> **Auto-Instrumentation**: ✅ Yes (via Traceloop SDK)
> **Vector Database**: Pinecone

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

The Pinecone instrumentation package automatically traces all Pinecone vector database operations, capturing query vectors, results, metadata filters, and usage metrics. It provides complete visibility into vector search performance and behavior.

**Key Features**:
- ✅ Complete operation coverage (query, upsert, delete)
- ✅ Vector and sparse vector tracking
- ✅ Metadata filtering capture
- ✅ Namespace support
- ✅ Batch operations
- ✅ Usage metrics (read/write units)
- ✅ Result tracking with scores and metadata

---

## Quick Start

### With Traceloop SDK (Recommended)

```python
from pinecone import Pinecone
from traceloop.sdk import Traceloop

# Initialize Traceloop - automatically instruments Pinecone
Traceloop.init(app_name="my-pinecone-app")

# Use Pinecone as normal - all operations automatically traced
pc = Pinecone(api_key="your-api-key")
index = pc.Index("your-index")

# Query - automatically traced
results = index.query(
    vector=[0.1, 0.2, 0.3, ...],  # Your query vector
    top_k=5,
    include_metadata=True
)

for match in results['matches']:
    print(f"ID: {match['id']}, Score: {match['score']}")
```

### Manual Instrumentation

```python
from opentelemetry.instrumentation.pinecone import PineconeInstrumentor

# Initialize instrumentation
PineconeInstrumentor().instrument()

# Use Pinecone - all operations are traced
pc = Pinecone(api_key="your-api-key")
index = pc.Index("your-index")
results = index.query(vector=[...], top_k=5)
```

---

## What's Auto-Instrumented

### Instrumented Operations

| Operation | Method | Span Name | Description |
|-----------|--------|-----------|-------------|
| **Query** | `Index.query()` | `pinecone.query` | Vector similarity search |
| **Upsert** | `Index.upsert()` | `pinecone.upsert` | Insert/update vectors |
| **Delete** | `Index.delete()` | `pinecone.delete` | Delete vectors |

**Note**: Both `Index` (HTTP) and `GRPCIndex` (gRPC) variants are instrumented.

### Supported Query Parameters

All query parameters are captured in spans:

| Parameter | Captured | Description |
|-----------|----------|-------------|
| `vector` | ✅ | Query vector (as event) |
| `id` | ✅ | Vector ID for lookup |
| `queries` | ✅ | Multiple query vectors (batch) |
| `top_k` | ✅ | Number of results |
| `namespace` | ✅ | Index namespace |
| `filter` | ✅ | Metadata filter (JSON) |
| `include_values` | ✅ | Include vector values in response |
| `include_metadata` | ✅ | Include metadata in response |
| `sparse_vector` | ✅ | Sparse vector for hybrid search |

---

## What's Captured

### Spans

Each operation creates a span with:
- **Span Name**: `pinecone.query`, `pinecone.upsert`, or `pinecone.delete`
- **Span Kind**: `CLIENT`
- **Duration**: Operation execution time
- **Status**: `OK` or `ERROR` with exception details

### Span Attributes

**Request Attributes** (Query operations):

| Attribute | Description | Example |
|-----------|-------------|---------|
| `server.address` | Pinecone index hostname | `"your-index-abc123.svc.pinecone.io"` |
| `vector_db.vendor` | Vector database provider | `"Pinecone"` |
| `pinecone.query.id` | Vector ID (if querying by ID) | `"vec_001"` |
| `pinecone.query.queries` | Multiple queries (batch) | `[...]` |
| `pinecone.query.top_k` | Result limit | `5` |
| `pinecone.query.namespace` | Index namespace | `"production"` |
| `pinecone.query.filter` | Metadata filter (JSON) | `"{\"genre\": \"sci-fi\"}"` |
| `pinecone.query.include_values` | Include vectors flag | `true` |
| `pinecone.query.include_metadata` | Include metadata flag | `true` |

**Response Attributes** (All operations):

| Attribute | Description | Example |
|-----------|-------------|---------|
| `pinecone.usage.read_units` | Read units consumed | `5` |
| `pinecone.usage.write_units` | Write units consumed | `10` |

**Note**: Usage units are only available on Pinecone Serverless indexes.

### Events

**Query Embeddings Events** (`db.query.embeddings`):

Logged for each query vector:

| Event Attribute | Description | Example |
|-----------------|-------------|---------|
| `db.query.embeddings.vector` | Query vector | `[0.1, 0.2, 0.3, ...]` |

**Query Result Events** (`db.pinecone.query.result`):

Logged for each match in the results:

| Event Attribute | Description | Example |
|-----------------|-------------|---------|
| `db.pinecone.query.result.id` | Vector ID | `"doc_123"` |
| `db.pinecone.query.result.score` | Similarity score | `0.95` |
| `db.pinecone.query.result.metadata` | Result metadata (JSON) | `"{\"title\": \"...\"}"` |
| `db.pinecone.query.result.vector` | Vector values (if included) | `[0.1, 0.2, ...]` |

### Metrics

**Histograms**:
- `db.pinecone.query.duration` - Query operation duration (seconds)
- `db.pinecone.query.scores` - Similarity scores from results

**Counters**:
- `db.pinecone.usage.read_units` - Total read units consumed
- `db.pinecone.usage.write_units` - Total write units consumed

**Common Attributes**: All metrics include `server.address` for filtering by index.

---

## Advanced Features

### Metadata Filtering

Metadata filters are fully captured as JSON:

```python
results = index.query(
    vector=[...],
    top_k=5,
    filter={
        "genre": {"$eq": "sci-fi"},
        "year": {"$gte": 2020}
    },
    include_metadata=True
)

# Filter captured as:
# pinecone.query.filter = '{"genre": {"$eq": "sci-fi"}, "year": {"$gte": 2020}}'
```

**What's Tracked**:
- ✅ Filter structure and complexity
- ✅ Logical operators ($eq, $ne, $gt, $gte, $lt, $lte, $in, $nin)
- ✅ Compound filters ($and, $or)

### Namespace Support

Namespaces are captured in query attributes:

```python
# Query specific namespace
results = index.query(
    vector=[...],
    top_k=5,
    namespace="production",
    include_metadata=True
)

# Captured as: pinecone.query.namespace = "production"
```

**Use Cases**:
- Multi-tenant applications
- Environment separation (dev/staging/prod)
- Data partitioning

### Sparse-Dense Hybrid Search

Hybrid search with sparse and dense vectors is fully instrumented:

```python
results = index.query(
    vector=[0.1, 0.2, ...],  # Dense vector
    sparse_vector={
        "indices": [10, 45, 16],
        "values": [0.5, 0.2, 0.8]
    },
    top_k=5
)

# Both dense and sparse vectors captured in db.query.embeddings events
```

**What's Tracked**:
- ✅ Dense query vector
- ✅ Sparse vector indices and values
- ✅ Hybrid search results

### Batch Operations

**Batch Upsert**:

```python
vectors = [
    {"id": "vec1", "values": [0.1, 0.2, ...], "metadata": {"title": "Doc 1"}},
    {"id": "vec2", "values": [0.3, 0.4, ...], "metadata": {"title": "Doc 2"}},
    # ... up to 100 vectors
]

index.upsert(vectors)

# Single span created for entire batch
# Write units tracked for batch
```

**Batch Query** (multiple vectors):

```python
results = index.query(
    queries=[
        {"vector": [0.1, 0.2, ...], "top_k": 3},
        {"vector": [0.5, 0.6, ...], "top_k": 3}
    ]
)

# Each query vector logged as separate db.query.embeddings event
```

### Usage Metrics (Serverless)

Pinecone Serverless indexes return usage metrics:

```python
results = index.query(vector=[...], top_k=5)

# Usage tracked automatically:
# - pinecone.usage.read_units (span attribute)
# - db.pinecone.usage.read_units (metric counter)
```

**Benefits**:
- ✅ Cost tracking per query
- ✅ Identify expensive operations
- ✅ Optimize query patterns

**Note**: Usage units are only available on Serverless indexes, not Pod-based indexes.

---

## Configuration Options

### Instrumentor Parameters

```python
from opentelemetry.instrumentation.pinecone import PineconeInstrumentor

PineconeInstrumentor().instrument(
    # Custom exception logger
    exception_logger=my_exception_handler
)
```

### Exception Logging

```python
def custom_exception_handler(exception):
    print(f"Pinecone instrumentation error: {exception}")
    # Log to monitoring service, etc.

instrumentor = PineconeInstrumentor(exception_logger=custom_exception_handler)
instrumentor.instrument()
```

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `TRACELOOP_METRICS_ENABLED` | `"true"` | Enable metrics collection |

**Disable Metrics**:

```bash
export TRACELOOP_METRICS_ENABLED=false
```

### Custom Providers

```python
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.metrics import MeterProvider

tracer_provider = TracerProvider()
meter_provider = MeterProvider()

PineconeInstrumentor().instrument(
    tracer_provider=tracer_provider,
    meter_provider=meter_provider
)
```

---

## Manual Instrumentation Scenarios

### When Auto-Instrumentation Isn't Enough

1. **Custom Workflow Context**: Add application-specific metadata

```python
from opentelemetry import trace
from traceloop.sdk.decorators import workflow, task

@task(name="semantic_search")
def search_documents(query_text, user_id):
    span = trace.get_current_span()
    span.set_attribute("user.id", user_id)
    span.set_attribute("query.length", len(query_text))

    # Generate embedding
    embedding = embed_text(query_text)

    # Search Pinecone (auto-instrumented)
    results = index.query(vector=embedding, top_k=5, include_metadata=True)

    return [r['metadata']['text'] for r in results['matches']]
```

2. **RAG Pipeline Tracking**: Use workflow decorators for multi-step retrieval

```python
from traceloop.sdk.decorators import workflow, task

@task(name="embed_query")
def embed_query(text):
    return embedding_model.encode(text)

@task(name="retrieve_context")
def retrieve_context(embedding):
    results = index.query(
        vector=embedding,
        top_k=3,
        include_metadata=True
    )
    return [r['metadata']['text'] for r in results['matches']]

@workflow(name="rag_pipeline")
def answer_question(question):
    embedding = embed_query(question)
    context = retrieve_context(embedding)
    answer = llm.generate(question, context)
    return answer
```

3. **Performance Monitoring**: Track search quality

```python
from opentelemetry import trace

def monitored_search(query_vector, expected_results=None):
    span = trace.get_current_span()

    results = index.query(
        vector=query_vector,
        top_k=10,
        include_metadata=True,
        include_values=True
    )

    # Add custom metrics
    top_score = results['matches'][0]['score'] if results['matches'] else 0
    span.set_attribute("search.top_score", top_score)
    span.set_attribute("search.results_count", len(results['matches']))

    # Track relevance if ground truth available
    if expected_results:
        found = sum(1 for m in results['matches'] if m['id'] in expected_results)
        precision = found / len(results['matches']) if results['matches'] else 0
        span.set_attribute("search.precision", precision)

    return results
```

---

## Sample Applications

All samples located in `packages/sample-app/sample_app/`

### Sample 1: RAG with OpenAI Embeddings

**File**: `pinecone_app.py`

**Features**:
- Index creation and batch upsert
- OpenAI text-embedding-ada-002 for embeddings
- Metadata inclusion
- Integration with LLM for RAG
- Workflow and task decorators

**Code**:
```python
from pinecone import Pinecone, ServerlessSpec
from openai import OpenAI
from traceloop.sdk import Traceloop
from traceloop.sdk.decorators import workflow, task

Traceloop.init(app_name="pinecone_app")

# Initialize clients
pc = Pinecone(api_key=os.getenv("PINECONE_API_KEY"))
openai_client = OpenAI()

# Create index
pc.create_index(
    name=index_name,
    dimension=1536,  # text-embedding-ada-002 dimensions
    metric='cosine',
    spec=ServerlessSpec(cloud='aws', region='us-east-1')
)

index = pc.Index(index_name)

# Batch upsert with embeddings
for batch in dataset.iter_documents(batch_size=100):
    index.upsert(batch)

@task(name="retrieve")
def retrieve(query):
    # Generate query embedding
    res = openai_client.embeddings.create(
        input=[query],
        model="text-embedding-ada-002"
    )
    xq = res.data[0].embedding

    # Query Pinecone
    res = index.query(vector=xq, top_k=3,
                      include_metadata=True,
                      include_values=True)

    # Extract contexts
    contexts = [x["metadata"]["text"] for x in res.matches]
    return "\n\n---\n\n".join(contexts)

@workflow(name="query_with_retrieve")
def query_with_retrieve(query):
    context = retrieve(query)
    # Use context with LLM
    return generate_answer(context, query)
```

### Sample 2: RAG with Sentence Transformers

**File**: `pinecone_app_sentence_transformers.py`

**Features**:
- Local embeddings (no API calls)
- Sentence Transformers model
- Console span exporter for debugging
- Batch insertion with metadata

**Code**:
```python
from sentence_transformers import SentenceTransformer
from opentelemetry.sdk.trace.export import ConsoleSpanExporter
from traceloop.sdk import Traceloop

# Initialize with console exporter for debugging
Traceloop.init(
    app_name="pinecone_st_app",
    exporter=ConsoleSpanExporter(),
    disable_batch=True
)

# Load local model
model = SentenceTransformer("intfloat/e5-small-v2")

# Generate embeddings locally
input_texts = ["Document 1 text", "Document 2 text", ...]
embeddings = model.encode(input_texts, normalize_embeddings=True)

# Prepare data with metadata
data_to_insert = [
    {
        "id": f"id{i}",
        "values": vector.tolist(),
        "metadata": {"description": f"Document {i}"}
    }
    for i, vector in enumerate(embeddings)
]

# Batch upsert
index.upsert(data_to_insert)

# Query
query_text = "search query"
query_embedding = model.encode([query_text], normalize_embeddings=True)[0]

result = index.query(
    vector=query_embedding.tolist(),
    top_k=3,
    include_metadata=True,
    include_values=True
)

for match in result['matches']:
    print(f"Score: {match['score']}, Metadata: {match['metadata']}")
```

### Running Samples

```bash
# Install dependencies
cd packages/sample-app
poetry install

# Set API keys
export PINECONE_API_KEY=your-key-here
export OPENAI_API_KEY=your-openai-key  # For OpenAI sample

# Run a sample
poetry run python sample_app/pinecone_app.py
```

---

## Common Patterns

### Pattern 1: Semantic Search

```python
from traceloop.sdk.decorators import workflow

@workflow(name="semantic_search")
def semantic_search(query_text, top_k=5):
    # Generate embedding
    embedding = embed_text(query_text)

    # Search Pinecone
    results = index.query(
        vector=embedding,
        top_k=top_k,
        include_metadata=True,
        filter={"status": "published"}  # Only published documents
    )

    return [
        {
            "id": m['id'],
            "score": m['score'],
            "text": m['metadata']['text']
        }
        for m in results['matches']
    ]
```

### Pattern 2: Multi-Namespace Search

```python
def search_all_namespaces(query_vector, namespaces):
    """Search across multiple namespaces and combine results."""
    all_results = []

    for namespace in namespaces:
        results = index.query(
            vector=query_vector,
            top_k=5,
            namespace=namespace,
            include_metadata=True
        )
        all_results.extend(results['matches'])

    # Sort by score
    all_results.sort(key=lambda x: x['score'], reverse=True)
    return all_results[:10]  # Top 10 across all namespaces
```

### Pattern 3: Filtered Search with Fallback

```python
def search_with_fallback(query_vector, filter_dict, top_k=5):
    """Try filtered search, fall back to unfiltered if no results."""
    # Try with filter
    results = index.query(
        vector=query_vector,
        top_k=top_k,
        filter=filter_dict,
        include_metadata=True
    )

    if not results['matches']:
        # Fallback to unfiltered search
        results = index.query(
            vector=query_vector,
            top_k=top_k,
            include_metadata=True
        )

    return results['matches']
```

### Pattern 4: Batch Upsert with Progress Tracking

```python
from traceloop.sdk.decorators import task

@task(name="batch_upsert")
def batch_upsert_vectors(vectors, batch_size=100):
    """Upsert vectors in batches with tracking."""
    total = len(vectors)

    for i in range(0, total, batch_size):
        batch = vectors[i:i + batch_size]
        index.upsert(batch)
        print(f"Upserted {min(i + batch_size, total)}/{total}")

    return total
```

### Pattern 5: Hybrid Search (Dense + Sparse)

```python
def hybrid_search(dense_vector, sparse_indices, sparse_values, top_k=10):
    """Hybrid search combining dense and sparse vectors."""
    results = index.query(
        vector=dense_vector,
        sparse_vector={
            "indices": sparse_indices,
            "values": sparse_values
        },
        top_k=top_k,
        include_metadata=True
    )

    return results['matches']
```

---

## Troubleshooting

### Issue: No spans appearing

**Symptoms**: Pinecone operations work but no traces visible

**Solutions**:
1. Verify initialization order:
   ```python
   from traceloop.sdk import Traceloop
   Traceloop.init()  # Must be called first

   from pinecone import Pinecone  # Then import Pinecone
   ```

2. Check instrumentation is active:
   ```python
   from opentelemetry.instrumentation.pinecone import PineconeInstrumentor
   print(PineconeInstrumentor().is_instrumented_by_opentelemetry)
   ```

3. Verify exporter configured:
   ```python
   from opentelemetry.sdk.trace.export import ConsoleSpanExporter
   Traceloop.init(exporter=ConsoleSpanExporter())
   ```

### Issue: Usage metrics missing

**Symptoms**: Spans exist but no read/write unit attributes

**Cause**: Usage units only available on Pinecone Serverless indexes.

**Solution**:
- Verify you're using a Serverless index (not Pod-based)
- Check Pinecone dashboard to confirm index type

### Issue: Query vectors not captured

**Symptoms**: Spans exist but no `db.query.embeddings` events

**Cause**: Events may not be visible depending on your telemetry backend.

**Solution**:
- Use console exporter to verify events are emitted:
  ```python
  from opentelemetry.sdk.trace.export import ConsoleSpanExporter
  Traceloop.init(exporter=ConsoleSpanExporter())
  ```
- Check backend supports OpenTelemetry events

### Issue: Metadata filters not showing

**Symptoms**: Filter attribute empty or missing

**Cause**: Filter might not be JSON-serializable.

**Solution**:
- Use dict-based filters:
  ```python
  # Good
  filter={"genre": "sci-fi"}

  # May not work
  filter=custom_filter_object
  ```

### Issue: High memory usage

**Symptoms**: Memory grows during large batch operations

**Cause**: Vector values captured in events can be large.

**Solution**:
- Disable `include_values` in queries:
  ```python
  results = index.query(
      vector=...,
      top_k=5,
      include_values=False,  # Reduce memory
      include_metadata=True
  )
  ```
- Process results in batches
- Consider disabling metrics if not needed:
  ```bash
  export TRACELOOP_METRICS_ENABLED=false
  ```

### Issue: Namespace not captured

**Symptoms**: `pinecone.query.namespace` attribute missing

**Cause**: Namespace not specified in query.

**Solution**:
- Explicitly specify namespace:
  ```python
  results = index.query(
      vector=...,
      namespace="production",  # Required for attribute
      top_k=5
  )
  ```

---

## Additional Resources

- **Pinecone Documentation**: https://docs.pinecone.io
- **Pinecone Python Client**: https://github.com/pinecone-io/pinecone-python-client
- **Semantic Conventions**: https://opentelemetry.io/docs/specs/semconv/database/
- **Traceloop SDK Guide**: [../../traceloop-sdk-guide.md](../../traceloop-sdk-guide.md)
- **Sample Applications**: [../../examples-index.md](../../examples-index.md)

---

**Last Updated**: 2025-11-22 (Session 4)
