# Vector Databases

## Descripción

Vector databases almacenan y recuperan embeddings eficientemente para similarity search. Esenciales para RAG, recommendation systems, semantic search. A diferencia de DBs tradicionales (exact match), vector DBs usan ANN (Approximate Nearest Neighbors) para búsqueda semántica. Principales: Pinecone (managed, escalable), Weaviate (open-source, GraphQL), Chroma (simple, local), Qdrant (Rust, rápido), FAISS (Meta, library), Milvus (cloud-native). Operaciones: insert vectors, similarity search (cosine, dot product, euclidean), filtering con metadata, hybrid search. Use cases: RAG knowledge bases, recommendation engines, duplicate detection, anomaly detection. Crítico: indexing strategy (HNSW, IVF), dimensionality (768, 1536), metadata filtering, scalability.

## Conceptos

### 1. **Embeddings**
- Dense vectors (768, 1536 dims típico)
- Sentence-BERT, OpenAI embeddings
- Semantic meaning capturado
- Normalizados para cosine similarity

### 2. **Similarity Metrics**
- Cosine similarity (más común)
- Dot product
- Euclidean distance
- Manhattan distance

### 3. **Indexing Algorithms**
- HNSW (Hierarchical Navigable Small World)
- IVF (Inverted File Index)
- LSH (Locality Sensitive Hashing)
- Trade-off speed vs accuracy

### 4. **Metadata Filtering**
- Pre-filtering vs post-filtering
- Combine vector + metadata search
- Namespace/collections
- Access control

### 5. **Hybrid Search**
- Vector + keyword (BM25)
- Dense + sparse embeddings
- Re-ranking
- Best of both worlds

## Opciones Populares

| Database | Tipo | Pros | Contras |
|----------|------|------|---------|
| Pinecone | Managed | Escalable, simple | Costo, vendor lock-in |
| Weaviate | Open-source | Flexible, GraphQL | Self-host complexity |
| Chroma | Local | Fácil setup | No distribuido |
| Qdrant | Open/Managed | Rápido, filtros buenos | Comunidad pequeña |
| FAISS | Library | Gratis, Meta-backed | No es DB completo |

## Recursos

**Pinecone**: https://www.pinecone.io/learn/
**Weaviate**: https://weaviate.io/developers/weaviate
**Chroma**: https://docs.trychroma.com/

## Ejemplos

### Ejemplo 1: Pinecone

```python
import pinecone
from sentence_transformers import SentenceTransformer

# Initialize
pinecone.init(api_key="your-key", environment="us-west1-gcp")

# Create index
if "my-index" not in pinecone.list_indexes():
    pinecone.create_index("my-index", dimension=384, metric="cosine")

index = pinecone.Index("my-index")

# Encoder
model = SentenceTransformer('all-MiniLM-L6-v2')

# Insert vectors
texts = ["AI is amazing", "Machine learning rocks", "Deep learning is cool"]
embeddings = model.encode(texts)

vectors = [(f"id-{i}", emb.tolist(), {"text": text}) 
           for i, (emb, text) in enumerate(zip(embeddings, texts))]

index.upsert(vectors=vectors)

# Search
query = "artificial intelligence"
query_emb = model.encode(query).tolist()

results = index.query(vector=query_emb, top_k=3, include_metadata=True)

for match in results.matches:
    print(f"Score: {match.score}, Text: {match.metadata['text']}")
```

### Ejemplo 2: Chroma

```python
import chromadb
from chromadb.config import Settings

# Client
client = chromadb.Client(Settings(
    chroma_db_impl="duckdb+parquet",
    persist_directory="./chroma_db"
))

# Collection
collection = client.create_collection("my_collection")

# Add documents
collection.add(
    documents=["AI is amazing", "ML rocks", "DL is cool"],
    metadatas=[{"topic": "AI"}, {"topic": "ML"}, {"topic": "DL"}],
    ids=["id1", "id2", "id3"]
)

# Query
results = collection.query(
    query_texts=["artificial intelligence"],
    n_results=2,
    where={"topic": "AI"}
)

print(results)
```

### Ejemplo 3: Weaviate

```python
import weaviate

# Client
client = weaviate.Client("http://localhost:8080")

# Schema
schema = {
    "class": "Document",
    "vectorizer": "text2vec-transformers",
    "properties": [
        {"name": "content", "dataType": ["text"]},
        {"name": "category", "dataType": ["string"]}
    ]
}

client.schema.create_class(schema)

# Add data
client.data_object.create(
    data_object={"content": "AI is amazing", "category": "tech"},
    class_name="Document"
)

# Search
result = client.query.get("Document", ["content", "category"]) \
    .with_near_text({"concepts": ["artificial intelligence"]}) \
    .with_limit(3) \
    .do()

print(result)
```

### Ejemplo 4: FAISS

```python
import faiss
import numpy as np

# Create index
dimension = 384
index = faiss.IndexFlatL2(dimension)

# Add vectors
vectors = np.random.random((1000, dimension)).astype('float32')
index.add(vectors)

# Search
query = np.random.random((1, dimension)).astype('float32')
distances, indices = index.search(query, k=5)

print(f"Top 5 indices: {indices[0]}")
print(f"Distances: {distances[0]}")
```

### Ejemplo 5: Qdrant

```python
from qdrant_client import QdrantClient
from qdrant_client.models import Distance, VectorParams, PointStruct

# Client
client = QdrantClient(":memory:")

# Create collection
client.create_collection(
    collection_name="my_collection",
    vectors_config=VectorParams(size=384, distance=Distance.COSINE)
)

# Insert
points = [
    PointStruct(
        id=i,
        vector=embedding.tolist(),
        payload={"text": text}
    )
    for i, (embedding, text) in enumerate(zip(embeddings, texts))
]

client.upsert(collection_name="my_collection", points=points)

# Search
results = client.search(
    collection_name="my_collection",
    query_vector=query_emb.tolist(),
    limit=3
)
```

## Mejores Prácticas

```python
# ✅ Normalize embeddings para cosine similarity
embeddings = embeddings / np.linalg.norm(embeddings, axis=1, keepdims=True)

# ✅ Use metadata filtering
results = index.query(vector=query, filter={"category": "tech"})

# ✅ Batch operations
index.upsert(vectors=batch_vectors)  # Faster than one-by-one

# ✅ Monitor performance
# Check query latency, index size

# ❌ No almacenar texto completo en vector DB
# Store reference/ID, fetch text desde DB relacional
```

## Cuándo Usar

**Vector DB cuando:**
- Similarity search crítico
- RAG system
- Recommendation engine
- >10K vectors

**SQL/NoSQL cuando:**
- Exact match suficiente
- Structured data queries
- ACID transactions
- Relational data

## Proyectos

1. **Semantic Search Engine**
   - Index documents
   - Pinecone/Weaviate
   - Web interface

2. **RAG Knowledge Base**
   - Company docs
   - Hybrid search
   - LangChain integration

3. **Recommendation System**
   - User/item embeddings
   - Collaborative filtering
   - Real-time updates

## Checklist

- [ ] Entender vector similarity
- [ ] Setup vector database
- [ ] Implement insert/search
- [ ] Add metadata filtering
- [ ] Optimize indexing
- [ ] Monitor performance
- [ ] Scale para producción

---
**Tiempo**: 2-3 semanas
