# Álgebra Lineal para AI

## Descripción

El álgebra lineal es la base matemática fundamental para entender cómo funcionan los algoritmos de Machine Learning y Deep Learning. Los datos se representan como vectores y matrices, y las transformaciones se realizan mediante operaciones lineales.

## Conceptos Clave

### 1. **Vectores**
- Representación de datos y características
- Operaciones: suma, producto escalar, norma
- Espacios vectoriales

### 2. **Matrices**
- Transformaciones lineales
- Operaciones: multiplicación, transposición, inversa
- Matrices especiales: identidad, diagonal, ortogonal

### 3. **Transformaciones Lineales**
- Proyecciones
- Rotaciones
- Escalado

### 4. **Valores y Vectores Propios (Eigenvalues/Eigenvectors)**
- PCA (Principal Component Analysis)
- Descomposición espectral
- Aplicaciones en reducción de dimensionalidad

### 5. **Descomposición de Matrices**
- SVD (Singular Value Decomposition)
- LU Decomposition
- QR Decomposition

## Aplicaciones en AI/ML

- **Redes Neuronales**: Los pesos son matrices, las entradas son vectores
- **PCA**: Reducción de dimensionalidad usando eigenvalues
- **Recommender Systems**: Factorización de matrices
- **Computer Vision**: Transformaciones de imágenes
- **NLP**: Word embeddings como vectores

## Recursos de Aprendizaje

### Cursos Online

1. **3Blue1Brown - Essence of Linear Algebra** (Gratis)
   - https://www.youtube.com/playlist?list=PLZHQObOWTQDPD3MizzM2xVFitgF8hE_ab
   - Visualizaciones increíbles, perfecto para intuición

2. **MIT OpenCourseWare - Linear Algebra** (Gratis)
   - https://ocw.mit.edu/courses/18-06-linear-algebra-spring-2010/
   - Profesor: Gilbert Strang
   - https://www.youtube.com/playlist?list=PL49CF3715CB9EF31D

3. **Khan Academy - Linear Algebra** (Gratis)
   - https://www.khanacademy.org/math/linear-algebra
   - Excelente para principiantes

4. **Coursera - Mathematics for Machine Learning: Linear Algebra** (Imperial College London)
   - https://www.coursera.org/learn/linear-algebra-machine-learning
   - Enfocado en ML

### Libros

1. **"Linear Algebra and Its Applications"** - Gilbert Strang
   - Libro clásico y muy completo
   - http://math.mit.edu/~gs/linearalgebra/

2. **"Introduction to Linear Algebra"** - Gilbert Strang
   - Versión más accesible del anterior

3. **"Linear Algebra Done Right"** - Sheldon Axler
   - Enfoque más teórico

4. **"No Bullshit Guide to Linear Algebra"** - Ivan Savov
   - Práctico y directo al punto

### Documentación y Referencias

1. **NumPy Linear Algebra**
   - https://numpy.org/doc/stable/reference/routines.linalg.html
   - Documentación oficial de funciones de álgebra lineal

2. **SciPy Linear Algebra**
   - https://docs.scipy.org/doc/scipy/reference/linalg.html
   - Funciones avanzadas

3. **Wolfram MathWorld - Linear Algebra**
   - https://mathworld.wolfram.com/LinearAlgebra.html
   - Referencia matemática completa

### Videos Recomendados

1. **3Blue1Brown Series** (Imprescindible)
   - https://www.youtube.com/watch?v=fNk_zzaMoSs (Vectors)
   - https://www.youtube.com/watch?v=kYB8IZa5AuE (Linear transformations)
   - https://www.youtube.com/watch?v=PFDu9oVAE-g (Matrix multiplication)

2. **StatQuest - Linear Algebra Basics**
   - https://www.youtube.com/watch?v=fNk_zzaMoSs
   - Explicaciones simples y claras

3. **Khan Academy Videos**
   - https://www.youtube.com/user/khanacademy

## Herramientas y Bibliotecas

### Python

```python
import numpy as np
from scipy import linalg

# Crear vectores y matrices
v = np.array([1, 2, 3])
A = np.array([[1, 2], [3, 4]])

# Operaciones básicas
dot_product = np.dot(v, v)
matrix_mult = A @ A.T

# Eigenvalues y eigenvectors
eigenvalues, eigenvectors = np.linalg.eig(A)

# SVD
U, S, Vt = np.linalg.svd(A)

# Inversa de matriz
A_inv = np.linalg.inv(A)

# Norma de vector
norm = np.linalg.norm(v)
```

### Bibliotecas Principales

1. **NumPy** - Operaciones básicas
2. **SciPy** - Funciones avanzadas
3. **SymPy** - Álgebra simbólica
4. **JAX** - Diferenciación automática

## Ejemplos Prácticos

### Ejemplo 1: PCA desde Cero

