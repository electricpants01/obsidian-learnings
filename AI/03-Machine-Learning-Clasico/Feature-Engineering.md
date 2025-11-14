# Feature Engineering - Ingeniería de Características

## Descripción

Feature Engineering es el proceso de transformar datos crudos en features (características) que mejoren significativamente el rendimiento de modelos de machine learning. Es frecuentemente más importante que la elección del algoritmo mismo - un modelo simple con excelentes features supera a un modelo complejo con features pobres. El proceso incluye: creación de features desde datos existentes, transformación para cumplir assumptions del modelo, selección de las más relevantes, y encoding de variables categóricas. Feature engineering requiere conocimiento del dominio para identificar qué información es predictiva. Por ejemplo, en predicción de precios de casas, crear features como "años desde construcción" o "ratio precio/metro cuadrado" puede ser más útil que usar fecha de construcción o precio bruto. En series temporales, features de lag y rolling statistics capturan patrones temporales. En NLP, TF-IDF extrae información de texto. El feature engineering efectivo combina creatividad, conocimiento del dominio, análisis exploratorio, y experimentación iterativa. Dominar estas técnicas separa a data scientists principiantes de expertos.

## Conceptos Clave

### 1. **Feature Scaling**
- StandardScaler (z-score normalization)
- MinMaxScaler (0-1 scaling)
- RobustScaler (usa mediana/IQR, robusto a outliers)
- Normalizer (L1/L2 normalization)
- Crítico para distance-based algorithms

### 2. **Encoding Categórico**
- One-Hot Encoding para nominales
- Label Encoding para ordinales
- Target Encoding (mean encoding)
- Frequency Encoding
- Binary Encoding para alta cardinalidad

### 3. **Feature Creation**
- Polynomial features (interacciones)
- Domain-specific features
- Agregaciones y estadísticas
- Ratios y diferencias
- Binning/discretización

### 4. **Feature Selection**
- Filter methods (correlación, mutual information)
- Wrapper methods (RFE)
- Embedded methods (L1 regularization, feature importance)
- Variance threshold
- Remove multicollinearity

### 5. **Temporal Features**
- Date decomposition (año, mes, día, día semana)
- Lag features
- Rolling statistics (mean, std, min, max)
- Time since event
- Seasonal indicators

### 6. **Text Features**
- Bag of Words
- TF-IDF
- N-grams
- Word embeddings (Word2Vec, GloVe)
- Sentiment scores

### 7. **Missing Data Handling**
- Simple imputation (mean, median, mode)
- Indicator variables
- KNN imputation
- MICE (Multiple Imputation)
- Forward/backward fill para series temporales

## Recursos de Aprendizaje

### Libros

1. **"Feature Engineering for Machine Learning"** - Alice Zheng & Amanda Casari
   - O'Reilly, 100% dedicado al tema
   - Ejemplos prácticos extensos
   - Casos de uso reales

2. **"Feature Engineering and Selection"** - Max Kuhn & Kjell Johnson  
   - Gratis: http://www.feat.engineering/
   - Enfoque estadístico profundo
   - Ejemplos en R pero conceptos universales

3. **"Hands-On Machine Learning"** - Aurélien Géron
   - Capítulos sobre feature engineering
   - Integrado con proyectos completos

### Cursos

1. **Feature Engineering - Kaggle Learn** (Gratis)
   - https://www.kaggle.com/learn/feature-engineering
   - Práctico, enfocado en competiciones

2. **Feature Engineering for Machine Learning - Udacity** 
   - https://www.udacity.com/course/feature-engineering--ud732

3. **Feature Engineering for Machine Learning - Coursera**
   - https://www.coursera.org/learn/feature-engineering

### Videos

1. **Kaggle - Feature Engineering Videos**
   - https://www.youtube.com/c/Kaggle
   - Casos de competiciones reales

2. **Data Science Dojo - Feature Engineering**
   - https://www.youtube.com/c/Datasciencedojo

## Ejemplos Prácticos

### Ejemplo 1: Feature Scaling

