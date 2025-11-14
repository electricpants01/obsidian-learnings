# Pandas - Análisis y Manipulación de Datos

## Descripción

Pandas es la biblioteca principal para análisis y manipulación de datos en Python. Proporciona estructuras de datos flexibles y eficientes (Series y DataFrames) diseñadas para trabajar con datos estructurados y series temporales. Es fundamental para cualquier tarea de ciencia de datos, machine learning y análisis estadístico.

## Conceptos Clave

### 1. **Series y DataFrames**
- Series: array unidimensional con índice
- DataFrame: tabla bidimensional con etiquetas
- Index: sistema de etiquetado
- Operaciones vectorizadas

### 2. **Indexing y Selección**
- loc: selección basada en etiquetas
- iloc: selección basada en posición
- Boolean indexing
- MultiIndex (índices jerárquicos)

### 3. **Data Cleaning**
- Manejo de valores faltantes (NaN)
- Duplicados
- Tipo de datos (dtypes)
- Conversión de tipos

### 4. **Transformaciones**
- Apply, map, applymap
- Groupby operations
- Pivot tables
- Merge, join, concatenate

### 5. **Time Series**
- DatetimeIndex
- Resampling
- Rolling windows
- Time zone handling

### 6. **I/O Operations**
- CSV, Excel, JSON, SQL
- Parquet, HDF5
- APIs y web scraping
- Optimización de lectura/escritura

### 7. **Visualización**
- Integración con Matplotlib
- Plot methods
- Visualizaciones básicas

## Recursos de Aprendizaje

### Documentación Oficial

1. **Pandas Documentation**
   - https://pandas.pydata.org/docs/
   - Referencia completa y guías

2. **10 Minutes to Pandas**
   - https://pandas.pydata.org/docs/user_guide/10min.html
   - Quick start oficial

3. **Pandas User Guide**
   - https://pandas.pydata.org/docs/user_guide/index.html

### Cursos Online

1. **DataCamp - Pandas Foundations** (Suscripción)
   - https://www.datacamp.com/courses/pandas-foundations
   - Curso interactivo completo

2. **Kaggle Learn - Pandas** (Gratis)
   - https://www.kaggle.com/learn/pandas
   - Micro-curso práctico

3. **Coursera - Applied Data Science with Python**
   - https://www.coursera.org/specializations/data-science-python
   - Universidad de Michigan

### Libros

1. **"Python for Data Analysis"** - Wes McKinney
   - Creador de Pandas
   - Libro definitivo
   - 3ra edición actualizada

2. **"Pandas 1.x Cookbook"** - Matt Harrison
   - Recetas prácticas
   - Casos de uso reales

3. **"Effective Pandas"** - Matt Harrison
   - Mejores prácticas
   - Código idiomático

### Videos Recomendados

1. **Corey Schafer - Pandas Tutorials**
   - https://www.youtube.com/playlist?list=PL-osiE80TeTsWmV9i9c58mdDCSskIFdDS
   - Serie completa

2. **Keith Galli - Complete Python Pandas Data Science Tutorial**
   - https://www.youtube.com/watch?v=vmEHCJofslg
   - Tutorial de 1 hora

3. **Data School - Pandas Q&A**
   - https://www.youtube.com/user/dataschool
   - Videos cortos y prácticos

## Instalación y Setup

```bash
# Instalación
pip install pandas

# Con dependencias opcionales
pip install pandas[all]

# Verificar instalación
python -c "import pandas as pd; print(pd.__version__)"

# Imports comunes
import pandas as pd
import numpy as np
```

## Ejemplos Prácticos

### Ejemplo 1: Crear DataFrames

