# Python para Machine Learning y AI

## Descripción

Python es el lenguaje dominante en AI/ML por su sintaxis clara, rico ecosistema de bibliotecas científicas, y amplia comunidad. Dominar Python avanzado es esencial para implementar algoritmos eficientemente, trabajar con datos a gran escala, y construir sistemas de producción.

## Conceptos Clave

### 1. **Fundamentos Avanzados**
- List/Dict/Set comprehensions
- Generators y iterators
- Decorators
- Context managers (`with` statement)
- `*args` y `**kwargs`

### 2. **Programación Orientada a Objetos**
- Classes y inheritance
- Magic methods (`__init__`, `__str__`, `__repr__`)
- Properties y descriptors
- Abstract base classes
- Dataclasses

### 3. **Programación Funcional**
- Lambda functions
- `map()`, `filter()`, `reduce()`
- `functools` module
- Closures
- Partial functions

### 4. **Manejo de Datos**
- File I/O (CSV, JSON, pickle)
- Context managers para recursos
- Pathlib para rutas
- Regular expressions

### 5. **Concurrencia y Paralelismo**
- Threading vs Multiprocessing
- `concurrent.futures`
- `async`/`await` (asyncio)
- GIL (Global Interpreter Lock)

### 6. **Testing y Debugging**
- `unittest` y `pytest`
- Assertions
- Debugging con `pdb`
- Logging

### 7. **Tipos y Type Hints**
- Type annotations
- `typing` module
- Mypy para type checking
- Protocolos

## Recursos de Aprendizaje

### Cursos Online

1. **Real Python** (Suscripción)
   - https://realpython.com/
   - Tutoriales de alta calidad
   - Enfoque práctico

2. **Python.org Tutorial** (Gratis)
   - https://docs.python.org/3/tutorial/
   - Documentación oficial
   - Completo y actualizado

3. **Coursera - Python for Everybody**
   - https://www.coursera.org/specializations/python
   - Universidad de Michigan

4. **Corey Schafer - YouTube** (Gratis)
   - https://www.youtube.com/c/Coreyms
   - Excelentes tutoriales en video

### Libros

1. **"Fluent Python"** - Luciano Ramalho
   - Python avanzado e idiomático
   - Imprescindible para nivel intermedio-avanzado

2. **"Python Cookbook"** - David Beazley & Brian K. Jones
   - Recetas prácticas
   - Soluciones a problemas comunes

3. **"Effective Python"** - Brett Slatkin
   - 90 formas de escribir mejor Python
   - Mejores prácticas

4. **"Python for Data Analysis"** - Wes McKinney
   - Creador de Pandas
   - Enfoque en análisis de datos

### Documentación

1. **Python Documentation**
   - https://docs.python.org/3/
   - Referencia oficial completa

2. **PEP 8 - Style Guide**
   - https://pep8.org/
   - Guía de estilo oficial

3. **Python Enhancement Proposals (PEPs)**
   - https://www.python.org/dev/peps/
   - Evolución del lenguaje

### Videos y Canales

1. **Corey Schafer**
   - https://www.youtube.com/c/Coreyms

2. **mCoding**
   - https://www.youtube.com/c/mCodingWithJamesMurphy
   - Python avanzado

3. **ArjanCodes**
   - https://www.youtube.com/c/ArjanCodes
   - Design patterns y arquitectura

## Herramientas Esenciales

### Entornos Virtuales

```bash
# venv (built-in)
python -m venv myenv
source myenv/bin/activate  # Linux/Mac
myenv\Scripts\activate  # Windows

# conda
conda create -n myenv python=3.10
conda activate myenv

# poetry (gestión de dependencias moderna)
poetry new myproject
poetry add numpy pandas
poetry install
```

### Package Management

```bash
# pip
pip install numpy
pip install -r requirements.txt
pip freeze > requirements.txt

# pip-tools (para dependency resolution)
pip-compile requirements.in
pip-sync requirements.txt

# poetry
poetry add numpy@^1.24.0
poetry lock
poetry export -f requirements.txt --output requirements.txt
```

### IDEs y Editores

1. **VS Code** - Editor ligero y extensible
2. **PyCharm** - IDE completo para Python
3. **Jupyter Lab** - Notebooks interactivos
4. **Vim/Neovim** - Editor de terminal

## Ejemplos Prácticos

### Ejemplo 1: Comprehensions Avanzadas

