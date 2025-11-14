# PyTorch y TensorFlow - Frameworks de Deep Learning

## Descripción

PyTorch y TensorFlow son los dos frameworks dominantes para deep learning. PyTorch, desarrollado por Meta (Facebook), se ha convertido en el favorito de investigadores por su diseño dinámico e intuitivo que facilita la experimentación rápida. TensorFlow, creado por Google, domina en producción gracias a su ecosistema maduro (TF Serving, TF Lite, TensorBoard) y herramientas robustas de deployment. Ambos frameworks proporcionan abstracciones de alto nivel para construir redes neuronales, operaciones tensoriales aceleradas por GPU/TPU, y diferenciación automática (autograd) para el entrenamiento eficiente. La elección entre ellos depende del caso de uso: PyTorch para investigación, prototipado rápido y flexibilidad máxima; TensorFlow para sistemas de producción escalables y deployment móvil/edge. Las diferencias se han reducido: PyTorch 2.0 mejoró performance con torch.compile, mientras TensorFlow 2.x adoptó eager execution similar a PyTorch. Dominar al menos uno (idealmente ambos) es esencial para cualquier AI/ML engineer. PyTorch tiene 75% del share en papers de investigación, TensorFlow domina en producción enterprise.

## Conceptos Clave

### 1. **Tensores**
- Arrays multidimensionales (similar a NumPy pero con GPU)
- Operaciones vectorizadas y broadcasting
- Autograd tracking para gradientes
- Device management (CPU/GPU/TPU)
- Dtype y memory management

### 2. **Autograd - Diferenciación Automática**
- Computational graph dinámico (PyTorch) vs estático (TF)
- Backward pass automático
- Gradient accumulation
- torch.no_grad() context
- Custom autograd functions

### 3. **Construcción de Modelos**
- nn.Module (PyTorch) / Model API (TF/Keras)
- Sequential vs Functional API
- Custom layers y operations
- Parameter initialization
- Model composition

### 4. **Training Loop**
- Forward propagation
- Loss computation
- Backpropagation (loss.backward())
- Optimizer step
- Learning rate scheduling

### 5. **Data Loading**
- Dataset y DataLoader (PyTorch)
- tf.data API (TensorFlow)
- Data augmentation
- Batching y shuffling
- Multi-process data loading

### 6. **Model Saving & Loading**
- State dictionaries (PyTorch)
- SavedModel format (TensorFlow)
- Checkpointing strategies
- Transfer learning workflows
- ONNX export

### 7. **GPU/TPU Acceleration**
- CUDA tensors (PyTorch)
- tf.device context (TensorFlow)
- Mixed precision training (FP16)
- Multi-GPU training (DDP, DataParallel)
- Distributed training

## Recursos de Aprendizaje

### Documentación Oficial

1. **PyTorch Documentation**
   - https://pytorch.org/docs/stable/index.html
   - Documentación excelente con ejemplos

2. **PyTorch Tutorials**
   - https://pytorch.org/tutorials/
   - 60 Minute Blitz, Transfer Learning, Production

3. **TensorFlow Documentation**
   - https://www.tensorflow.org/guide
   - Guías completas y API reference

4. **TensorFlow Tutorials**
   - https://www.tensorflow.org/tutorials
   - Quickstart, Keras, Advanced

5. **PyTorch Lightning**
   - https://lightning.ai/docs/pytorch/
   - High-level wrapper para PyTorch

### Cursos Online

1. **Deep Learning with PyTorch - Udacity** (Gratis)
   - https://www.udacity.com/course/deep-learning-pytorch--ud188
   - Por Meta/Facebook, muy práctico

2. **fast.ai - Practical Deep Learning**
   - https://course.fast.ai/
   - Approach top-down, usa PyTorch

3. **DeepLearning.AI TensorFlow Specialization**
   - https://www.coursera.org/specializations/tensorflow-in-practice
   - 4 cursos por Andrew Ng

4. **PyTorch for Deep Learning - Zero to Mastery**
   - https://zerotomastery.io/courses/learn-pytorch/

5. **Deep Learning Specialization - Coursera**
   - https://www.coursera.org/specializations/deep-learning
   - Por Andrew Ng, usa TensorFlow

### Libros

1. **"Deep Learning with PyTorch"** - Eli Stevens, Luca Antiga
   - Libro oficial de PyTorch
   - Ejemplos prácticos extensos
   - Casos de uso reales

