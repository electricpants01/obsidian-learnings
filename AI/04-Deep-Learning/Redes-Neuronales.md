# Redes Neuronales - Fundamentos

## Descripción

Las redes neuronales artificiales son el fundamento del deep learning moderno, inspiradas en el funcionamiento del cerebro humano pero con arquitecturas matemáticas específicas. Una neurona artificial toma inputs ponderados, suma un bias, y aplica una función de activación no lineal. Las capas conectadas forman redes que aprenden representaciones jerárquicas: primeras capas detectan features simples (bordes, colores), capas profundas conceptos complejos (caras, objetos). El algoritmo de backpropagation, junto con gradient descent, ajusta los pesos mediante la cadena de derivadas. Las arquitecturas evolucionaron: perceptrones simples (1950s), MLPs (1980s), CNNs para visión (1990s), RNNs/LSTMs para secuencias (2000s), Transformers para todo (2017+). Conceptos clave: activaciones (ReLU, sigmoid, tanh), inicialización de pesos (Xavier, He), regularización (dropout, batch normalization, L1/L2), y optimización (SGD, Adam, learning rate scheduling). El Universal Approximation Theorem garantiza que redes con suficientes neuronas pueden aproximar cualquier función continua. Dominar estos fundamentos es esencial antes de arquitecturas avanzadas.

## Conceptos Clave

### 1. **Neurona Artificial**
- Suma ponderada de inputs: z = Σ(wi·xi) + b
- Función de activación: a = f(z)
- Forward propagation
- Pesos y bias entrenables

### 2. **Funciones de Activación**
- ReLU: f(x) = max(0, x) - más usada
- Sigmoid: f(x) = 1/(1+e^-x) - output [0,1]
- Tanh: f(x) = (e^x - e^-x)/(e^x + e^-x) - output [-1,1]
- Leaky ReLU, ELU, Swish - variantes

### 3. **Arquitectura de Redes**
- Input layer, hidden layers, output layer
- Fully connected (dense) layers
- Profundidad vs anchura
- Skip connections, residual blocks

### 4. **Backpropagation**
- Cadena de derivadas (chain rule)
- Gradiente del error respecto a pesos
- Backward pass capa por capa
- Computational graph

### 5. **Inicialización de Pesos**
- Random pequeño (evitar simetría)
- Xavier/Glorot initialization
- He initialization (para ReLU)
- Transfer learning weights

### 6. **Regularización**
- Dropout: apagar neuronas aleatoriamente
- Batch Normalization: normalizar activaciones
- L1/L2 regularization en loss
- Early stopping
- Data augmentation

### 7. **Optimización**
- Gradient Descent y variantes (SGD, Mini-batch)
- Momentum, RMSprop, Adam
- Learning rate scheduling
- Gradient clipping
- Batch size effects

## Recursos de Aprendizaje

### Cursos

1. **Neural Networks and Deep Learning - Coursera**
   - https://www.coursera.org/learn/neural-networks-deep-learning
   - Por Andrew Ng, fundamentos sólidos

2. **Deep Learning Specialization - Coursera**
   - https://www.coursera.org/specializations/deep-learning
   - 5 cursos completos

3. **CS231n: CNN for Visual Recognition - Stanford**
   - https://cs231n.github.io/
   - Excelentes notas, muy técnico

### Libros

1. **"Deep Learning"** - Goodfellow, Bengio, Courville
   - Gratis: https://www.deeplearningbook.org/
   - La biblia del deep learning

2. **"Neural Networks and Deep Learning"** - Michael Nielsen
   - Gratis: http://neuralnetworksanddeeplearning.com/
   - Muy didáctico, con visualizaciones

### Videos

1. **3Blue1Brown - Neural Networks**
   - https://www.youtube.com/watch?v=aircAruvnKk
   - Visualizaciones excepcionales

2. **StatQuest - Neural Networks**
   - https://www.youtube.com/c/joshstarmer

## Ejemplos Prácticos

### Ejemplo 1: Neurona Simple

