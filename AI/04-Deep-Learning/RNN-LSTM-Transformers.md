# RNN, LSTM y Transformers - Secuencias y Atención

## Descripción

Las arquitecturas para datos secuenciales evolucionaron drásticamente: RNNs (1980s) procesan secuencias con memoria pero sufren vanishing gradients, LSTMs (1997) solucionaron esto con gates permitiendo memoria a largo plazo, GRUs simplificaron LSTMs, y Transformers (2017) revolucionaron todo eliminando recurrencia usando self-attention. RNNs son feedforward networks con conexiones recurrentes que mantienen estado oculto, pero training es difícil en secuencias largas. LSTMs usan gates (input, forget, output) para controlar flujo de información resolviendo vanishing gradients. Transformers usan multi-head attention procesando toda la secuencia en paralelo, siendo mucho más eficientes y escalables. Aplicaciones: NLP (traducción, generación texto, chatbots), series temporales, speech recognition, video analysis. Transformers dominan actualmente: BERT, GPT, T5, todos basados en atención. Conceptos: hidden states, gates, attention mechanisms, positional encoding, encoder-decoder. Son fundamentales para IA moderna.

## Conceptos Clave

### 1. **RNN Básicas**
- Hidden state: memoria de secuencia
- Backpropagation Through Time (BPTT)
- Vanishing/Exploding gradients
- Aplicaciones limitadas por memoria corta

### 2. **LSTM (Long Short-Term Memory)**
- Cell state: memoria a largo plazo
- Gates: forget, input, output
- Resuelve vanishing gradients
- Más parámetros que RNN simple

### 3. **GRU (Gated Recurrent Unit)**
- Simplificación de LSTM
- Solo 2 gates: reset, update
- Menos parámetros, similar performance
- Más rápido de entrenar

### 4. **Attention Mechanism**
- Enfoque en partes relevantes
- Query, Key, Value
- Attention scores (softmax)
- Seq2Seq con atención

### 5. **Transformers**
- Self-attention multi-head
- Positional encoding
- Encoder-decoder architecture
- Paralelización completa

### 6. **Aplicaciones**
- Machine translation
- Text generation
- Time series forecasting
- Speech recognition
- Video understanding

## Recursos de Aprendizaje

### Cursos
1. **Sequence Models - Coursera** (Andrew Ng)
   - https://www.coursera.org/learn/nlp-sequence-models

2. **CS224n: NLP with Deep Learning - Stanford**
   - http://web.stanford.edu/class/cs224n/

### Papers Clave
1. **LSTM** - Hochreiter & Schmidhuber (1997)
2. **Attention Is All You Need** - Vaswani et al. (2017)
3. **BERT** - Devlin et al. (2018)
4. **GPT** - Radford et al. (2018-2023)

### Libros
1. **"Speech and Language Processing"** - Jurafsky & Martin
   - Gratis: https://web.stanford.edu/~jurafsky/slp3/

## Ejemplos Prácticos

### Ejemplo 1: RNN Simple

```python
import torch
import torch.nn as nn

class SimpleRNN(nn.Module):
    def __init__(self, input_size, hidden_size, output_size):
        super().__init__()
        self.hidden_size = hidden_size
        self.rnn = nn.RNN(input_size, hidden_size, batch_first=True)
        self.fc = nn.Linear(hidden_size, output_size)
    
    def forward(self, x):
        # x: (batch, seq_len, input_size)
        h0 = torch.zeros(1, x.size(0), self.hidden_size)
        out, hn = self.rnn(x, h0)
        # out: (batch, seq_len, hidden_size)
        out = self.fc(out[:, -1, :])  # Last time step
        return out

# Uso
model = SimpleRNN(input_size=10, hidden_size=20, output_size=5)
x = torch.randn(32, 15, 10)  # batch=32, seq=15, features=10
output = model(x)
print(output.shape)  # [32, 5]
```

### Ejemplo 2: LSTM para Predicción

```python
class LSTMPredictor(nn.Module):
    def __init__(self, input_dim, hidden_dim, num_layers, output_dim):
        super().__init__()
        self.hidden_dim = hidden_dim
        self.num_layers = num_layers
        
        self.lstm = nn.LSTM(input_dim, hidden_dim, num_layers, 
                           batch_first=True, dropout=0.2)
        self.fc = nn.Linear(hidden_dim, output_dim)
    
    def forward(self, x):
        h0 = torch.zeros(self.num_layers, x.size(0), self.hidden_dim)
        c0 = torch.zeros(self.num_layers, x.size(0), self.hidden_dim)
        
        out, (hn, cn) = self.lstm(x, (h0, c0))
        out = self.fc(out[:, -1, :])
        return out

# Time series prediction
model = LSTMPredictor(input_dim=1, hidden_dim=64, 
                      num_layers=2, output_dim=1)
```

### Ejemplo 3: Attention Mechanism

