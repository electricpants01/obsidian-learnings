# Airflow - Workflow Orchestration

## Descripción
Airflow orquesta data pipelines complejos: DAGs (Directed Acyclic Graphs), scheduling, monitoring, retry logic. ML use: training pipelines, feature engineering, batch predictions, data refresh. Alternative: Prefect, Dagster. Operators: PythonOperator, BashOperator, custom.

## DAG Example
```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime

with DAG('ml_training', start_date=datetime(2024,1,1), schedule='@daily') as dag:
    extract = PythonOperator(task_id='extract', python_callable=extract_data)
    train = PythonOperator(task_id='train', python_callable=train_model)
    deploy = PythonOperator(task_id='deploy', python_callable=deploy_model)
    extract >> train >> deploy
```
---
**Tiempo**: 2 semanas
