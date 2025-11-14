# Cálculo para AI y Machine Learning

## Descripción

El cálculo es esencial para entender cómo los algoritmos de ML "aprenden". El concepto de derivadas permite optimizar funciones de pérdida mediante gradient descent, que es la base del entrenamiento de redes neuronales y la mayoría de algoritmos de ML.

## Conceptos Clave

### 1. **Derivadas**
- Tasa de cambio instantánea
- Reglas de derivación
- Derivadas parciales
- Interpretación geométrica: pendiente de la tangente

### 2. **Gradientes**
- Vector de derivadas parciales
- Dirección de máximo crecimiento
- Aplicación en optimización

### 3. **Chain Rule (Regla de la Cadena)**
- Composición de funciones
- Base de backpropagation
- Diferenciación automática

### 4. **Optimización**
- Puntos críticos: máximos y mínimos
- Derivada segunda: concavidad
- Matriz Hessiana

### 5. **Integrales** (Menos crítico pero útil)
- Área bajo la curva
- Probabilidades acumuladas
- Esperanza matemática

## Aplicaciones en AI/ML

- **Gradient Descent**: Minimización de función de pérdida
- **Backpropagation**: Calcular gradientes en redes neuronales
- **Optimización de hiperparámetros**: Encontrar configuración óptima
- **Learning Rate**: Control de la velocidad de convergencia
- **Regularización**: Derivadas para penalización L1/L2

## Recursos de Aprendizaje

### Cursos Online

1. **3Blue1Brown - Essence of Calculus** (Gratis)
   - https://www.youtube.com/playlist?list=PLZHQObOWTQDMsr9K-rj53DwVRMYO3t5Yr
   - La mejor introducción visual al cálculo

2. **Khan Academy - Calculus** (Gratis)
   - https://www.khanacademy.org/math/calculus-1
   - https://www.khanacademy.org/math/multivariable-calculus
   - Completo y bien estructurado

3. **MIT OpenCourseWare - Single Variable Calculus**
   - https://ocw.mit.edu/courses/18-01-single-variable-calculus-fall-2006/
   - Curso universitario completo

4. **Coursera - Mathematics for Machine Learning: Multivariate Calculus**
   - https://www.coursera.org/learn/multivariate-calculus-machine-learning
   - Imperial College London

### Libros

1. **"Calculus"** - James Stewart
   - Libro universitario estándar
   - Muy completo con muchos ejercicios

2. **"Thomas' Calculus"** - George B. Thomas
   - Otro clásico universitario

3. **"The Calculus Lifesaver"** - Adrian Banner
   - Más accesible, perfecto para self-study

4. **"Calculus Made Easy"** - Silvanus P. Thompson
   - Clásico de 1910, sorprendentemente relevante
   - Gratis: https://calculusmadeeasy.org/

### Documentación y Referencias

1. **Wolfram MathWorld - Calculus**
   - https://mathworld.wolfram.com/Calculus.html
   - Referencia completa

2. **Paul's Online Math Notes**
   - https://tutorial.math.lamar.edu/Classes/CalcI/CalcI.aspx
   - Excelente recurso gratuito

3. **Better Explained - Calculus**
   - https://betterexplained.com/guides/calculus/
   - Explicaciones intuitivas

### Videos Recomendados

1. **3Blue1Brown - Essence of Calculus Series**
   - https://www.youtube.com/watch?v=WUvTyaaNkzM (Derivative paradox)
   - https://www.youtube.com/watch?v=9vKqVkMQHKk (Chain rule)
   - https://www.youtube.com/watch?v=rfG8ce4nNh0 (Implicit differentiation)

2. **StatQuest - Gradient Descent**
   - https://www.youtube.com/watch?v=sDv4f4s2SB8
   - Explicación clara del concepto

3. **MIT OpenCourseWare - Highlights of Calculus**
   - https://www.youtube.com/playlist?list=PLBE9407EA64E2C318
   - Gilbert Strang

## Herramientas y Bibliotecas

### Python - Cálculo Simbólico

```python
import sympy as sp
import numpy as np
from scipy import optimize
import matplotlib.pyplot as plt

# Definir variable simbólica
x = sp.Symbol('x')

# Definir función
f = x**3 - 6*x**2 + 9*x + 1

# Calcular derivada
f_prime = sp.diff(f, x)
print(f"f'(x) = {f_prime}")

# Segunda derivada
f_double_prime = sp.diff(f, x, 2)
print(f"f''(x) = {f_double_prime}")

# Encontrar puntos críticos
critical_points = sp.solve(f_prime, x)
print(f"Puntos críticos: {critical_points}")

# Evaluar en un punto
value = f.subs(x, 2)
print(f"f(2) = {value}")
```