2. **"Programming PyTorch for Deep Learning"** - Ian Pointer
   - O'Reilly, muy práctico
   - Production-focused

3. **"Hands-On Machine Learning"** - Aurélien Géron
   - Parte 2 cubre TensorFlow/Keras
   - Excelente para comenzar

4. **"Deep Learning with Python"** - François Chollet
   - Por creador de Keras
   - TensorFlow/Keras focused

### Videos

1. **PyTorch Official YouTube**
   - https://www.youtube.com/c/PyTorch
   - Tutoriales oficiales

2. **Aladdin Persson - PyTorch Tutorials**
   - https://www.youtube.com/c/AladdinPersson
   - Muy didáctico, implementaciones desde cero

3. **TensorFlow YouTube Channel**
   - https://www.youtube.com/c/TensorFlow

## Instalación

```bash
# PyTorch (CPU)
pip install torch torchvision torchaudio

# PyTorch (GPU - CUDA 11.8)
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118

# PyTorch (GPU - CUDA 12.1)
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121

# TensorFlow (CPU y GPU)
pip install tensorflow

# TensorFlow con Metal (Apple Silicon)
pip install tensorflow-metal

# Verificar instalación PyTorch
python -c "import torch; print(f'PyTorch: {torch.__version__}'); print(f'CUDA available: {torch.cuda.is_available()}')"

# Verificar instalación TensorFlow
python -c "import tensorflow as tf; print(f'TensorFlow: {tf.__version__}'); print(f'GPUs: {tf.config.list_physical_devices(\"GPU\")}')"

# PyTorch Lightning (opcional pero recomendado)
pip install lightning

# TensorBoard
pip install tensorboard
```

## Ejemplos Prácticos

### Ejemplo 1: Tensores Básicos - PyTorch

```python
import torch
import numpy as np

# Crear tensores
x = torch.tensor([1, 2, 3])
y = torch.zeros(3, 4)
z = torch.randn(2, 3)  # Normal distribution
w = torch.ones(2, 3, dtype=torch.float32)

print(f"x: {x}")
print(f"y shape: {y.shape}")
print(f"z:\n{z}")

# Desde NumPy (share memory)
np_array = np.array([[1, 2], [3, 4]])
tensor_from_numpy = torch.from_numpy(np_array)

# A NumPy
tensor_to_numpy = z.numpy()

# Operaciones
a = torch.tensor([1.0, 2.0, 3.0])
b = torch.tensor([4.0, 5.0, 6.0])

print(f"a + b = {a + b}")
print(f"a * b = {a * b}")  # Element-wise
print(f"a @ b = {torch.dot(a, b)}")  # Dot product

# Matrix operations
A = torch.randn(3, 4)
B = torch.randn(4, 5)
C = A @ B  # Matrix multiplication
print(f"Matrix mult shape: {C.shape}")

# Reshape
original = torch.arange(12)
reshaped = original.reshape(3, 4)
print(f"Reshaped:\n{reshaped}")

# View (shares memory)
viewed = original.view(2, 6)

# GPU operations
if torch.cuda.is_available():
    device = torch.device('cuda')
    x_gpu = x.to(device)
    print(f"x on GPU: {x_gpu.device}")
    
    # Operations on GPU
    y_gpu = torch.randn(3).to(device)
    z_gpu = x_gpu + y_gpu
    
    # Back to CPU
    z_cpu = z_gpu.cpu()
```

### Ejemplo 2: Autograd y Gradientes

```python
import torch

# Require gradients
x = torch.tensor([2.0, 3.0], requires_grad=True)
y = torch.tensor([4.0, 5.0], requires_grad=True)

# Operations
z = x * y
loss = z.sum()

print(f"z: {z}")
print(f"loss: {loss}")

# Backpropagation
loss.backward()

print(f"x.grad: {x.grad}")  # dL/dx
print(f"y.grad: {y.grad}")  # dL/dy

# Detach from graph
with torch.no_grad():
    result = x * 2  # No gradient tracking

# Manual gradient example
x = torch.tensor(3.0, requires_grad=True)
y = x ** 2 + 2 * x + 1

y.backward()
print(f"dy/dx at x=3: {x.grad}")  # Should be 2*3 + 2 = 8
```

### Ejemplo 3: Red Neuronal Simple - PyTorch