```python
import numpy as np

class Neuron:
    def __init__(self, n_inputs):
        # Inicialización Xavier
        self.weights = np.random.randn(n_inputs) * np.sqrt(2.0 / n_inputs)
        self.bias = 0.0
    
    def forward(self, inputs):
        # z = w·x + b
        z = np.dot(self.weights, inputs) + self.bias
        # Activación ReLU
        self.output = max(0, z)
        return self.output
    
    def sigmoid(self, z):
        return 1 / (1 + np.exp(-z))
    
    def tanh(self, z):
        return np.tanh(z)

# Usar neurona
neuron = Neuron(n_inputs=3)
inputs = np.array([1.0, 2.0, 3.0])
output = neuron.forward(inputs)
print(f"Output: {output}")
```

### Ejemplo 2: MLP desde Cero

```python
import numpy as np

class NeuralNetwork:
    def __init__(self, layers):
        self.layers = layers
        self.weights = []
        self.biases = []
        
        # Inicializar pesos y biases
        for i in range(len(layers) - 1):
            w = np.random.randn(layers[i], layers[i+1]) * np.sqrt(2.0 / layers[i])
            b = np.zeros((1, layers[i+1]))
            self.weights.append(w)
            self.biases.append(b)
    
    def relu(self, Z):
        return np.maximum(0, Z)
    
    def relu_derivative(self, Z):
        return (Z > 0).astype(float)
    
    def forward(self, X):
        self.activations = [X]
        self.z_values = []
        
        A = X
        for i in range(len(self.weights)):
            Z = np.dot(A, self.weights[i]) + self.biases[i]
            self.z_values.append(Z)
            
            if i < len(self.weights) - 1:
                A = self.relu(Z)  # Hidden layers
            else:
                A = Z  # Output layer (linear)
            
            self.activations.append(A)
        
        return A
    
    def backward(self, X, y, learning_rate=0.01):
        m = X.shape[0]
        
        # Output layer gradient
        dA = self.activations[-1] - y
        
        # Backpropagate
        for i in reversed(range(len(self.weights))):
            dZ = dA
            if i < len(self.weights) - 1:
                dZ = dA * self.relu_derivative(self.z_values[i])
            
            dW = (1/m) * np.dot(self.activations[i].T, dZ)
            db = (1/m) * np.sum(dZ, axis=0, keepdims=True)
            
            if i > 0:
                dA = np.dot(dZ, self.weights[i].T)
            
            # Update weights
            self.weights[i] -= learning_rate * dW
            self.biases[i] -= learning_rate * db
    
    def train(self, X, y, epochs=1000, learning_rate=0.01):
        for epoch in range(epochs):
            # Forward pass
            predictions = self.forward(X)
            
            # Compute loss (MSE)
            loss = np.mean((predictions - y) ** 2)
            
            # Backward pass
            self.backward(X, y, learning_rate)
            
            if epoch % 100 == 0:
                print(f'Epoch {epoch}, Loss: {loss:.4f}')

# Ejemplo de uso
X = np.random.randn(100, 2)
y = (X[:, 0] + X[:, 1] > 0).reshape(-1, 1).astype(float)

nn = NeuralNetwork([2, 4, 4, 1])
nn.train(X, y, epochs=1000, learning_rate=0.1)
```

### Ejemplo 3: Funciones de Activación

```python
import numpy as np
import matplotlib.pyplot as plt

def relu(x):
    return np.maximum(0, x)

def sigmoid(x):
    return 1 / (1 + np.exp(-np.clip(x, -500, 500)))

def tanh(x):
    return np.tanh(x)

def leaky_relu(x, alpha=0.01):
    return np.where(x > 0, x, alpha * x)

def elu(x, alpha=1.0):
    return np.where(x > 0, x, alpha * (np.exp(x) - 1))

# Visualizar
x = np.linspace(-5, 5, 100)

plt.figure(figsize=(15, 10))

activations = [
    ('ReLU', relu),
    ('Sigmoid', sigmoid),
    ('Tanh', tanh),
    ('Leaky ReLU', leaky_relu),
    ('ELU', elu)
]

for idx, (name, func) in enumerate(activations, 1):
    plt.subplot(2, 3, idx)
    plt.plot(x, func(x))
    plt.title(name)
    plt.grid(True)
    plt.axhline(y=0, color='k', linewidth=0.5)
    plt.axvline(x=0, color='k', linewidth=0.5)

plt.tight_layout()
plt.show()
```

### Ejemplo 4: Batch Normalization

