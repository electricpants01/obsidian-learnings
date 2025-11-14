# Convolutional Neural Networks (CNN)

## Descripción

Las Redes Neuronales Convolucionales revolucionaron computer vision al aplicar el concepto de campos receptivos locales inspirados en la corteza visual. A diferencia de MLPs que procesan imágenes como vectores planos perdiendo estructura espacial, las CNNs preservan relaciones espaciales mediante operaciones de convolución que detectan features locales (bordes, texturas, patrones) compartiendo pesos a través de la imagen. La arquitectura típica alterna capas convolucionales (feature extraction) con pooling (downsampling) seguidas de capas fully connected (clasificación). Las primeras capas detectan features simples (bordes horizontales/verticales), capas medias patrones complejos (texturas, formas), capas profundas conceptos de alto nivel (ojos, ruedas, rostros). LeNet-5 (1998) demostró el concepto en dígitos, AlexNet (2012) lo escaló ganando ImageNet, ResNet (2015) introdujo skip connections permitiendo redes de 100+ capas. CNNs dominan: clasificación de imágenes, detección de objetos, segmentación semántica, reconocimiento facial, imágenes médicas. Conceptos clave: convoluciones, pooling, padding, stride, receptive field, transfer learning, data augmentation. Son fundamentales para cualquier tarea de visión por computadora.

## Conceptos Clave

### 1. **Operación de Convolución**
- Kernel/filter deslizándose sobre imagen
- Producto punto local feature maps
- Weight sharing reduce parámetros
- Translation invariance

### 2. **Pooling**
- Max pooling: toma valor máximo
- Average pooling: promedio
- Downsampling para reducir dimensionalidad
- Invarianza a pequeños desplazamientos

### 3. **Parámetros de Convolución**
- Kernel size (3x3, 5x5, 7x7)
- Stride: paso del kernel
- Padding: zeros alrededor (same/valid)
- Dilation: espaciado del kernel

### 4. **Arquitecturas Clásicas**
- LeNet-5: pionera (dígitos)
- AlexNet: breakthrough ImageNet 2012
- VGGNet: bloques simples repetidos
- ResNet: skip connections
- Inception: multi-scale

### 5. **Feature Maps**
- Salida de cada capa convolucional
- Múltiples canales/filtros
- Profundidad aumenta, tamaño espacial disminuye
- Visualización de activaciones

### 6. **Transfer Learning**
- Pre-training en ImageNet
- Fine-tuning para tarea específica
- Feature extraction con pesos congelados
- Domain adaptation

### 7. **Data Augmentation**
- Rotaciones, flips, crops
- Color jittering
- Mixup, CutMix
- Augmentación automática (AutoAugment)

## Recursos de Aprendizaje

### Cursos

1. **CS231n: CNN for Visual Recognition - Stanford**
   - https://cs231n.github.io/
   - El mejor curso de CNNs, notas excelentes

2. **Convolutional Neural Networks - Coursera**
   - https://www.coursera.org/learn/convolutional-neural-networks
   - Por Andrew Ng

### Libros

1. **"Deep Learning for Computer Vision"** - Rajalingappaa Shanmugamani
   - Práctico, con implementaciones

2. **"Dive into Deep Learning"** - Zhang et al.
   - Gratis: https://d2l.ai/
   - Capítulos sobre CNNs

### Papers Clave

1. **LeNet-5** - LeCun et al. (1998)
2. **AlexNet** - Krizhevsky et al. (2012)
3. **VGG** - Simonyan & Zisserman (2014)
4. **ResNet** - He et al. (2015)
5. **Inception** - Szegedy et al. (2015)

## Ejemplos Prácticos

### Ejemplo 1: Convolución Manual

```python
import numpy as np

def conv2d(image, kernel, stride=1, padding=0):
    """
    Convolución 2D simple
    """
    # Agregar padding
    if padding > 0:
        image = np.pad(image, padding, mode='constant')
    
    i_h, i_w = image.shape
    k_h, k_w = kernel.shape
    
    # Tamaño de salida
    out_h = (i_h - k_h) // stride + 1
    out_w = (i_w - k_w) // stride + 1
    
    output = np.zeros((out_h, out_w))
    
    # Aplicar convolución
    for i in range(0, out_h):
        for j in range(0, out_w):
            h_start = i * stride
            w_start = j * stride
            
            region = image[h_start:h_start+k_h, w_start:w_start+k_w]
            output[i, j] = np.sum(region * kernel)
    
    return output

# Ejemplo
image = np.array([
    [1, 2, 3, 4],
    [5, 6, 7, 8],
    [9, 10, 11, 12],
    [13, 14, 15, 16]
])

# Detecta bordes verticales
kernel_vertical = np.array([
    [-1, 0, 1],
    [-1, 0, 1],
    [-1, 0, 1]
])

result = conv2d(image, kernel_vertical, stride=1, padding=0)
print("Convolución:\n", result)
```

