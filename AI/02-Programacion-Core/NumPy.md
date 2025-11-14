# NumPy - Computación Numérica en Python

## Descripción

NumPy (Numerical Python) es la biblioteca fundamental para computación científica en Python. Proporciona arrays multidimensionales de alto rendimiento y herramientas para trabajar con ellos. Es la base sobre la que se construyen bibliotecas como Pandas, scikit-learn, TensorFlow y PyTorch.

## Conceptos Clave

### 1. **Arrays de NumPy (ndarray)**
- Arrays multidimensionales homogéneos
- Más rápidos que listas de Python
- Operaciones vectorizadas
- Broadcasting

### 2. **Indexing y Slicing**
- Indexación básica y avanzada
- Fancy indexing
- Boolean indexing
- Multidimensional slicing

### 3. **Operaciones Matriciales**
- Productos escalares, vectoriales y matriciales
- Transposición
- Determinantes e inversas
- Eigenvalues y eigenvectors

### 4. **Broadcasting**
- Operaciones entre arrays de diferentes dimensiones
- Reglas de broadcasting
- Optimización de memoria

### 5. **Funciones Universales (ufuncs)**
- Operaciones element-wise
- Agregaciones (sum, mean, std)
- Funciones matemáticas

### 6. **Álgebra Lineal**
- `np.linalg` module
- Operaciones matriciales avanzadas
- Descomposiciones (SVD, QR, Cholesky)

### 7. **Random Numbers**
- Generación de números aleatorios
- Distribuciones de probabilidad
- Semillas para reproducibilidad

## Recursos de Aprendizaje

### Documentación Oficial

1. **NumPy Documentation**
   - https://numpy.org/doc/stable/
   - Referencia completa y tutoriales

2. **NumPy User Guide**
   - https://numpy.org/doc/stable/user/index.html
   - Guía para usuarios

3. **NumPy for MATLAB Users**
   - https://numpy.org/doc/stable/user/numpy-for-matlab-users.html
   - Transición desde MATLAB

### Cursos Online

1. **NumPy Official Tutorial** (Gratis)
   - https://numpy.org/learn/
   - Tutorial oficial interactivo

2. **DataCamp - Introduction to NumPy** 
   - https://www.datacamp.com/courses/introduction-to-numpy
   - Curso interactivo

3. **Coursera - Applied Data Science with Python**
   - https://www.coursera.org/specializations/data-science-python
   - Universidad de Michigan

### Libros

1. **"NumPy Beginner's Guide"** - Ivan Idris
   - Introducción completa

2. **"Guide to NumPy"** - Travis Oliphant
   - Por el creador de NumPy
   - Gratis: https://web.mit.edu/dvp/Public/numpybook.pdf

3. **"Python for Data Analysis"** - Wes McKinney
   - Capítulos sobre NumPy
   - Excelente referencia

### Videos Recomendados

1. **Keith Galli - Complete Python NumPy Tutorial**
   - https://www.youtube.com/watch?v=GB9ByFAIAH4
   - Tutorial completo en 1 hora

2. **freeCodeCamp - NumPy Tutorial**
   - https://www.youtube.com/watch?v=QUT1VHiLmmI
   - Curso completo

3. **Corey Schafer - NumPy Arrays**
   - https://www.youtube.com/watch?v=QUT1VHiLmmI

## Instalación y Setup

```bash
# Instalación con pip
pip install numpy

# Instalación con conda
conda install numpy

# Verificar instalación
python -c "import numpy as np; print(np.__version__)"

# Importar en código
import numpy as np
```

## Ejemplos Prácticos

### Ejemplo 1: Creación de Arrays

```python
import numpy as np

# Array desde lista
arr = np.array([1, 2, 3, 4, 5])
print(f"1D array: {arr}")

# Array 2D desde listas anidadas
matrix = np.array([[1, 2, 3], [4, 5, 6]])
print(f"Shape: {matrix.shape}")  # (2, 3)
print(f"Dtype: {matrix.dtype}")  # int64

# Arrays especiales
zeros = np.zeros((3, 4))  # Array de ceros
ones = np.ones((2, 3))     # Array de unos
identity = np.eye(3)       # Matriz identidad
random = np.random.rand(2, 3)  # Valores aleatorios [0, 1)

# Secuencias
arange = np.arange(0, 10, 2)  # [0, 2, 4, 6, 8]
linspace = np.linspace(0, 1, 5)  # [0, 0.25, 0.5, 0.75, 1]

# Array con tipo específico
float_arr = np.array([1, 2, 3], dtype=np.float32)

# Reshape
reshaped = np.arange(12).reshape(3, 4)
print(reshaped)
```

