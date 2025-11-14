# RAG - Retrieval Augmented Generation

## Descripción

RAG combina generación de LLMs con recuperación de información externa, solucionando limitaciones clave: conocimiento desactualizado (LLMs entrenados en datos estáticos), alucinaciones (inventa información), falta de fuentes verificables. El flujo: 1) Usuario hace pregunta, 2) Sistema recupera documentos relevantes de base de conocimiento, 3) LLM genera respuesta usando contexto recuperado. Vector databases almacenan embeddings de documentos para búsqueda semántica rápida. Ventajas sobre fine-tuning: actualización dinámica de conocimiento (agregar documentos sin reentrenar), cita fuentes, más económico, funciona con LLMs cerrados (GPT-4, Claude). Aplicaciones: chatbots empresariales, Q&A sobre documentos, asistentes de código, soporte técnico. Arquitecturas: naive RAG (simple retrieval), advanced RAG (re-ranking, hybrid search), modular RAG (agents, tools). Herramientas: LangChain, LlamaIndex, Haystack para orquestación; Pinecone, Weaviate, Chroma para vector storage. Crítico para producción de LLMs.

## Conceptos Clave

### 1. **Embeddings**
- Representación vectorial de texto
- Similarity search (cosine, dot product)
- Dense vs sparse embeddings
- Models: OpenAI, Sentence-BERT

### 2. **Vector Databases**
- Almacenamiento de embeddings
- Búsqueda por similaridad eficiente
- ANN (Approximate Nearest Neighbors)
- Pinecone, Weaviate, Chroma, FAISS

### 3. **Retrieval Strategies**
- Semantic search (embeddings)
- Keyword search (BM25)
- Hybrid search (combinar ambos)
- Re-ranking retrieved docs

### 4. **Chunking**
- Dividir documentos en chunks
- Tamaño óptimo (~500 tokens)
- Overlap entre chunks
- Semantic chunking

### 5. **Prompt Engineering para RAG**
- Context injection en prompt
- System prompt para grounding
- Citation enforcement
- Handling no relevant docs

### 6. **Evaluation**
- Retrieval metrics (Recall@k, MRR)
- Generation metrics (ROUGE, BLEU)
- Answer relevance
- Faithfulness to sources

## Recursos

### Cursos
1. **LangChain & Vector Databases - DeepLearning.AI**
   - https://www.deeplearning.ai/short-courses/

2. **Building RAG Applications - Pinecone**
   - https://www.pinecone.io/learn/

### Documentación
1. **LangChain Docs**
   - https://python.langchain.com/docs/

2. **LlamaIndex Docs**
   - https://docs.llamaindex.ai/

### Papers
1. **RAG: Retrieval-Augmented Generation** - Lewis et al. (2020)
2. **REALM** - Guu et al. (2020)

## Ejemplos Prácticos

### Ejemplo 1: RAG Básico con LangChain

```python
from langchain.embeddings import OpenAIEmbeddings
from langchain.vectorstores import Chroma
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.llms import OpenAI
from langchain.chains import RetrievalQA
from langchain.document_loaders import TextLoader

# 1. Cargar documentos
loader = TextLoader('documents.txt')
documents = loader.load()

# 2. Chunk documents
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,
    chunk_overlap=50
)
texts = text_splitter.split_documents(documents)

# 3. Crear embeddings y vector store
embeddings = OpenAIEmbeddings()
vectorstore = Chroma.from_documents(texts, embeddings)

# 4. Crear retriever
retriever = vectorstore.as_retriever(search_kwargs={"k": 3})

# 5. Crear QA chain
qa_chain = RetrievalQA.from_chain_type(
    llm=OpenAI(temperature=0),
    chain_type="stuff",
    retriever=retriever,
    return_source_documents=True
)

# 6. Query
query = "What is the main topic of the documents?"
result = qa_chain({"query": query})

print(f"Answer: {result['result']}")
print(f"\nSources: {result['source_documents']}")
```

### Ejemplo 2: RAG con Pinecone

```python
import pinecone
from langchain.vectorstores import Pinecone
from langchain.embeddings import OpenAIEmbeddings

# Inicializar Pinecone
pinecone.init(api_key="your-api-key", environment="us-west1-gcp")

# Crear index
index_name = "rag-demo"
if index_name not in pinecone.list_indexes():
    pinecone.create_index(
        name=index_name,
        dimension=1536,  # OpenAI embeddings dimension
        metric="cosine"
    )

# Embeddings
embeddings = OpenAIEmbeddings()

# Vector store
vectorstore = Pinecone.from_documents(
    documents=texts,
    embedding=embeddings,
    index_name=index_name
)

# Search
query = "What is RAG?"
docs = vectorstore.similarity_search(query, k=3)

for doc in docs:
    print(f"Content: {doc.page_content}")
    print(f"Metadata: {doc.metadata}\n")
```

### Ejemplo 3: Custom RAG Pipeline