```python
# List comprehension
squares = [x**2 for x in range(10)]

# Con condición
even_squares = [x**2 for x in range(10) if x % 2 == 0]

# Nested comprehension
matrix = [[i*j for j in range(5)] for i in range(5)]

# Dict comprehension
word_lengths = {word: len(word) for word in ['hello', 'world', 'python']}

# Set comprehension
unique_lengths = {len(word) for word in ['hello', 'world', 'python']}

# Generator expression (lazy evaluation)
sum_of_squares = sum(x**2 for x in range(1000000))  # Memory efficient
```

### Ejemplo 2: Decorators

```python
import time
from functools import wraps

def timing_decorator(func):
    """Decorator para medir tiempo de ejecución"""
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"{func.__name__} took {end - start:.4f} seconds")
        return result
    return wrapper

def memoize(func):
    """Decorator para cachear resultados"""
    cache = {}
    @wraps(func)
    def wrapper(*args):
        if args not in cache:
            cache[args] = func(*args)
        return cache[args]
    return wrapper

# Uso
@timing_decorator
@memoize
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

result = fibonacci(30)
```

### Ejemplo 3: Context Managers

```python
from contextlib import contextmanager
import time

class Timer:
    """Context manager para timing"""
    def __enter__(self):
        self.start = time.time()
        return self
    
    def __exit__(self, *args):
        self.end = time.time()
        self.elapsed = self.end - self.start
        print(f"Elapsed time: {self.elapsed:.4f} seconds")

# Uso
with Timer():
    # Código a cronometrar
    time.sleep(1)

# Con decorator @contextmanager
@contextmanager
def temporary_directory():
    """Context manager para directorio temporal"""
    import tempfile
    import shutil
    
    dirpath = tempfile.mkdtemp()
    try:
        yield dirpath
    finally:
        shutil.rmtree(dirpath)

# Uso
with temporary_directory() as tmpdir:
    print(f"Working in {tmpdir}")
    # Hacer operaciones en directorio temporal
```

### Ejemplo 4: Classes y OOP

```python
from dataclasses import dataclass
from typing import List, Optional
from abc import ABC, abstractmethod

# Dataclass (Python 3.7+)
@dataclass
class Point:
    x: float
    y: float
    
    def distance_to(self, other: 'Point') -> float:
        return ((self.x - other.x)**2 + (self.y - other.y)**2)**0.5

# Abstract base class
class Model(ABC):
    @abstractmethod
    def train(self, X, y):
        pass
    
    @abstractmethod
    def predict(self, X):
        pass

class LinearModel(Model):
    def __init__(self):
        self.weights = None
        
    def train(self, X, y):
        # Implementación
        import numpy as np
        self.weights = np.linalg.lstsq(X, y, rcond=None)[0]
    
    def predict(self, X):
        if self.weights is None:
            raise ValueError("Model not trained")
        return X @ self.weights

# Property decorator
class Circle:
    def __init__(self, radius):
        self._radius = radius
    
    @property
    def radius(self):
        return self._radius
    
    @radius.setter
    def radius(self, value):
        if value < 0:
            raise ValueError("Radius cannot be negative")
        self._radius = value
    
    @property
    def area(self):
        import math
        return math.pi * self._radius ** 2
```

### Ejemplo 5: Generators e Iterators

```python
# Generator function
def fibonacci_generator(n):
    """Generate Fibonacci sequence up to n terms"""
    a, b = 0, 1
    for _ in range(n):
        yield a
        a, b = b, a + b

# Uso
for num in fibonacci_generator(10):
    print(num, end=' ')

# Generator expression
squares_gen = (x**2 for x in range(1000000))  # No ocupa memoria hasta que se itera

# Custom iterator
class Countdown:
    def __init__(self, start):
        self.current = start
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.current <= 0:
            raise StopIteration
        self.current -= 1
        return self.current + 1

# Uso
for i in Countdown(5):
    print(i)

# itertools para operaciones eficientes
from itertools import islice, cycle, chain, combinations

# Tomar primeros 10 números de secuencia infinita
from itertools import count
first_10 = list(islice(count(), 10))

# Combinar iterables
combined = list(chain([1, 2, 3], [4, 5, 6]))

# Combinaciones
combos = list(combinations([1, 2, 3, 4], 2))
```

### Ejemplo 6: Async Programming

