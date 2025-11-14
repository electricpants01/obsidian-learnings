# Optimización para Machine Learning

## Descripción

La optimización es el corazón del aprendizaje automático. Entrenar un modelo de ML es esencialmente un problema de optimización: encontrar los parámetros que minimizan (o maximizan) una función objetivo. Gradient descent y sus variantes son los algoritmos fundamentales que permiten que las redes neuronales y otros modelos aprendan.

## Conceptos Clave

### 1. **Problema de Optimización**
- Función objetivo (loss function)
- Variables de decisión (parámetros del modelo)
- Restricciones (constraints)
- Mínimos locales vs globales

### 2. **Gradient Descent**
- Batch Gradient Descent
- Stochastic Gradient Descent (SGD)
- Mini-batch Gradient Descent
- Learning rate scheduling

### 3. **Momentum-based Methods**
- Momentum
- Nesterov Accelerated Gradient (NAG)
- Aceleración de convergencia

### 4. **Adaptive Learning Rate Methods**
- AdaGrad
- RMSprop
- Adam (Adaptive Moment Estimation)
- AdamW

### 5. **Second-Order Methods**
- Newton's Method
- Quasi-Newton (BFGS, L-BFGS)
- Matriz Hessiana
- Curvature information

### 6. **Constrained Optimization**
- Lagrange multipliers
- KKT conditions
- Projected gradient descent

### 7. **Regularización**
- L1 (Lasso) - induce sparsity
- L2 (Ridge) - weight decay
- Elastic Net
- Dropout como regularización

## Aplicaciones en AI/ML

- **Training Neural Networks**: Optimización de pesos
- **Hyperparameter Tuning**: Grid search, Random search, Bayesian optimization
- **Support Vector Machines**: Maximización de margen
- **Linear/Logistic Regression**: Minimización de error
- **Clustering**: K-means como optimización
- **Reinforcement Learning**: Policy optimization

## Recursos de Aprendizaje

### Cursos Online

1. **Stanford - Convex Optimization** (Gratis)
   - https://www.edx.org/learn/computer-science/stanford-university-convex-optimization
   - Professor Stephen Boyd
   - Curso clásico y fundamental

2. **Coursera - Optimization for Machine Learning**
   - https://www.coursera.org/learn/optimization-for-machine-learning

3. **MIT OpenCourseWare - Nonlinear Programming**
   - https://ocw.mit.edu/courses/15-084j-nonlinear-programming-spring-2004/

4. **fast.ai - Practical Deep Learning**
   - https://course.fast.ai/
   - Enfoque práctico en optimizers

### Libros

1. **"Convex Optimization"** - Stephen Boyd y Lieven Vandenberghe
   - El libro definitivo
   - Gratis: https://web.stanford.edu/~boyd/cvxbook/

2. **"Numerical Optimization"** - Nocedal & Wright
   - Muy completo y práctico
   - Referencia estándar

3. **"Algorithms for Optimization"** - Kochenderfer & Wheeler
   - Moderno, con código Julia
   - https://algorithmsbook.com/optimization/

4. **"Deep Learning"** - Goodfellow, Bengio & Courville
   - Capítulo 8: Optimization for Training Deep Models
   - Gratis: https://www.deeplearningbook.org/

### Documentación

1. **PyTorch Optimizers**
   - https://pytorch.org/docs/stable/optim.html
   - Documentación de todos los optimizers

2. **TensorFlow Optimizers**
   - https://www.tensorflow.org/api_docs/python/tf/keras/optimizers

3. **SciPy Optimize**
   - https://docs.scipy.org/doc/scipy/reference/optimize.html
   - Optimización numérica general

### Videos Recomendados

1. **StatQuest - Gradient Descent**
   - https://www.youtube.com/watch?v=sDv4f4s2SB8
   - Explicación visual clara

2. **Andrew Ng - Optimization Algorithms**
   - https://www.youtube.com/playlist?list=PLkDaE6sCZn6Hn0vK8co82zjQtt3T2Nkqc
   - De su curso de Deep Learning

3. **Sebastian Ruder - An Overview of Gradient Descent Optimization Algorithms**
   - Blog post clásico: https://ruder.io/optimizing-gradient-descent/

## Herramientas y Bibliotecas

