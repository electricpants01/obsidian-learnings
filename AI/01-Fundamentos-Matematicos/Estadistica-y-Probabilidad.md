# Estadística y Probabilidad para AI

## Descripción

La estadística y probabilidad son fundamentales para Machine Learning. Los modelos ML son inherentemente probabilísticos - hacen predicciones bajo incertidumbre, aprenden de datos ruidosos y utilizan conceptos como distribuciones de probabilidad, inferencia estadística y teorema de Bayes.

## Conceptos Clave

### 1. **Probabilidad Básica**
- Espacio muestral y eventos
- Probabilidad condicional: P(A|B)
- Independencia de eventos
- Regla de la suma y el producto

### 2. **Teorema de Bayes**
- P(A|B) = P(B|A) × P(A) / P(B)
- Prior, likelihood, posterior
- Aplicaciones en clasificadores bayesianos

### 3. **Variables Aleatorias**
- Discretas vs continuas
- Función de masa de probabilidad (PMF)
- Función de densidad de probabilidad (PDF)
- Función de distribución acumulada (CDF)

### 4. **Distribuciones de Probabilidad**
- **Discretas**: Bernoulli, Binomial, Poisson
- **Continuas**: Normal (Gaussiana), Uniforme, Exponencial
- Distribuciones multivariadas

### 5. **Esperanza y Varianza**
- Valor esperado: E[X]
- Varianza: Var(X) = E[(X - μ)²]
- Desviación estándar
- Covarianza y correlación

### 6. **Estimación Estadística**
- Estimadores puntuales
- Intervalos de confianza
- Maximum Likelihood Estimation (MLE)
- Maximum A Posteriori (MAP)

### 7. **Pruebas de Hipótesis**
- Hipótesis nula y alternativa
- p-valores
- Errores tipo I y tipo II
- Tests estadísticos comunes

### 8. **Información y Entropía**
- Entropía de Shannon
- Cross-entropy
- KL Divergence
- Mutual Information

## Aplicaciones en AI/ML

- **Clasificación Probabilística**: Naive Bayes, Regresión Logística
- **Funciones de Pérdida**: Cross-entropy loss
- **Incertidumbre**: Bayesian Neural Networks
- **Generative Models**: VAEs, GANs
- **Reinforcement Learning**: Políticas estocásticas
- **A/B Testing**: Validación de modelos
- **Regularización**: Prior bayesiano como regularización

## Recursos de Aprendizaje

### Cursos Online

1. **Khan Academy - Statistics and Probability** (Gratis)
   - https://www.khanacademy.org/math/statistics-probability
   - Excelente para principiantes

2. **StatQuest with Josh Starmer** (Gratis)
   - https://www.youtube.com/c/joshstarmer
   - Explicaciones visuales y claras

3. **MIT OpenCourseWare - Probabilistic Systems Analysis**
   - https://ocw.mit.edu/courses/6-041-probabilistic-systems-analysis-and-applied-probability-fall-2010/

4. **Coursera - Mathematics for Machine Learning: Probability and Statistics**
   - https://www.coursera.org/learn/probability-statistics-machine-learning

5. **Stanford - Probabilistic Graphical Models**
   - https://www.coursera.org/specializations/probabilistic-graphical-models

### Libros

1. **"Introduction to Probability"** - Dimitri P. Bertsekas y John N. Tsitsiklis
   - Libro del curso de MIT
   - Muy completo

2. **"All of Statistics"** - Larry Wasserman
   - Estadística para Computer Science
   - Conciso y práctico

3. **"Pattern Recognition and Machine Learning"** - Christopher Bishop
   - Capítulos 1-2 son excelentes para probabilidad en ML
   - Disponible gratis

4. **"Probabilistic Machine Learning"** - Kevin Murphy
   - Dos tomos: Introduction y Advanced Topics
   - Estado del arte

5. **"Think Stats"** - Allen B. Downey
   - Enfoque práctico con Python
   - Gratis: https://greenteapress.com/wp/think-stats-2e/

### Documentación

1. **SciPy Stats**
   - https://docs.scipy.org/doc/scipy/reference/stats.html
   - Distribuciones y tests estadísticos

2. **NumPy Random**
   - https://numpy.org/doc/stable/reference/random/index.html
   - Generación de números aleatorios

