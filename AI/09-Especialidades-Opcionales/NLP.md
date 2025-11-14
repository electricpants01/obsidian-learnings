# NLP - Natural Language Processing

## Descripción
NLP procesa lenguaje humano: sentiment analysis, NER, machine translation, text generation. Técnicas clásicas: tokenization, stemming, TF-IDF. Deep Learning: RNNs, Transformers (BERT, GPT). Libraries: NLTK, spaCy, Transformers. Casos: chatbots, search, summarization.

## Ejemplo
```python
from transformers import pipeline
classifier = pipeline("sentiment-analysis")
result = classifier("I love this product!")
# [{'label': 'POSITIVE', 'score': 0.9998}]
```
---
**Tiempo**: 4-6 semanas