### PyTorch Optimizers

```python
import torch
import torch.nn as nn
import torch.optim as optim

# Modelo simple
model = nn.Linear(10, 1)

# SGD
optimizer_sgd = optim.SGD(model.parameters(), lr=0.01, momentum=0.9)

# Adam
optimizer_adam = optim.Adam(model.parameters(), lr=0.001, betas=(0.9, 0.999))

# AdamW (Adam with weight decay)
optimizer_adamw = optim.AdamW(model.parameters(), lr=0.001, weight_decay=0.01)

# RMSprop
optimizer_rmsprop = optim.RMSprop(model.parameters(), lr=0.01, alpha=0.99)

# Training loop
for epoch in range(100):
    optimizer_adam.zero_grad()  # Clear gradients
    output = model(input_data)
    loss = criterion(output, target)
    loss.backward()  # Compute gradients
    optimizer_adam.step()  # Update parameters
```

### Learning Rate Schedulers

```python
import torch.optim.lr_scheduler as lr_scheduler

# Step decay
scheduler = lr_scheduler.StepLR(optimizer, step_size=30, gamma=0.1)

# Exponential decay
scheduler = lr_scheduler.ExponentialLR(optimizer, gamma=0.95)

# Cosine annealing
scheduler = lr_scheduler.CosineAnnealingLR(optimizer, T_max=100)

# Reduce on plateau
scheduler = lr_scheduler.ReduceLROnPlateau(optimizer, mode='min', 
                                           factor=0.1, patience=10)

# One Cycle Policy
scheduler = lr_scheduler.OneCycleLR(optimizer, max_lr=0.1, 
                                    steps_per_epoch=100, epochs=10)

# Training loop with scheduler
for epoch in range(100):
    train(...)
    validate(...)
    scheduler.step()  # Update learning rate
```

### SciPy Optimization

```python
from scipy.optimize import minimize, differential_evolution

# Función a minimizar
def rosenbrock(x):
    return sum(100.0*(x[1:]-x[:-1]**2.0)**2.0 + (1-x[:-1])**2.0)

# Punto inicial
x0 = [1.3, 0.7, 0.8, 1.9, 1.2]

# Método BFGS
result = minimize(rosenbrock, x0, method='BFGS')

# Nelder-Mead (no requiere gradientes)
result = minimize(rosenbrock, x0, method='Nelder-Mead')

# L-BFGS-B (con bounds)
bounds = [(0, 2)] * len(x0)
result = minimize(rosenbrock, x0, method='L-BFGS-B', bounds=bounds)

# Algoritmo genético para optimización global
result = differential_evolution(rosenbrock, bounds)

print(f"Minimum found at: {result.x}")
print(f"Function value: {result.fun}")
```

## Ejemplos Prácticos

### Ejemplo 1: Implementar Optimizers desde Cero

