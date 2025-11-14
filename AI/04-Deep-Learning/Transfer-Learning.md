# Transfer Learning - Reutilizando Conocimiento

## Descripción

Transfer learning reutiliza modelos pre-entrenados en grandes datasets (ImageNet, Wikipedia) para tareas específicas con datos limitados, siendo una de las técnicas más poderosas en deep learning moderno. En lugar de entrenar desde cero (requiere millones de imágenes y semanas de GPU), usamos pesos pre-entrenados que ya capturaron features generales (edges, textures, shapes) y los adaptamos. Estrategias: feature extraction (congelar todas las capas excepto última), fine-tuning (entrenar algunas capas finales), fine-tuning completo (todo el modelo con learning rate bajo). Funciona porque features de bajo nivel son transferibles entre dominios. ResNet50 entrenado en ImageNet reconoce edges/textures útiles para imágenes médicas, clasificación de productos, etc. Reduce dramáticamente: tiempo de entrenamiento (horas vs días), datos necesarios (cientos vs millones), costo computacional. Aplicaciones: CV (clasificación, detección, segmentación), NLP (BERT, GPT fine-tuning), speech recognition. Conceptos: domain adaptation, catastrophic forgetting, learning rate scheduling, gradual unfreezing. Es estándar en la industria.

## Conceptos Clave

### 1. **Feature Extraction**
- Congelar capas pre-entrenadas
- Solo entrenar classifier final
- Más rápido, previene overfitting
- Cuando dataset muy pequeño

### 2. **Fine-Tuning**
- Descongelar capas superiores
- Learning rates diferenciados
- Gradual unfreezing strategy
- Balance entre generalización y adaptación

### 3. **Domain Adaptation**
- Transferir entre dominios diferentes
- Natural images → Medical images
- English → Spanish (NLP)
- Discrepancy reduction

### 4. **Pre-trained Models**
- ImageNet models (ResNet, EfficientNet)
- BERT, GPT para NLP
- CLIP para vision-language
- Model zoos (torchvision, Hugging Face)

### 5. **Learning Rate Strategies**
- Lower LR para pre-trained layers
- Higher LR para nuevas capas
- Discriminative fine-tuning
- Gradual unfreezing

## Recursos

### Cursos
1. **Transfer Learning - Coursera**
   - https://www.coursera.org/learn/convolutional-neural-networks

2. **Fast.ai Practical Deep Learning**
   - https://course.fast.ai/
   - Heavy focus en transfer learning

### Papers
1. **ImageNet** - Deng et al. (2009)
2. **ResNet** - He et al. (2015)
3. **BERT** - Devlin et al. (2018)
4. **EfficientNet** - Tan & Le (2019)

## Ejemplos Prácticos

### Ejemplo 1: Feature Extraction

```python
import torch
import torchvision.models as models
import torch.nn as nn

# Cargar modelo pre-entrenado
resnet = models.resnet50(pretrained=True)

# Congelar TODAS las capas
for param in resnet.parameters():
    param.requires_grad = False

# Reemplazar última capa
num_ftrs = resnet.fc.in_features
resnet.fc = nn.Linear(num_ftrs, num_classes)

# Solo pesos de fc son entrenables
optimizer = torch.optim.Adam(resnet.fc.parameters(), lr=0.001)

# Training
for epoch in range(num_epochs):
    for images, labels in train_loader:
        outputs = resnet(images)
        loss = criterion(outputs, labels)
        
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()
```

### Ejemplo 2: Fine-Tuning con Learning Rates Diferenciados

```python
# Descongelar últimas capas
for param in resnet.layer4.parameters():
    param.requires_grad = True
for param in resnet.fc.parameters():
    param.requires_grad = True

# Diferentes learning rates
optimizer = torch.optim.Adam([
    {'params': resnet.layer4.parameters(), 'lr': 1e-4},
    {'params': resnet.fc.parameters(), 'lr': 1e-3}
])
```

### Ejemplo 3: Gradual Unfreezing

