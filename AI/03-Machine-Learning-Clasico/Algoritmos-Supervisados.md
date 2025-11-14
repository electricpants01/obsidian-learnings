# Algoritmos de Machine Learning Supervisado

## Descripción

El aprendizaje supervisado es la rama del machine learning donde los algoritmos aprenden de datos etiquetados para hacer predicciones sobre datos nuevos. Es la base de la mayoría de aplicaciones prácticas de ML, desde clasificación de spam hasta diagnóstico médico. Los algoritmos supervisados se dividen en dos categorías principales: clasificación (predecir categorías discretas como spam/no spam, gato/perro) y regresión (predecir valores continuos como precio de casa, temperatura). Cada algoritmo tiene sus fortalezas: regresión lineal para relaciones lineales simples, árboles de decisión para interpretabilidad, random forests para robustez, gradient boosting (XGBoost, LightGBM) para performance máxima, SVMs para problemas de alta dimensionalidad, y k-NN para simplicidad. La elección del algoritmo depende del tamaño del dataset, dimensionalidad de features, necesidad de interpretabilidad, y recursos computacionales disponibles. Dominar estos algoritmos y saber cuándo aplicar cada uno es fundamental para cualquier ML engineer.

## Conceptos Clave

### 1. **Regresión Lineal y Logística**
- Modelos lineales simples pero poderosos
- Regresión lineal para valores continuos
- Regresión logística para clasificación binaria
- Regularización (L1/L2) para prevenir overfitting
- Interpretabilidad de coeficientes

### 2. **Árboles de Decisión**
- Modelo basado en reglas if-then
- Criterion: Gini impurity vs Entropy
- Profundidad máxima y poda
- Feature importance
- Muy interpretables

### 3. **Random Forests**
- Ensemble de múltiples árboles
- Bagging para reducir varianza
- Out-of-bag error estimation
- Feature importance agregada
- Robusto a overfitting

### 4. **Gradient Boosting**
- XGBoost, LightGBM, CatBoost
- Boosting secuencial de árboles débiles
- Learning rate y número de estimadores
- Early stopping
- State-of-the-art en datos tabulares

### 5. **Support Vector Machines (SVM)**
- Maximizar margen entre clases
- Kernel trick para no linealidad
- C parameter (regularización)
- Gamma parameter (influencia de ejemplos)
- Eficiente en alta dimensionalidad

### 6. **K-Nearest Neighbors (k-NN)**
- Clasificación basada en vecinos cercanos
- Distance metrics (Euclidean, Manhattan)
- Elección de k
- Lazy learning (no training)
- Curse of dimensionality

### 7. **Naive Bayes**
- Basado en teorema de Bayes
- Assumption de independencia
- Gaussian NB, Multinomial NB, Bernoulli NB
- Rápido y eficiente
- Excelente para text classification

## Recursos de Aprendizaje

### Documentación Oficial

1. **Scikit-learn Supervised Learning**
   - https://scikit-learn.org/stable/supervised_learning.html
   - Guía completa de todos los algoritmos

2. **XGBoost Documentation**
   - https://xgboost.readthedocs.io/
   - Gradient boosting de alto rendimiento

3. **LightGBM Documentation**
   - https://lightgbm.readthedocs.io/
   - Gradient boosting optimizado

### Cursos Online

1. **Machine Learning by Andrew Ng - Coursera** (Gratis con auditoría)
   - https://www.coursera.org/learn/machine-learning
   - El curso clásico, fundamentos sólidos

2. **Applied Machine Learning in Python - Coursera**
   - https://www.coursera.org/learn/python-machine-learning
   - Universidad de Michigan, muy práctico

3. **Supervised Learning with scikit-learn - DataCamp**
   - https://www.datacamp.com/courses/supervised-learning-with-scikit-learn
   - Interactivo, enfoque en código

