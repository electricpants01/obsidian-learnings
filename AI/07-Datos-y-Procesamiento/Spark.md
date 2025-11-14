# Apache Spark - Big Data Processing

## Descripción

Spark procesa big data distribuido: terabytes/petabytes. RDD (Resilient Distributed Datasets), DataFrames, SQL API. ML con MLlib. Ventajas vs Hadoop: in-memory (100x faster), APIs Python/Scala/Java. Use cases: batch processing, streaming (Spark Streaming), ML training a escala, ETL pipelines. PySpark para ML engineers. Deployment: standalone, YARN, Kubernetes, Databricks. Critical para datasets que no caben en memoria

.

## Ejemplo

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("ML").getOrCreate()

# Read data
df = spark.read.parquet("s3://bucket/data")

# Transform
df_agg = df.groupBy("user_id").agg({"amount": "sum"})

# ML
from pyspark.ml.classification import LogisticRegression

lr = LogisticRegression(featuresCol="features", labelCol="label")
model = lr.fit(train_df)
predictions = model.transform(test_df)
```

---
**Tiempo**: 3-4 semanas
