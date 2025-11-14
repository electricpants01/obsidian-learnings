# Optimizadores - Algoritmos de Optimización

## Descripción

Los optimizadores son algoritmos que ajustan los pesos de la red neuronal para minimizar la función de loss. La elección del optimizador impacta dramáticamente en convergencia, velocidad de training y calidad del modelo final. SGD (Stochastic Gradient Descent) es el más simple: actualiza pesos en dirección opuesta al gradiente con learning rate fijo. Momentum agrega "inercia" acelerando en direcciones consistentes. RMSprop adapta learning rates por parámetro usando promedio móvil de gradientes al cuadrado. Adam combina Momentum y RMSprop siendo el optimizador más popular por su robustez y buen performance "out of the box". AdamW agrega weight decay correctamente. Optimizadores de segundo orden (L-BFGS) usan información de curvatura pero son costosos. Learning rate scheduling es crucial: decay exponencial, cosine annealing, one-cycle policy. Hyperparámetros: learning rate (más importante), momentum (0.9 típico), betas en Adam (0.9, 0.999). Warmup previene divergencia inicial. Gradient clipping previene exploding gradients. Lookahead y LAMB para training distribuido.

## Conceptos Clave

### 1. **SGD (Stochastic Gradient Descent)**
- Actualización: w = w - lr * gradient
- Simple pero efectivo
- Requiere tuning cuidadoso de LR
- Con momentum: mejor convergencia

### 2. **Momentum**
- Acumulación de gradientes previos
- Acelera en direcciones consistentes
- Beta típico: 0.9
- Nesterov momentum: look-ahead

### 3. **RMSprop**
- Adaptive learning rates
- Promedio móvil de gradientes²
- Bueno para RNNs
- Precursor de Adam

### 4. **Adam (Adaptive Moment Estimation)**
- Combina Momentum + RMSprop
- Learning rates adaptativos
- Beta1=0.9, Beta2=0.999
- Más popular en práctica

### 5. **AdamW**
- Adam con weight decay correcto
- Mejor regularización
- Preferido para Transformers
- Decoupled weight decay

### 6. **Learning Rate Scheduling**
- Step decay, exponential decay
- Cosine annealing
- ReduceLROnPlateau
- One-cycle policy
- Warmup

## Recursos

### Papers Fundamentales
1. **Adam** - Kingma & Ba (2014)
2. **AdamW** - Loshchilov & Hutter (2017)
3. **Lookahead** - Zhang et al. (2019)
4. **LARS/LAMB** - You et al. (2017, 2019)

### Cursos
1. **Deep Learning Specialization** - Coursera
   - https://www.coursera.org/specializations/deep-learning
   - Semana sobre optimization

### Documentación
1. **PyTorch Optimizers**
   - https://pytorch.org/docs/stable/optim.html

## Ejemplos Prácticos

### Ejemplo 1: Comparar Optimizadores

```python
import torch
import torch.nn as nn
import torch.optim as optim

model = SimpleNet()

# SGD básico
optimizer_sgd = optim.SGD(model.parameters(), lr=0.01)

# SGD con momentum
optimizer_sgd_momentum = optim.SGD(model.parameters(), lr=0.01, momentum=0.9)

# RMSprop
optimizer_rmsprop = optim.RMSprop(model.parameters(), lr=0.001)

# Adam
optimizer_adam = optim.Adam(model.parameters(), lr=0.001)

# AdamW (mejor para Transformers)
optimizer_adamw = optim.AdamW(model.parameters(), lr=0.001, weight_decay=0.01)

# Training loop
for epoch in range(num_epochs):
    for batch in dataloader:
        optimizer.zero_grad()
        loss = criterion(model(batch['x']), batch['y'])
        loss.backward()
        optimizer.step()
```

### Ejemplo 2: Learning Rate Scheduling

```python
# Cosine Annealing
scheduler = optim.lr_scheduler.CosineAnnealingLR(optimizer, T_max=100)

# Step Decay
scheduler = optim.lr_scheduler.StepLR(optimizer, step_size=30, gamma=0.1)

# Exponential Decay
scheduler = optim.lr_scheduler.ExponentialLR(optimizer, gamma=0.95)

# ReduceLROnPlateau (basado en métrica)
scheduler = optim.lr_scheduler.ReduceLROnPlateau(
    optimizer, mode='min', factor=0.5, patience=10
)

# One Cycle Policy
scheduler = optim.lr_scheduler.OneCycleLR(
    optimizer, max_lr=0.01, steps_per_epoch=len(train_loader), epochs=10
)

# Usar scheduler
for epoch in range(num_epochs):
    train(...)
    val_loss = validate(...)
    
    # Para ReduceLROnPlateau
    scheduler.step(val_loss)
    
    # Para otros
    # scheduler.step()
```

### Ejemplo 3: Warmup