```python
import asyncio
import aiohttp
import time

async def fetch_url(session, url):
    """Fetch URL asynchronously"""
    async with session.get(url) as response:
        return await response.text()

async def fetch_multiple_urls(urls):
    """Fetch multiple URLs concurrently"""
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_url(session, url) for url in urls]
        results = await asyncio.gather(*tasks)
        return results

# Comparar sync vs async
def sync_sleep():
    """Synchronous sleep"""
    start = time.time()
    for _ in range(5):
        time.sleep(1)
    print(f"Sync took: {time.time() - start:.2f}s")

async def async_sleep():
    """Asynchronous sleep"""
    start = time.time()
    await asyncio.gather(*[asyncio.sleep(1) for _ in range(5)])
    print(f"Async took: {time.time() - start:.2f}s")

# Ejecutar
# sync_sleep()  # ~5 seconds
# asyncio.run(async_sleep())  # ~1 second
```

### Ejemplo 7: Type Hints

```python
from typing import List, Dict, Tuple, Optional, Union, Callable, TypeVar, Generic

# Type hints básicos
def greet(name: str) -> str:
    return f"Hello, {name}"

# Collections
def process_numbers(numbers: List[int]) -> Dict[str, float]:
    return {
        'mean': sum(numbers) / len(numbers),
        'max': max(numbers),
        'min': min(numbers)
    }

# Optional (puede ser None)
def find_user(user_id: int) -> Optional[Dict[str, str]]:
    # Retorna dict o None
    pass

# Union (puede ser uno de varios tipos)
def process_input(data: Union[str, int, float]) -> str:
    return str(data)

# Callable
def apply_operation(numbers: List[int], 
                    operation: Callable[[int], int]) -> List[int]:
    return [operation(n) for n in numbers]

# Generic types
T = TypeVar('T')

class Stack(Generic[T]):
    def __init__(self):
        self.items: List[T] = []
    
    def push(self, item: T) -> None:
        self.items.append(item)
    
    def pop(self) -> T:
        return self.items.pop()

# Uso
int_stack: Stack[int] = Stack()
int_stack.push(1)
```

## Patrones de Diseño Comunes en ML

### Singleton Pattern

```python
class DatasetLoader:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance.data = None
        return cls._instance
    
    def load_data(self, path):
        if self.data is None:
            # Cargar datos una sola vez
            self.data = self._load_from_disk(path)
        return self.data
```

### Factory Pattern

```python
class ModelFactory:
    @staticmethod
    def create_model(model_type: str, **kwargs):
        if model_type == 'linear':
            from sklearn.linear_model import LinearRegression
            return LinearRegression(**kwargs)
        elif model_type == 'tree':
            from sklearn.tree import DecisionTreeRegressor
            return DecisionTreeRegressor(**kwargs)
        elif model_type == 'random_forest':
            from sklearn.ensemble import RandomForestRegressor
            return RandomForestRegressor(**kwargs)
        else:
            raise ValueError(f"Unknown model type: {model_type}")

# Uso
model = ModelFactory.create_model('random_forest', n_estimators=100)
```

### Builder Pattern

```python
class ModelPipeline:
    def __init__(self):
        self.steps = []
    
    def add_preprocessing(self, preprocessor):
        self.steps.append(('preprocess', preprocessor))
        return self
    
    def add_feature_engineering(self, transformer):
        self.steps.append(('features', transformer))
        return self
    
    def add_model(self, model):
        self.steps.append(('model', model))
        return self
    
    def build(self):
        from sklearn.pipeline import Pipeline
        return Pipeline(self.steps)

# Uso
pipeline = (ModelPipeline()
    .add_preprocessing(StandardScaler())
    .add_feature_engineering(PCA(n_components=10))
    .add_model(RandomForestClassifier())
    .build())
```

## Mejores Prácticas

### 1. PEP 8 - Style Guide

```python
# Nombrar variables y funciones: snake_case
user_name = "John"
def calculate_mean(numbers):
    pass

# Nombrar clases: PascalCase
class DataProcessor:
    pass

# Constantes: UPPERCASE
MAX_ITERATIONS = 1000
PI = 3.14159

# Imports
import os
import sys
from typing import List

import numpy as np
import pandas as pd

from myproject import mymodule

# Espaciado
x = 1  # Espacio alrededor de operadores
y = x * 2 + 1

# Longitud de línea: máximo 79 caracteres
# Usar \ para continuar línea larga
```

### 2. Docstrings

```python
def train_model(X, y, model_type='linear', **kwargs):
    """
    Train a machine learning model.
    
    Parameters
    ----------
    X : array-like of shape (n_samples, n_features)
        Training data
    y : array-like of shape (n_samples,)
        Target values
    model_type : str, default='linear'
        Type of model to train
    **kwargs : dict
        Additional parameters for the model
    
    Returns
    -------
    model : object
        Trained model
    
    Examples
    --------
    >>> X, y = load_data()
    >>> model = train_model(X, y, model_type='random_forest')
    """
    pass
```