### Python - Diferenciación Numérica

```python
import numpy as np
from scipy.misc import derivative

# Función de ejemplo
def f(x):
    return x**3 - 6*x**2 + 9*x + 1

# Derivada numérica
x = 2
df_dx = derivative(f, x, dx=1e-6)
print(f"f'(2) ≈ {df_dx}")

# Gradiente numérico para funciones multivariables
def g(params):
    x, y = params
    return x**2 + y**2

def numerical_gradient(f, params, eps=1e-5):
    grad = np.zeros_like(params)
    for i in range(len(params)):
        params_plus = params.copy()
        params_minus = params.copy()
        params_plus[i] += eps
        params_minus[i] -= eps
        grad[i] = (f(params_plus) - f(params_minus)) / (2 * eps)
    return grad

params = np.array([3.0, 4.0])
grad = numerical_gradient(g, params)
print(f"∇g = {grad}")
```

### Diferenciación Automática

```python
import torch

# PyTorch - Automatic Differentiation
x = torch.tensor([2.0], requires_grad=True)
y = x**3 - 6*x**2 + 9*x + 1

# Calcular gradiente
y.backward()
print(f"dy/dx en x=2: {x.grad.item()}")

# Múltiples variables
x = torch.tensor([1.0, 2.0], requires_grad=True)
y = (x**2).sum()
y.backward()
print(f"Gradiente: {x.grad}")
```

### JAX - High-Performance Autodiff

```python
import jax
import jax.numpy as jnp

def f(x):
    return x**3 - 6*x**2 + 9*x + 1

# Derivada automática
f_prime = jax.grad(f)
print(f"f'(2) = {f_prime(2.0)}")

# Gradiente de función multivariable
def g(params):
    x, y = params
    return x**2 + y**2

grad_g = jax.grad(g)
print(f"∇g = {grad_g(jnp.array([3.0, 4.0]))}")

# Hessiana
hessian_g = jax.hessian(g)
print(f"Hessian:\n{hessian_g(jnp.array([3.0, 4.0]))}")
```

## Ejemplos Prácticos

### Ejemplo 1: Gradient Descent desde Cero

```python
import numpy as np
import matplotlib.pyplot as plt

def f(x):
    """Función a minimizar"""
    return x**2 + 5*np.sin(x)

def df_dx(x):
    """Derivada de f"""
    return 2*x + 5*np.cos(x)

def gradient_descent(start, learning_rate=0.1, iterations=50):
    """Gradient descent simple"""
    x = start
    history = [x]
    
    for _ in range(iterations):
        gradient = df_dx(x)
        x = x - learning_rate * gradient
        history.append(x)
    
    return x, history

# Ejecutar gradient descent
final_x, history = gradient_descent(start=5.0, learning_rate=0.1)

# Visualizar
x_vals = np.linspace(-5, 5, 200)
plt.figure(figsize=(10, 6))
plt.plot(x_vals, f(x_vals), 'b-', label='f(x)')
plt.plot(history, [f(x) for x in history], 'ro-', label='GD path')
plt.xlabel('x')
plt.ylabel('f(x)')
plt.legend()
plt.title('Gradient Descent Optimization')
plt.grid(True)
plt.show()

print(f"Mínimo encontrado en x = {final_x:.4f}")
```

### Ejemplo 2: Backpropagation Simple

```python
import numpy as np

class SimpleNeuralNetwork:
    def __init__(self):
        # Inicializar pesos aleatoriamente
        np.random.seed(42)
        self.W1 = np.random.randn(2, 3)  # 2 inputs, 3 hidden
        self.W2 = np.random.randn(3, 1)  # 3 hidden, 1 output
        
    def sigmoid(self, x):
        return 1 / (1 + np.exp(-x))
    
    def sigmoid_derivative(self, x):
        return x * (1 - x)
    
    def forward(self, X):
        """Forward pass"""
        self.z1 = X @ self.W1
        self.a1 = self.sigmoid(self.z1)
        self.z2 = self.a1 @ self.W2
        self.a2 = self.sigmoid(self.z2)
        return self.a2
    
    def backward(self, X, y, output, learning_rate=0.1):
        """Backward pass usando chain rule"""
        m = X.shape[0]
        
        # Calcular gradientes usando chain rule
        dZ2 = output - y
        dW2 = (1/m) * self.a1.T @ dZ2
        
        dZ1 = (dZ2 @ self.W2.T) * self.sigmoid_derivative(self.a1)
        dW1 = (1/m) * X.T @ dZ1
        
        # Actualizar pesos
        self.W1 -= learning_rate * dW1
        self.W2 -= learning_rate * dW2
    
    def train(self, X, y, iterations=1000):
        """Entrenar la red"""
        for i in range(iterations):
            output = self.forward(X)
            self.backward(X, y, output)
            
            if i % 100 == 0:
                loss = np.mean((y - output)**2)
                print(f"Iteration {i}, Loss: {loss:.4f}")

# Datos de ejemplo (XOR problem)
X = np.array([[0, 0], [0, 1], [1, 0], [1, 1]])
y = np.array([[0], [1], [1], [0]])

# Entrenar
nn = SimpleNeuralNetwork()
nn.train(X, y, iterations=5000)

# Predicciones
predictions = nn.forward(X)
print("\nPredicciones:")
for i in range(len(X)):
    print(f"Input: {X[i]}, Predicted: {predictions[i][0]:.4f}, Actual: {y[i][0]}")
```