```python
import torch
import torch.nn as nn
import torch.optim as optim

# Definir modelo
class SimpleNN(nn.Module):
    def __init__(self, input_size, hidden_size, output_size):
        super(SimpleNN, self).__init__()
        self.fc1 = nn.Linear(input_size, hidden_size)
        self.relu = nn.ReLU()
        self.fc2 = nn.Linear(hidden_size, output_size)
    
    def forward(self, x):
        x = self.fc1(x)
        x = self.relu(x)
        x = self.fc2(x)
        return x

# Instanciar modelo
model = SimpleNN(input_size=784, hidden_size=128, output_size=10)
print(model)

# Loss y optimizer
criterion = nn.CrossEntropyLoss()
optimizer = optim.Adam(model.parameters(), lr=0.001)

# Datos de ejemplo
batch_size = 32
X_batch = torch.randn(batch_size, 784)
y_batch = torch.randint(0, 10, (batch_size,))

# Training step
optimizer.zero_grad()  # Reset gradients
outputs = model(X_batch)
loss = criterion(outputs, y_batch)
loss.backward()  # Compute gradients
optimizer.step()  # Update weights

print(f"Loss: {loss.item():.4f}")
```

### Ejemplo 4: CNN para Clasificación - PyTorch

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class SimpleCNN(nn.Module):
    def __init__(self):
        super(SimpleCNN, self).__init__()
        # Conv layers
        self.conv1 = nn.Conv2d(1, 32, kernel_size=3, padding=1)
        self.conv2 = nn.Conv2d(32, 64, kernel_size=3, padding=1)
        self.pool = nn.MaxPool2d(2, 2)
        
        # FC layers
        self.fc1 = nn.Linear(64 * 7 * 7, 128)
        self.fc2 = nn.Linear(128, 10)
        
        self.dropout = nn.Dropout(0.5)
    
    def forward(self, x):
        # Conv block 1
        x = self.pool(F.relu(self.conv1(x)))
        
        # Conv block 2
        x = self.pool(F.relu(self.conv2(x)))
        
        # Flatten
        x = x.view(-1, 64 * 7 * 7)
        
        # FC layers
        x = F.relu(self.fc1(x))
        x = self.dropout(x)
        x = self.fc2(x)
        
        return x

# Usar modelo
model = SimpleCNN()
x = torch.randn(4, 1, 28, 28)  # Batch of 4 MNIST images
output = model(x)
print(f"Output shape: {output.shape}")  # [4, 10]
```

### Ejemplo 5: DataLoader - PyTorch

```python
from torch.utils.data import Dataset, DataLoader
import torchvision.transforms as transforms

# Custom Dataset
class CustomDataset(Dataset):
    def __init__(self, data, labels, transform=None):
        self.data = data
        self.labels = labels
        self.transform = transform
    
    def __len__(self):
        return len(self.data)
    
    def __getitem__(self, idx):
        sample = self.data[idx]
        label = self.labels[idx]
        
        if self.transform:
            sample = self.transform(sample)
        
        return sample, label

# Crear dataset
X = torch.randn(1000, 28, 28)
y = torch.randint(0, 10, (1000,))

# Transformaciones
transform = transforms.Compose([
    transforms.Normalize((0.5,), (0.5,))
])

dataset = CustomDataset(X, y, transform=transform)

# DataLoader
dataloader = DataLoader(
    dataset,
    batch_size=32,
    shuffle=True,
    num_workers=2,
    pin_memory=True  # Faster GPU transfer
)

# Iterar
for batch_idx, (data, labels) in enumerate(dataloader):
    print(f"Batch {batch_idx}: data shape = {data.shape}, labels shape = {labels.shape}")
    if batch_idx == 2:
        break
```

### Ejemplo 6: Training Loop Completo - PyTorch

```python
import torch
import torch.nn as nn
from torch.utils.data import DataLoader
from torchvision import datasets, transforms

# Configuración
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
num_epochs = 10
batch_size = 64
learning_rate = 0.001

# Datos (MNIST example)
transform = transforms.Compose([
    transforms.ToTensor(),
    transforms.Normalize((0.1307,), (0.3081,))
])

train_dataset = datasets.MNIST(root='./data', train=True, transform=transform, download=True)
test_dataset = datasets.MNIST(root='./data', train=False, transform=transform)

train_loader = DataLoader(train_dataset, batch_size=batch_size, shuffle=True)
test_loader = DataLoader(test_dataset, batch_size=batch_size, shuffle=False)

# Modelo
model = SimpleCNN().to(device)
criterion = nn.CrossEntropyLoss()
optimizer = optim.Adam(model.parameters(), lr=learning_rate)