3. **StatsModels**
   - https://www.statsmodels.org/stable/index.html
   - Análisis estadístico en Python

### Videos Recomendados

1. **StatQuest - Statistics Fundamentals**
   - https://www.youtube.com/playlist?list=PLblh5JKOoLUK0FLuzwntyYI10UQFUhsY9
   - Serie completa de estadística básica

2. **3Blue1Brown - Bayes Theorem**
   - https://www.youtube.com/watch?v=HZGCoVF3YvM
   - Visualización del teorema de Bayes

3. **Khan Academy - Probability**
   - https://www.youtube.com/playlist?list=PLC58778F28211FA19

## Herramientas y Bibliotecas

### Python - Bibliotecas Principales

```python
import numpy as np
from scipy import stats
import matplotlib.pyplot as plt
import seaborn as sns
import pandas as pd
from statsmodels import api as sm

# Configuración para gráficos
sns.set_style("whitegrid")
```

### Distribuciones de Probabilidad

```python
from scipy.stats import norm, binom, poisson, expon

# Distribución Normal
mu, sigma = 0, 1
x = np.linspace(-4, 4, 100)
pdf = norm.pdf(x, mu, sigma)
cdf = norm.cdf(x, mu, sigma)

# Generar muestras
samples = norm.rvs(mu, sigma, size=1000)

# Distribución Binomial
n, p = 10, 0.5
k = np.arange(0, 11)
pmf = binom.pmf(k, n, p)

# Distribución Poisson
lambda_param = 3
k = np.arange(0, 15)
pmf_poisson = poisson.pmf(k, lambda_param)

# Estadísticas de distribución
mean = norm.mean(mu, sigma)
variance = norm.var(mu, sigma)
```

### Tests Estadísticos

```python
from scipy.stats import ttest_ind, chi2_contingency, kstest

# T-test (comparar medias)
group1 = np.random.normal(0, 1, 100)
group2 = np.random.normal(0.5, 1, 100)
t_stat, p_value = ttest_ind(group1, group2)
print(f"p-value: {p_value}")

# Chi-square test (independencia)
contingency_table = np.array([[10, 20], [30, 40]])
chi2, p_value, dof, expected = chi2_contingency(contingency_table)

# Kolmogorov-Smirnov test (normalidad)
data = np.random.normal(0, 1, 100)
ks_stat, p_value = kstest(data, 'norm')
```

## Ejemplos Prácticos

### Ejemplo 1: Teorema de Bayes - Email Spam

```python
import numpy as np

class NaiveBayesSpamClassifier:
    """
    Clasificador de spam usando Teorema de Bayes
    P(Spam|Words) = P(Words|Spam) * P(Spam) / P(Words)
    """
    
    def __init__(self):
        self.spam_prob = 0  # P(Spam)
        self.word_given_spam = {}  # P(Word|Spam)
        self.word_given_ham = {}  # P(Word|Ham)
        
    def train(self, emails, labels):
        """
        emails: lista de emails (cada uno es una lista de palabras)
        labels: 1 para spam, 0 para ham
        """
        spam_emails = [email for email, label in zip(emails, labels) if label == 1]
        ham_emails = [email for email, label in zip(emails, labels) if label == 0]
        
        # P(Spam)
        self.spam_prob = len(spam_emails) / len(emails)
        
        # Contar palabras
        spam_words = {}
        ham_words = {}
        
        for email in spam_emails:
            for word in email:
                spam_words[word] = spam_words.get(word, 0) + 1
                
        for email in ham_emails:
            for word in email:
                ham_words[word] = ham_words.get(word, 0) + 1
        
        # Calcular probabilidades (con Laplace smoothing)
        vocab = set(spam_words.keys()) | set(ham_words.keys())
        total_spam = sum(spam_words.values())
        total_ham = sum(ham_words.values())
        
        for word in vocab:
            # P(Word|Spam) con smoothing
            self.word_given_spam[word] = (spam_words.get(word, 0) + 1) / (total_spam + len(vocab))
            self.word_given_ham[word] = (ham_words.get(word, 0) + 1) / (total_ham + len(vocab))
    
    def predict(self, email):
        """Predice si un email es spam"""
        # Log probabilities para evitar underflow
        log_spam = np.log(self.spam_prob)
        log_ham = np.log(1 - self.spam_prob)
        
        for word in email:
            if word in self.word_given_spam:
                log_spam += np.log(self.word_given_spam[word])
                log_ham += np.log(self.word_given_ham[word])
        
        return 1 if log_spam > log_ham else 0

# Ejemplo de uso
emails = [
    ['win', 'money', 'now'],
    ['meeting', 'tomorrow', 'project'],
    ['free', 'money', 'win'],
    ['project', 'deadline', 'meeting']
]
labels = [1, 0, 1, 0]  # 1 = spam, 0 = ham

classifier = NaiveBayesSpamClassifier()
classifier.train(emails, labels)

test_email = ['free', 'meeting']
prediction = classifier.predict(test_email)
print(f"Email es {'spam' if prediction == 1 else 'ham'}")
```