```python
class BatchNormalization:
    def __init__(self, num_features, epsilon=1e-5, momentum=0.9):
        self.epsilon = epsilon
        self.momentum = momentum
        
        # Parámetros entrenables
        self.gamma = np.ones((1, num_features))
        self.beta = np.zeros((1, num_features))
        
        # Running statistics
        self.running_mean = np.zeros((1, num_features))
        self.running_var = np.ones((1, num_features))
    
    def forward(self, X, training=True):
        if training:
            # Compute batch statistics
            batch_mean = np.mean(X, axis=0, keepdims=True)
            batch_var = np.var(X, axis=0, keepdims=True)
            
            # Normalize
            X_normalized = (X - batch_mean) / np.sqrt(batch_var + self.epsilon)
            
            # Scale and shift
            out = self.gamma * X_normalized + self.beta
            
            # Update running statistics
            self.running_mean = self.momentum * self.running_mean + (1 - self.momentum) * batch_mean
            self.running_var = self.momentum * self.running_var + (1 - self.momentum) * batch_var
            
            # Cache for backward pass
            self.cache = (X, X_normalized, batch_mean, batch_var)
        else:
            # Use running statistics
            X_normalized = (X - self.running_mean) / np.sqrt(self.running_var + self.epsilon)
            out = self.gamma * X_normalized + self.beta
        
        return out

# Uso
bn = BatchNormalization(num_features=10)
X = np.random.randn(32, 10)
X_bn = bn.forward(X, training=True)
print(f"Original mean: {X.mean():.4f}, std: {X.std():.4f}")
print(f"After BN mean: {X_bn.mean():.4f}, std: {X_bn.std():.4f}")
```

### Ejemplo 5: Dropout

```python
class Dropout:
    def __init__(self, dropout_rate=0.5):
        self.dropout_rate = dropout_rate
        self.mask = None
    
    def forward(self, X, training=True):
        if training:
            # Create dropout mask
            self.mask = np.random.binomial(1, 1 - self.dropout_rate, size=X.shape)
            # Apply mask and scale
            return X * self.mask / (1 - self.dropout_rate)
        else:
            # No dropout during inference
            return X

# Ejemplo
dropout = Dropout(dropout_rate=0.5)
X = np.random.randn(10, 100)

X_train = dropout.forward(X, training=True)
X_test = dropout.forward(X, training=False)

print(f"Training: {np.sum(X_train == 0) / X_train.size * 100:.1f}% zeros")
print(f"Testing: {np.sum(X_test == 0) / X_test.size * 100:.1f}% zeros")
```

### Ejemplo 6: Inicialización de Pesos

```python
def initialize_weights(shape, method='xavier'):
    """
    Diferentes métodos de inicialización de pesos
    """
    if method == 'xavier':
        # Xavier/Glorot (para sigmoid/tanh)
        limit = np.sqrt(6.0 / (shape[0] + shape[1]))
        return np.random.uniform(-limit, limit, shape)
    
    elif method == 'he':
        # He initialization (para ReLU)
        std = np.sqrt(2.0 / shape[0])
        return np.random.randn(*shape) * std
    
    elif method == 'lecun':
        # LeCun (para SELU)
        std = np.sqrt(1.0 / shape[0])
        return np.random.randn(*shape) * std
    
    elif method == 'zeros':
        return np.zeros(shape)
    
    else:  # random_normal
        return np.random.randn(*shape) * 0.01

# Comparar inicializaciones
shapes = (100, 100)

xavier_w = initialize_weights(shapes, 'xavier')
he_w = initialize_weights(shapes, 'he')
lecun_w = initialize_weights(shapes, 'lecun')

print(f"Xavier - mean: {xavier_w.mean():.4f}, std: {xavier_w.std():.4f}")
print(f"He - mean: {he_w.mean():.4f}, std: {he_w.std():.4f}")
print(f"LeCun - mean: {lecun_w.mean():.4f}, std: {lecun_w.std():.4f}")
```

### Ejemplo 7: XOR Problem (Clásico)

