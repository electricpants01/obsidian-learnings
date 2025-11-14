# DVC - Data Version Control

## Descripción
DVC versiona datasets y modelos como Git para código: track grandes archivos, reproducibilidad, pipelines. Remote storage: S3, GCS, Azure. Benefits: compartir data sin subir a Git, reproduce experiments, track model versions. Integra con Git.

## Uso
```bash
dvc init
dvc add data/dataset.csv
git add data/dataset.csv.dvc .gitignore
dvc remote add -d storage s3://mybucket/dvc
dvc push
dvc pull
```
---
**Tiempo**: 1 semana