```python
import numpy as np
from sklearn.datasets import load_iris

# Cargar datos
iris = load_iris()
X = iris.data

# Centrar los datos
X_centered = X - np.mean(X, axis=0)

# Calcular matriz de covarianza
cov_matrix = np.cov(X_centered.T)

# Eigenvalues y eigenvectors
eigenvalues, eigenvectors = np.linalg.eig(cov_matrix)

# Ordenar por eigenvalues
idx = eigenvalues.argsort()[::-1]
eigenvalues = eigenvalues[idx]
eigenvectors = eigenvectors[:, idx]

# Proyectar a 2 componentes principales
k = 2
principal_components = eigenvectors[:, :k]
X_pca = X_centered @ principal_components

print(f"Varianza explicada: {eigenvalues[:k].sum() / eigenvalues.sum():.2%}")
```

### Ejemplo 2: Regresión Lineal con Álgebra

```python
import numpy as np

# Datos de ejemplo
X = np.array([[1, 1], [1, 2], [1, 3], [1, 4]])  # Agregar columna de 1s para bias
y = np.array([2, 4, 6, 8])

# Solución de mínimos cuadrados: β = (X^T X)^(-1) X^T y
beta = np.linalg.inv(X.T @ X) @ X.T @ y

print(f"Coeficientes: {beta}")
# Predicciones
y_pred = X @ beta
```

### Ejemplo 3: Sistema de Recomendación Simple

```python
import numpy as np

# Matrix de ratings (usuarios x películas)
# 0 significa no visto
R = np.array([
    [5, 3, 0, 1],
    [4, 0, 0, 1],
    [1, 1, 0, 5],
    [1, 0, 0, 4],
    [0, 1, 5, 4],
])

# SVD para factorización de matriz
U, S, Vt = np.linalg.svd(R)

# Reconstruir con k factores latentes
k = 2
R_reconstructed = U[:, :k] @ np.diag(S[:k]) @ Vt[:k, :]

print("Predicciones de ratings:")
print(R_reconstructed)
```

## Proyectos Sugeridos

1. **Implementar PCA desde cero**
   - No usar sklearn, solo NumPy
   - Comparar con sklearn.decomposition.PCA

2. **Sistema de Recomendación con Matrix Factorization**
   - Dataset: MovieLens
   - Implementar SVD

3. **Image Compression con SVD**
   - Comprimir imágenes usando diferentes números de valores singulares
   - Visualizar la calidad vs compresión

4. **Linear Regression desde cero**
   - Implementar usando ecuaciones normales
   - Comparar con gradient descent

5. **Face Recognition básico con Eigenfaces**
   - Usar PCA en imágenes de rostros
   - Dataset: LFW (Labeled Faces in the Wild)

## Mejores Prácticas

1. **Visualiza siempre que sea posible**
   - Usa matplotlib para graficar vectores y transformaciones
   - La intuición geométrica es clave

2. **Verifica dimensiones**
   - Asegúrate de que las dimensiones sean compatibles
   - Usa `shape` constantemente

3. **Estabilidad numérica**
   - Cuidado con matrices mal condicionadas
   - Usa `np.linalg.cond()` para verificar

4. **Eficiencia**
   - Usa operaciones vectorizadas
   - Evita loops cuando sea posible

5. **Broadcasting de NumPy**
   - Aprende a usar broadcasting para operaciones eficientes
   - https://numpy.org/doc/stable/user/basics.broadcasting.html

## Recursos Adicionales

### Interactive Tools

1. **Seeing Theory - Linear Algebra**
   - https://seeing-theory.brown.edu/linear-algebra/index.html
   - Visualizaciones interactivas

2. **Matrix Calculus**
   - http://www.matrixcalculus.org/
   - Calculadora de derivadas matriciales

### Papers y Artículos

1. **"A Tutorial on Principal Component Analysis"** - Jonathon Shlens
   - https://arxiv.org/abs/1404.1100

2. **"Matrix Factorization Techniques for Recommender Systems"**
   - https://datajobs.com/data-science-repo/Recommender-Systems-[Netflix].pdf

### Comunidades

1. **Math Stack Exchange**
   - https://math.stackexchange.com/questions/tagged/linear-algebra

2. **r/learnmath**
   - https://www.reddit.com/r/learnmath/

## Próximos Pasos

Después de dominar Álgebra Lineal:
1. Pasar a **Cálculo** (derivadas y gradientes)
2. Estudiar **Estadística y Probabilidad**
3. Combinar todo en **Optimización**

## Checklist de Aprendizaje

- [ ] Entender vectores y espacios vectoriales
- [ ] Dominar operaciones con matrices
- [ ] Comprender transformaciones lineales
- [ ] Calcular eigenvalues y eigenvectors
- [ ] Implementar PCA desde cero
- [ ] Realizar SVD y aplicaciones
- [ ] Resolver sistemas de ecuaciones lineales
- [ ] Aplicar en un proyecto real de ML

---
**Tiempo estimado de estudio**: 3-4 semanas dedicando 2-3 horas diarias
