# Scikit-learn - Machine Learning en Python

## Descripción

Scikit-learn es la biblioteca de machine learning más popular de Python. Proporciona implementaciones eficientes de algoritmos clásicos de ML con una API consistente y bien documentada. Es fundamental para cualquier proyecto de ciencia de datos y punto de partida antes de deep learning.

## Conceptos Clave

### 1. **API Consistente**
- fit(), predict(), transform()
- Estimators y transformers
- Pipelines
- Patrón de diseño unificado

### 2. **Preprocesamiento**
- StandardScaler, MinMaxScaler
- Encoding categórico
- Imputación de valores faltantes
- Feature selection

### 3. **Modelos Supervisados**
- Clasificación y regresión
- Linear models
- Tree-based models
- Ensemble methods

### 4. **Modelos No Supervisados**
- Clustering
- Dimensionality reduction
- Anomaly detection

### 5. **Model Selection**
- Train/test split
- Cross-validation
- Grid search
- Randomized search

### 6. **Métricas**
- Classification metrics
- Regression metrics
- Clustering metrics

### 7. **Pipelines**
- Encadenar transformaciones
- Reproducibilidad
- Evitar data leakage

## Recursos de Aprendizaje

### Documentación Oficial

1. **Scikit-learn Documentation**
   - https://scikit-learn.org/stable/
   - Excelente documentación con ejemplos

2. **User Guide**
   - https://scikit-learn.org/stable/user_guide.html
   - Guía completa de uso

3. **API Reference**
   - https://scikit-learn.org/stable/modules/classes.html

### Cursos Online

1. **Scikit-learn Course - DataCamp**
   - https://www.datacamp.com/courses/supervised-learning-with-scikit-learn

2. **Machine Learning with Python - Coursera**
   - https://www.coursera.org/learn/machine-learning-with-python

3. **Applied Machine Learning in Python - Coursera**
   - https://www.coursera.org/learn/python-machine-learning

### Libros

1. **"Hands-On Machine Learning"** - Aurélien Géron
   - Capítulos completos sobre scikit-learn
   - Proyectos prácticos

2. **"Introduction to Machine Learning with Python"** - Andreas Müller & Sarah Guido
   - Escrito por core developer de scikit-learn

3. **"Python Machine Learning"** - Sebastian Raschka
   - Fundamentos y práctica

### Videos

1. **Jake VanderPlas - Machine Learning with Scikit-learn**
   - https://www.youtube.com/watch?v=HC0J_SPm9co

2. **sentdex - Machine Learning with Python**
   - https://www.youtube.com/playlist?list=PLQVvvaa0QuDfKTOs3Keq_kaG2P55YRn5v

## Instalación

```bash
pip install scikit-learn

# Verificar
python -c "import sklearn; print(sklearn.__version__)"

# Con todas las dependencias
pip install scikit-learn[alldeps]
```

## Ejemplos Prácticos

### Ejemplo 1: Workflow Básico

```python
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import accuracy_score, classification_report
import numpy as np

# Cargar datos
from sklearn.datasets import load_iris
iris = load_iris()
X, y = iris.data, iris.target

# Split
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)

# Preprocesar
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

# Entrenar
model = LogisticRegression(random_state=42)
model.fit(X_train_scaled, y_train)

# Predecir
y_pred = model.predict(X_test_scaled)

# Evaluar
accuracy = accuracy_score(y_test, y_pred)
print(f"Accuracy: {accuracy:.3f}")
print(classification_report(y_test, y_pred))
```

### Ejemplo 2: Pipeline

```python
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.decomposition import PCA
from sklearn.ensemble import RandomForestClassifier

# Crear pipeline
pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('pca', PCA(n_components=2)),
    ('classifier', RandomForestClassifier(n_estimators=100))
])

# Entrenar pipeline completo
pipeline.fit(X_train, y_train)

# Predecir
y_pred = pipeline.predict(X_test)

# Acceder a componentes
pca_component = pipeline.named_steps['pca']
print(f"Explained variance: {pca_component.explained_variance_ratio_}")
```

