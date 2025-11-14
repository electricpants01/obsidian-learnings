# Monitoring - Model & System Observability

## Descripción

Monitoring detecta model drift, performance degradation, system issues post-deployment. Métricas ML: accuracy, precision, recall, latency, throughput. Infrastructure: CPU/GPU, memory, disk. Data drift: distribución input changes. Concept drift: relación X-y changes. Tools: Prometheus+Grafana (metrics), ELK stack (logs), Evidently AI, Whylabs (ML-specific). Alerting crítico: performance drops, high latency, errors. Dashboards para business stakeholders. Retraining triggers cuando drift detectado.

## Conceptos

**Data Drift**: Input distribution changes
**Concept Drift**: Target relationship changes
**Performance Metrics**: Accuracy, latency
**Logging**: Request/response tracking
**Alerts**: Automated notifications

## Ejemplo Prometheus + Grafana

```python
from prometheus_client import Counter, Histogram, start_http_server
import time

# Metrics
prediction_counter = Counter('predictions_total', 'Total predictions')
prediction_latency = Histogram('prediction_latency_seconds', 'Prediction latency')

@app.post("/predict")
async def predict(data: InputData):
    start = time.time()
    
    result = model.predict(data.features)
    
    # Track metrics
    prediction_counter.inc()
    prediction_latency.observe(time.time() - start)
    
    return {"prediction": result}

# Start metrics server
start_http_server(8001)
```

## Drift Detection

```python
from evidently.metric_preset import DataDriftPreset
from evidently.report import Report

report = Report(metrics=[DataDriftPreset()])
report.run(reference_data=train_df, current_data=production_df)
report.save_html("drift_report.html")
```

---
**Tiempo**: 2 semanas