### Ejemplo 3: Newton's Method

```python
import numpy as np

def newtons_method(f, df, d2f, x0, iterations=10):
    """
    Newton's method para encontrar raíces
    f: función
    df: primera derivada
    d2f: segunda derivada
    """
    x = x0
    history = [x]
    
    for i in range(iterations):
        x_new = x - df(x) / d2f(x)
        history.append(x_new)
        
        if abs(x_new - x) < 1e-10:
            break
            
        x = x_new
    
    return x, history

# Ejemplo: minimizar f(x) = x^4 - 3x^3 + 2
def f(x):
    return x**4 - 3*x**3 + 2

def df(x):
    return 4*x**3 - 9*x**2

def d2f(x):
    return 12*x**2 - 18*x

# Encontrar mínimo
minimum, history = newtons_method(f, df, d2f, x0=3.0)
print(f"Mínimo en x = {minimum:.6f}")
print(f"Valor mínimo f(x) = {f(minimum):.6f}")
```

## Proyectos Sugeridos

1. **Implementar Gradient Descent para Regresión Lineal**
   - Calcular gradientes manualmente
   - Comparar con solución analítica
   - Visualizar la convergencia

2. **Red Neuronal desde Cero**
   - Implementar forward y backward pass
   - Usar solo NumPy
   - Entrenar en dataset simple (XOR, MNIST)

3. **Optimizadores Avanzados**
   - Implementar SGD con momentum
   - Adam optimizer
   - Comparar convergencia

4. **Automatic Differentiation Framework Básico**
   - Crear sistema de computational graph
   - Implementar backward pass automático

5. **Análisis de Funciones de Pérdida**
   - Visualizar landscape de diferentes loss functions
   - Estudiar puntos críticos y saddle points

## Conceptos Importantes para ML

### Gradient Descent

```python
# Batch Gradient Descent
def batch_gradient_descent(X, y, learning_rate=0.01, iterations=1000):
    m, n = X.shape
    theta = np.zeros(n)
    
    for _ in range(iterations):
        predictions = X @ theta
        errors = predictions - y
        gradient = (1/m) * X.T @ errors
        theta = theta - learning_rate * gradient
    
    return theta

# Stochastic Gradient Descent
def sgd(X, y, learning_rate=0.01, iterations=1000):
    m, n = X.shape
    theta = np.zeros(n)
    
    for _ in range(iterations):
        for i in range(m):
            random_index = np.random.randint(m)
            xi = X[random_index:random_index+1]
            yi = y[random_index:random_index+1]
            
            prediction = xi @ theta
            error = prediction - yi
            gradient = xi.T @ error
            theta = theta - learning_rate * gradient
    
    return theta

# Mini-batch Gradient Descent
def mini_batch_gd(X, y, batch_size=32, learning_rate=0.01, iterations=1000):
    m, n = X.shape
    theta = np.zeros(n)
    
    for _ in range(iterations):
        indices = np.random.permutation(m)
        X_shuffled = X[indices]
        y_shuffled = y[indices]
        
        for i in range(0, m, batch_size):
            xi = X_shuffled[i:i+batch_size]
            yi = y_shuffled[i:i+batch_size]
            
            predictions = xi @ theta
            errors = predictions - yi
            gradient = (1/len(xi)) * xi.T @ errors
            theta = theta - learning_rate * gradient
    
    return theta
```

### Chain Rule en Profundidad

