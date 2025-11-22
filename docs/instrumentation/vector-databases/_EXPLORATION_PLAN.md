# Vector Database Instrumentation - Exploration Plan

> **Purpose**: Systematic recipe for analyzing vector database instrumentation packages
> **Category**: Vector Databases (7 packages: Pinecone, ChromaDB, Qdrant, Weaviate, Milvus, LanceDB, Marqo)
> **Reference**: See `/docs/instrumentation/EXPLORATION_GUIDE.md` for general methodology

---

## 📋 Vector Database Analysis Checklist

Use this checklist for each vector database instrumentation package:

### 1. Operation Coverage

- [ ] **Query/Search Operations**
  - Vector similarity search
  - Hybrid search (vector + keyword)
  - Filtered search (metadata filtering)
  - Batch queries
  - Span names created

- [ ] **Write Operations**
  - Insert/Add vectors
  - Upsert (insert or update)
  - Batch upsert
  - Metadata updates

- [ ] **Delete Operations**
  - Delete by ID
  - Delete by filter
  - Batch delete
  - Collection/namespace deletion

- [ ] **Update Operations**
  - Update vectors
  - Update metadata
  - Partial updates

- [ ] **Read Operations**
  - Fetch by ID
  - List operations
  - Get statistics

- [ ] **Index/Collection Management**
  - Create index/collection
  - Delete index/collection
  - List indexes/collections
  - Describe index/collection
  - Configure index settings

### 2. Data Capture Analysis

#### Span Basics
- [ ] Span names (e.g., `pinecone.query`, `chroma.add`)
- [ ] Span kind (CLIENT)
- [ ] Operation duration

#### Common Attributes
- [ ] `vector_db.vendor` - Database name
- [ ] `server.address` - Database endpoint
- [ ] `db.operation` - Operation type
- [ ] `db.namespace` - Namespace/collection name
- [ ] Database-specific attributes

#### Query-Specific Attributes
- [ ] `db.query.top_k` - Number of results requested
- [ ] `db.query.filter` - Metadata filter (JSON)
- [ ] `db.query.include_values` - Return vector values
- [ ] `db.query.include_metadata` - Return metadata
- [ ] Similarity metric used

#### Vector Embeddings
- [ ] How embeddings are captured (events vs attributes)
- [ ] Dense vectors
- [ ] Sparse vectors
- [ ] Embedding dimensions

#### Query Results
- [ ] Result IDs
- [ ] Similarity scores
- [ ] Metadata
- [ ] Vector values (optional)
- [ ] How results are captured (events vs attributes)

#### Usage Metrics
- [ ] Read units consumed
- [ ] Write units consumed
- [ ] Compute units
- [ ] Database-specific usage

### 3. Metrics Collection

- [ ] **Operation Duration**
  - Metric name
  - Metric type (histogram)
  - Dimensions

- [ ] **Query Scores**
  - Similarity scores distribution
  - Metric type (histogram)

- [ ] **Usage Counters**
  - Read/write units
  - API calls
  - Database-specific metrics

- [ ] **Error Metrics**
  - Exception counters
  - Error types

### 4. Implementation Analysis

- [ ] **Patching Mechanism**
  - Module paths wrapped
  - Class and method names
  - Use of `wrapt.wrap_function_wrapper`

- [ ] **Index/Collection Client Wrapping**
  - How client instances are wrapped
  - Methods intercepted
  - Context preservation

- [ ] **Error Handling**
  - Exception capture
  - `@dont_throw` decorator usage
  - Error metrics

- [ ] **Configuration**
  - Instrumentor init parameters
  - Environment variables
  - Custom callbacks

### 5. Special Features

- [ ] **Namespace/Collection Support**
  - How namespaces are handled
  - Namespace isolation in traces

- [ ] **Filtering Support**
  - Metadata filtering
  - Filter expression capture
  - Complex filter queries

- [ ] **Batch Operations**
  - Batch insert/upsert
  - Batch query
  - How batches are traced

- [ ] **Async Support**
  - Async client support
  - Context propagation
  - Async query/upsert

- [ ] **Hybrid Search**
  - Vector + keyword search
  - Sparse + dense vectors
  - How different search types are captured

