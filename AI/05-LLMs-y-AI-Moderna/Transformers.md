# Transformers - Arquitectura Revolucionaria

## Descripción

Transformers (Vaswani et al., 2017 "Attention Is All You Need") revolucionaron NLP y ahora dominan toda IA. A diferencia de RNNs que procesan secuencias secuencialmente, Transformers procesan toda la secuencia en paralelo usando self-attention para capturar dependencias de largo alcance. La arquitectura encoder-decoder usa multi-head attention, positional encoding, layer normalization y residual connections. BERT (2018) popularizó encoder-only para tareas de comprensión, GPT demostró decoder-only para generación, T5 unificó todo en texto-a-texto. Ventajas: paralelización completa (training mucho más rápido), captura dependencias largas, transferible a múltiples dominios. Ahora dominan: NLP (ChatGPT, Claude), CV (ViT), speech (Whisper), proteínas (AlphaFold). Conceptos: self-attention (Q,K,V), multi-head, positional encoding, feed-forward, layer norm. Hugging Face proporciona miles de modelos pre-entrenados. Son la base de LLMs modernos (GPT-4, Claude, Llama).

## Conceptos Clave

### 1. **Self-Attention**
- Query, Key, Value matrices
- Attention scores = softmax(QK^T/√d_k)
- Output = Attention × V
- Captura relaciones entre tokens

### 2. **Multi-Head Attention**
- Múltiples attention heads en paralelo
- Diferentes representaciones
- Concatenar y proyectar
- Típico: 8-16 heads

### 3. **Positional Encoding**
- Sin recurrencia, necesita posición
- Sinusoidal encoding
- Learned positional embeddings
- Relative position encoding

### 4. **Encoder-Decoder**
- Encoder: bidirectional context
- Decoder: autoregressive generation
- Cross-attention entre encoder-decoder
- BERT (encoder), GPT (decoder)

### 5. **Feed-Forward Networks**
- Dos capas lineales con ReLU
- Aplicado independientemente
- Expand-compress (típico 4x)
- Layer normalization

## Recursos

### Papers
1. **Attention Is All You Need** - Vaswani et al. (2017)
   - Paper original, fundamental

2. **BERT** - Devlin et al. (2018)
   - Bidirectional encoder

3. **GPT-3** - Brown et al. (2020)
   - Large language models

### Cursos
1. **CS224N: NLP with Deep Learning**
   - http://web.stanford.edu/class/cs224n/

2. **Hugging Face Course**
   - https://huggingface.co/learn/nlp-course
   - Gratis, muy práctico

### Libros
1. **"Natural Language Processing with Transformers"**
   - Lewis Tunstall et al., O'Reilly

## Ejemplos Prácticos

### Ejemplo 1: Self-Attention desde Cero

```python
import torch
import torch.nn as nn
import math

class SelfAttention(nn.Module):
    def __init__(self, embed_dim, num_heads):
        super().__init__()
        self.embed_dim = embed_dim
        self.num_heads = num_heads
        self.head_dim = embed_dim // num_heads
        
        assert embed_dim % num_heads == 0
        
        self.qkv = nn.Linear(embed_dim, embed_dim * 3)
        self.proj = nn.Linear(embed_dim, embed_dim)
    
    def forward(self, x):
        batch_size, seq_len, embed_dim = x.shape
        
        # QKV projection
        qkv = self.qkv(x).reshape(batch_size, seq_len, 3, self.num_heads, self.head_dim)
        qkv = qkv.permute(2, 0, 3, 1, 4)  # (3, batch, heads, seq, head_dim)
        q, k, v = qkv[0], qkv[1], qkv[2]
        
        # Attention scores
        scores = torch.matmul(q, k.transpose(-2, -1)) / math.sqrt(self.head_dim)
        attn = torch.softmax(scores, dim=-1)
        
        # Apply attention to values
        out = torch.matmul(attn, v)
        out = out.transpose(1, 2).reshape(batch_size, seq_len, embed_dim)
        
        return self.proj(out)

# Uso
attention = SelfAttention(embed_dim=512, num_heads=8)
x = torch.randn(32, 10, 512)  # batch=32, seq=10, dim=512
output = attention(x)
print(output.shape)  # torch.Size([32, 10, 512])
```

### Ejemplo 2: Transformer Block

```python
class TransformerBlock(nn.Module):
    def __init__(self, embed_dim, num_heads, ff_dim, dropout=0.1):
        super().__init__()
        
        # Multi-head attention
        self.attention = nn.MultiheadAttention(embed_dim, num_heads, dropout=dropout)
        
        # Feed-forward
        self.ff = nn.Sequential(
            nn.Linear(embed_dim, ff_dim),
            nn.ReLU(),
            nn.Dropout(dropout),
            nn.Linear(ff_dim, embed_dim)
        )
        
        # Layer normalization
        self.norm1 = nn.LayerNorm(embed_dim)
        self.norm2 = nn.LayerNorm(embed_dim)
        
        self.dropout = nn.Dropout(dropout)
    
    def forward(self, x):
        # Self-attention + residual
        attn_out, _ = self.attention(x, x, x)
        x = self.norm1(x + self.dropout(attn_out))
        
        # Feed-forward + residual
        ff_out = self.ff(x)
        x = self.norm2(x + self.dropout(ff_out))
        
        return x
```