```python
import openai
from sentence_transformers import SentenceTransformer
import faiss
import numpy as np

class SimpleRAG:
    def __init__(self, documents):
        # Cargar modelo embeddings
        self.encoder = SentenceTransformer('all-MiniLM-L6-v2')
        
        # Procesar documentos
        self.documents = documents
        self.embeddings = self.encoder.encode(documents)
        
        # Crear FAISS index
        dimension = self.embeddings.shape[1]
        self.index = faiss.IndexFlatL2(dimension)
        self.index.add(self.embeddings.astype('float32'))
    
    def retrieve(self, query, k=3):
        # Encode query
        query_embedding = self.encoder.encode([query])
        
        # Search
        distances, indices = self.index.search(
            query_embedding.astype('float32'), k
        )
        
        # Get relevant docs
        relevant_docs = [self.documents[i] for i in indices[0]]
        return relevant_docs, distances[0]
    
    def generate(self, query, context):
        # Construct prompt
        prompt = f"""Answer the question based on the context below.

Context:
{context}

Question: {query}

Answer:"""
        
        # Generate
        response = openai.ChatCompletion.create(
            model="gpt-3.5-turbo",
            messages=[{"role": "user", "content": prompt}],
            temperature=0
        )
        
        return response.choices[0].message.content
    
    def query(self, question, k=3):
        # Retrieve
        docs, scores = self.retrieve(question, k)
        
        # Format context
        context = "\n\n".join(docs)
        
        # Generate
        answer = self.generate(question, context)
        
        return {
            'answer': answer,
            'sources': docs,
            'scores': scores
        }

# Uso
documents = [
    "RAG stands for Retrieval Augmented Generation",
    "Vector databases store embeddings for similarity search",
    "LangChain helps build LLM applications"
]

rag = SimpleRAG(documents)
result = rag.query("What is RAG?")
print(f"Answer: {result['answer']}")
```

### Ejemplo 4: Re-ranking

```python
from sentence_transformers import CrossEncoder

class RAGWithReranking:
    def __init__(self, documents):
        self.encoder = SentenceTransformer('all-MiniLM-L6-v2')
        self.reranker = CrossEncoder('cross-encoder/ms-marco-MiniLM-L-6-v2')
        
        self.documents = documents
        self.embeddings = self.encoder.encode(documents)
        
        # FAISS index
        dimension = self.embeddings.shape[1]
        self.index = faiss.IndexFlatL2(dimension)
        self.index.add(self.embeddings.astype('float32'))
    
    def retrieve_and_rerank(self, query, k_initial=10, k_final=3):
        # Initial retrieval
        query_embedding = self.encoder.encode([query])
        distances, indices = self.index.search(
            query_embedding.astype('float32'), k_initial
        )
        
        candidates = [self.documents[i] for i in indices[0]]
        
        # Re-rank
        pairs = [[query, doc] for doc in candidates]
        scores = self.reranker.predict(pairs)
        
        # Get top k after reranking
        top_indices = np.argsort(scores)[::-1][:k_final]
        reranked_docs = [candidates[i] for i in top_indices]
        reranked_scores = [scores[i] for i in top_indices]
        
        return reranked_docs, reranked_scores

# Uso
rag_rerank = RAGWithReranking(documents)
docs, scores = rag_rerank.retrieve_and_rerank("What is RAG?")
```

### Ejemplo 5: Conversational RAG

```python
from langchain.memory import ConversationBufferMemory
from langchain.chains import ConversationalRetrievalChain

# Memory para mantener contexto
memory = ConversationBufferMemory(
    memory_key="chat_history",
    return_messages=True
)

# Conversational chain
qa = ConversationalRetrievalChain.from_llm(
    llm=OpenAI(temperature=0),
    retriever=vectorstore.as_retriever(),
    memory=memory
)

# Multi-turn conversation
questions = [
    "What is RAG?",
    "How does it work?",
    "What are its benefits?"
]

for question in questions:
    result = qa({"question": question})
    print(f"Q: {question}")
    print(f"A: {result['answer']}\n")
```

## Mejores Prácticas

### 1. Chunk Size Optimization

```python
# ✅ Probar diferentes tamaños
chunk_sizes = [200, 500, 1000]
for size in chunk_sizes:
    splitter = RecursiveCharacterTextSplitter(
        chunk_size=size,
        chunk_overlap=size//10
    )
    # Evaluar performance
```

### 2. Hybrid Search

```python
# ✅ Combinar semantic + keyword
from langchain.retrievers import EnsembleRetriever
from langchain.retrievers import BM25Retriever

# Semantic retriever
semantic_retriever = vectorstore.as_retriever()

# Keyword retriever
bm25_retriever = BM25Retriever.from_documents(documents)

# Ensemble
ensemble_retriever = EnsembleRetriever(
    retrievers=[semantic_retriever, bm25_retriever],
    weights=[0.5, 0.5]
)
```

### 3. Citation Enforcement

```python
# ✅ Prompt para citar fuentes
system_prompt = """Answer based ONLY on the provided context. 
Always cite your sources using [Source: X] format.
If you cannot answer from the context, say "I don't have enough information"."""
```

## Arquitecturas Avanzadas

1. **Self-RAG**: LLM decide cuándo recuperar
2. **CRAG**: Corrective RAG con refinamiento
3. **Graph RAG**: Knowledge graphs + RAG
4. **Agentic RAG**: Agents con tools

## Proyectos Sugeridos

1. **Document Q&A System**
   - PDF ingestion
   - RAG pipeline
   - Web interface (Streamlit)

2. **Code Assistant**
   - Index codebase
   - Semantic code search
   - Generate code from docs

3. **Customer Support Bot**
   - Knowledge base RAG
   - Multi-turn conversations
   - Escalation to human

## Checklist

- [ ] Entender RAG architecture
- [ ] Implementar basic RAG pipeline
- [ ] Usar vector database
- [ ] Aplicar chunking strategies
- [ ] Implement re-ranking
- [ ] Add citation enforcement
- [ ] Evaluate RAG system
- [ ] Deploy en producción

## Próximos Pasos

1. Estudiar **Advanced RAG patterns**
2. Aprender **Graph RAG**
3. Explorar **Agentic RAG**
4. Profundizar en **Evaluation frameworks**
5. Dominar **Production deployment**

---
**Tiempo estimado**: 4-6 semanas
