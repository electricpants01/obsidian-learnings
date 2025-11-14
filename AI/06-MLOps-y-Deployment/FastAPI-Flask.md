# FastAPI y Flask - Web Frameworks para ML APIs

## Descripción

FastAPI y Flask son frameworks Python para crear APIs web, críticos para serving ML models. FastAPI: moderno, async, auto-documentation (OpenAPI), type hints, rápido (comparable a Node.js). Flask: maduro, simple, flexible, gran ecosistema. ML use cases: model serving, batch predictions, webhooks. FastAPI preferido para nuevos proyectos (performance, async, validación automática). Flask para sistemas legacy o simplicidad. Ambos integran fácil con scikit-learn, PyTorch, TensorFlow. Deployment: Docker + Gunicorn/Uvicorn. Testing con pytest. Authentication (JWT), CORS, rate limiting críticos producción.

## Conceptos

**Endpoints**: Routes para requests
**Pydantic**: Data validation (FastAPI)
**Async**: Concurrency mejorada
**Middleware**: Pre/post processing
**Dependencies**: Injection pattern

## Ejemplos

### FastAPI ML API

```python
from fastapi import FastAPI
from pydantic import BaseModel
import joblib

app = FastAPI()
model = joblib.load('model.pkl')

class PredictionInput(BaseModel):
    features: list[float]

class PredictionOutput(BaseModel):
    prediction: float

@app.post("/predict", response_model=PredictionOutput)
async def predict(input_data: PredictionInput):
    prediction = model.predict([input_data.features])[0]
    return {"prediction": prediction}

@app.get("/health")
async def health():
    return {"status": "healthy"}
```

### Flask ML API

```python
from flask import Flask, request, jsonify
import joblib

app = Flask(__name__)
model = joblib.load('model.pkl')

@app.route('/predict', methods=['POST'])
def predict():
    data = request.json
    prediction = model.predict([data['features']])[0]
    return jsonify({'prediction': float(prediction)})

@app.route('/health')
def health():
    return jsonify({'status': 'healthy'})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8000)
```

### Testing

```python
from fastapi.testclient import TestClient

client = TestClient(app)

def test_predict():
    response = client.post("/predict", json={"features": [1.0, 2.0, 3.0]})
    assert response.status_code == 200
    assert "prediction" in response.json()
```

## Recursos

**FastAPI**: https://fastapi.tiangolo.com/
**Flask**: https://flask.palletsprojects.com/

---
**Tiempo**: 2 semanas