### Ejemplo 3: BERT con Hugging Face

```python
from transformers import BertTokenizer, BertModel

# Cargar modelo pre-entrenado
tokenizer = BertTokenizer.from_pretrained('bert-base-uncased')
model = BertModel.from_pretrained('bert-base-uncased')

# Tokenizar texto
text = "Transformers are amazing for NLP tasks"
inputs = tokenizer(text, return_tensors='pt', padding=True, truncation=True)

# Forward pass
with torch.no_grad():
    outputs = model(**inputs)

# Obtener embeddings
last_hidden_states = outputs.last_hidden_state  # [batch, seq_len, hidden_dim]
pooled_output = outputs.pooler_output  # [batch, hidden_dim]

print(f"Hidden states shape: {last_hidden_states.shape}")
print(f"Pooled output shape: {pooled_output.shape}")
```

### Ejemplo 4: GPT-2 Text Generation

```python
from transformers import GPT2LMHeadModel, GPT2Tokenizer

# Cargar modelo
tokenizer = GPT2Tokenizer.from_pretrained('gpt2')
model = GPT2LMHeadModel.from_pretrained('gpt2')

# Prompt
prompt = "Artificial intelligence will"
input_ids = tokenizer.encode(prompt, return_tensors='pt')

# Generar
output = model.generate(
    input_ids,
    max_length=100,
    num_return_sequences=3,
    temperature=0.7,
    top_k=50,
    top_p=0.95,
    do_sample=True
)

# Decode
for i, seq in enumerate(output):
    generated_text = tokenizer.decode(seq, skip_special_tokens=True)
    print(f"\nGeneration {i+1}:\n{generated_text}")
```

### Ejemplo 5: Fine-tuning para Clasificación

```python
from transformers import AutoModelForSequenceClassification, Trainer, TrainingArguments

# Cargar modelo para clasificación
model = AutoModelForSequenceClassification.from_pretrained(
    'bert-base-uncased',
    num_labels=2
)

# Training arguments
training_args = TrainingArguments(
    output_dir='./results',
    num_train_epochs=3,
    per_device_train_batch_size=16,
    warmup_steps=500,
    weight_decay=0.01,
    logging_dir='./logs',
    evaluation_strategy="epoch"
)

# Trainer
trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=train_dataset,
    eval_dataset=eval_dataset
)

# Train
trainer.train()

# Evaluate
results = trainer.evaluate()
print(results)
```

## Aplicaciones

1. **Text Classification**: BERT fine-tuning
2. **Text Generation**: GPT-based models
3. **Translation**: T5, mBART
4. **Summarization**: BART, Pegasus
5. **Question Answering**: BERT, RoBERTa
6. **Computer Vision**: ViT, CLIP
7. **Multimodal**: CLIP, Flamingo

## Mejores Prácticas

### 1. Use Pre-trained Models

```python
# ✅ Partir de pre-trained
from transformers import AutoModel
model = AutoModel.from_pretrained('bert-base-uncased')

# ❌ Entrenar desde cero (muy costoso)
```

### 2. Warmup Learning Rate

```python
# ✅ Warmup para Transformers
from transformers import get_linear_schedule_with_warmup

scheduler = get_linear_schedule_with_warmup(
    optimizer,
    num_warmup_steps=500,
    num_training_steps=total_steps
)
```

### 3. Gradient Checkpointing para Memoria

```python
# ✅ Cuando memoria limitada
model.gradient_checkpointing_enable()
```

## Proyectos Sugeridos

1. **Sentiment Analysis Fine-tuning**
   - BERT en IMDb reviews
   - Comparar con baseline
   - Deploy API

2. **Text Summarization**
   - Fine-tune BART/T5
   - News articles dataset
   - ROUGE evaluation

3. **Question Answering System**
   - SQuAD dataset
   - BERT-based QA
   - Web interface

4. **Custom Language Model**
   - Train pequeño GPT desde cero
   - Domain-specific corpus
   - Text generation

## Checklist

- [ ] Entender self-attention mechanism
- [ ] Implementar transformer block desde cero
- [ ] Usar Hugging Face library
- [ ] Fine-tune BERT para tarea específica
- [ ] Generar texto con GPT
- [ ] Optimizar para producción
- [ ] Aplicar en proyecto real

## Próximos Pasos

1. Dominar **Hugging Face Ecosystem**
2. Estudiar **Prompt Engineering**
3. Aprender **RAG (Retrieval Augmented Generation)**
4. Explorar **LoRA/QLoRA fine-tuning**
5. Profundizar en **LLM Deployment**

---
**Tiempo estimado**: 6-8 semanas
