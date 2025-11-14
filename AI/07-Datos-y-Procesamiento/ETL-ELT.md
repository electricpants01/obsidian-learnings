# ETL/ELT - Data Pipelines

## Descripción

ETL (Extract-Transform-Load) y ELT (Extract-Load-Transform) construyen data pipelines. ETL: transform antes de load (traditional data warehouses). ELT: load raw data primero, transform después (modern data lakes, BigQuery). Tools: Airflow (orchestration), dbt (transformations), Fivetran/Airbyte (ingestion). ML pipelines: feature extraction, data validation, incremental updates. Critical: idempotency, monitoring, error handling, scheduling.

## Tools

**Airflow**: Workflow orchestration, Python DAGs
**dbt**: SQL transformations, version control
**Prefect/Dagster**: Modern alternatives
**Fivetran**: Managed connectors

## Airflow Example

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime

def extract():
    # Extract data
    pass

def transform():
    # Transform data
    pass

def load():
    # Load to warehouse
    pass

with DAG('ml_pipeline', start_date=datetime(2024, 1, 1), schedule='@daily') as dag:
    extract_task = PythonOperator(task_id='extract', python_callable=extract)
    transform_task = PythonOperator(task_id='transform', python_callable=transform)
    load_task = PythonOperator(task_id='load', python_callable=load)
    
    extract_task >> transform_task >> load_task
```

---
**Tiempo**: 2-3 semanas