```python
class AttentionLayer(nn.Module):
    def __init__(self, hidden_dim):
        super().__init__()
        self.W = nn.Linear(hidden_dim, hidden_dim)
        self.V = nn.Linear(hidden_dim, 1)
    
    def forward(self, encoder_outputs):
        # encoder_outputs: (batch, seq_len, hidden_dim)
        
        # Compute attention scores
        scores = self.V(torch.tanh(self.W(encoder_outputs)))
        # scores: (batch, seq_len, 1)
        
        # Apply softmax
        attention_weights = torch.softmax(scores, dim=1)
        
        # Weighted sum
        context = torch.sum(attention_weights * encoder_outputs, dim=1)
        # context: (batch, hidden_dim)
        
        return context, attention_weights

# Uso
attention = AttentionLayer(hidden_dim=128)
encoder_out = torch.randn(32, 10, 128)
context, weights = attention(encoder_out)
```

### Ejemplo 4: Transformer Block Simplificado

```python
class TransformerBlock(nn.Module):
    def __init__(self, embed_dim, num_heads, ff_dim):
        super().__init__()
        self.attention = nn.MultiheadAttention(embed_dim, num_heads)
        self.norm1 = nn.LayerNorm(embed_dim)
        self.norm2 = nn.LayerNorm(embed_dim)
        
        self.ff = nn.Sequential(
            nn.Linear(embed_dim, ff_dim),
            nn.ReLU(),
            nn.Linear(ff_dim, embed_dim)
        )
    
    def forward(self, x):
        # Self-attention
        attn_out, _ = self.attention(x, x, x)
        x = self.norm1(x + attn_out)  # Residual
        
        # Feed-forward
        ff_out = self.ff(x)
        x = self.norm2(x + ff_out)  # Residual
        
        return x

# Uso
block = TransformerBlock(embed_dim=512, num_heads=8, ff_dim=2048)
x = torch.randn(10, 32, 512)  # (seq_len, batch, embed_dim)
output = block(x)
```

### Ejemplo 5: Text Generation con LSTM

```python
class CharRNN(nn.Module):
    def __init__(self, vocab_size, embed_dim, hidden_dim):
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, embed_dim)
        self.lstm = nn.LSTM(embed_dim, hidden_dim, batch_first=True)
        self.fc = nn.Linear(hidden_dim, vocab_size)
    
    def forward(self, x, hidden=None):
        x = self.embedding(x)
        out, hidden = self.lstm(x, hidden)
        out = self.fc(out)
        return out, hidden
    
    def generate(self, start_char, length=100, temperature=1.0):
        self.eval()
        chars = [start_char]
        hidden = None
        
        with torch.no_grad():
            for _ in range(length):
                x = torch.tensor([[chars[-1]]])
                out, hidden = self.forward(x, hidden)
                
                # Apply temperature
                probs = torch.softmax(out[0, -1] / temperature, dim=0)
                next_char = torch.multinomial(probs, 1).item()
                chars.append(next_char)
        
        return chars

# Training loop simplificado
model = CharRNN(vocab_size=100, embed_dim=128, hidden_dim=256)
optimizer = torch.optim.Adam(model.parameters())
criterion = nn.CrossEntropyLoss()

for epoch in range(num_epochs):
    for batch in dataloader:
        inputs, targets = batch
        
        outputs, _ = model(inputs)
        loss = criterion(outputs.view(-1, vocab_size), targets.view(-1))
        
        optimizer.zero_grad()
        loss.backward()
        torch.nn.utils.clip_grad_norm_(model.parameters(), 1.0)
        optimizer.step()
```

## Mejores Prácticas

### 1. Gradient Clipping

```python
# ✅ Prevenir exploding gradients
torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)
```

### 2. Bidirectional RNN para Mejor Contexto

```python
# ✅ Mejor para tareas donde el contexto futuro importa
self.lstm = nn.LSTM(input_size, hidden_size, bidirectional=True)
```

### 3. Teacher Forcing en Seq2Seq

```python
# ✅ Durante training
use_teacher_forcing = random.random() < teacher_forcing_ratio
```

## Proyectos Sugeridos

1. **Sentiment Analysis**
   - LSTM para clasificar texto
   - IMDb reviews dataset
   - Comparar con BERT

2. **Machine Translation**
   - Seq2Seq con attention
   - Transformer implementation
   - BLEU score evaluation

3. **Time Series Forecasting**
   - Stock price prediction
   - LSTM vs Transformer
   - Multi-step ahead

4. **Text Generation**
   - Character-level LSTM
   - Shakespeare text generation
   - Temperature sampling

5. **Named Entity Recognition**
   - Bi-LSTM + CRF
   - CoNLL dataset
   - F1 score evaluation

## Checklist de Aprendizaje

- [ ] Entender backpropagation through time
- [ ] Implementar RNN desde cero
- [ ] Dominar LSTM gates
- [ ] Aplicar attention mechanisms
- [ ] Usar Transformers (Hugging Face)
- [ ] Manejar secuencias variable-length
- [ ] Optimizar seq2seq models
- [ ] Fine-tune pre-trained transformers

## Próximos Pasos

1. Dominar **Hugging Face Transformers**
2. Estudiar **BERT, GPT architectures**
3. Aprender **Prompt Engineering**
4. Explorar **LoRA, QLoRA fine-tuning**
5. Profundizar en **RAG systems**

---
**Tiempo estimado**: 5-6 semanas