```python
def gradual_unfreeze(model, optimizer, train_loader, stages):
    """
    Descongelar gradualmente
    """
    # Stage 1: Solo FC
    for param in model.parameters():
        param.requires_grad = False
    for param in model.fc.parameters():
        param.requires_grad = True
    
    train_epochs(model, optimizer, train_loader, stages[0])
    
    # Stage 2: FC + Layer4
    for param in model.layer4.parameters():
        param.requires_grad = True
    
    optimizer.param_groups[0]['lr'] *= 0.1
    train_epochs(model, optimizer, train_loader, stages[1])
    
    # Stage 3: Todo
    for param in model.parameters():
        param.requires_grad = True
    
    optimizer.param_groups[0]['lr'] *= 0.1
    train_epochs(model, optimizer, train_loader, stages[2])
```

### Ejemplo 4: Transfer Learning para NLP

```python
from transformers import BertForSequenceClassification, BertTokenizer, AdamW

# Cargar BERT pre-entrenado
model = BertForSequenceClassification.from_pretrained(
    'bert-base-uncased',
    num_labels=2
)

# Congelar embeddings y primeras capas
for name, param in model.named_parameters():
    if 'bert.embeddings' in name or 'bert.encoder.layer.0' in name:
        param.requires_grad = False

# Optimizer con weight decay
optimizer = AdamW(model.parameters(), lr=2e-5, weight_decay=0.01)

# Training loop
model.train()
for batch in train_dataloader:
    input_ids = batch['input_ids']
    attention_mask = batch['attention_mask']
    labels = batch['labels']
    
    outputs = model(input_ids, attention_mask=attention_mask, labels=labels)
    loss = outputs.loss
    
    loss.backward()
    optimizer.step()
    optimizer.zero_grad()
```

### Ejemplo 5: Custom Head para Multi-task

```python
class CustomResNet(nn.Module):
    def __init__(self, num_classes_task1, num_classes_task2):
        super().__init__()
        # Pre-trained backbone
        resnet = models.resnet50(pretrained=True)
        self.backbone = nn.Sequential(*list(resnet.children())[:-1])
        
        # Custom heads
        self.fc_task1 = nn.Linear(2048, num_classes_task1)
        self.fc_task2 = nn.Linear(2048, num_classes_task2)
    
    def forward(self, x, task='task1'):
        features = self.backbone(x)
        features = features.view(features.size(0), -1)
        
        if task == 'task1':
            return self.fc_task1(features)
        else:
            return self.fc_task2(features)
```

## Mejores Prácticas

### 1. Empezar con Feature Extraction

```python
# ✅ Primero: solo entrenar classifier
# Luego: gradual unfreezing
```

### 2. Learning Rate Apropiado

```python
# ✅ LR bajo para pre-trained: 1e-5 a 1e-4
# ✅ LR alto para nuevas capas: 1e-3
```

### 3. Data Augmentation

```python
# ✅ Crítico cuando dataset pequeño
transform = transforms.Compose([
    transforms.RandomResizedCrop(224),
    transforms.RandomHorizontalFlip(),
    transforms.ColorJitter(),
    transforms.ToTensor(),
    transforms.Normalize([0.485, 0.456, 0.406], [0.229, 0.224, 0.225])
])
```

## Proyectos Sugeridos

1. **Clasificador de Imágenes Médicas**
   - ResNet pre-trained
   - Dataset pequeño (~1000 images)
   - Comparar: scratch vs transfer

2. **Fine-tuning BERT**
   - Sentiment analysis
   - Named Entity Recognition
   - Comparar con embeddings clásicos

3. **Object Detection Custom**
   - YOLOv5 o Faster R-CNN
   - Annotations propias
   - Transfer desde COCO

## Checklist

- [ ] Entender cuándo usar transfer learning
- [ ] Dominar feature extraction
- [ ] Aplicar fine-tuning efectivo
- [ ] Usar gradual unfreezing
- [ ] Optimizar learning rates
- [ ] Comparar con training desde cero
- [ ] Deploy modelo fine-tuned

## Próximos Pasos

1. Explorar **EfficientNet, Vision Transformers**
2. Estudiar **Few-shot Learning**
3. Aprender **Domain Adaptation** avanzada
4. Profundizar en **Meta-Learning**

---
**Tiempo estimado**: 3-4 semanas