### Ejemplo 2: Indexing y Slicing

```python
import numpy as np

arr = np.arange(10)
print(arr)  # [0 1 2 3 4 5 6 7 8 9]

# Indexing básico
print(arr[0])      # 0
print(arr[-1])     # 9
print(arr[2:5])    # [2 3 4]
print(arr[::2])    # [0 2 4 6 8]

# Array 2D
matrix = np.arange(12).reshape(3, 4)
print(matrix)
# [[ 0  1  2  3]
#  [ 4  5  6  7]
#  [ 8  9 10 11]]

# Indexing 2D
print(matrix[0, 1])      # 1
print(matrix[1, :])      # [4 5 6 7]  (fila 1)
print(matrix[:, 2])      # [2 6 10]   (columna 2)
print(matrix[0:2, 1:3])  # [[1 2], [5 6]]

# Boolean indexing
arr = np.array([1, 2, 3, 4, 5])
mask = arr > 2
print(arr[mask])  # [3 4 5]
print(arr[arr % 2 == 0])  # [2 4]

# Fancy indexing
arr = np.arange(10)
indices = [1, 3, 5]
print(arr[indices])  # [1 3 5]

# Where
arr = np.array([1, -2, 3, -4, 5])
print(np.where(arr > 0, arr, 0))  # [1 0 3 0 5]
```

### Ejemplo 3: Operaciones Vectorizadas

```python
import numpy as np

# Operaciones aritméticas (element-wise)
a = np.array([1, 2, 3, 4])
b = np.array([10, 20, 30, 40])

print(a + b)      # [11 22 33 44]
print(a * b)      # [10 40 90 160]
print(a ** 2)     # [1 4 9 16]
print(np.sqrt(a)) # [1. 1.41421356 1.73205081 2.]

# Comparaciones
print(a > 2)      # [False False  True  True]
print(a == b)     # [False False False False]

# Funciones universales
arr = np.array([0, np.pi/2, np.pi])
print(np.sin(arr))  # [0. 1. 0.]
print(np.exp(arr))  # [1. 4.81 23.14]
print(np.log(arr + 1))

# Agregaciones
data = np.random.randn(100)
print(f"Mean: {np.mean(data):.3f}")
print(f"Std: {np.std(data):.3f}")
print(f"Min: {np.min(data):.3f}")
print(f"Max: {np.max(data):.3f}")
print(f"Sum: {np.sum(data):.3f}")

# Agregaciones por eje
matrix = np.random.randn(3, 4)
print(f"Mean por columna: {np.mean(matrix, axis=0)}")
print(f"Sum por fila: {np.sum(matrix, axis=1)}")
```

### Ejemplo 4: Broadcasting

```python
import numpy as np

# Broadcasting básico
arr = np.array([[1, 2, 3], [4, 5, 6]])
print(arr + 10)
# [[11 12 13]
#  [14 15 16]]

# Broadcasting con arrays 1D
matrix = np.array([[1, 2, 3], [4, 5, 6]])
vector = np.array([10, 20, 30])
print(matrix + vector)
# [[11 22 33]
#  [14 25 36]]

# Broadcasting con reshape
a = np.arange(3).reshape(3, 1)  # shape (3, 1)
b = np.arange(3)                # shape (3,)
print(a + b)
# [[0 1 2]
#  [1 2 3]
#  [2 3 4]]

# Normalización usando broadcasting
data = np.random.randn(5, 3)
mean = np.mean(data, axis=0)
std = np.std(data, axis=0)
normalized = (data - mean) / std
print(f"Normalized mean: {np.mean(normalized, axis=0)}")  # ~[0, 0, 0]
print(f"Normalized std: {np.std(normalized, axis=0)}")    # ~[1, 1, 1]
```

### Ejemplo 5: Álgebra Lineal