```python
import pandas as pd
import numpy as np

# Desde diccionario
data = {
    'nombre': ['Ana', 'Luis', 'Carlos', 'María'],
    'edad': [25, 30, 35, 28],
    'ciudad': ['Madrid', 'Barcelona', 'Valencia', 'Sevilla'],
    'salario': [30000, 45000, 52000, 38000]
}
df = pd.DataFrame(data)
print(df)

# Desde lista de listas
data = [
    ['Ana', 25, 'Madrid'],
    ['Luis', 30, 'Barcelona'],
    ['Carlos', 35, 'Valencia']
]
df = pd.DataFrame(data, columns=['nombre', 'edad', 'ciudad'])

# Desde NumPy array
arr = np.random.randn(5, 3)
df = pd.DataFrame(arr, columns=['A', 'B', 'C'])

# Series
s = pd.Series([1, 2, 3, 4, 5], index=['a', 'b', 'c', 'd', 'e'])
print(s)

# Información del DataFrame
print(df.info())
print(df.describe())
print(df.head())
print(df.tail())
print(df.shape)
print(df.columns)
print(df.dtypes)
```

### Ejemplo 2: Indexing y Selección

```python
import pandas as pd

df = pd.DataFrame({
    'A': [1, 2, 3, 4],
    'B': [5, 6, 7, 8],
    'C': [9, 10, 11, 12]
}, index=['row1', 'row2', 'row3', 'row4'])

# Seleccionar columna
print(df['A'])  # Series
print(df[['A', 'B']])  # DataFrame

# loc: basado en etiquetas
print(df.loc['row1'])  # Fila
print(df.loc['row1', 'A'])  # Celda específica
print(df.loc['row1':'row3', ['A', 'C']])  # Rango

# iloc: basado en posición
print(df.iloc[0])  # Primera fila
print(df.iloc[0, 1])  # Primera fila, segunda columna
print(df.iloc[0:2, 1:3])  # Slicing

# Boolean indexing
print(df[df['A'] > 2])
print(df[(df['A'] > 1) & (df['B'] < 8)])

# Filtrado con query
print(df.query('A > 2 and B < 8'))

# Selección con at y iat (más rápido para valores individuales)
value = df.at['row1', 'A']
value = df.iat[0, 0]
```

### Ejemplo 3: Data Cleaning

```python
import pandas as pd
import numpy as np

# Crear DataFrame con datos faltantes
df = pd.DataFrame({
    'A': [1, 2, np.nan, 4, 5],
    'B': [np.nan, 2, 3, 4, 5],
    'C': [1, 2, 3, np.nan, 5]
})

# Detectar valores faltantes
print(df.isnull())
print(df.isnull().sum())

# Eliminar filas con NaN
df_dropped = df.dropna()  # Elimina filas con cualquier NaN
df_dropped = df.dropna(how='all')  # Solo si todos son NaN
df_dropped = df.dropna(thresh=2)  # Al menos 2 valores no-NaN

# Rellenar valores faltantes
df_filled = df.fillna(0)  # Con valor específico
df_filled = df.fillna(method='ffill')  # Forward fill
df_filled = df.fillna(method='bfill')  # Backward fill
df_filled = df.fillna(df.mean())  # Con la media

# Interpolar valores faltantes
df_interpolated = df.interpolate()

# Duplicados
df = pd.DataFrame({
    'A': [1, 2, 2, 3],
    'B': [4, 5, 5, 6]
})
print(df.duplicated())  # Boolean mask
df_unique = df.drop_duplicates()

# Cambiar tipos de datos
df['A'] = df['A'].astype('float')
df['B'] = pd.to_numeric(df['B'], errors='coerce')

# Renombrar columnas
df = df.rename(columns={'A': 'col_A', 'B': 'col_B'})

# Reemplazar valores
df = df.replace({1: 100, 2: 200})
```

### Ejemplo 4: Transformaciones y GroupBy