### Ejemplo 2: Maximum Likelihood Estimation

```python
import numpy as np
from scipy.stats import norm
import matplotlib.pyplot as plt

def mle_normal_distribution(data):
    """
    Estima parámetros de distribución normal usando MLE
    Para distribución normal: μ_MLE = mean(data), σ²_MLE = var(data)
    """
    mu_mle = np.mean(data)
    sigma_mle = np.std(data, ddof=0)  # MLE usa ddof=0
    
    return mu_mle, sigma_mle

# Generar datos de distribución normal conocida
true_mu, true_sigma = 5, 2
data = np.random.normal(true_mu, true_sigma, 1000)

# Estimar parámetros
mu_est, sigma_est = mle_normal_distribution(data)

print(f"True μ: {true_mu}, Estimated μ: {mu_est:.3f}")
print(f"True σ: {true_sigma}, Estimated σ: {sigma_est:.3f}")

# Visualizar
x = np.linspace(-5, 15, 200)
plt.figure(figsize=(10, 6))
plt.hist(data, bins=50, density=True, alpha=0.7, label='Data')
plt.plot(x, norm.pdf(x, true_mu, true_sigma), 'r-', lw=2, label='True Distribution')
plt.plot(x, norm.pdf(x, mu_est, sigma_est), 'g--', lw=2, label='MLE Estimate')
plt.xlabel('x')
plt.ylabel('Density')
plt.legend()
plt.title('Maximum Likelihood Estimation')
plt.grid(True)
plt.show()
```

### Ejemplo 3: A/B Testing

```python
import numpy as np
from scipy.stats import ttest_ind

def ab_test(control, treatment, alpha=0.05):
    """
    Realiza A/B test usando t-test
    
    control: resultados del grupo de control
    treatment: resultados del grupo de tratamiento
    alpha: nivel de significancia
    """
    # Estadísticas descriptivas
    control_mean = np.mean(control)
    treatment_mean = np.mean(treatment)
    
    # T-test
    t_stat, p_value = ttest_ind(control, treatment)
    
    # Decisión
    significant = p_value < alpha
    
    # Tamaño del efecto (Cohen's d)
    pooled_std = np.sqrt((np.var(control) + np.var(treatment)) / 2)
    cohens_d = (treatment_mean - control_mean) / pooled_std
    
    results = {
        'control_mean': control_mean,
        'treatment_mean': treatment_mean,
        'difference': treatment_mean - control_mean,
        'p_value': p_value,
        'significant': significant,
        'cohens_d': cohens_d
    }
    
    return results

# Simular experimento A/B
np.random.seed(42)
control = np.random.normal(10, 2, 1000)  # Tasa de conversión control: 10%
treatment = np.random.normal(10.5, 2, 1000)  # Tasa de conversión tratamiento: 10.5%

results = ab_test(control, treatment)

print("A/B Test Results:")
print(f"Control Mean: {results['control_mean']:.3f}")
print(f"Treatment Mean: {results['treatment_mean']:.3f}")
print(f"Difference: {results['difference']:.3f}")
print(f"P-value: {results['p_value']:.4f}")
print(f"Significant: {results['significant']}")
print(f"Cohen's d: {results['cohens_d']:.3f}")
```

### Ejemplo 4: Entropía y Cross-Entropy