### 3. Error Handling

```python
class DataValidationError(Exception):
    """Custom exception for data validation"""
    pass

def validate_data(data):
    """Validate input data"""
    if data is None:
        raise ValueError("Data cannot be None")
    
    if len(data) == 0:
        raise DataValidationError("Data is empty")
    
    try:
        # Intentar convertir a numpy array
        import numpy as np
        data = np.array(data)
    except Exception as e:
        raise DataValidationError(f"Cannot convert data: {e}")
    
    return data

# Uso con try-except-finally
try:
    data = validate_data(input_data)
    result = process_data(data)
except DataValidationError as e:
    print(f"Validation error: {e}")
    result = None
except Exception as e:
    print(f"Unexpected error: {e}")
    result = None
finally:
    # Siempre se ejecuta
    print("Cleaning up...")
```

### 4. Logging

```python
import logging

# Configurar logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('app.log'),
        logging.StreamHandler()
    ]
)

logger = logging.getLogger(__name__)

def train_model(X, y):
    logger.info("Starting model training")
    try:
        # Entrenar modelo
        logger.debug(f"Training with {len(X)} samples")
        # ...
        logger.info("Model training completed successfully")
    except Exception as e:
        logger.error(f"Error during training: {e}", exc_info=True)
        raise
```

## Proyectos Sugeridos

1. **Data Processing Pipeline**
   - Crear pipeline robusto para procesamiento
   - Usar OOP y design patterns
   - Implementar logging y error handling

2. **Custom ML Library**
   - Implementar algoritmos desde cero
   - Usar NumPy puro
   - API similar a scikit-learn

3. **CLI Tool para ML**
   - Usar `argparse` o `click`
   - Entrenar modelos desde terminal
   - Guardar/cargar modelos

4. **Async Data Fetcher**
   - Descargar datos de múltiples fuentes
   - Usar `asyncio` y `aiohttp`
   - Comparar performance vs sync

5. **Testing Suite**
   - Crear tests comprehensivos con pytest
   - Unit tests, integration tests
   - Configurar CI/CD

## Herramientas de Productividad

### Code Formatting

```bash
# Black - The uncompromising code formatter
pip install black
black myfile.py

# isort - Sort imports
pip install isort
isort myfile.py

# autopep8
pip install autopep8
autopep8 --in-place myfile.py
```

### Linting

```bash
# pylint
pip install pylint
pylint myfile.py

# flake8
pip install flake8
flake8 myfile.py

# mypy (type checking)
pip install mypy
mypy myfile.py
```

### Profiling

```python
import cProfile
import pstats

def my_function():
    # Código a perfilar
    pass

# Profile
cProfile.run('my_function()', 'output.prof')

# Analizar resultados
with open('output.prof', 'r') as f:
    stats = pstats.Stats('output.prof')
    stats.sort_stats('cumulative')
    stats.print_stats(10)

# Line profiler
# pip install line_profiler
# kernprof -l -v myfile.py
```

## Recursos Adicionales

### Bibliotecas Útiles

1. **rich** - Terminal output hermoso
2. **click** - CLI frameworks
3. **pydantic** - Data validation
4. **loguru** - Logging mejorado
5. **joblib** - Paralelización fácil

### Comunidades

1. **r/Python**
   - https://www.reddit.com/r/Python/

2. **Python Discord**
   - https://pythondiscord.com/

3. **Stack Overflow**
   - https://stackoverflow.com/questions/tagged/python

### Blogs

1. **Real Python Blog**
   - https://realpython.com/

2. **Python Software Foundation Blog**
   - https://pyfound.blogspot.com/

## Checklist de Aprendizaje

- [ ] Dominar comprehensions y generators
- [ ] Entender decorators y context managers
- [ ] Aplicar OOP correctamente
- [ ] Usar type hints
- [ ] Escribir tests unitarios
- [ ] Manejar errores apropiadamente
- [ ] Seguir PEP 8
- [ ] Implementar logging
- [ ] Usar async cuando sea apropiado
- [ ] Optimizar código (profiling)

## Próximos Pasos

Después de dominar Python:
1. Profundizar en **NumPy** para computación numérica
2. Aprender **Pandas** para análisis de datos
3. Dominar **Git** para control de versiones
4. Familiarizarse con **Linux/Unix** commands

---
**Tiempo estimado de estudio**: 4-6 semanas para fundamentos, continuo para dominio