### 6. Gaps & Manual Instrumentation

- [ ] **Operations NOT Instrumented**
  - List missing operations
  - Why they're not instrumented
  - Manual alternatives

- [ ] **Missing Data**
  - What's not captured
  - Privacy considerations

- [ ] **Manual Scenarios**
  - Index management operations
  - Complex workflows (RAG pipelines)
  - Custom retrieval logic
  - Post-processing

### 7. Sample Applications

- [ ] List all vector DB samples
- [ ] Simple query examples
- [ ] RAG pipeline examples
- [ ] Integration examples (with LangChain, LlamaIndex)
- [ ] What each demonstrates

---

## 🤖 AI Analysis Prompt Template

Use this prompt to analyze a vector database instrumentation package:

```
Analyze the opentelemetry-instrumentation-{VECTORDB} package in comprehensive detail.

**Package Location**: `/home/user/openllmetry/packages/opentelemetry-instrumentation-{vectordb}/`

I need a **Level 2 (Code Analysis)** covering:

## 1. Automatically Instrumented Operations

List ALL operations that are automatically instrumented:

### Data Operations
- Query/search operations (list all methods)
- Insert/upsert operations
- Delete operations
- Update operations
- Fetch operations

### Management Operations
- Index/collection creation
- Index/collection deletion
- Configuration operations

For each operation:
- Full method path (e.g., `{vectordb}.Index.query`)
- Span name created
- Whether it's instrumented (Yes/No)

## 2. Data Captured (Spans, Attributes, Events, Metrics)

### Span Attributes

**Common Attributes**:
- `vector_db.vendor`: `"{vectordb}"`
- `server.address`: Database endpoint
- Other common attributes

**Query-Specific Attributes**:
List ALL attributes for query operations:
- Top-k, filters, namespaces, etc.

**Result Attributes**:
How are results captured?
- As span attributes?
- As span events?
- Structure and format

**Embedding Capture**:
How are query vectors captured?
- As events (`db.query.embeddings`)?
- As attributes?
- Dense vs sparse vectors

### Metrics

List ALL metrics collected:
- Metric names
- Metric types (histogram, counter)
- Dimensions/attributes
- What each measures

## 3. Patching Mechanism

Explain:
- How the {VectorDB} client is patched
- Module paths wrapped
- Methods intercepted
- Wrapper pattern used

Example:
\`\`\`python
wrap_function_wrapper(
    "{vectordb}",
    "Index.query",
    _wrap(...)
)
\`\`\`

## 4. Special Features Support

Analyze:
- **Namespaces/Collections**: How are they handled in traces?
- **Filtering**: Metadata filter capture
- **Batch Operations**: How are batches traced?
- **Async Support**: Async client instrumentation
- **Hybrid Search**: Vector + keyword search

## 5. Scenarios Requiring Manual Instrumentation

Identify operations that are NOT automatically instrumented:
- Index/collection management (create, delete, list)
- Advanced operations (stats, describe)
- Why they're not instrumented
- How to manually instrument them

Provide code examples using:
\`\`\`python
from opentelemetry import trace

tracer = trace.get_tracer(__name__)

with tracer.start_as_current_span("create_index"):
    # Manual instrumentation
    index = client.create_index(...)
\`\`\`

## 6. Configuration Options

Document:
- Instrumentor initialization parameters
- Environment variables
- Custom callbacks (exception_logger, etc.)

## 7. Sample Applications

List all samples in `packages/sample-app/sample_app/` related to {vectordb}:
- File names and paths
- What each demonstrates (basic query, RAG, integration)
- Key patterns shown

## Analysis Files

Please analyze:
1. **Main instrumentor**: `opentelemetry/instrumentation/{vectordb}/__init__.py`
2. **Handler files**: Query handlers, span utilities
3. **Test files**: `tests/` directory
4. **Sample apps**: Look for {vectordb} in sample-app

Provide a comprehensive summary following the Vector Database Exploration Plan checklist.
```

---

## 📊 Documentation Template for Vector Databases

