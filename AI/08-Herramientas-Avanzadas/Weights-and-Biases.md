# Weights & Biases - Experiment Tracking

## Descripción
W&B: experiment tracking avanzado, visualización, hyperparameter tuning (Sweeps), model registry. Mejor UI que MLflow, colaboración team. Free para personal. Autologging frameworks populares.

## Uso
```python
import wandb
wandb.init(project="my-project")
wandb.config.lr = 0.01
for epoch in range(10):
    loss = train()
    wandb.log({"loss": loss, "epoch": epoch})
wandb.finish()
```
---
**Tiempo**: 1 semana