```python
from sklearn.preprocessing import (
    StandardScaler, MinMaxScaler, RobustScaler, Normalizer
)
import numpy as np
import pandas as pd

# Datos de ejemplo
data = pd.DataFrame({
    'age': [25, 30, 35, 40, 45],
    'salary': [30000, 45000, 52000, 65000, 80000],
    'years_exp': [2, 5, 7, 10, 15]
})

# StandardScaler (z-score)
scaler_std = StandardScaler()
data_std = pd.DataFrame(
    scaler_std.fit_transform(data),
    columns=data.columns
)
print("Standard Scaled:")
print(data_std.describe())

# MinMaxScaler (0-1)
scaler_minmax = MinMaxScaler()
data_minmax = pd.DataFrame(
    scaler_minmax.fit_transform(data),
    columns=data.columns
)
print("\nMinMax Scaled:")
print(data_minmax.describe())

# RobustScaler (robusto a outliers)
scaler_robust = RobustScaler()
data_robust = pd.DataFrame(
    scaler_robust.fit_transform(data),
    columns=data.columns
)

# Comparar con outlier
data_with_outlier = data.copy()
data_with_outlier.loc[5] = [25, 1000000, 2]  # Outlier en salary

print("\nWith outlier - Standard vs Robust:")
print("Standard:", StandardScaler().fit_transform(data_with_outlier)[:, 1][:3])
print("Robust:", RobustScaler().fit_transform(data_with_outlier)[:, 1][:3])
```

### Ejemplo 2: Encoding Categórico

```python
from sklearn.preprocessing import OneHotEncoder, LabelEncoder
import category_encoders as ce  # pip install category-encoders

# Datos de ejemplo
df = pd.DataFrame({
    'city': ['NYC', 'LA', 'Chicago', 'NYC', 'LA', 'NYC'],
    'color': ['red', 'blue', 'red', 'green', 'blue', 'red'],
    'size': ['S', 'M', 'L', 'M', 'S', 'L'],  # Ordinal
    'price': [100, 150, 200, 120, 140, 180]
})

# One-Hot Encoding
df_onehot = pd.get_dummies(df, columns=['city', 'color'], prefix=['city', 'color'])
print("One-Hot Encoded:")
print(df_onehot.head())

# Label Encoding (para ordinales)
label_encoder = LabelEncoder()
df['size_encoded'] = label_encoder.fit_transform(df['size'])
print("\nLabel Encoded (ordinal):")
print(df[['size', 'size_encoded']])

# Target Encoding (mean encoding)
target_encoder = ce.TargetEncoder(cols=['city'])
df['city_target_encoded'] = target_encoder.fit_transform(df['city'], df['price'])
print("\nTarget Encoded:")
print(df[['city', 'price', 'city_target_encoded']])

# Frequency Encoding
freq_encoding = df['color'].value_counts().to_dict()
df['color_freq'] = df['color'].map(freq_encoding)
print("\nFrequency Encoded:")
print(df[['color', 'color_freq']])

# Binary Encoding (para alta cardinalidad)
binary_encoder = ce.BinaryEncoder(cols=['city'])
df_binary = binary_encoder.fit_transform(df['city'])
print("\nBinary Encoded:")
print(df_binary)
```

### Ejemplo 3: Feature Creation

```python
from sklearn.preprocessing import PolynomialFeatures

# Datos de ejemplo
df = pd.DataFrame({
    'length': [10, 15, 20, 25, 30],
    'width': [5, 7, 10, 12, 15],
    'height': [3, 4, 5, 6, 7]
})

# Features derivadas simples
df['area'] = df['length'] * df['width']
df['volume'] = df['length'] * df['width'] * df['height']
df['aspect_ratio'] = df['length'] / df['width']
df['perimeter'] = 2 * (df['length'] + df['width'])

print("Derived Features:")
print(df.head())

# Polynomial Features (interacciones)
poly = PolynomialFeatures(degree=2, include_bias=False)
X_poly = poly.fit_transform(df[['length', 'width']])

feature_names = poly.get_feature_names_out(['length', 'width'])
df_poly = pd.DataFrame(X_poly, columns=feature_names)
print("\nPolynomial Features:")
print(df_poly.head())

# Binning/Discretización
df['length_binned'] = pd.cut(
    df['length'], 
    bins=[0, 15, 25, float('inf')],
    labels=['small', 'medium', 'large']
)
print("\nBinned Feature:")
print(df[['length', 'length_binned']])

# Quantile-based binning
df['length_quantile'] = pd.qcut(df['length'], q=3, labels=['Q1', 'Q2', 'Q3'])
```