4. **Machine Learning Crash Course - Google** (Gratis)
   - https://developers.google.com/machine-learning/crash-course
   - Por Google, ejercicios prácticos

### Libros

1. **"Introduction to Statistical Learning"** - James, Witten, Hastie, Tibshirani
   - Gratis: https://www.statlearning.com/
   - El libro perfecto para comenzar
   - Ejemplos en R pero conceptos universales

2. **"Hands-On Machine Learning"** - Aurélien Géron
   - Parte 1 cubre todos estos algoritmos
   - Ejemplos prácticos en Python
   - Muy didáctico

3. **"The Elements of Statistical Learning"** - Hastie, Tibshirani, Friedman
   - Gratis: https://web.stanford.edu/~hastie/ElemStatLearn/
   - Más teórico pero fundamental
   - La biblia del ML

4. **"Pattern Recognition and Machine Learning"** - Christopher Bishop
   - Enfoque bayesiano
   - Matemáticamente riguroso

### Videos Recomendados

1. **StatQuest with Josh Starmer**
   - https://www.youtube.com/c/joshstarmer
   - Excelentes explicaciones de cada algoritmo

2. **3Blue1Brown - Neural Networks**
   - https://www.youtube.com/c/3blue1brown
   - Visualizaciones increíbles

3. **Krish Naik - ML Algorithms Playlist**
   - https://www.youtube.com/user/krishnaik06

## Ejemplos Prácticos

### Ejemplo 1: Regresión Lineal

```python
from sklearn.linear_model import LinearRegression, Ridge, Lasso
from sklearn.model_selection import train_test_split
from sklearn.metrics import mean_squared_error, r2_score
import numpy as np
import matplotlib.pyplot as plt

# Generar datos sintéticos
np.random.seed(42)
X = np.random.rand(100, 1) * 10
y = 2.5 * X + 3 + np.random.randn(100, 1) * 2

# Split
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# Linear Regression
lr = LinearRegression()
lr.fit(X_train, y_train)
y_pred = lr.predict(X_test)

print(f"Coefficients: {lr.coef_}")
print(f"Intercept: {lr.intercept_}")
print(f"R² Score: {r2_score(y_test, y_pred):.3f}")
print(f"RMSE: {np.sqrt(mean_squared_error(y_test, y_pred)):.3f}")

# Ridge (L2 regularization)
ridge = Ridge(alpha=1.0)
ridge.fit(X_train, y_train)
y_pred_ridge = ridge.predict(X_test)

# Lasso (L1 regularization)
lasso = Lasso(alpha=0.1)
lasso.fit(X_train, y_train)
y_pred_lasso = lasso.predict(X_test)

# Visualizar
plt.scatter(X_test, y_test, label='Actual')
plt.plot(X_test, y_pred, 'r-', label='Linear Regression')
plt.plot(X_test, y_pred_ridge, 'g--', label='Ridge')
plt.plot(X_test, y_pred_lasso, 'b:', label='Lasso')
plt.legend()
plt.show()
```

### Ejemplo 2: Regresión Logística

```python
from sklearn.linear_model import LogisticRegression
from sklearn.datasets import make_classification
from sklearn.metrics import classification_report, confusion_matrix
import seaborn as sns

# Generar datos
X, y = make_classification(
    n_samples=1000, n_features=20, n_informative=15,
    n_redundant=5, random_state=42
)

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)

# Logistic Regression
log_reg = LogisticRegression(random_state=42, max_iter=1000)
log_reg.fit(X_train, y_train)

# Predicciones
y_pred = log_reg.predict(X_test)
y_pred_proba = log_reg.predict_proba(X_test)

# Evaluación
print("Classification Report:")
print(classification_report(y_test, y_pred))

# Confusion Matrix
cm = confusion_matrix(y_test, y_pred)
sns.heatmap(cm, annot=True, fmt='d')
plt.title('Confusion Matrix')
plt.ylabel('True Label')
plt.xlabel('Predicted Label')
plt.show()

# Probabilidades
print(f"Sample probabilities: {y_pred_proba[:5]}")
```

