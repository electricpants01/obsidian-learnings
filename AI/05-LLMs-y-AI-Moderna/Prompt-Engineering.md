# Prompt Engineering - Diseño de Prompts Efectivos

## Descripción

Prompt Engineering es el arte y ciencia de diseñar instrucciones (prompts) efectivas para obtener los mejores resultados de Large Language Models (LLMs). Es una habilidad esencial en la era de ChatGPT, GPT-4, Claude y otros LLMs, permitiendo maximizar su utilidad sin necesidad de fine-tuning.

## Conceptos Clave

### 1. **Anatomía de un Prompt**
- Instrucciones claras
- Contexto relevante
- Ejemplos (few-shot)
- Formato de salida esperado

### 2. **Técnicas Fundamentales**
- Zero-shot prompting
- Few-shot learning
- Chain-of-Thought (CoT)
- Tree of Thoughts

### 3. **Optimización**
- Temperatura y top-p
- System messages
- Delimitadores
- Role prompting

### 4. **Patrones Avanzados**
- ReAct (Reasoning + Acting)
- Self-consistency
- Constitutional AI
- Prompt chaining

## Recursos de Aprendizaje

### Cursos y Guías

1. **OpenAI Prompt Engineering Guide**
   - https://platform.openai.com/docs/guides/prompt-engineering
   
2. **Prompt Engineering Guide by DAIR.AI**
   - https://www.promptingguide.ai/
   
3. **Learn Prompting**
   - https://learnprompting.org/

### Papers Importantes

1. **"Chain-of-Thought Prompting"** - Wei et al. (2022)
   - https://arxiv.org/abs/2201.11903

2. **"ReAct: Synergizing Reasoning and Acting"**
   - https://arxiv.org/abs/2210.03629

## Técnicas Prácticas

### Zero-Shot vs Few-Shot

```python
# Zero-shot
prompt = """
Clasifica el sentimiento del siguiente texto:
"¡Me encantó la película!"
"""

# Few-shot
prompt = """
Clasifica el sentimiento de los siguientes textos:

Texto: "Excelente producto"
Sentimiento: Positivo

Texto: "Horrible experiencia"
Sentimiento: Negativo

Texto: "¡Me encantó la película!"
Sentimiento:
"""
```

### Chain-of-Thought

```python
prompt = """
Resuelve el problema paso a paso:

Problema: María tiene 3 manzanas. Compra 5 más y luego regala 2. 
¿Cuántas manzanas tiene ahora?

Razonamiento paso a paso:
1. María comienza con 3 manzanas
2. Compra 5 más: 3 + 5 = 8
3. Regala 2: 8 - 2 = 6

Respuesta: 6 manzanas
"""
```

### Formato de Salida

```python
prompt = """
Analiza la siguiente reseña y devuelve un JSON:

Reseña: "El producto es bueno pero caro"

Formato:
{
    "sentimiento": "positivo/negativo/neutro",
    "aspectos": ["calidad", "precio"],
    "puntuación": 1-5
}
"""
```

## Mejores Prácticas

### 1. Ser Específico

```python
# ❌ Vago
"Escribe sobre IA"

# ✅ Específico
"Escribe un párrafo de 100 palabras explicando transformers en IA 
para principiantes, usando analogías simples"
```

### 2. Usar Delimitadores

```python
prompt = """
Resume el siguiente texto entre comillas triples:

'''
[Texto largo aquí]
'''

Resume en 3 puntos clave.
"""
```

### 3. Especificar Formato

```python
prompt = """
Lista los beneficios del ejercicio.

Formato:
- Beneficio 1: [descripción]
- Beneficio 2: [descripción]
...
"""
```

## Patrones Avanzados

### ReAct Pattern

```python
prompt = """
Actúa como un asistente que piensa en voz alta:

Pregunta: ¿Cuál es la capital de Francia y su población?

Thought: Necesito buscar la capital de Francia
Action: search("capital de Francia")
Observation: París

Thought: Ahora necesito la población
Action: search("población de París")
Observation: ~2.2 millones

Answer: La capital de Francia es París con aproximadamente 2.2 millones 
de habitantes
"""
```

### Prompt Chaining

```python
# Paso 1: Generar ideas
prompt1 = "Genera 5 temas para un blog sobre IA"

# Paso 2: Expandir el mejor tema
prompt2 = f"Del siguiente tema: {mejor_tema}, crea un outline detallado"

# Paso 3: Escribir sección
prompt3 = f"Escribe la introducción para: {outline}"
```

## Herramientas

1. **LangChain** - Framework para aplicaciones con LLMs
2. **Promptfoo** - Testing de prompts
3. **OpenPrompt** - Biblioteca de prompts
4. **GPT Prompt Engineer** - Auto-optimización

## Proyectos Sugeridos

1. **Sistema de Q&A sobre documentos**
2. **Generador de código con explicaciones**
3. **Asistente de análisis de datos**
4. **Creador de contenido estructurado**
5. **Evaluador automático de respuestas**

## Checklist de Aprendizaje

- [ ] Dominar zero-shot y few-shot
- [ ] Aplicar Chain-of-Thought
- [ ] Controlar temperatura y parámetros
- [ ] Diseñar prompts estructurados
- [ ] Implementar prompt chaining
- [ ] Evaluar calidad de prompts
- [ ] Crear biblioteca de prompts reutilizables

---
**Tiempo estimado**: 2-3 semanas
