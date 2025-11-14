# Validación de Modelos y Métricas de Evaluación

## Descripción

La validación correcta y selección de métricas apropiadas son fundamentales para evaluar objetivamente el rendimiento de modelos de machine learning y evitar errores costosos en producción. Un modelo puede tener 99% accuracy pero ser completamente inútil si las clases están desbalanceadas 99:1. La validación determina qué tan bien generaliza un modelo a datos no vistos, mientras que las métricas cuantifican aspectos específicos del performance. Train-test split es el método más básico pero puede dar resultados variables; cross-validation provee estimaciones más robustas dividiendo datos en múltiples folds. Para clasificación, accuracy es intuitiva pero insuficiente - precision mide exactitud de predicciones positivas, recall captura cuántos positivos reales se detectan, F1-score balancea ambos, y ROC-AUC evalúa capacidad discriminativa. Para regresión, MSE penaliza errores grandes, MAE es más robusta a outliers, R² indica proporción de varianza explicada. Stratified sampling mantiene distribución de clases, time series split respeta orden temporal, nested CV evita data leakage en hyperparameter tuning. Dominar estas técnicas previene overfitting, underfitting y permite comparación justa entre modelos.

## Conceptos Clave

### 1. **Train-Test Split**
- División básica de datos
- Típicamente 70-30 o 80-20
- Stratified split para clases desbalanceadas
- Random state para reproducibilidad
- Limitación: resultados pueden variar

### 2. **Cross-Validation**
- K-Fold CV (típicamente k=5 o k=10)
- Stratified K-Fold para clasificación
- Leave-One-Out CV (LOOCV)
- Time Series Split para datos temporales
- Más robusto que simple split

### 3. **Métricas de Clasificación**
- Accuracy: proporción de predicciones correctas
- Precision: TP / (TP + FP)
- Recall (Sensitivity): TP / (TP + FN)
- F1-Score: media armónica de precision y recall
- ROC-AUC: área bajo curva ROC

### 4. **Métricas de Regresión**
- MSE (Mean Squared Error)
- RMSE (Root Mean Squared Error)
- MAE (Mean Absolute Error)
- R² Score (Coefficient of Determination)
- MAPE (Mean Absolute Percentage Error)

### 5. **Confusion Matrix**
- True Positives (TP)
- True Negatives (TN)
- False Positives (FP) - Type I Error
- False Negatives (FN) - Type II Error
- Base para otras métricas

### 6. **Curvas de Evaluación**
- ROC Curve (Receiver Operating Characteristic)
- Precision-Recall Curve
- Learning Curves
- Validation Curves
- Calibration Curves

### 7. **Validación para Producción**
- Hold-out test set
- Temporal validation
- A/B testing
- Online learning metrics
- Model monitoring

## Recursos de Aprendizaje

### Documentación Oficial

1. **Scikit-learn Model Evaluation**
   - https://scikit-learn.org/stable/model_selection.html
   - Guía completa de validación y métricas

2. **Scikit-learn Metrics**
   - https://scikit-learn.org/stable/modules/model_evaluation.html
   - Todas las métricas disponibles

### Cursos

1. **Model Validation - Kaggle Learn** (Gratis)
   - https://www.kaggle.com/learn/intro-to-machine-learning
   - Sección sobre validation

2. **Machine Learning - Coursera** (Andrew Ng)
   - https://www.coursera.org/learn/machine-learning
   - Excelente coverage de métricas

3. **Improving Deep Neural Networks - Coursera**
   - https://www.coursera.org/learn/deep-neural-network
   - Train/dev/test sets, bias-variance

### Libros

1. **"Hands-On Machine Learning"** - Aurélien Géron
   - Capítulo sobre performance measures
   - Ejemplos prácticos completos

2. **"Introduction to Statistical Learning"** - James et al.
   - Capítulo sobre resampling methods
   - Cross-validation en detalle

### Videos