### Ejemplo 3: Grid Search

```python
from sklearn.model_selection import GridSearchCV
from sklearn.svm import SVC

# Definir grid de hiperparámetros
param_grid = {
    'C': [0.1, 1, 10, 100],
    'gamma': [0.001, 0.01, 0.1, 1],
    'kernel': ['rbf', 'linear']
}

# Grid search con CV
grid_search = GridSearchCV(
    SVC(),
    param_grid,
    cv=5,
    scoring='accuracy',
    n_jobs=-1,
    verbose=1
)

grid_search.fit(X_train_scaled, y_train)

# Mejores parámetros
print(f"Best parameters: {grid_search.best_params_}")
print(f"Best score: {grid_search.best_score_:.3f}")

# Usar mejor modelo
best_model = grid_search.best_estimator_
```

### Ejemplo 4: Cross-Validation

```python
from sklearn.model_selection import cross_val_score, cross_validate

model = RandomForestClassifier(n_estimators=100)

# Simple CV
scores = cross_val_score(model, X, y, cv=5, scoring='accuracy')
print(f"CV Accuracy: {scores.mean():.3f} (+/- {scores.std() * 2:.3f})")

# CV con múltiples métricas
scoring = ['accuracy', 'precision_macro', 'recall_macro']
scores = cross_validate(model, X, y, cv=5, scoring=scoring)

print(f"Accuracy: {scores['test_accuracy'].mean():.3f}")
print(f"Precision: {scores['test_precision_macro'].mean():.3f}")
print(f"Recall: {scores['test_recall_macro'].mean():.3f}")
```

### Ejemplo 5: Feature Engineering

```python
from sklearn.preprocessing import PolynomialFeatures, OneHotEncoder
from sklearn.compose import ColumnTransformer

# Datos de ejemplo
import pandas as pd
df = pd.DataFrame({
    'age': [25, 30, 35, 40],
    'salary': [50000, 60000, 70000, 80000],
    'city': ['NY', 'LA', 'NY', 'SF']
})

# Column Transformer
preprocessor = ColumnTransformer(
    transformers=[
        ('num', StandardScaler(), ['age', 'salary']),
        ('cat', OneHotEncoder(), ['city'])
    ])

# Pipeline completo
pipeline = Pipeline([
    ('preprocessor', preprocessor),
    ('classifier', LogisticRegression())
])

# Features polinomiales
poly = PolynomialFeatures(degree=2, include_bias=False)
X_poly = poly.fit_transform(df[['age', 'salary']])
```

### Ejemplo 6: Ensemble Methods

```python
from sklearn.ensemble import (
    RandomForestClassifier,
    GradientBoostingClassifier,
    VotingClassifier,
    StackingClassifier
)

# Voting Classifier
clf1 = LogisticRegression()
clf2 = RandomForestClassifier()
clf3 = GradientBoostingClassifier()

voting_clf = VotingClassifier(
    estimators=[('lr', clf1), ('rf', clf2), ('gb', clf3)],
    voting='soft'
)

voting_clf.fit(X_train, y_train)

# Stacking
estimators = [
    ('rf', RandomForestClassifier(n_estimators=10)),
    ('gb', GradientBoostingClassifier(n_estimators=10))
]

stacking_clf = StackingClassifier(
    estimators=estimators,
    final_estimator=LogisticRegression()
)

stacking_clf.fit(X_train, y_train)
```

### Ejemplo 7: Model Persistence

```python
import joblib
import pickle

# Guardar modelo
joblib.dump(model, 'model.joblib')

# Cargar modelo
loaded_model = joblib.load('model.joblib')

# Predecir con modelo cargado
predictions = loaded_model.predict(X_test)

# Guardar pipeline completo
joblib.dump(pipeline, 'pipeline.joblib')
```

## Mejores Prácticas

### 1. Siempre Usar Pipelines

