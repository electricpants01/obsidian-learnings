# Computer Vision - Visión por Computadora

## Descripción
CV analiza imágenes/videos: object detection, segmentation, face recognition, OCR. Arquitecturas: CNNs (ResNet, EfficientNet), Vision Transformers. Libraries: OpenCV, PIL, torchvision. Frameworks: YOLO, Detectron2. Casos: autonomous vehicles, medical imaging, surveillance.

## Ejemplo
```python
from torchvision import models, transforms
model = models.resnet50(pretrained=True)
img = transforms.Compose([transforms.Resize(256), transforms.ToTensor()])(image)
output = model(img.unsqueeze(0))
```
---
**Tiempo**: 4-6 semanas
