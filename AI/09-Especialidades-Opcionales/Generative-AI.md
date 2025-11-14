# Generative AI - IA Generativa

## Descripción
Generative AI crea contenido nuevo: text (GPT), images (DALL-E, Stable Diffusion), audio, video. Técnicas: GANs, VAEs, Diffusion models, Transformers. Casos: content creation, data augmentation, synthetic data, art. Ethics: deepfakes, copyright, bias.

## Ejemplo
```python
from diffusers import StableDiffusionPipeline
pipe = StableDiffusionPipeline.from_pretrained("runwayml/stable-diffusion-v1-5")
image = pipe("A fantasy landscape").images[0]
```
---
**Tiempo**: 3-4 semanas