### Ejemplo 3: Decision Tree

```python
from sklearn.tree import DecisionTreeClassifier, plot_tree
from sklearn.datasets import load_iris

# Cargar datos
iris = load_iris()
X, y = iris.data, iris.target

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# Decision Tree
dt = DecisionTreeClassifier(
    max_depth=3,
    min_samples_split=10,
    min_samples_leaf=5,
    criterion='gini',
    random_state=42
)

dt.fit(X_train, y_train)

# Evaluación
train_score = dt.score(X_train, y_train)
test_score = dt.score(X_test, y_test)

print(f"Train Accuracy: {train_score:.3f}")
print(f"Test Accuracy: {test_score:.3f}")

# Feature Importance
feature_importance = pd.DataFrame({
    'feature': iris.feature_names,
    'importance': dt.feature_importances_
}).sort_values('importance', ascending=False)

print("\nFeature Importance:")
print(feature_importance)

# Visualizar árbol
plt.figure(figsize=(20, 10))
plot_tree(
    dt,
    feature_names=iris.feature_names,
    class_names=iris.target_names,
    filled=True,
    rounded=True
)
plt.show()
```

### Ejemplo 4: Random Forest

```python
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import cross_val_score

# Random Forest
rf = RandomForestClassifier(
    n_estimators=100,
    max_depth=5,
    min_samples_split=10,
    random_state=42,
    n_jobs=-1
)

rf.fit(X_train, y_train)

# Cross-validation
cv_scores = cross_val_score(rf, X, y, cv=5)
print(f"CV Scores: {cv_scores}")
print(f"Mean CV Score: {cv_scores.mean():.3f} (+/- {cv_scores.std() * 2:.3f})")

# Out-of-bag score (si oob_score=True)
rf_oob = RandomForestClassifier(
    n_estimators=100,
    oob_score=True,
    random_state=42
)
rf_oob.fit(X_train, y_train)
print(f"OOB Score: {rf_oob.oob_score_:.3f}")

# Feature Importance
importances = pd.DataFrame({
    'feature': iris.feature_names,
    'importance': rf.feature_importances_,
    'std': np.std([tree.feature_importances_ for tree in rf.estimators_], axis=0)
}).sort_values('importance', ascending=False)

print("\nRandom Forest Feature Importance:")
print(importances)

# Plot importance
importances.plot(x='feature', y='importance', kind='barh', xerr='std')
plt.title('Feature Importance')
plt.show()
```

### Ejemplo 5: XGBoost

```python
import xgboost as xgb
from sklearn.metrics import accuracy_score

# Preparar datos en formato DMatrix
dtrain = xgb.DMatrix(X_train, label=y_train)
dtest = xgb.DMatrix(X_test, label=y_test)

# Parámetros
params = {
    'objective': 'multi:softmax',
    'num_class': 3,
    'max_depth': 3,
    'learning_rate': 0.1,
    'n_estimators': 100,
    'subsample': 0.8,
    'colsample_bytree': 0.8,
    'eval_metric': 'merror'
}

# Entrenar con early stopping
evals = [(dtrain, 'train'), (dtest, 'test')]
bst = xgb.train(
    params,
    dtrain,
    num_boost_round=100,
    evals=evals,
    early_stopping_rounds=10,
    verbose_eval=10
)

# Predicción
y_pred = bst.predict(dtest)
accuracy = accuracy_score(y_test, y_pred)
print(f"XGBoost Accuracy: {accuracy:.3f}")

# Feature Importance
xgb.plot_importance(bst)
plt.show()

# Usar scikit-learn API (más conveniente)
xgb_clf = xgb.XGBClassifier(
    n_estimators=100,
    max_depth=3,
    learning_rate=0.1,
    random_state=42
)
xgb_clf.fit(X_train, y_train)
y_pred_sklearn = xgb_clf.predict(X_test)
```