```python
import numpy as np

class SGD:
    def __init__(self, lr=0.01, momentum=0.0):
        self.lr = lr
        self.momentum = momentum
        self.velocity = None
        
    def update(self, params, grads):
        if self.velocity is None:
            self.velocity = np.zeros_like(params)
        
        self.velocity = self.momentum * self.velocity - self.lr * grads
        params += self.velocity
        return params

class Adam:
    def __init__(self, lr=0.001, beta1=0.9, beta2=0.999, epsilon=1e-8):
        self.lr = lr
        self.beta1 = beta1
        self.beta2 = beta2
        self.epsilon = epsilon
        self.m = None  # First moment
        self.v = None  # Second moment
        self.t = 0     # Time step
        
    def update(self, params, grads):
        if self.m is None:
            self.m = np.zeros_like(params)
            self.v = np.zeros_like(params)
        
        self.t += 1
        
        # Update biased first moment estimate
        self.m = self.beta1 * self.m + (1 - self.beta1) * grads
        
        # Update biased second moment estimate
        self.v = self.beta2 * self.v + (1 - self.beta2) * (grads ** 2)
        
        # Bias correction
        m_hat = self.m / (1 - self.beta1 ** self.t)
        v_hat = self.v / (1 - self.beta2 ** self.t)
        
        # Update parameters
        params -= self.lr * m_hat / (np.sqrt(v_hat) + self.epsilon)
        return params

# Comparar optimizers
def compare_optimizers(f, grad_f, x0, optimizers, n_iterations=100):
    """
    Compara diferentes optimizers
    """
    import matplotlib.pyplot as plt
    
    results = {}
    for name, optimizer in optimizers.items():
        x = x0.copy()
        history = [x.copy()]
        
        for _ in range(n_iterations):
            grads = grad_f(x)
            x = optimizer.update(x, grads)
            history.append(x.copy())
        
        results[name] = np.array(history)
    
    # Visualizar
    plt.figure(figsize=(12, 5))
    
    # Trajectories
    plt.subplot(1, 2, 1)
    for name, history in results.items():
        plt.plot(history[:, 0], history[:, 1], 'o-', label=name, alpha=0.7)
    plt.xlabel('x1')
    plt.ylabel('x2')
    plt.legend()
    plt.title('Optimization Trajectories')
    plt.grid(True)
    
    # Convergence
    plt.subplot(1, 2, 2)
    for name, history in results.items():
        values = [f(x) for x in history]
        plt.plot(values, label=name)
    plt.xlabel('Iteration')
    plt.ylabel('Function Value')
    plt.yscale('log')
    plt.legend()
    plt.title('Convergence')
    plt.grid(True)
    
    plt.tight_layout()
    plt.show()

# Ejemplo: Rosenbrock function
def rosenbrock(x):
    return (1 - x[0])**2 + 100 * (x[1] - x[0]**2)**2

def rosenbrock_grad(x):
    dx0 = -2 * (1 - x[0]) - 400 * x[0] * (x[1] - x[0]**2)
    dx1 = 200 * (x[1] - x[0]**2)
    return np.array([dx0, dx1])

x0 = np.array([-1.0, 1.0])

optimizers = {
    'SGD': SGD(lr=0.001),
    'SGD+Momentum': SGD(lr=0.001, momentum=0.9),
    'Adam': Adam(lr=0.01)
}

compare_optimizers(rosenbrock, rosenbrock_grad, x0, optimizers)
```

### Ejemplo 2: Learning Rate Finder

```python
import numpy as np
import matplotlib.pyplot as plt

class LRFinder:
    """
    Encuentra el learning rate óptimo
    Método de Leslie Smith
    """
    def __init__(self, model, optimizer, criterion):
        self.model = model
        self.optimizer = optimizer
        self.criterion = criterion
        
    def range_test(self, train_loader, end_lr=10, num_iter=100, 
                   smooth_f=0.05, diverge_th=5):
        """
        Prueba un rango de learning rates
        """
        lrs = []
        losses = []
        best_loss = float('inf')
        
        # Start with very small lr
        lr = end_lr / num_iter
        self.optimizer.param_groups[0]['lr'] = lr
        
        # Exponential increase of lr
        mult = (end_lr / lr) ** (1 / num_iter)
        
        for iteration, (inputs, targets) in enumerate(train_loader):
            if iteration >= num_iter:
                break
            
            # Forward pass
            outputs = self.model(inputs)
            loss = self.criterion(outputs, targets)
            
            # Smooth the loss
            if iteration == 0:
                avg_loss = loss.item()
            else:
                avg_loss = smooth_f * loss.item() + (1 - smooth_f) * avg_loss
            
            # Record the best loss
            if avg_loss < best_loss:
                best_loss = avg_loss
            
            # Stop if loss is exploding
            if avg_loss > diverge_th * best_loss:
                print(f"Stopping early at iteration {iteration}")
                break
            
            # Record
            lrs.append(lr)
            losses.append(avg_loss)
            
            # Backward pass
            self.optimizer.zero_grad()
            loss.backward()
            self.optimizer.step()
            
            # Update learning rate
            lr *= mult
            self.optimizer.param_groups[0]['lr'] = lr
        
        return lrs, losses
    
    def plot(self, lrs, losses, skip_start=10, skip_end=5):
        """
        Plot learning rate vs loss
        """
        if skip_end == 0:
            lrs = lrs[skip_start:]
            losses = losses[skip_start:]
        else:
            lrs = lrs[skip_start:-skip_end]
            losses = losses[skip_start:-skip_end]
        
        plt.figure(figsize=(10, 6))
        plt.plot(lrs, losses)
        plt.xscale('log')
        plt.xlabel('Learning Rate')
        plt.ylabel('Loss')
        plt.title('Learning Rate Finder')
        plt.grid(True)
        
        # Find the point with steepest descent
        min_grad_idx = np.gradient(np.array(losses)).argmin()
        suggested_lr = lrs[min_grad_idx]
        plt.axvline(x=suggested_lr, color='r', linestyle='--', 
                   label=f'Suggested LR: {suggested_lr:.2e}')
        plt.legend()
        plt.show()
        
        return suggested_lr

# Uso:
# lr_finder = LRFinder(model, optimizer, criterion)
# lrs, losses = lr_finder.range_test(train_loader)
# suggested_lr = lr_finder.plot(lrs, losses)
```