### Ejemplo 4: Temporal Features

```python
import pandas as pd

# Datos con fechas
df = pd.DataFrame({
    'date': pd.date_range('2023-01-01', periods=100, freq='D'),
    'sales': np.random.randint(100, 1000, 100)
})

# Date decomposition
df['year'] = df['date'].dt.year
df['month'] = df['date'].dt.month
df['day'] = df['date'].dt.day
df['day_of_week'] = df['date'].dt.dayofweek
df['day_name'] = df['date'].dt.day_name()
df['is_weekend'] = df['date'].dt.dayofweek.isin([5, 6]).astype(int)
df['is_month_start'] = df['date'].dt.is_month_start.astype(int)
df['is_month_end'] = df['date'].dt.is_month_end.astype(int)
df['quarter'] = df['date'].dt.quarter
df['week_of_year'] = df['date'].dt.isocalendar().week

print("Date Features:")
print(df.head())

# Lag features
df['sales_lag_1'] = df['sales'].shift(1)
df['sales_lag_7'] = df['sales'].shift(7)
df['sales_lag_30'] = df['sales'].shift(30)

# Rolling statistics
df['sales_rolling_mean_7'] = df['sales'].rolling(window=7).mean()
df['sales_rolling_std_7'] = df['sales'].rolling(window=7).std()
df['sales_rolling_min_7'] = df['sales'].rolling(window=7).min()
df['sales_rolling_max_7'] = df['sales'].rolling(window=7).max()

# Expanding statistics
df['sales_expanding_mean'] = df['sales'].expanding().mean()
df['sales_cumsum'] = df['sales'].cumsum()

# Time since event
event_date = pd.Timestamp('2023-01-15')
df['days_since_event'] = (df['date'] - event_date).dt.days

print("\nTemporal Features:")
print(df[['date', 'sales', 'sales_lag_1', 'sales_rolling_mean_7']].head(10))
```

### Ejemplo 5: Feature Selection

```python
from sklearn.feature_selection import (
    SelectKBest, f_classif, mutual_info_classif,
    RFE, SelectFromModel, VarianceThreshold
)
from sklearn.ensemble import RandomForestClassifier
from sklearn.linear_model import LassoCV

# Generar datos
from sklearn.datasets import make_classification
X, y = make_classification(
    n_samples=1000, n_features=20, n_informative=10,
    n_redundant=5, random_state=42
)

# Variance Threshold (remover features de baja varianza)
selector_var = VarianceThreshold(threshold=0.1)
X_high_var = selector_var.fit_transform(X)
print(f"Features after variance threshold: {X_high_var.shape[1]}")

# SelectKBest (filter method)
selector_kbest = SelectKBest(f_classif, k=10)
X_kbest = selector_kbest.fit_transform(X, y)
scores = selector_kbest.scores_
print(f"\nTop 10 features by F-score:")
print(pd.DataFrame({
    'feature': range(len(scores)),
    'score': scores
}).sort_values('score', ascending=False).head(10))

# Mutual Information
selector_mi = SelectKBest(mutual_info_classif, k=10)
X_mi = selector_mi.fit_transform(X, y)

# RFE (Recursive Feature Elimination)
rfe = RFE(RandomForestClassifier(n_estimators=100, random_state=42), n_features_to_select=10)
X_rfe = rfe.fit_transform(X, y)
print(f"\nFeatures selected by RFE: {np.where(rfe.support_)[0]}")

# Feature Importance (embedded method)
rf = RandomForestClassifier(n_estimators=100, random_state=42)
rf.fit(X, y)

feature_importance = pd.DataFrame({
    'feature': range(X.shape[1]),
    'importance': rf.feature_importances_
}).sort_values('importance', ascending=False)

print("\nFeature Importance:")
print(feature_importance.head(10))

# L1-based selection (Lasso)
lasso = LassoCV(cv=5, random_state=42)
selector_l1 = SelectFromModel(lasso)
X_l1 = selector_l1.fit_transform(X, y)
print(f"\nFeatures selected by L1: {X_l1.shape[1]}")

# Correlation-based selection
corr_matrix = pd.DataFrame(X).corr().abs()
upper_triangle = corr_matrix.where(
    np.triu(np.ones(corr_matrix.shape), k=1).astype(bool)
)

# Features con correlación > 0.95
high_corr_features = [
    column for column in upper_triangle.columns 
    if any(upper_triangle[column] > 0.95)
]
print(f"\nHighly correlated features to remove: {high_corr_features}")
```