```python
import numpy as np

def entropy(p):
    """
    Calcula entropía de Shannon: H(p) = -Σ p(x) log(p(x))
    """
    p = np.array(p)
    p = p[p > 0]  # Eliminar ceros para evitar log(0)
    return -np.sum(p * np.log2(p))

def cross_entropy(p, q):
    """
    Calcula cross-entropy: H(p, q) = -Σ p(x) log(q(x))
    p: distribución verdadera
    q: distribución predicha
    """
    p = np.array(p)
    q = np.array(q)
    q = np.clip(q, 1e-10, 1)  # Evitar log(0)
    return -np.sum(p * np.log2(q))

def kl_divergence(p, q):
    """
    KL Divergence: D_KL(p||q) = H(p, q) - H(p)
    Mide cuánta información se pierde al aproximar p con q
    """
    return cross_entropy(p, q) - entropy(p)

# Ejemplo
p = np.array([0.5, 0.3, 0.2])  # Distribución verdadera
q = np.array([0.4, 0.4, 0.2])  # Distribución predicha

print(f"Entropy H(p): {entropy(p):.4f} bits")
print(f"Cross-Entropy H(p,q): {cross_entropy(p, q):.4f} bits")
print(f"KL Divergence D_KL(p||q): {kl_divergence(p, q):.4f} bits")

# Cross-entropy loss en clasificación
def cross_entropy_loss(y_true, y_pred):
    """
    Cross-entropy loss para clasificación multiclase
    y_true: one-hot encoded labels
    y_pred: predicted probabilities
    """
    epsilon = 1e-10
    y_pred = np.clip(y_pred, epsilon, 1 - epsilon)
    return -np.sum(y_true * np.log(y_pred))

# Ejemplo clasificación
y_true = np.array([0, 1, 0])  # Clase 1
y_pred = np.array([0.1, 0.7, 0.2])  # Predicción

loss = cross_entropy_loss(y_true, y_pred)
print(f"\nCross-Entropy Loss: {loss:.4f}")
```

### Ejemplo 5: Bootstrap y Confidence Intervals

```python
import numpy as np
from scipy import stats

def bootstrap_confidence_interval(data, statistic=np.mean, n_bootstrap=10000, confidence=0.95):
    """
    Calcula intervalo de confianza usando bootstrap
    
    data: datos originales
    statistic: función estadística (mean, median, etc.)
    n_bootstrap: número de muestras bootstrap
    confidence: nivel de confianza
    """
    bootstrap_stats = []
    n = len(data)
    
    for _ in range(n_bootstrap):
        # Muestreo con reemplazo
        sample = np.random.choice(data, size=n, replace=True)
        bootstrap_stats.append(statistic(sample))
    
    bootstrap_stats = np.array(bootstrap_stats)
    
    # Percentiles para intervalo de confianza
    alpha = 1 - confidence
    lower = np.percentile(bootstrap_stats, 100 * alpha / 2)
    upper = np.percentile(bootstrap_stats, 100 * (1 - alpha / 2))
    
    return {
        'statistic': statistic(data),
        'ci_lower': lower,
        'ci_upper': upper,
        'bootstrap_distribution': bootstrap_stats
    }

# Ejemplo
data = np.random.exponential(scale=2, size=100)

# Media con intervalo de confianza
mean_ci = bootstrap_confidence_interval(data, statistic=np.mean)
print(f"Mean: {mean_ci['statistic']:.3f}")
print(f"95% CI: [{mean_ci['ci_lower']:.3f}, {mean_ci['ci_upper']:.3f}]")

# Mediana con intervalo de confianza
median_ci = bootstrap_confidence_interval(data, statistic=np.median)
print(f"\nMedian: {median_ci['statistic']:.3f}")
print(f"95% CI: [{median_ci['ci_lower']:.3f}, {median_ci['ci_upper']:.3f}]")
```

## Proyectos Sugeridos

1. **Naive Bayes Classifier desde Cero**
   - Implementar sin sklearn
   - Entrenar en dataset de texto
   - Comparar con sklearn.naive_bayes

2. **A/B Testing Framework**
   - Crear herramienta para análisis A/B
   - Implementar múltiples tests estadísticos
   - Calcular poder estadístico y tamaño de muestra