# Training loop
for epoch in range(num_epochs):
    model.train()
    train_loss = 0.0
    correct = 0
    total = 0
    
    for batch_idx, (data, targets) in enumerate(train_loader):
        data, targets = data.to(device), targets.to(device)
        
        # Forward
        outputs = model(data)
        loss = criterion(outputs, targets)
        
        # Backward
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()
        
        # Métricas
        train_loss += loss.item()
        _, predicted = outputs.max(1)
        total += targets.size(0)
        correct += predicted.eq(targets).sum().item()
        
        if batch_idx % 100 == 0:
            print(f'Epoch [{epoch+1}/{num_epochs}], Step [{batch_idx}/{len(train_loader)}], '
                  f'Loss: {loss.item():.4f}')
    
    # Epoch summary
    avg_loss = train_loss / len(train_loader)
    accuracy = 100. * correct / total
    print(f'Epoch [{epoch+1}/{num_epochs}], Avg Loss: {avg_loss:.4f}, Accuracy: {accuracy:.2f}%')
    
    # Validation
    model.eval()
    test_loss = 0.0
    correct = 0
    total = 0
    
    with torch.no_grad():
        for data, targets in test_loader:
            data, targets = data.to(device), targets.to(device)
            outputs = model(data)
            loss = criterion(outputs, targets)
            
            test_loss += loss.item()
            _, predicted = outputs.max(1)
            total += targets.size(0)
            correct += predicted.eq(targets).sum().item()
    
    avg_test_loss = test_loss / len(test_loader)
    test_accuracy = 100. * correct / total
    print(f'Test Loss: {avg_test_loss:.4f}, Test Accuracy: {test_accuracy:.2f}%\n')
```

### Ejemplo 7: TensorFlow/Keras Equivalente

```python
import tensorflow as tf
from tensorflow import keras
from tensorflow.keras import layers

# Definir modelo con Keras
model = keras.Sequential([
    layers.Conv2D(32, (3, 3), activation='relu', input_shape=(28, 28, 1)),
    layers.MaxPooling2D((2, 2)),
    layers.Conv2D(64, (3, 3), activation='relu'),
    layers.MaxPooling2D((2, 2)),
    layers.Flatten(),
    layers.Dense(128, activation='relu'),
    layers.Dropout(0.5),
    layers.Dense(10, activation='softmax')
])

# Compilar
model.compile(
    optimizer='adam',
    loss='sparse_categorical_crossentropy',
    metrics=['accuracy']
)

# Cargar datos
(x_train, y_train), (x_test, y_test) = keras.datasets.mnist.load_data()
x_train = x_train.reshape(-1, 28, 28, 1).astype('float32') / 255
x_test = x_test.reshape(-1, 28, 28, 1).astype('float32') / 255

# Entrenar
history = model.fit(
    x_train, y_train,
    batch_size=64,
    epochs=10,
    validation_data=(x_test, y_test),
    callbacks=[
        keras.callbacks.ModelCheckpoint('best_model.h5', save_best_only=True),
        keras.callbacks.EarlyStopping(patience=3, restore_best_weights=True)
    ]
)

# Evaluar
test_loss, test_acc = model.evaluate(x_test, y_test)
print(f'Test accuracy: {test_acc:.4f}')

# Predicción
predictions = model.predict(x_test[:5])
print(f'Predictions shape: {predictions.shape}')
```

### Ejemplo 8: Model Saving & Loading

```python
# PyTorch - Save/Load

# Guardar solo state dict (recomendado)
torch.save(model.state_dict(), 'model_weights.pth')

# Cargar
model = SimpleCNN()
model.load_state_dict(torch.load('model_weights.pth'))
model.eval()

# Guardar modelo completo
torch.save(model, 'model_complete.pth')
model = torch.load('model_complete.pth')

# Guardar checkpoint con optimizer
checkpoint = {
    'epoch': epoch,
    'model_state_dict': model.state_dict(),
    'optimizer_state_dict': optimizer.state_dict(),
    'loss': loss
}
torch.save(checkpoint, 'checkpoint.pth')

# Cargar checkpoint
checkpoint = torch.load('checkpoint.pth')
model.load_state_dict(checkpoint['model_state_dict'])
optimizer.load_state_dict(checkpoint['optimizer_state_dict'])
epoch = checkpoint['epoch']
loss = checkpoint['loss']

# TensorFlow/Keras - Save/Load

# Guardar modelo completo
model.save('my_model.h5')

