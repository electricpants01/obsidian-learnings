# Model Serving - Production Inference

## Descripción

Model serving expone modelos para inference en producción. Opciones: REST API (FastAPI/Flask), gRPC (performance), batch (scheduled jobs), streaming (Kafka). Specialized frameworks: TensorFlow Serving, TorchServe, Triton Inference Server, BentoML. Optimization: quantization, ONNX export, model compression, caching. Scaling: load balancing, auto-scaling, GPU optimization. Edge deployment: TensorFlow Lite, ONNX Runtime Mobile. Critical: latency SLAs, throughput, cost optimization, versioning, A/B testing.

## Opciones

**TensorFlow Serving**: Production-ready, gRPC/REST
**TorchServe**: PyTorch oficial
**Triton**: NVIDIA, multi-framework
**BentoML**: Python-friendly, containerization automática
**Custom API**: FastAPI/Flask para flexibilidad

## Ejemplo TorchServe

```bash
# Package model
torch-model-archiver --model-name resnet18 \
  --version 1.0 \
  --model-file model.py \
  --serialized-file model.pth \
  --handler image_classifier

# Start server
torchserve --start --model-store model_store --models resnet18=resnet18.mar

# Inference
curl -X POST http://localhost:8080/predictions/resnet18 -T image.jpg
```

## BentoML Example

```python
import bentoml
from bentoml.io import JSON

@bentoml.service(name="iris_classifier")
class IrisClassifier:
    @bentoml.api
    def predict(self, input_data: JSON) -> JSON:
        model = bentoml.sklearn.load_model("iris_model:latest")
        prediction = model.predict([input_data["features"]])
        return {"class": int(prediction[0])}

# Build & serve
bentoml build
bentoml containerize iris_classifier:latest
docker run -p 3000:3000 iris_classifier:latest
```

---
**Tiempo**: 2-3 semanas