### Ejemplo 6: Support Vector Machine

```python
from sklearn.svm import SVC
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline

# SVM necesita features escaladas
pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('svm', SVC(kernel='rbf', C=1.0, gamma='scale', random_state=42))
])

pipeline.fit(X_train, y_train)

# Predicción
y_pred = pipeline.predict(X_test)
accuracy = accuracy_score(y_test, y_pred)
print(f"SVM Accuracy: {accuracy:.3f}")

# Probar diferentes kernels
kernels = ['linear', 'rbf', 'poly']
for kernel in kernels:
    svm = SVC(kernel=kernel, random_state=42)
    svm.fit(X_train, y_train)
    score = svm.score(X_test, y_test)
    print(f"SVM {kernel} kernel: {score:.3f}")

# Grid Search para optimizar
from sklearn.model_selection import GridSearchCV

param_grid = {
    'C': [0.1, 1, 10],
    'gamma': ['scale', 'auto', 0.1, 1],
    'kernel': ['rbf', 'poly']
}

grid_search = GridSearchCV(
    SVC(random_state=42),
    param_grid,
    cv=5,
    scoring='accuracy',
    n_jobs=-1
)

grid_search.fit(X_train, y_train)
print(f"Best params: {grid_search.best_params_}")
print(f"Best score: {grid_search.best_score_:.3f}")
```

### Ejemplo 7: K-Nearest Neighbors

```python
from sklearn.neighbors import KNeighborsClassifier

# Probar diferentes valores de k
k_values = range(1, 21)
train_scores = []
test_scores = []

for k in k_values:
    knn = KNeighborsClassifier(n_neighbors=k)
    knn.fit(X_train, y_train)
    
    train_scores.append(knn.score(X_train, y_train))
    test_scores.append(knn.score(X_test, y_test))

# Plot
plt.plot(k_values, train_scores, label='Train')
plt.plot(k_values, test_scores, label='Test')
plt.xlabel('k')
plt.ylabel('Accuracy')
plt.legend()
plt.title('k-NN: k vs Accuracy')
plt.show()

# Mejor k
best_k = k_values[np.argmax(test_scores)]
print(f"Best k: {best_k}")

# Entrenar con mejor k
knn_best = KNeighborsClassifier(n_neighbors=best_k)
knn_best.fit(X_train, y_train)

# Distance metrics
from sklearn.metrics import pairwise_distances

distances = pairwise_distances(X_test[:1], X_train, metric='euclidean')
nearest_indices = np.argsort(distances[0])[:best_k]
print(f"Nearest neighbors: {nearest_indices}")
print(f"Their labels: {y_train[nearest_indices]}")
```

## Comparación de Algoritmos

```python
from sklearn.model_selection import cross_validate
import time

# Definir modelos
models = {
    'Logistic Regression': LogisticRegression(),
    'Decision Tree': DecisionTreeClassifier(random_state=42),
    'Random Forest': RandomForestClassifier(n_estimators=100, random_state=42),
    'XGBoost': xgb.XGBClassifier(n_estimators=100, random_state=42),
    'SVM': SVC(random_state=42),
    'k-NN': KNeighborsClassifier()
}

# Comparar
results = []

for name, model in models.items():
    start_time = time.time()
    
    scores = cross_validate(
        model, X, y,
        cv=5,
        scoring=['accuracy', 'precision_macro', 'recall_macro', 'f1_macro'],
        n_jobs=-1
    )
    
    training_time = time.time() - start_time
    
    results.append({
        'Model': name,
        'Accuracy': scores['test_accuracy'].mean(),
        'Precision': scores['test_precision_macro'].mean(),
        'Recall': scores['test_recall_macro'].mean(),
        'F1': scores['test_f1_macro'].mean(),
        'Time': training_time
    })

# Resultados
results_df = pd.DataFrame(results).sort_values('Accuracy', ascending=False)
print(results_df)
```