```python
# Ejemplo de chain rule con múltiples capas
def chain_rule_example():
    """
    Si y = f(g(h(x))), entonces:
    dy/dx = df/dg * dg/dh * dh/dx
    """
    
    # Definir funciones
    def h(x):
        return x**2
    
    def g(h_val):
        return np.sin(h_val)
    
    def f(g_val):
        return np.exp(g_val)
    
    # Derivadas
    def dh_dx(x):
        return 2*x
    
    def dg_dh(h_val):
        return np.cos(h_val)
    
    def df_dg(g_val):
        return np.exp(g_val)
    
    # Evaluar en x = 2
    x = 2
    h_val = h(x)
    g_val = g(h_val)
    f_val = f(g_val)
    
    # Chain rule
    dy_dx = df_dg(g_val) * dg_dh(h_val) * dh_dx(x)
    
    print(f"x = {x}")
    print(f"y = f(g(h(x))) = {f_val}")
    print(f"dy/dx = {dy_dx}")
    
    return dy_dx

chain_rule_example()
```

## Mejores Prácticas

1. **Verifica tus gradientes**
   - Usa gradient checking con diferencias finitas
   - Compara autodiff con cálculo manual

2. **Learning Rate**
   - Empieza con valores pequeños (0.001 - 0.1)
   - Usa learning rate decay
   - Considera adaptive learning rates (Adam)

3. **Visualización**
   - Grafica la función de pérdida
   - Visualiza el landscape de optimización
   - Monitorea la convergencia

4. **Estabilidad Numérica**
   - Cuidado con overflow/underflow
   - Usa log-sum-exp trick cuando sea necesario
   - Normaliza inputs

5. **Debugging**
   - Implementa gradient checking
   - Verifica dimensiones constantemente
   - Empieza con ejemplos simples

## Recursos Adicionales

### Interactive Tools

1. **Desmos Graphing Calculator**
   - https://www.desmos.com/calculator
   - Visualiza derivadas interactivamente

2. **GeoGebra**
   - https://www.geogebra.org/calculator
   - Excelente para visualización 3D

3. **Wolfram Alpha**
   - https://www.wolframalpha.com/
   - Calcula derivadas, límites, integrales

### Papers y Artículos

1. **"Automatic Differentiation in Machine Learning: a Survey"**
   - https://arxiv.org/abs/1502.05767

2. **"An overview of gradient descent optimization algorithms"**
   - https://arxiv.org/abs/1609.04747

3. **"Adam: A Method for Stochastic Optimization"**
   - https://arxiv.org/abs/1412.6980

### Blogs

1. **Towards Data Science - Calculus**
   - https://towardsdatascience.com/tagged/calculus

2. **Distill.pub - Differentiation**
   - https://distill.pub/

3. **Christopher Olah's Blog**
   - https://colah.github.io/posts/2015-08-Backprop/

## Fórmulas Importantes

### Derivadas Comunes

```
d/dx(x^n) = nx^(n-1)
d/dx(e^x) = e^x
d/dx(ln x) = 1/x
d/dx(sin x) = cos x
d/dx(cos x) = -sin x
d/dx(1/x) = -1/x^2
```

### Reglas de Derivación

```
(f + g)' = f' + g'
(f · g)' = f'g + fg'  (Product rule)
(f/g)' = (f'g - fg')/g²  (Quotient rule)
(f(g(x)))' = f'(g(x)) · g'(x)  (Chain rule)
```

### Funciones de Activación y sus Derivadas

```python
# Sigmoid
def sigmoid(x):
    return 1 / (1 + np.exp(-x))

def sigmoid_derivative(x):
    s = sigmoid(x)
    return s * (1 - s)

# Tanh
def tanh_derivative(x):
    return 1 - np.tanh(x)**2

# ReLU
def relu(x):
    return np.maximum(0, x)

def relu_derivative(x):
    return (x > 0).astype(float)

# Leaky ReLU
def leaky_relu(x, alpha=0.01):
    return np.where(x > 0, x, alpha * x)

def leaky_relu_derivative(x, alpha=0.01):
    return np.where(x > 0, 1, alpha)
```

## Próximos Pasos

Después de dominar Cálculo:
1. Combinar con **Álgebra Lineal** para gradientes matriciales
2. Estudiar **Optimización** avanzada
3. Profundizar en **Estadística y Probabilidad**

## Checklist de Aprendizaje

- [ ] Entender concepto de derivada
- [ ] Dominar reglas de derivación
- [ ] Calcular derivadas parciales
- [ ] Comprender chain rule
- [ ] Implementar gradient descent
- [ ] Implementar backpropagation
- [ ] Usar diferenciación automática (PyTorch/JAX)
- [ ] Aplicar en proyecto de ML real

---
**Tiempo estimado de estudio**: 3-4 semanas dedicando 2-3 horas diarias