1. **StatQuest - Machine Learning Metrics**
   - https://www.youtube.com/c/joshstarmer
   - ROC, AUC, Confusion Matrix explicados

2. **Krish Naik - Model Evaluation**
   - https://www.youtube.com/user/krishnaik06

## Ejemplos Prácticos

### Ejemplo 1: Train-Test Split

```python
from sklearn.model_selection import train_test_split
from sklearn.datasets import load_iris
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score

# Cargar datos
iris = load_iris()
X, y = iris.data, iris.target

# Simple split
X_train, X_test, y_train, y_test = train_test_split(
    X, y, 
    test_size=0.2, 
    random_state=42
)

print(f"Training set size: {len(X_train)}")
print(f"Test set size: {len(X_test)}")

# Entrenar y evaluar
model = RandomForestClassifier(random_state=42)
model.fit(X_train, y_train)

y_pred = model.predict(X_test)
accuracy = accuracy_score(y_test, y_pred)
print(f"Accuracy: {accuracy:.3f}")

# Stratified split (mantiene proporción de clases)
X_train_strat, X_test_strat, y_train_strat, y_test_strat = train_test_split(
    X, y,
    test_size=0.2,
    stratify=y,  # Importante para clases desbalanceadas
    random_state=42
)

print("\nClass distribution in original:", np.bincount(y) / len(y))
print("Class distribution in train:", np.bincount(y_train_strat) / len(y_train_strat))
print("Class distribution in test:", np.bincount(y_test_strat) / len(y_test_strat))
```

### Ejemplo 2: Cross-Validation

```python
from sklearn.model_selection import (
    cross_val_score, cross_validate, 
    KFold, StratifiedKFold
)
import numpy as np

# K-Fold Cross-Validation
kfold = KFold(n_splits=5, shuffle=True, random_state=42)
scores = cross_val_score(model, X, y, cv=kfold, scoring='accuracy')

print("Cross-validation scores:", scores)
print(f"Mean CV accuracy: {scores.mean():.3f} (+/- {scores.std() * 2:.3f})")

# Stratified K-Fold (recomendado para clasificación)
skfold = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
scores_stratified = cross_val_score(model, X, y, cv=skfold, scoring='accuracy')

print(f"\nStratified CV accuracy: {scores_stratified.mean():.3f}")

# Cross-validate con múltiples métricas
scoring = ['accuracy', 'precision_macro', 'recall_macro', 'f1_macro']
cv_results = cross_validate(
    model, X, y, 
    cv=skfold, 
    scoring=scoring,
    return_train_score=True
)

for metric in scoring:
    train_scores = cv_results[f'train_{metric}']
    test_scores = cv_results[f'test_{metric}']
    print(f"\n{metric}:")
    print(f"  Train: {train_scores.mean():.3f} (+/- {train_scores.std():.3f})")
    print(f"  Test:  {test_scores.mean():.3f} (+/- {test_scores.std():.3f})")

# Leave-One-Out CV (para datasets pequeños)
from sklearn.model_selection import LeaveOneOut

loo = LeaveOneOut()
scores_loo = cross_val_score(model, X, y, cv=loo)
print(f"\nLOOCV accuracy: {scores_loo.mean():.3f}")
```

### Ejemplo 3: Métricas de Clasificación