```python
import pandas as pd

# Sample data
df = pd.DataFrame({
    'category': ['A', 'B', 'A', 'B', 'A', 'B'],
    'value': [10, 20, 30, 40, 50, 60],
    'score': [1.1, 2.2, 3.3, 4.4, 5.5, 6.6]
})

# Apply function to column
df['value_squared'] = df['value'].apply(lambda x: x**2)

# Apply function to multiple columns
df['total'] = df.apply(lambda row: row['value'] + row['score'], axis=1)

# Map values
mapping = {'A': 'Category A', 'B': 'Category B'}
df['category_full'] = df['category'].map(mapping)

# GroupBy operations
grouped = df.groupby('category')

# Agregaciones
print(grouped.sum())
print(grouped.mean())
print(grouped.agg(['sum', 'mean', 'std']))

# Múltiples agregaciones por columna
print(grouped.agg({
    'value': ['sum', 'mean'],
    'score': ['min', 'max']
}))

# Transform (mantiene el shape original)
df['value_normalized'] = grouped['value'].transform(lambda x: (x - x.mean()) / x.std())

# Filter groups
filtered = grouped.filter(lambda x: x['value'].sum() > 50)

# Pivot table
pivot = df.pivot_table(
    values='value',
    index='category',
    aggfunc='mean'
)
print(pivot)
```

### Ejemplo 5: Merge, Join, Concatenate

```python
import pandas as pd

# Sample DataFrames
df1 = pd.DataFrame({
    'key': ['A', 'B', 'C', 'D'],
    'value1': [1, 2, 3, 4]
})

df2 = pd.DataFrame({
    'key': ['B', 'D', 'E', 'F'],
    'value2': [5, 6, 7, 8]
})

# Merge (similar a SQL JOIN)
# Inner join (default)
merged = pd.merge(df1, df2, on='key', how='inner')
print(merged)

# Left join
merged = pd.merge(df1, df2, on='key', how='left')

# Right join
merged = pd.merge(df1, df2, on='key', how='right')

# Outer join
merged = pd.merge(df1, df2, on='key', how='outer')

# Merge on index
df1_indexed = df1.set_index('key')
df2_indexed = df2.set_index('key')
joined = df1_indexed.join(df2_indexed, how='outer')

# Concatenate
# Vertical (rows)
df_concat = pd.concat([df1, df2], axis=0, ignore_index=True)

# Horizontal (columns)
df_concat = pd.concat([df1, df2], axis=1)

# Append (añadir filas)
df_appended = df1.append(df2, ignore_index=True)
```

### Ejemplo 6: Time Series

```python
import pandas as pd
import numpy as np

# Crear DatetimeIndex
dates = pd.date_range('2024-01-01', periods=100, freq='D')
ts = pd.Series(np.random.randn(100), index=dates)

# Selección por fecha
print(ts['2024-01'])  # Todo enero
print(ts['2024-01-15':'2024-01-20'])  # Rango

# Resampling
# Downsample (mayor frecuencia a menor)
monthly = ts.resample('M').mean()
weekly = ts.resample('W').sum()

# Upsample (menor frecuencia a mayor)
daily = monthly.resample('D').ffill()

# Rolling window
rolling_mean = ts.rolling(window=7).mean()
rolling_std = ts.rolling(window=7).std()

# Expanding window
expanding_mean = ts.expanding().mean()

# Shift (desplazar valores)
ts_shifted = ts.shift(1)  # Shift forward
ts_lag = ts.shift(-1)  # Shift backward

# Diferencias
ts_diff = ts.diff()  # Primera diferencia
ts_pct = ts.pct_change()  # Cambio porcentual

# Time zones
ts_utc = ts.tz_localize('UTC')
ts_ny = ts_utc.tz_convert('America/New_York')
```

### Ejemplo 7: I/O Operations