```python
import numpy as np
import matplotlib.pyplot as plt

# XOR dataset
X = np.array([[0, 0], [0, 1], [1, 0], [1, 1]])
y = np.array([[0], [1], [1], [0]])

class XORNetwork:
    def __init__(self):
        # 2 -> 2 -> 1 architecture
        self.W1 = np.random.randn(2, 2) * 0.5
        self.b1 = np.zeros((1, 2))
        self.W2 = np.random.randn(2, 1) * 0.5
        self.b2 = np.zeros((1, 1))
    
    def sigmoid(self, z):
        return 1 / (1 + np.exp(-np.clip(z, -500, 500)))
    
    def sigmoid_derivative(self, z):
        s = self.sigmoid(z)
        return s * (1 - s)
    
    def forward(self, X):
        self.z1 = np.dot(X, self.W1) + self.b1
        self.a1 = self.sigmoid(self.z1)
        self.z2 = np.dot(self.a1, self.W2) + self.b2
        self.a2 = self.sigmoid(self.z2)
        return self.a2
    
    def backward(self, X, y, learning_rate=1.0):
        m = X.shape[0]
        
        # Output layer
        dz2 = self.a2 - y
        dW2 = (1/m) * np.dot(self.a1.T, dz2)
        db2 = (1/m) * np.sum(dz2, axis=0, keepdims=True)
        
        # Hidden layer
        dz1 = np.dot(dz2, self.W2.T) * self.sigmoid_derivative(self.z1)
        dW1 = (1/m) * np.dot(X.T, dz1)
        db1 = (1/m) * np.sum(dz1, axis=0, keepdims=True)
        
        # Update
        self.W2 -= learning_rate * dW2
        self.b2 -= learning_rate * db2
        self.W1 -= learning_rate * dW1
        self.b1 -= learning_rate * db1
    
    def train(self, X, y, epochs=10000):
        losses = []
        for epoch in range(epochs):
            output = self.forward(X)
            loss = np.mean((output - y) ** 2)
            losses.append(loss)
            
            self.backward(X, y)
            
            if epoch % 1000 == 0:
                print(f'Epoch {epoch}, Loss: {loss:.4f}')
        
        return losses

# Entrenar
model = XORNetwork()
losses = model.train(X, y, epochs=10000)

# Resultados
predictions = model.forward(X)
print("\nPredictions:")
for i in range(len(X)):
    print(f"Input: {X[i]}, Target: {y[i][0]}, Prediction: {predictions[i][0]:.4f}")
```

## Mejores Prácticas

### 1. Inicialización Apropiada

```python
# ✅ Bueno - Xavier para sigmoid/tanh
W = np.random.randn(n_in, n_out) * np.sqrt(1.0 / n_in)

# ✅ Bueno - He para ReLU
W = np.random.randn(n_in, n_out) * np.sqrt(2.0 / n_in)

# ❌ Malo - todos ceros (simetría)
W = np.zeros((n_in, n_out))
```

### 2. Normalización de Datos

```python
# ✅ Bueno - normalizar inputs
X_mean = X_train.mean(axis=0)
X_std = X_train.std(axis=0)
X_train_normalized = (X_train - X_mean) / X_std
X_test_normalized = (X_test - X_mean) / X_std

# ✅ Usar Batch Normalization en capas profundas
```

### 3. Evitar Overfitting

```python
# ✅ Dropout
# ✅ L2 regularization
# ✅ Early stopping
# ✅ Data augmentation
# ✅ Reduce model complexity
```

## Proyectos Sugeridos

1. **Clasificador de Dígitos MNIST**
   - MLP desde cero
   - Comparar activaciones
   - Optimizar arquitectura

2. **Aproximador de Funciones**
   - Universal approximation
   - Diferentes funciones (sin, cos, etc.)
   - Visualizar decision boundaries

3. **Autoencoder Simple**
   - Comprimir y reconstruir datos
   - Visualizar espacio latente
   - PCA vs Autoencoder

4. **Predictor de Series Temporales**
   - MLP para forecasting
   - Feature engineering temporal
   - Comparar con modelos clásicos

## Checklist de Aprendizaje

- [ ] Entender neurona artificial
- [ ] Implementar backpropagation desde cero
- [ ] Dominar funciones de activación
- [ ] Aplicar inicialización correcta
- [ ] Usar regularización efectivamente
- [ ] Optimizar hiperparámetros
- [ ] Detectar y solucionar vanishing/exploding gradients
- [ ] Visualizar activaciones y pesos
- [ ] Implementar MLP completo
- [ ] Resolver XOR problem

## Próximos Pasos

1. Estudiar **CNNs** para computer vision
2. Aprender **RNNs/LSTMs** para secuencias
3. Profundizar en **Optimización Avanzada**
4. Explorar **Transformers** 
5. Dominar **Transfer Learning**

---
**Tiempo estimado**: 4-6 semanas