```python
import numpy as np

# Producto escalar (dot product)
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])
dot_product = np.dot(a, b)  # 1*4 + 2*5 + 3*6 = 32
print(f"Dot product: {dot_product}")

# Multiplicación de matrices
A = np.array([[1, 2], [3, 4]])
B = np.array([[5, 6], [7, 8]])
C = A @ B  # Equivalente a np.matmul(A, B)
print(f"Matrix multiplication:\n{C}")

# Transposición
print(f"Transpose:\n{A.T}")

# Determinante
det = np.linalg.det(A)
print(f"Determinant: {det}")

# Inversa
inv = np.linalg.inv(A)
print(f"Inverse:\n{inv}")
print(f"A @ inv(A):\n{A @ inv}")  # Debería dar identidad

# Eigenvalues y eigenvectors
eigenvalues, eigenvectors = np.linalg.eig(A)
print(f"Eigenvalues: {eigenvalues}")
print(f"Eigenvectors:\n{eigenvectors}")

# SVD (Singular Value Decomposition)
U, s, Vt = np.linalg.svd(A)
print(f"U:\n{U}")
print(f"Singular values: {s}")
print(f"Vt:\n{Vt}")

# Resolver sistema de ecuaciones Ax = b
A = np.array([[3, 1], [1, 2]])
b = np.array([9, 8])
x = np.linalg.solve(A, b)
print(f"Solution x: {x}")
print(f"Verification Ax: {A @ x}")  # Debería ser igual a b

# Normas
vector = np.array([3, 4])
l2_norm = np.linalg.norm(vector)  # sqrt(3^2 + 4^2) = 5
print(f"L2 norm: {l2_norm}")
```

### Ejemplo 6: Números Aleatorios

```python
import numpy as np

# Seed para reproducibilidad
np.random.seed(42)

# Distribución uniforme [0, 1)
uniform = np.random.rand(5)
print(f"Uniform: {uniform}")

# Distribución uniforme en rango específico
uniform_range = np.random.uniform(low=10, high=20, size=5)
print(f"Uniform [10, 20): {uniform_range}")

# Distribución normal (Gaussiana)
normal = np.random.randn(5)  # mean=0, std=1
print(f"Normal (0, 1): {normal}")

# Normal con parámetros específicos
normal_custom = np.random.normal(loc=100, scale=15, size=5)
print(f"Normal (100, 15): {normal_custom}")

# Integers aleatorios
integers = np.random.randint(low=1, high=100, size=10)
print(f"Random integers: {integers}")

# Choice (selección aleatoria)
options = ['apple', 'banana', 'cherry']
choices = np.random.choice(options, size=5, replace=True)
print(f"Random choices: {choices}")

# Shuffle (mezclar in-place)
arr = np.arange(10)
np.random.shuffle(arr)
print(f"Shuffled: {arr}")

# Permutation (nueva copia mezclada)
perm = np.random.permutation(10)
print(f"Permutation: {perm}")

# Distribuciones específicas
exponential = np.random.exponential(scale=2.0, size=1000)
binomial = np.random.binomial(n=10, p=0.5, size=1000)
poisson = np.random.poisson(lam=3.0, size=1000)

# Nueva API de random (recomendada)
rng = np.random.default_rng(seed=42)
samples = rng.normal(size=5)
print(f"New API normal: {samples}")
```

### Ejemplo 7: Operaciones Avanzadas

```python
import numpy as np

# Stack arrays
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])

# Vertical stack (vstack)
v_stack = np.vstack([a, b])
print(f"Vstack:\n{v_stack}")

# Horizontal stack (hstack)
h_stack = np.hstack([a, b])
print(f"Hstack: {h_stack}")

# Concatenate
concat = np.concatenate([a, b])
print(f"Concatenate: {concat}")

# Split
arr = np.arange(12)
splits = np.split(arr, 3)  # Divide en 3 partes iguales
print(f"Splits: {splits}")

# Meshgrid (útil para plotting)
x = np.linspace(-5, 5, 5)
y = np.linspace(-5, 5, 5)
X, Y = np.meshgrid(x, y)
print(f"X:\n{X}")
print(f"Y:\n{Y}")

# Vectorize (aplicar función a cada elemento)
def my_function(x, y):
    return x**2 + y**2

vectorized = np.vectorize(my_function)
result = vectorized(X, Y)
print(f"Vectorized result shape: {result.shape}")

# Einsum (Einstein summation) - operaciones tensoriales eficientes
A = np.random.randn(3, 4)
B = np.random.randn(4, 5)

# Multiplicación de matrices
C = np.einsum('ik,kj->ij', A, B)
# Equivalente a A @ B

# Traza de matriz
trace = np.einsum('ii', A[:3, :3])

# Producto elemento a elemento y suma
arr1 = np.array([1, 2, 3])
arr2 = np.array([4, 5, 6])
dot = np.einsum('i,i->', arr1, arr2)  # dot product
```

## Optimización y Performance

### Memory Layout