### Ejemplo 6: Text Features

```python
from sklearn.feature_extraction.text import CountVectorizer, TfidfVectorizer
from sklearn.decomposition import LatentDirichletAllocation

# Datos de texto
documents = [
    "machine learning is great",
    "deep learning with neural networks",
    "machine learning and deep learning",
    "python programming for data science",
    "data science with python"
]

# Bag of Words
vectorizer_bow = CountVectorizer()
X_bow = vectorizer_bow.fit_transform(documents)
bow_features = vectorizer_bow.get_feature_names_out()

print("Bag of Words:")
print(pd.DataFrame(X_bow.toarray(), columns=bow_features))

# TF-IDF
vectorizer_tfidf = TfidfVectorizer()
X_tfidf = vectorizer_tfidf.fit_transform(documents)

print("\nTF-IDF:")
print(pd.DataFrame(X_tfidf.toarray(), columns=vectorizer_tfidf.get_feature_names_out()))

# N-grams
vectorizer_ngram = CountVectorizer(ngram_range=(1, 2))
X_ngram = vectorizer_ngram.fit_transform(documents)
print(f"\nN-gram features: {len(vectorizer_ngram.get_feature_names_out())}")

# Character-level features
df_text = pd.DataFrame({'text': documents})
df_text['length'] = df_text['text'].str.len()
df_text['word_count'] = df_text['text'].str.split().str.len()
df_text['avg_word_length'] = df_text['text'].apply(
    lambda x: np.mean([len(word) for word in x.split()])
)
df_text['uppercase_count'] = df_text['text'].str.count(r'[A-Z]')

print("\nText Statistics:")
print(df_text)
```

### Ejemplo 7: Missing Data Handling

```python
from sklearn.impute import SimpleImputer, KNNImputer
from sklearn.experimental import enable_iterative_imputer
from sklearn.impute import IterativeImputer

# Crear datos con missings
np.random.seed(42)
df = pd.DataFrame({
    'A': [1, 2, np.nan, 4, 5, np.nan, 7, 8],
    'B': [np.nan, 2, 3, np.nan, 5, 6, 7, 8],
    'C': [1, 2, 3, 4, 5, 6, 7, 8]
})

print("Original data with NaN:")
print(df)

# Simple Imputation - Mean
imputer_mean = SimpleImputer(strategy='mean')
df_mean = pd.DataFrame(
    imputer_mean.fit_transform(df),
    columns=df.columns
)
print("\nMean Imputation:")
print(df_mean)

# Simple Imputation - Median (robusto a outliers)
imputer_median = SimpleImputer(strategy='median')
df_median = pd.DataFrame(
    imputer_median.fit_transform(df),
    columns=df.columns
)

# KNN Imputation
imputer_knn = KNNImputer(n_neighbors=3)
df_knn = pd.DataFrame(
    imputer_knn.fit_transform(df),
    columns=df.columns
)
print("\nKNN Imputation:")
print(df_knn)

# MICE (Multiple Imputation by Chained Equations)
imputer_mice = IterativeImputer(random_state=42)
df_mice = pd.DataFrame(
    imputer_mice.fit_transform(df),
    columns=df.columns
)
print("\nMICE Imputation:")
print(df_mice)

# Indicator variables
df_with_indicator = df.copy()
df_with_indicator['A_missing'] = df['A'].isna().astype(int)
df_with_indicator['B_missing'] = df['B'].isna().astype(int)
df_with_indicator.fillna(df.mean(), inplace=True)

print("\nWith Missing Indicators:")
print(df_with_indicator)

# Forward/Backward Fill (time series)
df_ffill = df.fillna(method='ffill')  # Forward fill
df_bfill = df.fillna(method='bfill')  # Backward fill
```

## Pipeline Completo de Feature Engineering