### Ejemplo 2: CNN Simple con PyTorch

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class SimpleCNN(nn.Module):
    def __init__(self, num_classes=10):
        super(SimpleCNN, self).__init__()
        
        # Convolutional layers
        self.conv1 = nn.Conv2d(1, 32, kernel_size=3, padding=1)
        self.conv2 = nn.Conv2d(32, 64, kernel_size=3, padding=1)
        self.conv3 = nn.Conv2d(64, 128, kernel_size=3, padding=1)
        
        # Pooling
        self.pool = nn.MaxPool2d(2, 2)
        
        # Fully connected
        self.fc1 = nn.Linear(128 * 3 * 3, 256)
        self.fc2 = nn.Linear(256, num_classes)
        
        # Dropout
        self.dropout = nn.Dropout(0.5)
    
    def forward(self, x):
        # Conv block 1: 28x28 -> 14x14
        x = self.pool(F.relu(self.conv1(x)))
        
        # Conv block 2: 14x14 -> 7x7
        x = self.pool(F.relu(self.conv2(x)))
        
        # Conv block 3: 7x7 -> 3x3
        x = self.pool(F.relu(self.conv3(x)))
        
        # Flatten
        x = x.view(-1, 128 * 3 * 3)
        
        # FC layers
        x = F.relu(self.fc1(x))
        x = self.dropout(x)
        x = self.fc2(x)
        
        return x

# Usar modelo
model = SimpleCNN(num_classes=10)
x = torch.randn(4, 1, 28, 28)  # Batch de 4 imágenes MNIST
output = model(x)
print(f"Output shape: {output.shape}")  # [4, 10]
```

### Ejemplo 3: ResNet Block

```python
class ResidualBlock(nn.Module):
    def __init__(self, in_channels, out_channels, stride=1):
        super(ResidualBlock, self).__init__()
        
        # Main path
        self.conv1 = nn.Conv2d(in_channels, out_channels, 
                               kernel_size=3, stride=stride, padding=1)
        self.bn1 = nn.BatchNorm2d(out_channels)
        
        self.conv2 = nn.Conv2d(out_channels, out_channels,
                               kernel_size=3, stride=1, padding=1)
        self.bn2 = nn.BatchNorm2d(out_channels)
        
        # Skip connection
        self.skip = nn.Sequential()
        if stride != 1 or in_channels != out_channels:
            self.skip = nn.Sequential(
                nn.Conv2d(in_channels, out_channels, 
                         kernel_size=1, stride=stride),
                nn.BatchNorm2d(out_channels)
            )
    
    def forward(self, x):
        # Main path
        out = F.relu(self.bn1(self.conv1(x)))
        out = self.bn2(self.conv2(out))
        
        # Skip connection
        out += self.skip(x)
        out = F.relu(out)
        
        return out

# Test
resblock = ResidualBlock(64, 128, stride=2)
x = torch.randn(4, 64, 32, 32)
out = resblock(x)
print(f"Input: {x.shape}, Output: {out.shape}")
```

### Ejemplo 4: Transfer Learning

```python
import torchvision.models as models

# Cargar modelo pre-entrenado
resnet50 = models.resnet50(pretrained=True)

# Congelar todas las capas
for param in resnet50.parameters():
    param.requires_grad = False

# Reemplazar última capa para nueva tarea
num_classes = 5
resnet50.fc = nn.Linear(resnet50.fc.in_features, num_classes)

# Solo entrenar última capa
optimizer = torch.optim.Adam(resnet50.fc.parameters(), lr=0.001)

# Feature extraction (sin fine-tuning)
resnet50.eval()
with torch.no_grad():
    features = resnet50(images)

# Fine-tuning (descongelar algunas capas)
for param in resnet50.layer4.parameters():
    param.requires_grad = True

optimizer = torch.optim.Adam(
    filter(lambda p: p.requires_grad, resnet50.parameters()),
    lr=0.0001
)
```

### Ejemplo 5: Data Augmentation

```python
from torchvision import transforms

# Training transforms
train_transform = transforms.Compose([
    transforms.RandomResizedCrop(224),
    transforms.RandomHorizontalFlip(p=0.5),
    transforms.RandomRotation(15),
    transforms.ColorJitter(brightness=0.2, contrast=0.2, 
                          saturation=0.2, hue=0.1),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406],
                        std=[0.229, 0.224, 0.225])
])

# Test transforms (no augmentation)
test_transform = transforms.Compose([
    transforms.Resize(256),
    transforms.CenterCrop(224),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406],
                        std=[0.229, 0.224, 0.225])
])

# Aplicar
from torchvision.datasets import ImageFolder

