# LangChain y LlamaIndex - Frameworks para LLM Apps

## Descripción

LangChain y LlamaIndex son frameworks para construir aplicaciones con LLMs. LangChain (más general) proporciona chains, agents, memory para orquestar LLMs con tools externos. LlamaIndex (especializado en RAG) optimiza indexing y retrieval de datos. Ambos simplifican: conectar LLMs con datos, crear workflows complejos, manejar memoria conversacional, integrar tools (APIs, databases). LangChain: modular (chains componibles), agents (reasoning loops), memory (contexto conversacional). LlamaIndex: indexing eficiente, query engines avanzados, optimizado para documentos. Casos de uso: chatbots, Q&A systems, agents autónomos, RAG pipelines. Production-ready con observability (LangSmith), deployment (LangServe). Son estándares para LLM apps.

## Conceptos Clave

### 1. **Chains (LangChain)**
- Sequences de llamadas
- LLMChain, SequentialChain
- Componibles y reutilizables
- LCEL (LangChain Expression Language)

### 2. **Agents (LangChain)**
- Reasoning loops
- Tool use (ReAct pattern)
- Autonomous decision making
- Built-in agents (Zero-shot, Conversational)

### 3. **Memory**
- Conversation history
- Buffer, Summary, Entity memory
- Vector store memory
- Window memory

### 4. **Indexes (LlamaIndex)**
- VectorStoreIndex, ListIndex
- Tree indexes
- Keyword Table Index
- Graph indexes

### 5. **Query Engines (LlamaIndex)**
- Retrieval strategies
- Response synthesizers
- Router query engines
- Sub-question query engines

## Recursos

**LangChain**: https://python.langchain.com/docs/
**LlamaIndex**: https://docs.llamaindex.ai/
**Curso DeepLearning.AI**: https://www.deeplearning.ai/short-courses/

## Ejemplos

### Ejemplo 1: Simple Chain

```python
from langchain.llms import OpenAI
from langchain.prompts import PromptTemplate
from langchain.chains import LLMChain

llm = OpenAI(temperature=0.7)

prompt = PromptTemplate(
    input_variables=["product"],
    template="What is a good name for a company that makes {product}?"
)

chain = LLMChain(llm=llm, prompt=prompt)
result = chain.run("eco-friendly water bottles")
print(result)
```

### Ejemplo 2: Agent con Tools

```python
from langchain.agents import load_tools, initialize_agent, AgentType
from langchain.llms import OpenAI

llm = OpenAI(temperature=0)

tools = load_tools(["serpapi", "llm-math"], llm=llm)

agent = initialize_agent(
    tools,
    llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True
)

agent.run("What was the high temperature in SF yesterday? What is that number raised to the .023 power?")
```

### Ejemplo 3: Conversational Memory

```python
from langchain.memory import ConversationBufferMemory
from langchain.chains import ConversationChain

memory = ConversationBufferMemory()

conversation = ConversationChain(
    llm=OpenAI(),
    memory=memory,
    verbose=True
)

conversation.predict(input="Hi, my name is Alice")
conversation.predict(input="What is my name?")
```

### Ejemplo 4: LlamaIndex RAG

```python
from llama_index import VectorStoreIndex, SimpleDirectoryReader

# Load documents
documents = SimpleDirectoryReader('data').load_data()

# Create index
index = VectorStoreIndex.from_documents(documents)

# Query
query_engine = index.as_query_engine()
response = query_engine.query("What is the main topic?")
print(response)
```

### Ejemplo 5: Custom Tool

```python
from langchain.tools import Tool
from langchain.agents import initialize_agent

def get_word_length(word: str) -> int:
    """Returns the length of a word."""
    return len(word)

tools = [
    Tool(
        name="Word Length",
        func=get_word_length,
        description="Useful for getting the length of a word"
    )
]

agent = initialize_agent(tools, llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION)
agent.run("What is the length of the word 'artificial'?")
```

## LangChain vs LlamaIndex

| Feature | LangChain | LlamaIndex |
|---------|-----------|------------|
| Foco | General LLM apps | RAG optimizado |
| Agents | Sí, robusto | Limitado |
| RAG | Bueno | Excelente |
| Indexing | Básico | Avanzado |
| Learning Curve | Medio | Más simple |

## Proyectos

1. **Chatbot con Memory**
   - Conversational agent
   - Tools integration
   - Deploy con LangServe

2. **RAG System**
   - LlamaIndex para indexing
   - Advanced retrieval
   - Streamlit UI

3. **Autonomous Agent**
   - Multi-step reasoning
   - External APIs
   - Error handling

## Checklist

- [ ] Dominar chains básicos
- [ ] Crear agents con tools
- [ ] Implementar memory
- [ ] Usar LlamaIndex para RAG
- [ ] Build production app
- [ ] Deploy con LangServe

---
**Tiempo**: 3-4 semanas