```python
import numpy as np

# C-contiguous (row-major) vs F-contiguous (column-major)
arr_c = np.array([[1, 2, 3], [4, 5, 6]], order='C')
arr_f = np.array([[1, 2, 3], [4, 5, 6]], order='F')

print(f"C-contiguous: {arr_c.flags['C_CONTIGUOUS']}")
print(f"F-contiguous: {arr_f.flags['F_CONTIGUOUS']}")

# Views vs Copies
arr = np.arange(10)
view = arr[::2]  # View (no copia)
copy = arr[::2].copy()  # Copia explícita

view[0] = 999  # Modifica arr original
print(f"Original arr: {arr}")

# Verificar si es view o copy
print(f"Is view: {view.base is arr}")
print(f"Is copy: {copy.base is None}")
```

### Vectorización vs Loops

```python
import numpy as np
import time

# Comparar performance: loops vs vectorización
n = 1000000

# Con loops (lento)
start = time.time()
result_loop = []
for i in range(n):
    result_loop.append(i ** 2)
result_loop = np.array(result_loop)
time_loop = time.time() - start

# Vectorizado (rápido)
start = time.time()
arr = np.arange(n)
result_vec = arr ** 2
time_vec = time.time() - start

print(f"Loop time: {time_loop:.4f}s")
print(f"Vectorized time: {time_vec:.4f}s")
print(f"Speedup: {time_loop / time_vec:.1f}x")
```

## Casos de Uso en ML/AI

### Normalización de Datos

```python
import numpy as np

def normalize_data(X, method='standard'):
    """
    Normaliza datos usando diferentes métodos
    
    Parameters:
    X: array de shape (n_samples, n_features)
    method: 'standard', 'minmax', o 'l2'
    """
    if method == 'standard':
        # Z-score normalization
        mean = np.mean(X, axis=0)
        std = np.std(X, axis=0)
        return (X - mean) / (std + 1e-8)
    
    elif method == 'minmax':
        # Min-max scaling a [0, 1]
        min_val = np.min(X, axis=0)
        max_val = np.max(X, axis=0)
        return (X - min_val) / (max_val - min_val + 1e-8)
    
    elif method == 'l2':
        # L2 normalization
        norms = np.linalg.norm(X, axis=1, keepdims=True)
        return X / (norms + 1e-8)

# Ejemplo
X = np.random.randn(100, 5)
X_normalized = normalize_data(X, method='standard')
print(f"Mean after normalization: {np.mean(X_normalized, axis=0)}")
print(f"Std after normalization: {np.std(X_normalized, axis=0)}")
```

### One-Hot Encoding

```python
import numpy as np

def one_hot_encode(labels, num_classes=None):
    """
    Convierte labels a one-hot encoding
    
    Parameters:
    labels: array de labels (n_samples,)
    num_classes: número de clases (opcional)
    """
    if num_classes is None:
        num_classes = labels.max() + 1
    
    one_hot = np.zeros((len(labels), num_classes))
    one_hot[np.arange(len(labels)), labels] = 1
    return one_hot

# Ejemplo
labels = np.array([0, 1, 2, 1, 0])
one_hot = one_hot_encode(labels, num_classes=3)
print(one_hot)
```

### Batch Processing

```python
import numpy as np

def create_batches(X, y, batch_size=32, shuffle=True):
    """
    Crea batches para training
    
    Parameters:
    X: features (n_samples, n_features)
    y: labels (n_samples,)
    batch_size: tamaño del batch
    shuffle: mezclar datos
    """
    n_samples = len(X)
    indices = np.arange(n_samples)
    
    if shuffle:
        np.random.shuffle(indices)
    
    for start_idx in range(0, n_samples, batch_size):
        end_idx = min(start_idx + batch_size, n_samples)
        batch_indices = indices[start_idx:end_idx]
        yield X[batch_indices], y[batch_indices]

# Ejemplo de uso
X = np.random.randn(1000, 10)
y = np.random.randint(0, 5, size=1000)

for batch_X, batch_y in create_batches(X, y, batch_size=32):
    # Entrenar con batch
    print(f"Batch shape: {batch_X.shape}")
    break  # Solo mostrar primer batch
```

## Mejores Prácticas

### 1. Evitar Loops

```python
# ❌ Malo
result = []
for i in range(len(arr)):
    result.append(arr[i] ** 2)

# ✅ Bueno
result = arr ** 2
```

### 2. Usar Broadcasting

