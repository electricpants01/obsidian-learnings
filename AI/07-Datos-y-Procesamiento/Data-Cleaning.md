# Data Cleaning - Preparación de Datos

## Descripción

Data cleaning es 80% del trabajo ML: manejar missing values, outliers, duplicates, inconsistencies. Técnicas: imputation (mean/median/mode, KNN), outlier detection (IQR, Z-score, isolation forest), deduplication, normalization. Libraries: Pandas, NumPy. Critical: domain knowledge, no eliminar información valiosa, documentar decisiones, mantener raw data separado.

## Técnicas

```python
import pandas as pd
import numpy as np

# Missing values
df['age'].fillna(df['age'].median(), inplace=True)
df.dropna(subset=['critical_col'], inplace=True)

# Outliers (IQR method)
Q1 = df['value'].quantile(0.25)
Q3 = df['value'].quantile(0.75)
IQR = Q3 - Q1
df = df[(df['value'] >= Q1 - 1.5*IQR) & (df['value'] <= Q3 + 1.5*IQR)]

# Duplicates
df.drop_duplicates(subset=['id'], inplace=True)

# Normalization
from sklearn.preprocessing import StandardScaler
scaler = StandardScaler()
df[['feature1', 'feature2']] = scaler.fit_transform(df[['feature1', 'feature2']])
```

---
**Tiempo**: 2 semanas