```python
from sklearn.metrics import (
    accuracy_score, precision_score, recall_score, 
    f1_score, classification_report, confusion_matrix
)
from sklearn.datasets import make_classification

# Generar datos desbalanceados
X, y = make_classification(
    n_samples=1000, n_classes=2, weights=[0.9, 0.1],
    n_features=20, random_state=42
)

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, stratify=y, random_state=42
)

model = RandomForestClassifier(random_state=42)
model.fit(X_train, y_train)
y_pred = model.predict(X_test)

# Métricas básicas
print("Accuracy:", accuracy_score(y_test, y_pred))
print("Precision:", precision_score(y_test, y_pred))
print("Recall:", recall_score(y_test, y_pred))
print("F1-Score:", f1_score(y_test, y_pred))

# Classification Report (todas las métricas)
print("\nClassification Report:")
print(classification_report(y_test, y_pred))

# Confusion Matrix
cm = confusion_matrix(y_test, y_pred)
print("\nConfusion Matrix:")
print(cm)

# Visualizar Confusion Matrix
import seaborn as sns
import matplotlib.pyplot as plt

plt.figure(figsize=(8, 6))
sns.heatmap(cm, annot=True, fmt='d', cmap='Blues')
plt.xlabel('Predicted')
plt.ylabel('Actual')
plt.title('Confusion Matrix')
plt.show()

# Métricas detalladas desde confusion matrix
tn, fp, fn, tp = cm.ravel()
print(f"\nTrue Negatives: {tn}")
print(f"False Positives: {fp}")
print(f"False Negatives: {fn}")
print(f"True Positives: {tp}")

# Calcular métricas manualmente
precision_manual = tp / (tp + fp)
recall_manual = tp / (tp + fn)
f1_manual = 2 * (precision_manual * recall_manual) / (precision_manual + recall_manual)

print(f"\nManual Precision: {precision_manual:.3f}")
print(f"Manual Recall: {recall_manual:.3f}")
print(f"Manual F1: {f1_manual:.3f}")
```

### Ejemplo 4: ROC Curve y AUC

```python
from sklearn.metrics import roc_curve, roc_auc_score, auc

# Obtener probabilidades
y_pred_proba = model.predict_proba(X_test)[:, 1]

# ROC Curve
fpr, tpr, thresholds = roc_curve(y_test, y_pred_proba)
roc_auc = auc(fpr, tpr)

# Plot ROC Curve
plt.figure(figsize=(10, 6))
plt.plot(fpr, tpr, color='darkorange', lw=2, 
         label=f'ROC curve (AUC = {roc_auc:.2f})')
plt.plot([0, 1], [0, 1], color='navy', lw=2, linestyle='--', label='Random')
plt.xlim([0.0, 1.0])
plt.ylim([0.0, 1.05])
plt.xlabel('False Positive Rate')
plt.ylabel('True Positive Rate')
plt.title('Receiver Operating Characteristic (ROC) Curve')
plt.legend(loc="lower right")
plt.grid(alpha=0.3)
plt.show()

# AUC Score
auc_score = roc_auc_score(y_test, y_pred_proba)
print(f"ROC-AUC Score: {auc_score:.3f}")

# Encontrar mejor threshold
optimal_idx = np.argmax(tpr - fpr)
optimal_threshold = thresholds[optimal_idx]
print(f"Optimal threshold: {optimal_threshold:.3f}")

# Precision-Recall Curve
from sklearn.metrics import precision_recall_curve, average_precision_score

precision, recall, thresholds_pr = precision_recall_curve(y_test, y_pred_proba)
ap_score = average_precision_score(y_test, y_pred_proba)

plt.figure(figsize=(10, 6))
plt.plot(recall, precision, color='blue', lw=2,
         label=f'PR curve (AP = {ap_score:.2f})')
plt.xlabel('Recall')
plt.ylabel('Precision')
plt.title('Precision-Recall Curve')
plt.legend(loc="best")
plt.grid(alpha=0.3)
plt.show()
```

### Ejemplo 5: Métricas de Regresión