## Mejores Prácticas

### 1. Siempre Escalar Features para SVM y k-NN

```python
# ✅ Bueno - pipeline con scaling
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline

pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('svm', SVC())
])

# ❌ Malo - olvidar escalar
svm = SVC()
svm.fit(X_train, y_train)  # Features no escaladas
```

### 2. Usar Random State para Reproducibilidad

```python
# ✅ Bueno
model = RandomForestClassifier(random_state=42)

# ❌ Malo
model = RandomForestClassifier()  # Resultados diferentes cada vez
```

### 3. Prevent Overfitting

```python
# Decision Tree: limitar profundidad
dt = DecisionTreeClassifier(max_depth=5, min_samples_leaf=10)

# Random Forest: usar out-of-bag
rf = RandomForestClassifier(n_estimators=100, oob_score=True)

# XGBoost: early stopping
xgb.train(params, dtrain, early_stopping_rounds=10)

# Regularización en regresión
ridge = Ridge(alpha=1.0)  # L2
lasso = Lasso(alpha=1.0)  # L1
```

### 4. Use Cross-Validation

```python
# ✅ Bueno
from sklearn.model_selection import cross_val_score
scores = cross_val_score(model, X, y, cv=5)
print(f"Mean: {scores.mean():.3f} (+/- {scores.std() * 2:.3f})")

# ❌ Malo - single split
model.fit(X_train, y_train)
score = model.score(X_test, y_test)
```

### 5. Handle Imbalanced Data

```python
# Class weights
from sklearn.utils.class_weight import compute_class_weight

class_weights = compute_class_weight('balanced', classes=np.unique(y), y=y)
model = LogisticRegression(class_weight='balanced')

# SMOTE para oversampling
from imblearn.over_sampling import SMOTE

smote = SMOTE(random_state=42)
X_resampled, y_resampled = smote.fit_resample(X, y)
```

## Proyectos Sugeridos

1. **Predictor de Precios de Casas**
   - Dataset: Kaggle House Prices
   - Comparar regresión lineal, random forest, XGBoost
   - Feature engineering extensivo
   - Optimización de hiperparámetros

2. **Clasificador de Enfermedades Cardíacas**
   - Dataset: UCI Heart Disease
   - Múltiples algoritmos de clasificación
   - Handle class imbalance
   - Feature importance analysis

3. **Sistema de Recomendación de Productos**
   - Collaborative filtering con k-NN
   - Matrix factorization
   - Hybrid approaches
   - Evaluation metrics

4. **Detección de Fraude en Transacciones**
   - Dataset altamente desbalanceado
   - Anomaly detection
   - Precision/Recall optimization
   - Real-time scoring

5. **Predictor de Churn de Clientes**
   - Classification problem
   - Feature engineering de datos temporales
   - Cost-sensitive learning
   - Interpretable models para business

## Checklist de Aprendizaje

- [ ] Implementar regresión lineal desde cero
- [ ] Entender bias-variance tradeoff
- [ ] Dominar árboles de decisión y visualización
- [ ] Comparar bagging vs boosting
- [ ] Optimizar hiperparámetros con Grid Search
- [ ] Manejar datos desbalanceados
- [ ] Interpretar feature importance
- [ ] Elegir algoritmo apropiado por caso de uso
- [ ] Implementar ensemble voting/stacking
- [ ] Completar proyecto end-to-end

## Próximos Pasos

1. Estudiar **Feature Engineering** en detalle
2. Profundizar en **Hyperparameter Tuning**
3. Aprender **Ensemble Methods** avanzados
4. Explorar **Deep Learning** para comparar
5. Dominar **Model Interpretation** (SHAP, LIME)

---
**Tiempo estimado de estudio**: 4-6 semanas dedicando 3-4 horas diarias