### Ejemplo 3: Grid Search vs Random Search

```python
import numpy as np
from sklearn.model_selection import GridSearchCV, RandomizedSearchCV
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import make_classification

# Generar datos
X, y = make_classification(n_samples=1000, n_features=20, 
                           n_informative=15, n_redundant=5, random_state=42)

# Grid Search
param_grid = {
    'n_estimators': [50, 100, 200],
    'max_depth': [5, 10, 15, None],
    'min_samples_split': [2, 5, 10],
    'min_samples_leaf': [1, 2, 4]
}

grid_search = GridSearchCV(
    RandomForestClassifier(random_state=42),
    param_grid,
    cv=5,
    scoring='accuracy',
    n_jobs=-1,
    verbose=1
)

grid_search.fit(X, y)
print(f"Best parameters (Grid): {grid_search.best_params_}")
print(f"Best score (Grid): {grid_search.best_score_:.4f}")

# Random Search
from scipy.stats import randint, uniform

param_distributions = {
    'n_estimators': randint(50, 500),
    'max_depth': randint(5, 50),
    'min_samples_split': randint(2, 20),
    'min_samples_leaf': randint(1, 10),
    'max_features': uniform(0.1, 0.9)
}

random_search = RandomizedSearchCV(
    RandomForestClassifier(random_state=42),
    param_distributions,
    n_iter=50,  # Number of parameter settings sampled
    cv=5,
    scoring='accuracy',
    n_jobs=-1,
    verbose=1,
    random_state=42
)

random_search.fit(X, y)
print(f"\nBest parameters (Random): {random_search.best_params_}")
print(f"Best score (Random): {random_search.best_score_:.4f}")
```

### Ejemplo 4: Bayesian Optimization

```python
from sklearn.gaussian_process import GaussianProcessRegressor
from sklearn.gaussian_process.kernels import Matern
import numpy as np

class BayesianOptimization:
    """
    Optimización bayesiana simple para tuning de hiperparámetros
    """
    def __init__(self, f, bounds, n_init=5):
        self.f = f  # Black-box function to optimize
        self.bounds = np.array(bounds)
        self.n_params = len(bounds)
        
        # Initialize GP
        kernel = Matern(nu=2.5)
        self.gp = GaussianProcessRegressor(kernel=kernel, 
                                          alpha=1e-6, 
                                          n_restarts_optimizer=10)
        
        # Sample initial points
        self.X = np.random.uniform(self.bounds[:, 0], self.bounds[:, 1], 
                                  size=(n_init, self.n_params))
        self.y = np.array([self.f(x) for x in self.X])
        
    def acquisition_function(self, X, xi=0.01):
        """
        Expected Improvement acquisition function
        """
        mu, sigma = self.gp.predict(X, return_std=True)
        
        with np.errstate(divide='warn'):
            imp = mu - np.max(self.y) - xi
            Z = imp / sigma
            ei = imp * norm.cdf(Z) + sigma * norm.pdf(Z)
            ei[sigma == 0.0] = 0.0
        
        return ei
    
    def propose_location(self):
        """
        Propone siguiente punto a evaluar
        """
        from scipy.optimize import minimize
        from scipy.stats import norm
        
        # Random restart optimization
        min_val = float('inf')
        min_x = None
        
        for _ in range(10):
            x0 = np.random.uniform(self.bounds[:, 0], self.bounds[:, 1])
            
            res = minimize(lambda x: -self.acquisition_function(x.reshape(1, -1)),
                          x0, bounds=self.bounds, method='L-BFGS-B')
            
            if res.fun < min_val:
                min_val = res.fun
                min_x = res.x
        
        return min_x
    
    def optimize(self, n_iter=20):
        """
        Run Bayesian optimization
        """
        for i in range(n_iter):
            # Fit GP
            self.gp.fit(self.X, self.y)
            
            # Get next point
            x_next = self.propose_location()
            
            # Evaluate
            y_next = self.f(x_next)
            
            # Add to data
            self.X = np.vstack([self.X, x_next])
            self.y = np.append(self.y, y_next)
            
            print(f"Iteration {i+1}: f({x_next}) = {y_next:.4f}")
        
        # Return best
        idx = np.argmax(self.y)
        return self.X[idx], self.y[idx]

# Ejemplo
def objective(x):
    """Función objetivo a maximizar"""
    return -(x[0] - 2)**2 - (x[1] - 3)**2 + 10

bounds = [(-5, 5), (-5, 5)]
bo = BayesianOptimization(objective, bounds)
best_x, best_y = bo.optimize(n_iter=20)

print(f"\nBest parameters: {best_x}")
print(f"Best value: {best_y:.4f}")
```