```python
from sklearn.metrics import (
    mean_squared_error, mean_absolute_error, 
    r2_score, mean_absolute_percentage_error
)
from sklearn.linear_model import LinearRegression
from sklearn.datasets import make_regression

# Generar datos de regresión
X, y = make_regression(n_samples=100, n_features=10, noise=10, random_state=42)

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# Entrenar modelo
reg_model = LinearRegression()
reg_model.fit(X_train, y_train)
y_pred = reg_model.predict(X_test)

# Métricas de regresión
mse = mean_squared_error(y_test, y_pred)
rmse = np.sqrt(mse)
mae = mean_absolute_error(y_test, y_pred)
r2 = r2_score(y_test, y_pred)
mape = mean_absolute_percentage_error(y_test, y_pred)

print("Regression Metrics:")
print(f"MSE: {mse:.2f}")
print(f"RMSE: {rmse:.2f}")
print(f"MAE: {mae:.2f}")
print(f"R² Score: {r2:.3f}")
print(f"MAPE: {mape:.3f}")

# Visualizar predicciones vs reales
plt.figure(figsize=(10, 6))
plt.scatter(y_test, y_pred, alpha=0.5)
plt.plot([y_test.min(), y_test.max()], [y_test.min(), y_test.max()], 
         'r--', lw=2)
plt.xlabel('Actual Values')
plt.ylabel('Predicted Values')
plt.title('Actual vs Predicted')
plt.grid(alpha=0.3)
plt.show()

# Residuals plot
residuals = y_test - y_pred

plt.figure(figsize=(10, 6))
plt.scatter(y_pred, residuals, alpha=0.5)
plt.axhline(y=0, color='r', linestyle='--')
plt.xlabel('Predicted Values')
plt.ylabel('Residuals')
plt.title('Residual Plot')
plt.grid(alpha=0.3)
plt.show()
```

### Ejemplo 6: Time Series Validation

```python
from sklearn.model_selection import TimeSeriesSplit

# Time Series Split (respeta orden temporal)
tscv = TimeSeriesSplit(n_splits=5)

# Visualizar splits
fig, axes = plt.subplots(5, 1, figsize=(15, 10))

for idx, (train_idx, test_idx) in enumerate(tscv.split(X)):
    axes[idx].plot(train_idx, [0] * len(train_idx), 'o', label='Train', markersize=2)
    axes[idx].plot(test_idx, [0] * len(test_idx), 'o', label='Test', markersize=2)
    axes[idx].set_title(f'Fold {idx + 1}')
    axes[idx].legend()
    axes[idx].set_yticks([])

plt.tight_layout()
plt.show()

# Validación temporal
scores_ts = cross_val_score(model, X, y, cv=tscv)
print(f"Time Series CV scores: {scores_ts}")
print(f"Mean: {scores_ts.mean():.3f} (+/- {scores_ts.std():.3f})")
```

### Ejemplo 7: Learning Curves

```python
from sklearn.model_selection import learning_curve

# Learning curves
train_sizes, train_scores, test_scores = learning_curve(
    model, X, y,
    cv=5,
    n_jobs=-1,
    train_sizes=np.linspace(0.1, 1.0, 10),
    scoring='accuracy'
)

train_mean = train_scores.mean(axis=1)
train_std = train_scores.std(axis=1)
test_mean = test_scores.mean(axis=1)
test_std = test_scores.std(axis=1)

# Plot learning curves
plt.figure(figsize=(10, 6))
plt.plot(train_sizes, train_mean, label='Training score', color='blue', marker='o')
plt.fill_between(train_sizes, train_mean - train_std, train_mean + train_std, 
                 alpha=0.15, color='blue')
plt.plot(train_sizes, test_mean, label='Cross-validation score', 
         color='red', marker='o')
plt.fill_between(train_sizes, test_mean - test_std, test_mean + test_std, 
                 alpha=0.15, color='red')

plt.xlabel('Training Set Size')
plt.ylabel('Accuracy Score')
plt.title('Learning Curves')
plt.legend(loc='best')
plt.grid(alpha=0.3)
plt.show()

# Diagnóstico
if train_mean[-1] > test_mean[-1] + 0.1:
    print("Possible overfitting detected")
elif train_mean[-1] < 0.8 and test_mean[-1] < 0.8:
    print("Possible underfitting detected")
else:
    print("Model seems well-fitted")
```

