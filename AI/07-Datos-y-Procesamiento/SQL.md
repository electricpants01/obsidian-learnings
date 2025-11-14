# SQL - Structured Query Language

## Descripción

SQL es fundamental para data engineering y ML: extraer datos, transformaciones, feature engineering, análisis exploratorio. Databases relacionales: PostgreSQL, MySQL, SQL Server. Data warehouses: BigQuery, Redshift, Snowflake. Operaciones críticas: SELECT (queries), JOIN (combinar tables), GROUP BY (agregaciones), window functions, CTEs. ML use: preparar training data, feature extraction, data validation. ORMs (SQLAlchemy) para Python integration. Query optimization crítica para grandes datasets.

## Conceptos

**SELECT**: Retrieve data
**JOIN**: Combine tables (INNER, LEFT, RIGHT, FULL)
**GROUP BY**: Aggregations
**Window Functions**: Analytic queries
**CTEs**: Common Table Expressions
**Indexes**: Performance optimization

## Ejemplos

```sql
-- Basic query
SELECT customer_id, SUM(amount) as total_spent
FROM orders
WHERE order_date >= '2024-01-01'
GROUP BY customer_id
HAVING SUM(amount) > 1000;

-- JOIN
SELECT o.order_id, c.name, p.product_name
FROM orders o
JOIN customers c ON o.customer_id = c.id
JOIN products p ON o.product_id = p.id;

-- Window function
SELECT 
    customer_id,
    order_date,
    amount,
    SUM(amount) OVER (PARTITION BY customer_id ORDER BY order_date) as running_total
FROM orders;

-- CTE
WITH high_value_customers AS (
    SELECT customer_id, SUM(amount) as total
    FROM orders
    GROUP BY customer_id
    HAVING SUM(amount) > 10000
)
SELECT * FROM high_value_customers;
```

## Python Integration

```python
import pandas as pd
from sqlalchemy import create_engine

engine = create_engine('postgresql://user:pass@localhost/dbname')

# Query to DataFrame
df = pd.read_sql("SELECT * FROM customers WHERE active = true", engine)

# Write DataFrame to SQL
df.to_sql('predictions', engine, if_exists='append', index=False)
```

---
**Tiempo**: 2-3 semanas