```python
from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder

# Datos de ejemplo
data = pd.DataFrame({
    'age': [25, 30, 35, np.nan, 45],
    'salary': [30000, 45000, 52000, 65000, 80000],
    'city': ['NYC', 'LA', 'Chicago', 'NYC', 'LA'],
    'education': ['BS', 'MS', 'PhD', 'BS', 'MS'],
    'years_exp': [2, 5, 7, 10, 15],
    'target': [0, 0, 1, 1, 1]
})

# Separar features y target
X = data.drop('target', axis=1)
y = data['target']

# Definir transformaciones por tipo de feature
numeric_features = ['age', 'salary', 'years_exp']
categorical_features = ['city', 'education']

# Pipeline para numéricas
numeric_transformer = Pipeline(steps=[
    ('imputer', SimpleImputer(strategy='median')),
    ('scaler', StandardScaler())
])

# Pipeline para categóricas
categorical_transformer = Pipeline(steps=[
    ('imputer', SimpleImputer(strategy='constant', fill_value='missing')),
    ('onehot', OneHotEncoder(handle_unknown='ignore'))
])

# Combinar transformaciones
preprocessor = ColumnTransformer(
    transformers=[
        ('num', numeric_transformer, numeric_features),
        ('cat', categorical_transformer, categorical_features)
    ])

# Pipeline completo
full_pipeline = Pipeline(steps=[
    ('preprocessor', preprocessor),
    ('classifier', RandomForestClassifier(random_state=42))
])

# Fit y predict
full_pipeline.fit(X, y)
predictions = full_pipeline.predict(X)

print("Pipeline completo ejecutado")
print(f"Predictions: {predictions}")
```

## Mejores Prácticas

### 1. Evitar Data Leakage

```python
# ❌ Malo - fit en todo el dataset
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)  # Leak!
X_train, X_test = train_test_split(X_scaled)

# ✅ Bueno - fit solo en train
X_train, X_test = train_test_split(X)
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

# ✅ Mejor - usar Pipeline
pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('model', RandomForestClassifier())
])
pipeline.fit(X_train, y_train)
```

### 2. Feature Scaling Apropiado

```python
# ✅ StandardScaler para distribución normal
# ✅ MinMaxScaler para rango específico [0,1]
# ✅ RobustScaler cuando hay outliers
# ✅ Sin scaling para tree-based models (RF, XGBoost)
```

### 3. Validate Features

```python
# ✅ Verificar correlación con target
correlations = df.corr()['target'].abs().sort_values(ascending=False)
print(correlations)

# ✅ Verificar multicolinealidad
from statsmodels.stats.outliers_influence import variance_inflation_factor

vif_data = pd.DataFrame()
vif_data["feature"] = X.columns
vif_data["VIF"] = [variance_inflation_factor(X.values, i) for i in range(len(X.columns))]
print(vif_data[vif_data['VIF'] > 10])  # High multicollinearity
```

## Proyectos Sugeridos

1. **Kaggle Competition Feature Engineering**
   - Participar en competición
   - Feature engineering iterativo
   - Documentar qué features funcionan

2. **Time Series Forecasting**
   - Crear lag features
   - Rolling statistics
   - Seasonal decomposition

3. **NLP Feature Pipeline**
   - TF-IDF
   - Sentiment analysis
   - Custom text features

4. **Customer Churn Prediction**
   - RFM features
   - Behavioral features
   - Temporal patterns

5. **Image Feature Extraction**
   - Transfer learning features
   - Image statistics
   - Color histograms

## Checklist de Aprendizaje

- [ ] Dominar feature scaling
- [ ] Aplicar encoding categórico apropiado
- [ ] Crear features polinomiales
- [ ] Manejar missing data correctamente
- [ ] Extraer features temporales
- [ ] Aplicar feature selection
- [ ] Evitar data leakage
- [ ] Crear pipelines reproducibles
- [ ] Validar features con domain knowledge
- [ ] Iterar basándose en model performance

## Próximos Pasos

1. Estudiar **Automated Feature Engineering** (Featuretools)
2. Aprender **Feature Importance** interpretation (SHAP)
3. Explorar **Deep Feature Synthesis**
4. Dominar **Time Series Features** avanzadas
5. Practicar en **Kaggle competitions**

---
**Tiempo estimado**: 4-6 semanas