### Ejemplo 5: Constrained Optimization

```python
from scipy.optimize import minimize

def objective(x):
    """Función objetivo"""
    return x[0]**2 + x[1]**2

def constraint1(x):
    """Constraint: x[0] + x[1] >= 1"""
    return x[0] + x[1] - 1

def constraint2(x):
    """Constraint: x[0]**2 + x[1]**2 <= 4"""
    return 4 - (x[0]**2 + x[1]**2)

# Define constraints
cons = [
    {'type': 'ineq', 'fun': constraint1},
    {'type': 'ineq', 'fun': constraint2}
]

# Initial guess
x0 = [2, 2]

# Optimize
result = minimize(objective, x0, method='SLSQP', constraints=cons)

print(f"Optimum: {result.x}")
print(f"Objective value: {result.fun}")
print(f"Success: {result.success}")
```

## Proyectos Sugeridos

1. **Implementar Optimizers Library**
   - SGD, Momentum, Adam, RMSprop desde cero
   - Comparar convergencia en diferentes problemas
   - Visualizar trayectorias de optimización

2. **Hyperparameter Tuning Framework**
   - Implementar Grid Search, Random Search
   - Integrar Bayesian Optimization
   - Crear visualizaciones de performance

3. **Learning Rate Schedule Explorer**
   - App interactiva (Streamlit)
   - Probar diferentes schedulers
   - Visualizar impacto en entrenamiento

4. **Constrained Optimization Solver**
   - Implementar métodos de penalty
   - Lagrange multipliers
   - Aplicar a problemas reales

5. **Neural Architecture Search (NAS)**
   - Usar optimización para encontrar arquitecturas
   - Implementar versión simple de NAS
   - Evolutionary algorithms o Bayesian optimization

## Conceptos Avanzados

### Convexidad

```python
import numpy as np
import matplotlib.pyplot as plt

def is_convex_numerical(f, bounds, n_samples=1000):
    """
    Verifica numéricamente si función es convexa
    Una función es convexa si f(λx + (1-λ)y) <= λf(x) + (1-λ)f(y)
    """
    d = len(bounds)
    
    for _ in range(n_samples):
        # Sample two random points
        x = np.random.uniform([b[0] for b in bounds], [b[1] for b in bounds])
        y = np.random.uniform([b[0] for b in bounds], [b[1] for b in bounds])
        
        # Random lambda
        lam = np.random.uniform(0, 1)
        
        # Check convexity condition
        z = lam * x + (1 - lam) * y
        if f(z) > lam * f(x) + (1 - lam) * f(y) + 1e-6:
            return False
    
    return True

# Ejemplo
def quadratic(x):
    return x[0]**2 + x[1]**2

def rosenbrock(x):
    return (1 - x[0])**2 + 100 * (x[1] - x[0]**2)**2

bounds = [(-2, 2), (-2, 2)]

print(f"Quadratic is convex: {is_convex_numerical(quadratic, bounds)}")
print(f"Rosenbrock is convex: {is_convex_numerical(rosenbrock, bounds)}")
```