```python
# ✅ Bueno
pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('model', RandomForestClassifier())
])

# ❌ Malo - riesgo de data leakage
X_scaled = scaler.fit_transform(X)  # Fit en todo X
X_train, X_test = train_test_split(X_scaled)
```

### 2. Separar Datos Antes de Cualquier Preprocesamiento

```python
# ✅ Bueno
X_train, X_test = train_test_split(X, y)
scaler.fit(X_train)
X_train_scaled = scaler.transform(X_train)
X_test_scaled = scaler.transform(X_test)

# ❌ Malo
X_scaled = scaler.fit_transform(X)
X_train, X_test = train_test_split(X_scaled)
```

### 3. Usar Cross-Validation

```python
# ✅ Bueno
scores = cross_val_score(model, X, y, cv=5)

# ❌ Malo - single split
model.fit(X_train, y_train)
score = model.score(X_test, y_test)
```

### 4. Set Random State

```python
# ✅ Bueno - reproducible
model = RandomForestClassifier(random_state=42)
X_train, X_test = train_test_split(X, y, random_state=42)
```

## Algoritmos Principales

```python
# Clasificación
from sklearn.linear_model import LogisticRegression
from sklearn.tree import DecisionTreeClassifier
from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier
from sklearn.svm import SVC
from sklearn.naive_bayes import GaussianNB
from sklearn.neighbors import KNeighborsClassifier

# Regresión
from sklearn.linear_model import LinearRegression, Ridge, Lasso
from sklearn.tree import DecisionTreeRegressor
from sklearn.ensemble import RandomForestRegressor, GradientBoostingRegressor
from sklearn.svm import SVR

# Clustering
from sklearn.cluster import KMeans, DBSCAN, AgglomerativeClustering
from sklearn.mixture import GaussianMixture

# Dimensionality Reduction
from sklearn.decomposition import PCA, TruncatedSVD
from sklearn.manifold import TSNE
```

## Proyectos Sugeridos

1. **Predictor de Precios de Casas**
   - Dataset de Kaggle (House Prices)
   - Feature engineering
   - Ensemble de modelos

2. **Clasificador de Sentimientos**
   - Tweets o reviews
   - TF-IDF vectorization
   - Comparar algoritmos

3. **Sistema de Recomendación**
   - Collaborative filtering
   - Content-based filtering
   - Hybrid approach

4. **Detección de Fraude**
   - Imbalanced classes
   - Anomaly detection
   - Feature importance

5. **Customer Segmentation**
   - K-means clustering
   - PCA para visualización
   - Perfiles de clientes

## Recursos Adicionales

### Cheat Sheets

1. **Scikit-learn Cheat Sheet**
   - https://scikit-learn.org/stable/tutorial/machine_learning_map/

2. **DataCamp Cheat Sheet**
   - https://www.datacamp.com/cheat-sheet/scikit-learn-cheat-sheet-python-machine-learning

### Comunidades

1. **Stack Overflow - scikit-learn**
   - https://stackoverflow.com/questions/tagged/scikit-learn

2. **Scikit-learn Discussions**
   - https://github.com/scikit-learn/scikit-learn/discussions

## Checklist de Aprendizaje

- [ ] Entender el API de scikit-learn
- [ ] Usar preprocesadores correctamente
- [ ] Implementar pipelines
- [ ] Realizar train/test split
- [ ] Aplicar cross-validation
- [ ] Hacer grid search
- [ ] Evaluar modelos con métricas apropiadas
- [ ] Guardar y cargar modelos
- [ ] Evitar data leakage
- [ ] Completar proyecto end-to-end

## Próximos Pasos

1. Profundizar en **Algoritmos Supervisados**
2. Estudiar **Feature Engineering**
3. Dominar **Validación y Métricas**
4. Avanzar a **Deep Learning** con PyTorch/TensorFlow

---
**Tiempo estimado de estudio**: 3-4 semanas dedicando 2-3 horas diarias