```python
# ❌ Malo
result = np.zeros((5, 3))
for i in range(5):
    result[i] = arr[i] + vector

# ✅ Bueno
result = arr + vector  # Broadcasting automático
```

### 3. Preallocate Arrays

```python
# ❌ Malo
result = []
for i in range(1000):
    result.append(some_computation(i))
result = np.array(result)

# ✅ Bueno
result = np.empty(1000)
for i in range(1000):
    result[i] = some_computation(i)
```

### 4. Usar Funciones In-Place

```python
# Cuando sea posible, usar operaciones in-place
arr += 10  # Más eficiente que arr = arr + 10
arr *= 2
```

### 5. Verificar dtype

```python
# Usar el dtype apropiado para ahorrar memoria
arr_int8 = np.array([1, 2, 3], dtype=np.int8)   # 1 byte por elemento
arr_float16 = np.array([1, 2, 3], dtype=np.float16)  # 2 bytes por elemento
```

## Proyectos Sugeridos

1. **Image Processing Library**
   - Implementar filtros (blur, edge detection)
   - Transformaciones (rotate, scale)
   - Solo usando NumPy

2. **Neural Network desde Cero**
   - Forward propagation
   - Backpropagation
   - Training loop
   - Solo NumPy, sin frameworks

3. **Linear Algebra Visualizer**
   - Visualizar transformaciones matriciales
   - Eigenvalues/eigenvectors
   - SVD decomposition

4. **Data Augmentation Tool**
   - Para imágenes
   - Rotaciones, flips, crops
   - Batch processing

5. **Statistical Analysis Tool**
   - Calcular estadísticas descriptivas
   - Correlaciones
   - Distribuciones

## Recursos Adicionales

### Tutoriales Interactivos

1. **NumPy Exercises**
   - https://github.com/rougier/numpy-100
   - 100 ejercicios NumPy

2. **NumPy Tutorial - W3Schools**
   - https://www.w3schools.com/python/numpy/default.asp

### Papers y Artículos

1. **"Array programming with NumPy"** (Nature, 2020)
   - https://www.nature.com/articles/s41586-020-2649-2
   - Paper oficial de NumPy

### Cheat Sheets

1. **DataCamp NumPy Cheat Sheet**
   - https://www.datacamp.com/cheat-sheet/numpy-cheat-sheet-data-analysis-in-python

2. **NumPy Quickstart**
   - https://numpy.org/doc/stable/user/quickstart.html

## Funciones Útiles de Referencia

```python
# Creación
np.array(), np.zeros(), np.ones(), np.empty(), np.eye()
np.arange(), np.linspace(), np.logspace()
np.random.rand(), np.random.randn(), np.random.randint()

# Shape manipulation
arr.reshape(), arr.flatten(), arr.ravel()
np.expand_dims(), np.squeeze()
np.transpose(), arr.T

# Combining/Splitting
np.concatenate(), np.vstack(), np.hstack()
np.split(), np.vsplit(), np.hsplit()

# Indexing
arr[indices], arr[mask], np.where()

# Math operations
np.add(), np.subtract(), np.multiply(), np.divide()
np.sum(), np.mean(), np.std(), np.var()
np.min(), np.max(), np.argmin(), np.argmax()
np.abs(), np.sqrt(), np.exp(), np.log()
np.sin(), np.cos(), np.tan()

# Linear algebra
np.dot(), np.matmul(), arr @ arr2
np.linalg.inv(), np.linalg.det(), np.linalg.eig()
np.linalg.svd(), np.linalg.solve()
np.linalg.norm()

# Logic operations
np.all(), np.any()
np.logical_and(), np.logical_or(), np.logical_not()
```

## Checklist de Aprendizaje

- [ ] Crear y manipular arrays multidimensionales
- [ ] Dominar indexing y slicing
- [ ] Entender y usar broadcasting
- [ ] Realizar operaciones vectorizadas
- [ ] Trabajar con álgebra lineal
- [ ] Generar números aleatorios
- [ ] Optimizar código (evitar loops)
- [ ] Implementar algoritmo de ML con NumPy puro
- [ ] Entender memory layout (C vs F order)
- [ ] Usar NumPy en proyecto real

## Próximos Pasos

Después de dominar NumPy:
1. Aprender **Pandas** para manipulación de datos estructurados
2. Profundizar en **SciPy** para computación científica avanzada
3. Usar NumPy con **Matplotlib** para visualización
4. Aplicar en **Machine Learning** con scikit-learn

---
**Tiempo estimado de estudio**: 2-3 semanas dedicando 2-3 horas diarias
