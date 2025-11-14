# Hugging Face - Ecosistema de LLMs

## Descripción

Hugging Face es el GitHub de ML, el hub central para modelos, datasets y spaces pre-entrenados. Proporciona la biblioteca Transformers (más usada para NLP), Model Hub (200K+ modelos), Datasets library, Inference API, y Spaces para demos. Democratiza acceso a LLMs state-of-the-art: BERT, GPT, T5, LLaMA, Mistral disponibles con 3 líneas de código. Ventajas: no entrenar desde cero (costoso), fine-tuning simplificado, comunidad activa, integraciones (PyTorch, TensorFlow, JAX). Aplicaciones: cualquier tarea NLP/CV. AutoModel detecta tarea automáticamente, pipelines simplifican uso, Trainer API facilita fine-tuning. Accelerate para training distribuido, PEFT para fine-tuning eficiente (LoRA, QLoRA). Critical para producción: Inference Endpoints, optimización con ONNX. Es estándar de la industria para LLMs.

## Conceptos Clave

### 1. **Transformers Library**
- AutoModel, AutoTokenizer
- Pipelines para tareas comunes
- from_pretrained() para modelos
- Compatible PyTorch/TensorFlow

### 2. **Model Hub**
- 200K+ modelos pre-entrenados
- Community contributions
- Model cards con documentación
- Versionado con Git

### 3. **Datasets Library**
- 50K+ datasets
- Streaming grandes datasets
- map() para procesamiento
- Arrow backend eficiente

### 4. **Trainer API**
- Simplifica training loop
- Logging, checkpointing automático
- Integration con TensorBoard/W&B
- Distributed training

### 5. **PEFT (Parameter-Efficient Fine-Tuning)**
- LoRA: Low-Rank Adaptation
- QLoRA: Quantized LoRA
- Adapter layers
- Reduce memory requirements

## Recursos

### Documentación
1. **Hugging Face Docs** - https://huggingface.co/docs
2. **Transformers Course** - https://huggingface.co/learn/nlp-course (Gratis)
3. **PEFT Docs** - https://huggingface.co/docs/peft

### Tutoriales
1. **Official YouTube** - https://www.youtube.com/@HuggingFace

## Ejemplos

### Ejemplo 1: Text Classification Pipeline

```python
from transformers import pipeline

# Pipeline automático
classifier = pipeline("sentiment-analysis")

result = classifier("I love Hugging Face!")
print(result)  # [{'label': 'POSITIVE', 'score': 0.9998}]

# Batch processing
texts = ["Great!", "Terrible!", "OK"]
results = classifier(texts)
```

### Ejemplo 2: Load y Fine-tune BERT

```python
from transformers import AutoModelForSequenceClassification, AutoTokenizer, Trainer, TrainingArguments
from datasets import load_dataset

# Load model y tokenizer
model = AutoModelForSequenceClassification.from_pretrained("bert-base-uncased", num_labels=2)
tokenizer = AutoTokenizer.from_pretrained("bert-base-uncased")

# Load dataset
dataset = load_dataset("imdb")

# Tokenize
def tokenize_function(examples):
    return tokenizer(examples["text"], padding="max_length", truncation=True)

tokenized_datasets = dataset.map(tokenize_function, batched=True)

# Training arguments
training_args = TrainingArguments(
    output_dir="./results",
    evaluation_strategy="epoch",
    learning_rate=2e-5,
    per_device_train_batch_size=8,
    num_train_epochs=3,
)

# Trainer
trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=tokenized_datasets["train"],
    eval_dataset=tokenized_datasets["test"],
)

# Train
trainer.train()
```

### Ejemplo 3: Text Generation

```python
from transformers import AutoModelForCausalLM, AutoTokenizer

model = AutoModelForCausalLM.from_pretrained("gpt2")
tokenizer = AutoTokenizer.from_pretrained("gpt2")

prompt = "Artificial intelligence will"
inputs = tokenizer(prompt, return_tensors="pt")

outputs = model.generate(
    **inputs,
    max_length=100,
    num_return_sequences=3,
    temperature=0.8,
    top_p=0.9
)

for i, output in enumerate(outputs):
    print(f"Generation {i+1}: {tokenizer.decode(output)}")
```

### Ejemplo 4: LoRA Fine-tuning

```python
from peft import LoraConfig, get_peft_model, TaskType

# Base model
base_model = AutoModelForCausalLM.from_pretrained("gpt2")

# LoRA config
lora_config = LoraConfig(
    task_type=TaskType.CAUSAL_LM,
    r=8,  # rank
    lora_alpha=32,
    lora_dropout=0.1,
    target_modules=["c_attn"]  # modules to apply LoRA
)

# Get PEFT model
model = get_peft_model(base_model, lora_config)

# Only ~1% parameters trainable
model.print_trainable_parameters()

# Train como normal con Trainer
```

### Ejemplo 5: Upload Model al Hub

```python
from huggingface_hub import login, HfApi

# Login
login(token="your_token")

# Save model locally
model.save_pretrained("./my-model")
tokenizer.save_pretrained("./my-model")

# Push to hub
model.push_to_hub("username/my-model")
tokenizer.push_to_hub("username/my-model")

# Or use API
api = HfApi()
api.upload_folder(
    folder_path="./my-model",
    repo_id="username/my-model",
    repo_type="model",
)
```

## Mejores Prácticas

```python
# ✅ Use pipelines para prototyping rápido
classifier = pipeline("text-classification", model="bert-base-uncased")

# ✅ Cache models localmente
model = AutoModel.from_pretrained("model-name", cache_dir="./cache")

# ✅ Use AutoClasses
from transformers import AutoTokenizer, AutoModel

# ❌ No hardcodear model class
from transformers import BertModel  # Menos flexible
```

## Proyectos Sugeridos

1. **Fine-tune para Domain Specific**
   - Medical/Legal text classification
   - Custom NER
   - Deploy en Hugging Face Spaces

2. **LoRA Fine-tuning de LLaMA**
   - QLoRA para 4-bit
   - Instruction tuning
   - Comparar con base model

3. **Multi-modal Application**
   - CLIP para image-text
   - Deploy inference endpoint
   - Build Gradio demo

## Checklist

- [ ] Dominar transformers library
- [ ] Usar pipelines efectivamente
- [ ] Fine-tune modelo con Trainer
- [ ] Aplicar LoRA/QLoRA
- [ ] Upload modelo al Hub
- [ ] Deploy inference endpoint
- [ ] Build Gradio app

---
**Tiempo**: 3-4 semanas