# Cargar
model = keras.models.load_model('my_model.h5')

# SavedModel format (recomendado para producción)
model.save('saved_model/')
model = keras.models.load_model('saved_model/')

# Solo pesos
model.save_weights('model_weights.h5')
model.load_weights('model_weights.h5')
```

## PyTorch vs TensorFlow - Comparación

```python
# PYTORCH - Ejemplo
import torch
import torch.nn as nn

class PyTorchModel(nn.Module):
    def __init__(self):
        super().__init__()
        self.fc = nn.Linear(10, 5)
    
    def forward(self, x):
        return self.fc(x)

model = PyTorchModel()
x = torch.randn(32, 10)
y = model(x)

# TENSORFLOW - Equivalente
import tensorflow as tf

model = tf.keras.Sequential([
    tf.keras.layers.Dense(5, input_shape=(10,))
])

x = tf.random.normal((32, 10))
y = model(x)
```

## Mejores Prácticas

### 1. Use GPU Eficientemente

```python
# ✅ Bueno - mover modelo y datos a GPU
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
model = model.to(device)

for data, labels in dataloader:
    data, labels = data.to(device), labels.to(device)
    # Training...

# ❌ Malo - mover tensor repetidamente
for i in range(1000):
    x = torch.randn(10).cuda()  # Slow!
```

### 2. Gradient Management

```python
# ✅ Bueno - zero_grad antes de backward
optimizer.zero_grad()
loss.backward()
optimizer.step()

# ✅ Bueno - usar context manager para inference
with torch.no_grad():
    predictions = model(test_data)
```

### 3. Memory Management

```python
# ✅ Bueno - liberar memoria de gradientes
with torch.no_grad():
    # Inference code

# ✅ Bueno - usar .item() para scalars
loss_value = loss.item()  # Evita memory leak

# ❌ Malo - guardar tensores en listas sin detach
losses = []
for batch in dataloader:
    loss = criterion(model(data), labels)
    losses.append(loss)  # Memory leak!

# ✅ Correcto
losses.append(loss.item())
```

### 4. Model Checkpointing

```python
# ✅ Bueno - guardar mejor modelo
best_loss = float('inf')

for epoch in range(num_epochs):
    # Training...
    
    if val_loss < best_loss:
        best_loss = val_loss
        torch.save(model.state_dict(), 'best_model.pth')
```

### 5. Learning Rate Scheduling

```python
# ✅ Bueno - usar scheduler
from torch.optim.lr_scheduler import ReduceLROnPlateau

scheduler = ReduceLROnPlateau(optimizer, mode='min', patience=5)

for epoch in range(num_epochs):
    # Training...
    scheduler.step(val_loss)
```

## Proyectos Sugeridos

1. **MNIST Digit Classification**
   - Implementar CNN desde cero
   - Comparar PyTorch vs TensorFlow
   - Optimizar performance
   - Deploy con TorchServe/TF Serving

2. **Transfer Learning con ResNet**
   - Fine-tuning en dataset custom
   - Feature extraction
   - Data augmentation
   - Comparar diferentes backbones

3. **Image Segmentation**
   - U-Net architecture
   - Medical imaging o semantic segmentation
   - Custom loss functions
   - Post-processing

4. **Text Generation con RNN/LSTM**
   - Character-level model
   - Shakespeare text generation
   - Temperature sampling
   - Beam search

5. **GAN para Image Generation**
   - DCGAN implementation
   - Mode collapse handling
   - Progressive training
   - Evaluation metrics (FID)

## Checklist de Aprendizaje

- [ ] Dominar operaciones tensoriales básicas
- [ ] Entender computational graphs y autograd
- [ ] Construir modelos con nn.Module/Keras
- [ ] Implementar training loop completo
- [ ] Usar DataLoader eficientemente
- [ ] Manejar GPU/CPU correctamente
- [ ] Aplicar regularización (dropout, batch norm)
- [ ] Implementar callbacks y checkpointing
- [ ] Optimizar performance y memory
- [ ] Deploy modelo en producción

## Próximos Pasos

1. Profundizar en **Arquitecturas Avanzadas** (ResNet, Transformers)
2. Estudiar **Optimización** (learning rate scheduling, gradient clipping)
3. Aprender **Distributed Training** (DDP, Horovod)
4. Explorar **Model Quantization** para edge deployment
5. Dominar **TensorBoard** y **Weights & Biases** para tracking

---
**Tiempo estimado**: 6-8 semanas dedicando 4-5 horas diarias