3. **Bayesian Parameter Estimation**
   - Implementar inferencia bayesiana
   - Usar MCMC (Markov Chain Monte Carlo)
   - Biblioteca: PyMC3 o Stan

4. **Anomaly Detection**
   - Usar distribuciones de probabilidad
   - Identificar outliers
   - Comparar métodos estadísticos vs ML

5. **Confidence Interval Visualizer**
   - Crear app interactiva (Streamlit)
   - Visualizar diferentes métodos
   - Bootstrap, t-distribution, etc.

## Conceptos Avanzados

### Teorema del Límite Central

```python
import numpy as np
import matplotlib.pyplot as plt

def central_limit_theorem_demo(population_dist='uniform', sample_size=30, n_samples=1000):
    """
    Demuestra el Teorema del Límite Central
    """
    # Generar población
    if population_dist == 'uniform':
        population = np.random.uniform(0, 1, 100000)
    elif population_dist == 'exponential':
        population = np.random.exponential(2, 100000)
    else:
        population = np.random.normal(0, 1, 100000)
    
    # Tomar múltiples muestras y calcular medias
    sample_means = []
    for _ in range(n_samples):
        sample = np.random.choice(population, size=sample_size)
        sample_means.append(np.mean(sample))
    
    # Visualizar
    fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(14, 5))
    
    # Población original
    ax1.hist(population, bins=50, density=True, alpha=0.7, edgecolor='black')
    ax1.set_title(f'Population Distribution ({population_dist})')
    ax1.set_xlabel('Value')
    ax1.set_ylabel('Density')
    
    # Distribución de medias muestrales
    ax2.hist(sample_means, bins=50, density=True, alpha=0.7, edgecolor='black', label='Sample Means')
    
    # Superponer distribución normal teórica
    mu = np.mean(sample_means)
    sigma = np.std(sample_means)
    x = np.linspace(min(sample_means), max(sample_means), 100)
    ax2.plot(x, stats.norm.pdf(x, mu, sigma), 'r-', lw=2, label='Normal Fit')
    
    ax2.set_title(f'Distribution of Sample Means (n={sample_size})')
    ax2.set_xlabel('Sample Mean')
    ax2.set_ylabel('Density')
    ax2.legend()
    
    plt.tight_layout()
    plt.show()
    
    print(f"Mean of sample means: {mu:.4f}")
    print(f"Std of sample means: {sigma:.4f}")
    print(f"Theoretical std (σ/√n): {np.std(population)/np.sqrt(sample_size):.4f}")

# Ejecutar demo
central_limit_theorem_demo('exponential', sample_size=30)
```

### Bayesian Inference

```python
import numpy as np
import matplotlib.pyplot as plt

def bayesian_coin_flip(n_flips, n_heads):
    """
    Inferencia bayesiana para probabilidad de cara en moneda
    Prior: Beta(1, 1) = Uniform(0, 1)
    Likelihood: Binomial
    Posterior: Beta(heads + 1, tails + 1)
    """
    # Prior Beta(α, β)
    alpha_prior = 1
    beta_prior = 1
    
    # Actualizar con datos: Posterior Beta(α + heads, β + tails)
    alpha_post = alpha_prior + n_heads
    beta_post = beta_prior + (n_flips - n_heads)
    
    # Generar distribuciones
    p = np.linspace(0, 1, 1000)
    prior = stats.beta.pdf(p, alpha_prior, beta_prior)
    posterior = stats.beta.pdf(p, alpha_post, beta_post)
    
    # Likelihood (proporcional)
    likelihood = stats.binom.pmf(n_heads, n_flips, p)
    likelihood = likelihood / likelihood.max() * posterior.max()  # Normalizar para visualización
    
    # Visualizar
    plt.figure(figsize=(10, 6))
    plt.plot(p, prior, 'b--', label='Prior', lw=2)
    plt.plot(p, likelihood, 'g:', label='Likelihood', lw=2)
    plt.plot(p, posterior, 'r-', label='Posterior', lw=2)
    plt.xlabel('Probability of Heads (p)')
    plt.ylabel('Density')
    plt.title(f'Bayesian Inference: {n_heads} heads in {n_flips} flips')
    plt.legend()
    plt.grid(True, alpha=0.3)
    plt.show()
    
    # Estadísticas del posterior
    mean_post = alpha_post / (alpha_post + beta_post)
    mode_post = (alpha_post - 1) / (alpha_post + beta_post - 2) if alpha_post > 1 and beta_post > 1 else None
    
    # Intervalo de credibilidad al 95%
    ci_lower = stats.beta.ppf(0.025, alpha_post, beta_post)
    ci_upper = stats.beta.ppf(0.975, alpha_post, beta_post)
    
    print(f"Posterior mean: {mean_post:.4f}")
    if mode_post:
        print(f"Posterior mode (MAP): {mode_post:.4f}")
    print(f"95% Credible interval: [{ci_lower:.4f}, {ci_upper:.4f}]")
    
    return alpha_post, beta_post

# Ejemplo: 7 caras en 10 lanzamientos
bayesian_coin_flip(10, 7)
```