## Validation Strategies Comparison

```python
from sklearn.model_selection import ShuffleSplit, RepeatedStratifiedKFold
import pandas as pd

# Comparar diferentes estrategias
strategies = {
    'Simple Split': None,
    '5-Fold CV': KFold(n_splits=5, shuffle=True, random_state=42),
    'Stratified 5-Fold': StratifiedKFold(n_splits=5, shuffle=True, random_state=42),
    'Repeated CV': RepeatedStratifiedKFold(n_splits=5, n_repeats=3, random_state=42),
    'Shuffle Split': ShuffleSplit(n_splits=10, test_size=0.2, random_state=42)
}

results = []

for name, cv_strategy in strategies.items():
    if cv_strategy is None:
        # Simple split
        model.fit(X_train, y_train)
        score = model.score(X_test, y_test)
        scores_array = [score]
    else:
        scores_array = cross_val_score(model, X, y, cv=cv_strategy)
    
    results.append({
        'Strategy': name,
        'Mean Score': scores_array.mean(),
        'Std Score': scores_array.std(),
        'Min Score': scores_array.min(),
        'Max Score': scores_array.max()
    })

results_df = pd.DataFrame(results)
print(results_df)
```

## Mejores Prácticas

### 1. Siempre Usar Stratified Split

```python
# ❌ Malo - puede tener distribución desigual
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)

# ✅ Bueno - mantiene proporción de clases
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, stratify=y, random_state=42
)
```

### 2. Elegir Métrica Apropiada

```python
# Para clases desbalanceadas
# ✅ Usar F1-score, ROC-AUC, no accuracy

# Para minimizar falsos positivos
# ✅ Optimizar precision

# Para minimizar falsos negativos  
# ✅ Optimizar recall

# Balanceado
# ✅ Usar F1-score
```

### 3. Cross-Validation Apropiada

```python
# Para time series
# ✅ TimeSeriesSplit
cv = TimeSeriesSplit(n_splits=5)

# Para clasificación
# ✅ StratifiedKFold
cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)

# Para datasets pequeños
# ✅ LeaveOneOut o RepeatedKFold
```

### 4. Evitar Data Leakage

```python
# ❌ Malo - scaling antes de split
X_scaled = scaler.fit_transform(X)
X_train, X_test = train_test_split(X_scaled)

# ✅ Bueno - split primero
X_train, X_test = train_test_split(X)
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)
```

## Proyectos Sugeridos

1. **Imbalanced Classification**
   - Dataset desbalanceado
   - Comparar métricas (accuracy vs F1)
   - SMOTE, class weights

2. **Model Comparison Framework**
   - Múltiples modelos
   - Cross-validation rigurosa
   - Statistical testing

3. **Time Series Validation**
   - Datos temporales
   - Walk-forward validation
   - Out-of-time testing

4. **A/B Testing Simulator**
   - Comparar versiones de modelo
   - Statistical significance
   - Business metrics

5. **Automated Model Validation**
   - Pipeline de validación
   - Múltiples métricas
   - Reporting automático

## Checklist de Aprendizaje

- [ ] Dominar train-test split y stratification
- [ ] Implementar cross-validation correctamente
- [ ] Calcular todas las métricas de clasificación
- [ ] Interpretar confusion matrix
- [ ] Entender ROC curves y AUC
- [ ] Elegir métricas apropiadas por problema
- [ ] Detectar overfitting con learning curves
- [ ] Aplicar time series validation
- [ ] Evitar data leakage
- [ ] Comparar modelos estadísticamente

## Próximos Pasos

1. Aprender **Statistical Testing** para comparar modelos
2. Estudiar **Hyperparameter Tuning** con GridSearch/RandomSearch
3. Explorar **Nested Cross-Validation**
4. Dominar **Business Metrics** vs ML metrics
5. Profundizar en **Model Monitoring** en producción

---
**Tiempo estimado**: 3-4 semanas
