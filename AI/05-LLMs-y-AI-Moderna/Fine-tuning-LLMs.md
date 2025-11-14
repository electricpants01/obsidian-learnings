# Fine-tuning de LLMs

## Descripción

Fine-tuning adapta LLMs pre-entrenados a tareas específicas usando datos propios. Más efectivo que prompting para: dominios especializados (legal, médico), estilos específicos, tareas complejas, reducir tokens en producción. Métodos: full fine-tuning (ajustar todos parámetros, costoso), LoRA (Low-Rank Adaptation, eficiente), QLoRA (quantized LoRA, 4-bit), adapter layers. Ventajas vs prompting: mejor performance, menos tokens, no expone data en prompts. Desventajas: requiere datos, riesgo catastrophic forgetting, costoso mantener. Aplicaciones: chatbots empresariales, code generation, domain-specific Q&A. Plataformas: OpenAI fine-tuning API, Hugging Face PEFT, Axolotl. Datasets: instruction following (Alpaca format), conversational. Evaluation crítica: hold-out set, human evaluation, A/B testing.

## Conceptos Clave

### 1. **Full Fine-tuning**
- Actualiza todos los parámetros
- Requiere GPU potente
- Mejor performance
- Risk catastrophic forgetting

### 2. **LoRA (Low-Rank Adaptation)**
- Entrena matrices de bajo rango
- ~1% parámetros trainables
- Merge con base model después
- Eficiente en memoria

### 3. **QLoRA**
- 4-bit quantization
- Aún más eficiente
- Permite fine-tune 70B en 1 GPU
- Minimal accuracy loss

### 4. **Instruction Tuning**
- Format: Instruction + Input + Output
- Mejora following instructions
- Datasets: Alpaca, Dolly, FLAN
- Critical para chatbots

### 5. **RLHF (Reinforcement Learning from Human Feedback)**
- Align con preferencias humanas
- Reward modeling
- PPO training
- Usado en ChatGPT, Claude

## Recursos

**Papers**: LoRA (Hu et al. 2021), QLoRA (Dettmers et al. 2023)
**Cursos**: https://www.deeplearning.ai/short-courses/finetuning-large-language-models/
**Herramientas**: Axolotl, Unsloth, H2O LLM Studio

## Ejemplos

### Ejemplo 1: LoRA con Hugging Face

```python
from transformers import AutoModelForCausalLM, AutoTokenizer, TrainingArguments, Trainer
from peft import LoraConfig, get_peft_model, TaskType
from datasets import load_dataset

# Load base model
model = AutoModelForCausalLM.from_pretrained("gpt2")
tokenizer = AutoTokenizer.from_pretrained("gpt2")

# LoRA config
lora_config = LoraConfig(
    task_type=TaskType.CAUSAL_LM,
    r=8,
    lora_alpha=32,
    lora_dropout=0.1,
    target_modules=["c_attn", "c_proj"]
)

# Apply LoRA
model = get_peft_model(model, lora_config)
model.print_trainable_parameters()

# Prepare dataset
dataset = load_dataset("your-dataset")

# Training
training_args = TrainingArguments(
    output_dir="./lora-model",
    num_train_epochs=3,
    per_device_train_batch_size=4,
    learning_rate=2e-4,
    fp16=True
)

trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=dataset["train"],
    eval_dataset=dataset["test"]
)

trainer.train()
```

### Ejemplo 2: QLoRA Fine-tuning

```python
import torch
from transformers import AutoModelForCausalLM, BitsAndBytesConfig

# 4-bit quantization config
bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_compute_dtype=torch.float16,
    bnb_4bit_use_double_quant=True
)

# Load quantized model
model = AutoModelForCausalLM.from_pretrained(
    "meta-llama/Llama-2-7b-hf",
    quantization_config=bnb_config,
    device_map="auto"
)

# Apply LoRA
lora_config = LoraConfig(
    r=16,
    lora_alpha=32,
    target_modules=["q_proj", "v_proj"],
    lora_dropout=0.05,
    task_type=TaskType.CAUSAL_LM
)

model = get_peft_model(model, lora_config)
```

### Ejemplo 3: Instruction Dataset Format

```python
# Alpaca format
instruction_data = [
    {
        "instruction": "Summarize the following text",
        "input": "Long text here...",
        "output": "Summary here..."
    },
    {
        "instruction": "Translate to Spanish",
        "input": "Hello world",
        "output": "Hola mundo"
    }
]

# Format para training
def format_instruction(example):
    prompt = f"""### Instruction:
{example['instruction']}

### Input:
{example['input']}

### Response:
{example['output']}"""
    return tokenizer(prompt, truncation=True)

formatted_dataset = dataset.map(format_instruction)
```

### Ejemplo 4: OpenAI Fine-tuning API

```python
import openai

# Prepare training file (JSONL)
training_data = [
    {"prompt": "Question: What is AI? ->", "completion": " AI is artificial intelligence."},
    {"prompt": "Question: What is ML? ->", "completion": " ML is machine learning."}
]

# Upload file
file = openai.File.create(
    file=open("training_data.jsonl"),
    purpose="fine-tune"
)

# Create fine-tune job
fine_tune = openai.FineTune.create(
    training_file=file.id,
    model="gpt-3.5-turbo"
)

# Use fine-tuned model
completion = openai.ChatCompletion.create(
    model=fine_tune.fine_tuned_model,
    messages=[{"role": "user", "content": "What is AI?"}]
)
```

## Mejores Prácticas

```python
# ✅ Use LoRA/QLoRA para efficiency
# ✅ Start con small dataset (~1K examples)
# ✅ Validate con hold-out set
# ✅ Monitor for overfitting

# ❌ No fine-tune sin evaluar prompting primero
# ❌ No usar dataset muy pequeño (<100)
```

## Cuándo Fine-tune vs Prompting

**Fine-tune cuando**:
- Muchos datos disponibles (>1K examples)
- Domain muy específico
- Latencia/costo crítico
- Data sensitiva (no puede ir en prompts)

**Prompting cuando**:
- Pocos datos
- Tarea general
- Necesita flexibilidad
- Prototipando

## Proyectos

1. **Domain-Specific Chatbot**
   - Legal/Medical assistant
   - Instruction tuning
   - LoRA fine-tuning

2. **Code Assistant**
   - Fine-tune CodeLlama
   - Company codebase
   - Compare con GitHub Copilot

3. **Custom Style Writer**
   - Specific writing style
   - Few-shot to fine-tune
   - A/B test con base model

## Checklist

- [ ] Entender LoRA/QLoRA
- [ ] Preparar instruction dataset
- [ ] Fine-tune modelo con PEFT
- [ ] Evaluar vs base model
- [ ] Deploy fine-tuned model
- [ ] Monitor performance

---
**Tiempo**: 4-5 semanas
