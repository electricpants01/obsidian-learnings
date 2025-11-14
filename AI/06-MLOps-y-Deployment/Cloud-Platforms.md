# Cloud Platforms - AWS, GCP, Azure

## Descripción

Cloud providers ofrecen infraestructura ML escalable: compute (EC2, Compute Engine, VMs), storage (S3, GCS, Blob), managed ML (SageMaker, Vertex AI, Azure ML). Benefits: pay-as-you-go, auto-scaling, managed services, global infrastructure. ML services: training (GPUs/TPUs), inference endpoints, AutoML, notebooks (SageMaker Studio, Vertex AI Workbench). Data services: warehouses (BigQuery, Redshift), lakes (S3+Glue, GCS+Dataflow). IAM crítico para security. Cost optimization: spot instances, reserved capacity, rightsizing.

## Servicios ML

**AWS**: SageMaker (end-to-end ML), EC2 (flexible compute), Lambda (serverless)
**GCP**: Vertex AI (unified ML), AI Platform, BigQuery ML
**Azure**: Azure ML, Databricks, Synapse Analytics

## Ejemplo SageMaker

```python
import sagemaker
from sagemaker.sklearn import SKLearn

# Training
sklearn_estimator = SKLearn(
    entry_point='train.py',
    role=role,
    instance_type='ml.m5.xlarge',
    framework_version='0.23-1'
)
sklearn_estimator.fit({'train': 's3://bucket/data'})

# Deploy
predictor = sklearn_estimator.deploy(
    initial_instance_count=1,
    instance_type='ml.m5.large'
)
```

---
**Tiempo**: 3-4 semanas