train_dataset = ImageFolder('path/to/train', transform=train_transform)
test_dataset = ImageFolder('path/to/test', transform=test_transform)
```

### Ejemplo 6: Visualizar Feature Maps

```python
def visualize_feature_maps(model, image):
    activation = {}
    
    def get_activation(name):
        def hook(model, input, output):
            activation[name] = output.detach()
        return hook
    
    # Registrar hooks
    model.conv1.register_forward_hook(get_activation('conv1'))
    model.conv2.register_forward_hook(get_activation('conv2'))
    
    # Forward pass
    output = model(image.unsqueeze(0))
    
    # Visualizar
    import matplotlib.pyplot as plt
    
    for layer_name, feature_maps in activation.items():
        n_features = min(16, feature_maps.shape[1])
        
        fig, axes = plt.subplots(4, 4, figsize=(12, 12))
        for i, ax in enumerate(axes.flat):
            if i < n_features:
                ax.imshow(feature_maps[0, i].cpu(), cmap='viridis')
                ax.axis('off')
        
        plt.suptitle(f'{layer_name} Feature Maps')
        plt.show()

# Uso
model = SimpleCNN()
image = torch.randn(1, 28, 28)
visualize_feature_maps(model, image)
```

### Ejemplo 7: Gradual Unfreezing

```python
def gradual_unfreezing(model, train_loader, num_epochs_per_stage=5):
    """
    Estrategia de unfreezing gradual para fine-tuning
    """
    # Stage 1: Solo última capa
    for param in model.parameters():
        param.requires_grad = False
    for param in model.fc.parameters():
        param.requires_grad = True
    
    optimizer = torch.optim.Adam(model.fc.parameters(), lr=0.001)
    train(model, train_loader, optimizer, num_epochs_per_stage)
    
    # Stage 2: Descongelar layer4
    for param in model.layer4.parameters():
        param.requires_grad = True
    
    optimizer = torch.optim.Adam(
        filter(lambda p: p.requires_grad, model.parameters()),
        lr=0.0001
    )
    train(model, train_loader, optimizer, num_epochs_per_stage)
    
    # Stage 3: Todo el modelo
    for param in model.parameters():
        param.requires_grad = True
    
    optimizer = torch.optim.Adam(model.parameters(), lr=0.00001)
    train(model, train_loader, optimizer, num_epochs_per_stage)
```

## Mejores Prácticas

### 1. Batch Normalization después de Conv

```python
# ✅ Bueno
self.conv1 = nn.Conv2d(3, 64, 3)
self.bn1 = nn.BatchNorm2d(64)

x = self.bn1(F.relu(self.conv1(x)))
```

### 2. Data Augmentation Apropiada

```python
# ✅ Para ImageNet/natural images
transforms.RandomResizedCrop(224)
transforms.RandomHorizontalFlip()
transforms.ColorJitter()

# ✅ Para medical images
transforms.RandomAffine(degrees=10, translate=(0.1, 0.1))
transforms.RandomHorizontalFlip()  # Cuidado con vertical flip
```

### 3. Learning Rate Scheduling

```python
# ✅ Bueno
scheduler = torch.optim.lr_scheduler.ReduceLROnPlateau(
    optimizer, mode='min', patience=5, factor=0.5
)

for epoch in range(num_epochs):
    train_loss = train(...)
    val_loss = validate(...)
    scheduler.step(val_loss)
```

## Proyectos Sugeridos

1. **Clasificador de Imágenes CIFAR-10**
   - CNN desde cero
   - Comparar arquitecturas
   - Data augmentation
   - Alcanzar >90% accuracy

2. **Transfer Learning para Dataset Custom**
   - ResNet50 pre-entrenado
   - Fine-tuning strategy
   - Grad-CAM para interpretabilidad

3. **Object Detection**
   - YOLOv5 o Faster R-CNN
   - Custom dataset annotation
   - mAP evaluation

4. **Image Segmentation**
   - U-Net para segmentación
   - Medical imaging o semantic segmentation
   - Dice coefficient metric

## Checklist de Aprendizaje

- [ ] Entender operación de convolución
- [ ] Implementar conv2d desde cero
- [ ] Dominar arquitecturas clásicas
- [ ] Aplicar transfer learning efectivamente
- [ ] Usar data augmentation apropiadamente
- [ ] Visualizar feature maps
- [ ] Optimizar hiperparámetros (kernel size, filters)
- [ ] Fine-tune modelos pre-entrenados
- [ ] Interpretar activaciones con Grad-CAM
- [ ] Deploy modelo en producción

## Próximos Pasos

1. Estudiar **Object Detection** (YOLO, Faster R-CNN)
2. Aprender **Image Segmentation** (U-Net, Mask R-CNN)
3. Explorar **Vision Transformers** (ViT)
4. Profundizar en **Neural Architecture Search**
5. Dominar **Model Interpretation** (Grad-CAM, LIME)

---
**Tiempo estimado**: 4-6 semanas
