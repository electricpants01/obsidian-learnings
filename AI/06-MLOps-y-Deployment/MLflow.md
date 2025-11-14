# MLflow - Experiment Tracking & Model Management

## Descripción

MLflow gestiona el ciclo de vida ML: tracking experiments, reproducibility, model registry, deployment. Componentes: Tracking (log params/metrics), Projects (formato reproducible), Models (packaging estándar), Registry (versionado centralizado). Benefits: comparar experiments, reproducir runs, deploy modelos, colaboración equipo. Integra con PyTorch, TensorFlow, scikit-learn, Keras. UI web para visualizar. Critical para MLOps: sin tracking manual, versionado automático, roll-back fácil. Alternatives: Weights & Biases, Neptune, ClearML. Open-source con version enterprise.

## Conceptos

**Experiment**: Agrupa runs relacionados
**Run**: Ejecución única con params/metrics  
**Model Registry**: Versionado centralizado
**Artifacts**: Outputs (models, plots)
**Autologging**: Logging automático

## Ejemplos

```python
import mlflow
import mlflow.sklearn
from sklearn.ensemble import RandomForestClassifier

# Start run
with mlflow.start_run():
    # Log params
    mlflow.log_param("n_estimators", 100)
    mlflow.log_param("max_depth", 10)
    
    # Train model
    model = RandomForestClassifier(n_estimators=100, max_depth=10)
    model.fit(X_train, y_train)
    
    # Log metrics
    accuracy = model.score(X_test, y_test)
    mlflow.log_metric("accuracy", accuracy)
    
    # Log model
    mlflow.sklearn.log_model(model, "model")

# Autologging
mlflow.sklearn.autolog()
model.fit(X_train, y_train)  # Auto logs todo

# Load model
model = mlflow.sklearn.load_model("runs:/<run_id>/model")

# Model Registry
mlflow.register_model("runs:/<run_id>/model", "MyModel")
```

## Recursos

**Docs**: https://mlflow.org/docs/latest/index.html

---
**Tiempo**: 2 semanas