```markdown
# {VectorDB Name} Instrumentation

> **Exploration Status**: ✅ Complete | 🚧 In Progress | 📋 TODO
> **Last Updated**: {Date}
> **Package**: `opentelemetry-instrumentation-{vectordb}`
> **Instrumented Library**: `{vectordb}` Python client
> **Package Path**: `packages/opentelemetry-instrumentation-{vectordb}/`

## Table of Contents
- [Overview](#overview)
- [Quick Start](#quick-start)
- [What's Auto-Instrumented](#whats-auto-instrumented)
- [What's Captured](#whats-captured)
- [Manual Instrumentation](#manual-instrumentation)
- [Configuration](#configuration)
- [Examples](#examples)
- [Common Patterns](#common-patterns)
- [Troubleshooting](#troubleshooting)

## Overview

Brief description of the vector database and instrumentation capabilities.

## Quick Start

\`\`\`python
from traceloop.sdk import Traceloop
from {vectordb} import Client

# Initialize with auto-instrumentation
Traceloop.init(app_name="my-app")

# All {VectorDB} operations are automatically traced
client = Client(...)
index = client.Index("my-index")

# Query - automatically instrumented
results = index.query(
    vector=[0.1, 0.2, ...],
    top_k=10
)
# ✅ Span created: "{vectordb}.query"
# ✅ Query vectors, results, and scores captured
\`\`\`

**Sample Reference**: [`packages/sample-app/sample_app/{vectordb}_app.py`](../../packages/sample-app/sample_app/{vectordb}_app.py)

## What's Auto-Instrumented

### Data Operations

| Operation | Method | Span Name | Auto-Instrumented |
|-----------|--------|-----------|-------------------|
| Query | `Index.query()` | `{vectordb}.query` | ✅ Yes |
| Upsert | `Index.upsert()` | `{vectordb}.upsert` | ✅ Yes |
| Delete | `Index.delete()` | `{vectordb}.delete` | ✅ Yes |
| Fetch | `Index.fetch()` | - | ❌ No (manual) |

### Management Operations

| Operation | Method | Auto-Instrumented |
|-----------|--------|-------------------|
| Create Index | `create_index()` | ❌ No (manual) |
| Delete Index | `delete_index()` | ❌ No (manual) |
| List Indexes | `list_indexes()` | ❌ No (manual) |

## What's Captured

### Span Attributes

#### Common Attributes (All Operations)

| Attribute | Description | Example |
|-----------|-------------|---------|
| `vector_db.vendor` | Database name | `"{VectorDB}"` |
| `server.address` | Database endpoint | `"api.{vectordb}.io"` |
| `db.namespace` | Namespace/collection | `"my-namespace"` |

#### Query-Specific Attributes

| Attribute | Description | Example |
|-----------|-------------|---------|
| `db.query.top_k` | Results requested | `10` |
| `db.query.filter` | Metadata filter | `{"category": "tech"}` |
| `db.query.include_values` | Return vectors | `true` |
| `db.query.include_metadata` | Return metadata | `true` |

### Query Embeddings (Events)

Query vectors are captured as span events:

**Event Name**: `db.query.embeddings`

| Attribute | Description | Format |
|-----------|-------------|--------|
| `db.query.embeddings.vector` | Query vector | Array of floats |

### Query Results (Events)

Each result is captured as a span event:

**Event Name**: `db.{vectordb}.query.result`

| Attribute | Description | Example |
|-----------|-------------|---------|
| `db.{vectordb}.query.result.id` | Result ID | `"doc-123"` |
| `db.{vectordb}.query.result.score` | Similarity score | `0.95` |
| `db.{vectordb}.query.result.metadata` | Metadata | `{"title": "..."}` |
| `db.{vectordb}.query.result.vector` | Vector values | `[0.1, 0.2, ...]` |

### Metrics

| Metric Name | Type | Description | Dimensions |
|-------------|------|-------------|------------|
| `db.{vectordb}.query.duration` | Histogram | Query latency | `operation`, `namespace` |
| `db.{vectordb}.query.scores` | Histogram | Similarity scores | - |
| `db.{vectordb}.usage.read_units` | Counter | Read units consumed | `namespace` |
| `db.{vectordb}.usage.write_units` | Counter | Write units consumed | `namespace` |

## Manual Instrumentation

### Index Management Operations

Index/collection management operations are NOT automatically instrumented. Use manual spans:

\`\`\`python
from opentelemetry import trace
from traceloop.sdk.decorators import task

tracer = trace.get_tracer(__name__)

# Option 1: Using OpenTelemetry tracer
with tracer.start_as_current_span("create_{vectordb}_index") as span:
    index = client.create_index(
        name="my-index",
        dimension=1536
    )
    span.set_attribute("db.index.name", "my-index")
    span.set_attribute("db.index.dimension", 1536)

# Option 2: Using Traceloop decorator
@task(name="create_index")
def create_index(name, dimension):
    return client.create_index(name=name, dimension=dimension)
\`\`\`

**Sample Reference**: [`packages/sample-app/sample_app/{vectordb}_app.py:L23-39`](../../packages/sample-app/sample_app/{vectordb}_app.py)

### RAG Workflow Example

\`\`\`python
from traceloop.sdk.decorators import workflow, task

@workflow(name="rag_pipeline")
def rag_pipeline(query: str):
    # 1. Generate embedding (auto-instrumented if using OpenAI/etc)
    embedding = generate_embedding(query)

    # 2. Query vector DB (auto-instrumented)
    results = index.query(vector=embedding, top_k=5)

    # 3. Generate response (auto-instrumented if using LLM)
    return generate_response(query, results)
\`\`\`

## Configuration

\`\`\`python
from opentelemetry.instrumentation.{vectordb} import {VectorDB}Instrumentor

{VectorDB}Instrumentor(
    exception_logger=custom_logger,
    # ... other options
).instrument()
\`\`\`

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `TRACELOOP_METRICS_ENABLED` | Enable metrics collection | `true` |

## Examples

### Basic Query

\`\`\`python
from traceloop.sdk import Traceloop
from {vectordb} import Client

Traceloop.init(app_name="vector-search")

client = Client(...)
index = client.Index("my-index")

# Automatically traced
results = index.query(
    vector=[0.1, 0.2, 0.3, ...],
    top_k=10,
    filter={"category": "tech"}
)

for match in results['matches']:
    print(f"ID: {match['id']}, Score: {match['score']}")
\`\`\`

### RAG with {VectorDB}

{Code example integrating with LLM}

## Sample Applications

| Sample File | Demonstrates | Complexity |
|-------------|--------------|------------|
| `{vectordb}_app.py` | Basic query, upsert | Simple |
| `{vectordb}_rag_app.py` | RAG pipeline | Medium |

## Common Patterns

### Pattern 1: Semantic Search

{Best practice example}

### Pattern 2: Hybrid Search (if applicable)

{Best practice example}

## Troubleshooting

### Issue: Query vectors not captured

**Cause**: Metrics disabled or content tracing disabled
**Solution**: Ensure `TRACELOOP_TRACE_CONTENT=true`

### Issue: High cardinality in metrics

**Cause**: Namespace/collection names in metric dimensions
**Solution**: Use aggregation or custom metric attributes
```

---

## 🎯 Priority Matrix for Vector Databases

| Database | Priority | Reason |
|----------|----------|--------|
| Pinecone | 🟡 Medium | Popular managed service |
| ChromaDB | 🟡 Medium | Popular open source, RAG common |
| Qdrant | 🟡 Medium | Growing adoption |
| Weaviate | 🟢 Lower | Enterprise use cases |
| Milvus | 🟢 Lower | Enterprise, large scale |
| LanceDB | 🟢 Lower | Newer, embedded |
| Marqo | 🟢 Lower | Multimodal search |

---

## ✅ Completion Criteria

A vector database documentation is complete when:

- [ ] All auto-instrumented operations documented
- [ ] Complete attribute reference for query, upsert, delete
- [ ] Embedding capture explained (events/attributes)
- [ ] Result capture explained (events/attributes)
- [ ] Metrics documented with dimensions
- [ ] Manual instrumentation for management ops documented
- [ ] Configuration options documented
- [ ] RAG workflow example provided
- [ ] All sample apps referenced
- [ ] Common patterns documented

---

**End of Vector Database Exploration Plan**