## Mejores Prácticas

1. **Visualiza Siempre**
   - Histogramas para entender distribuciones
   - QQ-plots para verificar normalidad
   - Box plots para outliers

2. **Verifica Supuestos**
   - Normalidad (Shapiro-Wilk, Kolmogorov-Smirnov)
   - Homocedasticidad
   - Independencia

3. **Múltiples Comparaciones**
   - Corrección de Bonferroni
   - False Discovery Rate (FDR)
   - Cuidado con p-hacking

4. **Tamaño de Muestra**
   - Calcula poder estadístico
   - Usa fórmulas o simulaciones
   - Más datos no siempre es mejor

5. **Interpretación Correcta**
   - P-valor NO es probabilidad de hipótesis
   - Correlación ≠ Causalidad
   - Significancia estadística ≠ significancia práctica

## Recursos Adicionales

### Interactive Tools

1. **Seeing Theory**
   - https://seeing-theory.brown.edu/
   - Visualizaciones interactivas de probabilidad

2. **Statistical Distributions**
   - https://distribution-explorer.github.io/
   - Explorador de distribuciones

3. **P-value Calculator**
   - https://www.socscistatistics.com/pvalues/

### Papers Clásicos

1. **"On the Mathematical Foundations of Learning"** - Cucker & Smale
   - https://www.ams.org/journals/bull/2002-39-01/S0273-0979-01-00923-5/

2. **"Elements of Information Theory"** - Cover & Thomas

### Blogs y Tutoriales

1. **Towards Data Science - Statistics**
   - https://towardsdatascience.com/tagged/statistics

2. **Count Bayesie**
   - https://www.countbayesie.com/
   - Bayesian statistics explicado

3. **Probability Course**
   - https://www.probabilitycourse.com/
   - Curso completo gratis

## Fórmulas Importantes

### Probabilidad
```
P(A ∪ B) = P(A) + P(B) - P(A ∩ B)
P(A ∩ B) = P(A|B) × P(B)
P(A|B) = P(B|A) × P(A) / P(B)  (Bayes)
```

### Esperanza y Varianza
```
E[X] = Σ x × P(X = x)  (discreto)
E[X] = ∫ x × f(x) dx  (continuo)
Var(X) = E[X²] - (E[X])²
Var(aX + b) = a² Var(X)
```

### Distribuciones Comunes
```
Bernoulli: P(X=1) = p, P(X=0) = 1-p
Binomial: P(X=k) = C(n,k) × p^k × (1-p)^(n-k)
Normal: f(x) = (1/σ√(2π)) × exp(-(x-μ)²/(2σ²))
```

## Próximos Pasos

Después de dominar Estadística y Probabilidad:
1. Aplicar en **Machine Learning Clásico**
2. Profundizar en **Inferencia Bayesiana**
3. Estudiar **Procesos Estocásticos**
4. Combinar con **Optimización**

## Checklist de Aprendizaje

- [ ] Entender probabilidad condicional y Bayes
- [ ] Dominar distribuciones comunes
- [ ] Calcular esperanza y varianza
- [ ] Realizar tests de hipótesis
- [ ] Implementar MLE y MAP
- [ ] Entender entropía y cross-entropy
- [ ] Aplicar bootstrap y intervalos de confianza
- [ ] Usar en proyecto ML real

---
**Tiempo estimado de estudio**: 4-6 semanas dedicando 2-3 horas diarias