### Gradient Clipping

```python
import torch

def gradient_clipping_demo():
    """
    Demuestra gradient clipping para estabilidad
    """
    # Modelo y datos
    model = torch.nn.Linear(10, 1)
    optimizer = torch.optim.SGD(model.parameters(), lr=0.01)
    
    # Dummy data
    x = torch.randn(32, 10)
    y = torch.randn(32, 1)
    
    # Training step
    optimizer.zero_grad()
    output = model(x)
    loss = torch.nn.functional.mse_loss(output, y)
    loss.backward()
    
    # Gradient clipping by norm
    torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)
    
    # Or by value
    torch.nn.utils.clip_grad_value_(model.parameters(), clip_value=0.5)
    
    optimizer.step()
```

## Mejores Prácticas

1. **Elección de Optimizer**
   - **Adam**: Buena opción por default
   - **SGD + Momentum**: Mejor generalización en algunos casos
   - **AdamW**: Adam con weight decay apropiado

2. **Learning Rate**
   - Usa learning rate finder
   - Considera learning rate schedules
   - OneCycleLR suele funcionar bien

3. **Batch Size**
   - Más grande = más estable pero menos regularización
   - Pequeño = más ruido pero mejor generalización
   - Encuentra balance según memoria disponible

4. **Monitoring**
   - Visualiza loss curves (train y validation)
   - Monitorea gradientes (no deben explotar o desvanecerse)
   - Verifica que no haya overfitting

5. **Regularización**
   - Weight decay (L2)
   - Dropout
   - Early stopping
   - Data augmentation

## Recursos Adicionales

### Papers Fundamentales

1. **"Adam: A Method for Stochastic Optimization"**
   - https://arxiv.org/abs/1412.6980
   - Kingma & Ba, 2014

2. **"An Overview of Gradient Descent Optimization Algorithms"**
   - https://arxiv.org/abs/1609.04747
   - Sebastian Ruder

3. **"Cyclical Learning Rates for Training Neural Networks"**
   - https://arxiv.org/abs/1506.01186
   - Leslie Smith

4. **"Decoupled Weight Decay Regularization"** (AdamW)
   - https://arxiv.org/abs/1711.05101

### Herramientas

1. **Optuna** - Hyperparameter optimization framework
   - https://optuna.org/
   - Bayesian optimization, pruning

2. **Ray Tune** - Scalable hyperparameter tuning
   - https://docs.ray.io/en/latest/tune/index.html

3. **Weights & Biases** - Experiment tracking
   - https://wandb.ai/

### Blogs

1. **Sebastian Ruder's Blog**
   - https://ruder.io/
   - Excelentes posts sobre optimización

2. **Distill.pub - Why Momentum Really Works**
   - https://distill.pub/2017/momentum/

## Comparación de Optimizers

| Optimizer | Ventajas | Desventajas | Cuándo Usar |
|-----------|----------|-------------|-------------|
| **SGD** | Simple, bien entendido | Lento, sensible a LR | Baseline, investigación |
| **SGD + Momentum** | Más rápido que SGD | Requiere tuning | Cuando SGD es muy lento |
| **Adam** | Adaptativo, funciona bien por default | Puede overfittear | Primera opción, prototipos |
| **AdamW** | Adam + weight decay correcto | Más hiperparámetros | Production, mejora sobre Adam |
| **RMSprop** | Bueno para RNNs | No tan popular | Secuencias, RNNs |

## Checklist de Aprendizaje

- [ ] Entender gradient descent y variantes
- [ ] Implementar SGD y Adam desde cero
- [ ] Usar learning rate scheduling
- [ ] Aplicar gradient clipping
- [ ] Realizar hyperparameter tuning
- [ ] Entender trade-offs entre optimizers
- [ ] Implementar constrained optimization
- [ ] Aplicar en proyecto real de ML

## Próximos Pasos

Con los fundamentos matemáticos completos:
1. Aplicar en **Machine Learning Clásico**
2. Profundizar en **Deep Learning**
3. Explorar **Second-Order Methods**
4. Estudiar **Distributed Optimization**

---
**Tiempo estimado de estudio**: 4-5 semanas dedicando 2-3 horas diarias