```python
import pandas as pd

# CSV
df = pd.read_csv('data.csv')
df = pd.read_csv('data.csv', sep=';', encoding='utf-8', parse_dates=['date'])
df.to_csv('output.csv', index=False)

# Excel
df = pd.read_excel('data.xlsx', sheet_name='Sheet1')
df.to_excel('output.xlsx', sheet_name='Results', index=False)

# JSON
df = pd.read_json('data.json')
df.to_json('output.json', orient='records')

# SQL
import sqlite3
conn = sqlite3.connect('database.db')
df = pd.read_sql('SELECT * FROM table', conn)
df.to_sql('new_table', conn, if_exists='replace', index=False)

# Parquet (formato columnar eficiente)
df = pd.read_parquet('data.parquet')
df.to_parquet('output.parquet', compression='gzip')

# Clipboard
df = pd.read_clipboard()
df.to_clipboard()

# HTML
tables = pd.read_html('https://example.com/table.html')
df = tables[0]

# Multiple files
import glob
files = glob.glob('data_*.csv')
df_list = [pd.read_csv(f) for f in files]
df_combined = pd.concat(df_list, ignore_index=True)

# Optimización de lectura
df = pd.read_csv('large_file.csv',
                 usecols=['col1', 'col2'],  # Solo columnas necesarias
                 dtype={'col1': 'category'},  # Especificar tipos
                 chunksize=10000)  # Leer en chunks
```

## Optimización y Performance

### Memory Optimization

```python
import pandas as pd

# Reducir uso de memoria
def reduce_mem_usage(df):
    """Reduce memory usage of DataFrame"""
    start_mem = df.memory_usage().sum() / 1024**2
    
    for col in df.columns:
        col_type = df[col].dtype
        
        if col_type != object:
            c_min = df[col].min()
            c_max = df[col].max()
            
            if str(col_type)[:3] == 'int':
                if c_min > np.iinfo(np.int8).min and c_max < np.iinfo(np.int8).max:
                    df[col] = df[col].astype(np.int8)
                elif c_min > np.iinfo(np.int16).min and c_max < np.iinfo(np.int16).max:
                    df[col] = df[col].astype(np.int16)
                elif c_min > np.iinfo(np.int32).min and c_max < np.iinfo(np.int32).max:
                    df[col] = df[col].astype(np.int32)
            else:
                if c_min > np.finfo(np.float16).min and c_max < np.finfo(np.float16).max:
                    df[col] = df[col].astype(np.float16)
                elif c_min > np.finfo(np.float32).min and c_max < np.finfo(np.float32).max:
                    df[col] = df[col].astype(np.float32)
    
    end_mem = df.memory_usage().sum() / 1024**2
    print(f'Memory usage decreased from {start_mem:.2f}MB to {end_mem:.2f}MB')
    return df

# Usar categorías para strings repetitivos
df['category'] = df['category'].astype('category')

# Verificar uso de memoria
print(df.memory_usage(deep=True))
print(df.info(memory_usage='deep'))
```

### Vectorización

```python
# ❌ Malo: usar loops
total = 0
for value in df['column']:
    total += value * 2

# ✅ Bueno: operaciones vectorizadas
total = (df['column'] * 2).sum()

# ❌ Malo: apply con loop implícito
df['result'] = df.apply(lambda row: row['A'] + row['B'], axis=1)

# ✅ Bueno: operaciones vectorizadas
df['result'] = df['A'] + df['B']
```

## Casos de Uso en ML/AI

### Feature Engineering

```python
import pandas as pd
from sklearn.preprocessing import LabelEncoder, StandardScaler

# One-Hot Encoding
df_encoded = pd.get_dummies(df, columns=['category'], prefix='cat')

# Label Encoding
le = LabelEncoder()
df['category_encoded'] = le.fit_transform(df['category'])

# Binning
df['age_group'] = pd.cut(df['age'], bins=[0, 18, 35, 50, 100], 
                         labels=['young', 'adult', 'middle', 'senior'])

# Date features
df['date'] = pd.to_datetime(df['date'])
df['year'] = df['date'].dt.year
df['month'] = df['date'].dt.month
df['day_of_week'] = df['date'].dt.dayofweek
df['is_weekend'] = df['date'].dt.dayofweek.isin([5, 6])

# Rolling features
df['rolling_mean_7d'] = df['value'].rolling(window=7).mean()
df['rolling_std_7d'] = df['value'].rolling(window=7).std()

# Lag features
df['value_lag_1'] = df['value'].shift(1)
df['value_lag_7'] = df['value'].shift(7)
```

### Train-Test Split

