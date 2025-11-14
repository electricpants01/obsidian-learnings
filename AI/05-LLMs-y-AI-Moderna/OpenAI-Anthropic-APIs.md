# OpenAI y Anthropic APIs

## Descripción

OpenAI (GPT-4, GPT-3.5) y Anthropic (Claude) proveen APIs para integrar LLMs state-of-the-art en aplicaciones. Ventajas: no mantener infraestructura, modelos actualizados, escalabilidad automática. OpenAI: GPT-4 (razonamiento superior), GPT-3.5-turbo (rápido/barato), embeddings, Whisper (speech), DALL-E. Claude: context window largo (100K tokens), mejor en tareas analíticas, más seguro. Ambos: chat completions, streaming, function calling, JSON mode. Costos: por tokens (input/output separados). Rate limits críticos. Embeddings para similarity search. Function calling permite tool use. Vision APIs para imágenes. Crítico: prompt engineering, error handling, cost optimization, security (no exponer API keys).

## Conceptos

### 1. **Chat Completions**
- Messages format (system, user, assistant)
- Temperature, top_p para variabilidad
- max_tokens para límite
- Stream responses

### 2. **Function Calling**
- Define tools/functions
- LLM decide cuándo llamar
- Structured outputs
- Multi-step workflows

### 3. **Embeddings**
- text-embedding-ada-002 (OpenAI)
- Vector representations
- Similarity search
- Clustering, classification

### 4. **Context Management**
- Token counting
- Truncation strategies
- Sliding window
- Summarization for long convs

### 5. **Cost Optimization**
- Cache prompts
- Use GPT-3.5 cuando posible
- Batch requests
- Monitor usage

## Recursos

**Docs**: https://platform.openai.com/docs, https://docs.anthropic.com/
**Pricing**: https://openai.com/pricing

## Ejemplos

### Ejemplo 1: Chat Completion

```python
import openai

openai.api_key = "your-key"

response = openai.ChatCompletion.create(
    model="gpt-4",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Explain quantum computing"}
    ],
    temperature=0.7,
    max_tokens=500
)

print(response.choices[0].message.content)
```

### Ejemplo 2: Streaming

```python
response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[{"role": "user", "content": "Write a story"}],
    stream=True
)

for chunk in response:
    if chunk.choices[0].delta.get("content"):
        print(chunk.choices[0].delta.content, end="")
```

### Ejemplo 3: Function Calling

```python
functions = [
    {
        "name": "get_weather",
        "description": "Get current weather",
        "parameters": {
            "type": "object",
            "properties": {
                "location": {"type": "string"},
                "unit": {"type": "string", "enum": ["celsius", "fahrenheit"]}
            },
            "required": ["location"]
        }
    }
]

response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[{"role": "user", "content": "What's the weather in Boston?"}],
    functions=functions,
    function_call="auto"
)

if response.choices[0].message.get("function_call"):
    function_args = json.loads(response.choices[0].message.function_call.arguments)
    weather = get_weather(**function_args)
```

### Ejemplo 4: Claude API

```python
import anthropic

client = anthropic.Anthropic(api_key="your-key")

message = client.messages.create(
    model="claude-3-opus-20240229",
    max_tokens=1024,
    messages=[
        {"role": "user", "content": "Analyze this data..."}
    ]
)

print(message.content[0].text)
```

### Ejemplo 5: Embeddings

```python
response = openai.Embedding.create(
    model="text-embedding-ada-002",
    input="Your text here"
)

embedding = response['data'][0]['embedding']
# Use for similarity search
```

## Mejores Prácticas

```python
# ✅ Error handling
from openai.error import RateLimitError, APIError

try:
    response = openai.ChatCompletion.create(...)
except RateLimitError:
    # Retry with backoff
    time.sleep(60)
except APIError as e:
    # Log error
    logger.error(f"API error: {e}")

# ✅ Cost tracking
import tiktoken

def count_tokens(text, model="gpt-3.5-turbo"):
    encoding = tiktoken.encoding_for_model(model)
    return len(encoding.encode(text))

# ✅ Secure API keys
# Use environment variables
import os
openai.api_key = os.getenv("OPENAI_API_KEY")
```

## Comparación

| Feature | OpenAI GPT-4 | Claude 3 Opus |
|---------|--------------|---------------|
| Context | 128K tokens | 200K tokens |
| Razonamiento | Excelente | Excelente |
| Velocidad | Medio | Medio |
| Costo | $$$ | $$$ |
| Function calling | Sí | Limitado |

## Proyectos

1. **AI Assistant**
   - Function calling
   - Multi-turn conversation
   - Deploy FastAPI

2. **Document Analysis**
   - Claude para textos largos
   - Summarization
   - Q&A system

3. **Semantic Search**
   - OpenAI embeddings
   - Vector DB
   - Search interface

---
**Tiempo**: 2-3 semanas