```python
class WarmupScheduler:
    def __init__(self, optimizer, warmup_epochs, total_epochs, base_lr, target_lr):
        self.optimizer = optimizer
        self.warmup_epochs = warmup_epochs
        self.total_epochs = total_epochs
        self.base_lr = base_lr
        self.target_lr = target_lr
    
    def step(self, epoch):
        if epoch < self.warmup_epochs:
            # Linear warmup
            lr = self.base_lr + (self.target_lr - self.base_lr) * epoch / self.warmup_epochs
        else:
            # Cosine decay
            progress = (epoch - self.warmup_epochs) / (self.total_epochs - self.warmup_epochs)
            lr = self.target_lr * 0.5 * (1 + np.cos(np.pi * progress))
        
        for param_group in self.optimizer.param_groups:
            param_group['lr'] = lr

# Uso
scheduler = WarmupScheduler(optimizer, warmup_epochs=5, total_epochs=100, 
                           base_lr=1e-7, target_lr=1e-3)
```

### Ejemplo 4: Gradient Clipping

```python
# Por norma (prevenir exploding gradients)
torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)

# Por valor
torch.nn.utils.clip_grad_value_(model.parameters(), clip_value=0.5)

# En training loop
for batch in dataloader:
    optimizer.zero_grad()
    loss = criterion(model(batch['x']), batch['y'])
    loss.backward()
    
    # Gradient clipping
    torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)
    
    optimizer.step()
```

### Ejemplo 5: Learning Rate Finder

```python
class LRFinder:
    def __init__(self, model, optimizer, criterion, device):
        self.model = model
        self.optimizer = optimizer
        self.criterion = criterion
        self.device = device
    
    def find(self, train_loader, start_lr=1e-7, end_lr=10, num_iter=100):
        lrs = []
        losses = []
        
        lr = start_lr
        self.optimizer.param_groups[0]['lr'] = lr
        multiplier = (end_lr / start_lr) ** (1/num_iter)
        
        for batch_idx, (data, target) in enumerate(train_loader):
            if batch_idx >= num_iter:
                break
            
            data, target = data.to(self.device), target.to(self.device)
            
            self.optimizer.zero_grad()
            output = self.model(data)
            loss = self.criterion(output, target)
            loss.backward()
            self.optimizer.step()
            
            lrs.append(lr)
            losses.append(loss.item())
            
            lr *= multiplier
            self.optimizer.param_groups[0]['lr'] = lr
        
        # Plot
        plt.plot(lrs, losses)
        plt.xscale('log')
        plt.xlabel('Learning Rate')
        plt.ylabel('Loss')
        plt.show()
        
        return lrs, losses
```

### Ejemplo 6: Different LRs para Different Layers

```python
# Discriminative fine-tuning
optimizer = optim.Adam([
    {'params': model.layer1.parameters(), 'lr': 1e-5},
    {'params': model.layer2.parameters(), 'lr': 1e-4},
    {'params': model.layer3.parameters(), 'lr': 1e-3},
    {'params': model.fc.parameters(), 'lr': 1e-2}
])
```

## Mejores Prácticas

### 1. Elección de Optimizador

```python
# ✅ Para CNNs: SGD con momentum
optimizer = optim.SGD(model.parameters(), lr=0.1, momentum=0.9)

# ✅ Para Transformers/NLP: AdamW
optimizer = optim.AdamW(model.parameters(), lr=1e-4, weight_decay=0.01)

# ✅ Para RNNs: Adam o RMSprop
optimizer = optim.Adam(model.parameters(), lr=0.001)

# ✅ General purpose: Adam
optimizer = optim.Adam(model.parameters(), lr=0.001)
```

### 2. Learning Rate

```python
# ✅ Usar LR Finder
# ✅ Start small (1e-4 a 1e-3 para Adam)
# ✅ Usar warmup para Transformers
# ✅ Decay over time
```

### 3. Weight Decay

```python
# ✅ AdamW con weight decay
optimizer = optim.AdamW(model.parameters(), lr=0.001, weight_decay=0.01)

# ❌ Evitar L2 con Adam (usar AdamW)
```

## Comparación

| Optimizador | Pros | Contras | Uso Típico |
|-------------|------|---------|------------|
| SGD | Simple, generaliza bien | Lento, requiere tuning | CNNs, ResNets |
| SGD+Momentum | Más rápido que SGD | Aún requiere tuning | CNNs |
| RMSprop | Adaptativo, bueno para RNNs | Menos popular | RNNs |
| Adam | Robusto, funciona bien | A veces overfits | General purpose |
| AdamW | Mejor regularización | - | Transformers, NLP |

## Proyectos

1. **Benchmark Optimizers**
   - Mismo modelo, diferentes optimizers
   - Comparar convergencia
   - Visualizar learning curves

2. **Implement Adam desde Cero**
   - Entender funcionamiento interno
   - Comparar con PyTorch

3. **Optimal LR Search**
   - LR finder implementation
   - One-cycle policy
   - Comparar resultados

## Checklist

- [ ] Entender SGD y momentum
- [ ] Dominar Adam/AdamW
- [ ] Aplicar LR scheduling
- [ ] Usar gradient clipping
- [ ] Implementar warmup
- [ ] Comparar optimizers experimentalmente
- [ ] Optimizar hyperparámetros

## Próximos Pasos

1. Estudiar **Second-Order Methods**
2. Aprender **Distributed Optimization** (LARS, LAMB)
3. Explorar **Meta-Learning optimizers**
4. Profundizar en **Automatic Hyperparameter Tuning**

---
**Tiempo estimado**: 2-3 semanas