```python
from sklearn.model_selection import train_test_split

# Separar features y target
X = df.drop('target', axis=1)
y = df['target']

# Split
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)
```

## Mejores Prácticas

### 1. Evitar Loops

```python
# ❌ Malo
for idx, row in df.iterrows():
    df.at[idx, 'new_col'] = row['A'] + row['B']

# ✅ Bueno
df['new_col'] = df['A'] + df['B']
```

### 2. Method Chaining

```python
# ✅ Bueno: method chaining
result = (df
    .query('age > 18')
    .groupby('category')
    .agg({'value': 'mean'})
    .sort_values('value', ascending=False)
    .head(10)
)
```

### 3. Usar Categorías

```python
# Para columnas con valores repetidos
df['status'] = df['status'].astype('category')
```

### 4. Copy vs View

```python
# Siempre usar .copy() cuando quieras un DataFrame independiente
df_filtered = df[df['A'] > 0].copy()
```

## Proyectos Sugeridos

1. **Data Analysis Dashboard**
   - Analizar dataset de Kaggle
   - Generar insights
   - Visualizaciones

2. **Time Series Analysis**
   - Stock prices o weather data
   - Forecasting
   - Seasonal decomposition

3. **Data Cleaning Pipeline**
   - Pipeline automatizado
   - Manejo de missing values
   - Outlier detection

4. **ETL Pipeline**
   - Extract de múltiples fuentes
   - Transform data
   - Load a database

5. **Customer Segmentation**
   - RFM analysis
   - Clustering
   - Perfiles de clientes

## Recursos Adicionales

### Cheat Sheets

1. **Pandas Cheat Sheet**
   - https://pandas.pydata.org/Pandas_Cheat_Sheet.pdf

2. **DataCamp Pandas Cheat Sheet**
   - https://www.datacamp.com/cheat-sheet/pandas-cheat-sheet-for-data-science-in-python

### Blogs y Tutoriales

1. **Real Python - Pandas Tutorials**
   - https://realpython.com/learning-paths/pandas-data-science/

2. **Towards Data Science - Pandas**
   - https://towardsdatascience.com/tagged/pandas

### Comunidades

1. **Stack Overflow - Pandas Tag**
   - https://stackoverflow.com/questions/tagged/pandas

2. **Reddit - r/pandas**
   - https://www.reddit.com/r/pandas/

## Funciones Más Usadas

```python
# Creación
pd.DataFrame(), pd.Series(), pd.read_csv(), pd.read_excel()

# Inspección
df.head(), df.tail(), df.info(), df.describe(), df.shape, df.columns

# Selección
df['col'], df[['col1', 'col2']], df.loc[], df.iloc[], df.query()

# Limpieza
df.dropna(), df.fillna(), df.drop_duplicates(), df.replace()

# Transformación
df.groupby(), df.apply(), df.map(), df.pivot_table()

# Combinación
pd.merge(), pd.concat(), df.join(), df.append()

# Time Series
pd.date_range(), df.resample(), df.rolling(), df.shift()

# I/O
df.to_csv(), df.to_excel(), df.to_json(), df.to_sql()
```

## Checklist de Aprendizaje

- [ ] Crear y manipular DataFrames
- [ ] Dominar indexing (loc, iloc, boolean)
- [ ] Limpiar datos (NaN, duplicados)
- [ ] Usar groupby eficientemente
- [ ] Realizar merges y joins
- [ ] Trabajar con time series
- [ ] Optimizar performance
- [ ] Leer/escribir diferentes formatos
- [ ] Aplicar en proyecto de ML
- [ ] Usar method chaining efectivamente

## Próximos Pasos

Después de dominar Pandas:
1. Profundizar en **SQL** para bases de datos
2. Aprender **Matplotlib/Seaborn** para visualización avanzada
3. Usar Pandas con **scikit-learn** para ML
4. Explorar **Dask** para datasets grandes

---
**Tiempo estimado de estudio**: 3-4 semanas dedicando 2-3 horas diarias
