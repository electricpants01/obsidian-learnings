# Algoritmos de Machine Learning No Supervisado

## Descripción

El aprendizaje no supervisado trabaja con datos sin etiquetas, buscando patrones, estructuras ocultas y agrupaciones naturales en los datos. A diferencia del aprendizaje supervisado, no hay "respuestas correctas" - el objetivo es descubrir la estructura intrínseca de los datos. Las aplicaciones son vastas: segmentación de clientes, detección de anomalías, compresión de datos, sistemas de recomendación, y análisis exploratorio. Los principales tipos de aprendizaje no supervisado incluyen clustering (agrupar datos similares), reducción de dimensionalidad (comprimir datos manteniendo información), y detección de anomalías (identificar outliers). K-means es el algoritmo de clustering más popular por su simplicidad y eficiencia. PCA (Principal Component Analysis) domina la reducción dimensional lineal. DBSCAN excela en encontrar clusters de formas arbitrarias. t-SNE es excelente para visualización de alta dimensionalidad. Autoencoders combinan neural networks con unsupervised learning. Estos algoritmos son fundamentales para análisis exploratorio de datos, preprocesamiento para modelos supervisados, y cuando no hay labels disponibles.

## Conceptos Clave

### 1. **K-Means Clustering**
- Algoritmo de partición basado en centroides
- Minimiza suma de distancias al cuadrado
- Requiere especificar k (número de clusters)
- Sensible a inicialización
- Escalable y rápido

### 2. **Hierarchical Clustering**
- Crea dendrogramas de clusters anidados
- Agglomerative (bottom-up) vs Divisive (top-down)
- Linkage methods: single, complete, average, Ward
- No requiere especificar k a priori
- Computacionalmente costoso

### 3. **DBSCAN**
- Density-Based Spatial Clustering
- Encuentra clusters de forma arbitraria
- Parámetros: eps (radio) y min_samples
- Identifica outliers automáticamente
- No requiere especificar número de clusters

### 4. **PCA (Principal Component Analysis)**
- Reducción de dimensionalidad lineal
- Maximiza varianza explicada
- Eigenvalues y eigenvectors
- Decorrelaciona features
- Útil para visualización

### 5. **t-SNE**
- t-Distributed Stochastic Neighbor Embedding
- Reducción no lineal para visualización
- Preserva estructura local
- Parámetro perplexity crítico
- Computacionalmente intensivo

### 6. **Autoencoders**
- Neural networks para aprendizaje de representaciones
- Encoder comprime, decoder reconstruye
- Útil para reducción dimensional y anomaly detection
- Variational Autoencoders (VAE) para generación

### 7. **Gaussian Mixture Models (GMM)**
- Clustering probabilístico
- Asume mezcla de distribuciones gaussianas
- EM algorithm para training
- Soft clustering (probabilidades)
- Más flexible que K-means

## Recursos de Aprendizaje

### Documentación Oficial

1. **Scikit-learn Unsupervised Learning**
   - https://scikit-learn.org/stable/unsupervised_learning.html
   - Guía completa de clustering y reducción dimensional

2. **Scikit-learn Clustering**
   - https://scikit-learn.org/stable/modules/clustering.html
   - Comparación de algoritmos

### Cursos Online

1. **Unsupervised Learning - Coursera**
   - https://www.coursera.org/learn/unsupervised-learning-recommenders-reinforcement-learning
   - Por Andrew Ng, parte de ML Specialization

2. **Clustering & Retrieval - Coursera**
   - https://www.coursera.org/learn/ml-clustering-and-retrieval
   - Universidad de Washington

3. **Unsupervised Learning in Python - DataCamp**
   - https://www.datacamp.com/courses/unsupervised-learning-in-python

### Libros

1. **"Hands-On Unsupervised Learning"** - Ankur Patel
   - O'Reilly, enfoque práctico
   - Python y scikit-learn

2. **"Python Machine Learning"** - Sebastian Raschka
   - Capítulos sobre clustering y PCA
   - Teoría y práctica balanceadas

### Videos

1. **StatQuest - PCA, K-means, Hierarchical**
   - https://www.youtube.com/c/joshstarmer
   - Explicaciones visuales excelentes

2. **Normalized Nerd - Unsupervised Learning**
   - https://www.youtube.com/c/NormalizedNerd

## Ejemplos Prácticos

### Ejemplo 1: K-Means Clustering

```python
from sklearn.cluster import KMeans
from sklearn.datasets import make_blobs
import matplotlib.pyplot as plt

# Generar datos sintéticos
X, y_true = make_blobs(n_samples=300, centers=4, 
                       cluster_std=0.60, random_state=0)

# K-Means
kmeans = KMeans(n_clusters=4, random_state=42)
y_kmeans = kmeans.fit_predict(X)

# Visualizar
plt.scatter(X[:, 0], X[:, 1], c=y_kmeans, s=50, cmap='viridis')
centers = kmeans.cluster_centers_
plt.scatter(centers[:, 0], centers[:, 1], c='red', s=200, alpha=0.5, marker='X')
plt.title('K-Means Clustering')
plt.show()

# Elbow method para encontrar k óptimo
inertias = []
K_range = range(1, 11)

for k in K_range:
    kmeans = KMeans(n_clusters=k, random_state=42)
    kmeans.fit(X)
    inertias.append(kmeans.inertia_)

plt.plot(K_range, inertias, 'bo-')
plt.xlabel('Number of clusters (k)')
plt.ylabel('Inertia')
plt.title('Elbow Method')
plt.show()

# Silhouette score
from sklearn.metrics import silhouette_score

silhouette_scores = []
for k in range(2, 11):
    kmeans = KMeans(n_clusters=k, random_state=42)
    labels = kmeans.fit_predict(X)
    score = silhouette_score(X, labels)
    silhouette_scores.append(score)
    
best_k = silhouette_scores.index(max(silhouette_scores)) + 2
print(f"Best k: {best_k}")
```

### Ejemplo 2: Hierarchical Clustering

```python
from sklearn.cluster import AgglomerativeClustering
from scipy.cluster.hierarchy import dendrogram, linkage
from scipy.spatial.distance import pdist

# Hierarchical Clustering
hierarchical = AgglomerativeClustering(n_clusters=4, linkage='ward')
y_hierarchical = hierarchical.fit_predict(X)

# Visualizar resultados
plt.scatter(X[:, 0], X[:, 1], c=y_hierarchical, s=50, cmap='viridis')
plt.title('Hierarchical Clustering')
plt.show()

# Crear dendrograma
linkage_matrix = linkage(X, method='ward')

plt.figure(figsize=(12, 6))
dendrogram(linkage_matrix)
plt.title('Dendrogram')
plt.xlabel('Sample index')
plt.ylabel('Distance')
plt.show()

# Comparar diferentes linkage methods
linkage_methods = ['single', 'complete', 'average', 'ward']

fig, axes = plt.subplots(2, 2, figsize=(12, 10))
axes = axes.ravel()

for idx, method in enumerate(linkage_methods):
    hc = AgglomerativeClustering(n_clusters=4, linkage=method)
    labels = hc.fit_predict(X)
    
    axes[idx].scatter(X[:, 0], X[:, 1], c=labels, s=50, cmap='viridis')
    axes[idx].set_title(f'Linkage: {method}')

plt.tight_layout()
plt.show()
```

### Ejemplo 3: DBSCAN

```python
from sklearn.cluster import DBSCAN
from sklearn.preprocessing import StandardScaler

# Escalar datos (importante para DBSCAN)
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# DBSCAN
dbscan = DBSCAN(eps=0.3, min_samples=5)
y_dbscan = dbscan.fit_predict(X_scaled)

# Número de clusters (excluyendo ruido -1)
n_clusters = len(set(y_dbscan)) - (1 if -1 in y_dbscan else 0)
n_noise = list(y_dbscan).count(-1)

print(f"Estimated number of clusters: {n_clusters}")
print(f"Estimated number of noise points: {n_noise}")

# Visualizar
plt.scatter(X_scaled[:, 0], X_scaled[:, 1], c=y_dbscan, s=50, cmap='viridis')
plt.title(f'DBSCAN Clustering (eps={0.3}, min_samples={5})')
plt.show()

# Encontrar eps óptimo usando k-distance graph
from sklearn.neighbors import NearestNeighbors

neighbors = NearestNeighbors(n_neighbors=5)
neighbors_fit = neighbors.fit(X_scaled)
distances, indices = neighbors_fit.kneighbors(X_scaled)

distances = np.sort(distances[:, -1], axis=0)
plt.plot(distances)
plt.ylabel('5-NN distance')
plt.xlabel('Data points sorted by distance')
plt.title('K-distance Graph')
plt.show()
```

### Ejemplo 4: PCA

```python
from sklearn.decomposition import PCA
from sklearn.datasets import load_digits

# Cargar dataset de dígitos (64 dimensiones)
digits = load_digits()
X = digits.data
y = digits.target

# PCA
pca = PCA(n_components=2)
X_pca = pca.fit_transform(X)

# Varianza explicada
print(f"Explained variance ratio: {pca.explained_variance_ratio_}")
print(f"Total variance explained: {sum(pca.explained_variance_ratio_):.3f}")

# Visualizar
plt.figure(figsize=(10, 8))
scatter = plt.scatter(X_pca[:, 0], X_pca[:, 1], c=y, cmap='tab10', s=50, alpha=0.6)
plt.colorbar(scatter)
plt.xlabel('First Principal Component')
plt.ylabel('Second Principal Component')
plt.title('PCA of Digits Dataset')
plt.show()

# Scree plot - varianza vs componentes
pca_full = PCA()
pca_full.fit(X)

plt.plot(range(1, len(pca_full.explained_variance_ratio_) + 1),
         np.cumsum(pca_full.explained_variance_ratio_))
plt.xlabel('Number of Components')
plt.ylabel('Cumulative Explained Variance')
plt.title('Scree Plot')
plt.axhline(y=0.95, color='r', linestyle='--', label='95% variance')
plt.legend()
plt.show()

# Determinar componentes para 95% varianza
n_components_95 = np.argmax(np.cumsum(pca_full.explained_variance_ratio_) >= 0.95) + 1
print(f"Components needed for 95% variance: {n_components_95}")
```

### Ejemplo 5: t-SNE

```python
from sklearn.manifold import TSNE

# t-SNE (computacionalmente costoso)
tsne = TSNE(n_components=2, perplexity=30, random_state=42)
X_tsne = tsne.fit_transform(X[:1000])  # Usar subset para velocidad

# Visualizar
plt.figure(figsize=(10, 8))
scatter = plt.scatter(X_tsne[:, 0], X_tsne[:, 1], c=y[:1000], 
                     cmap='tab10', s=50, alpha=0.6)
plt.colorbar(scatter)
plt.title('t-SNE Visualization of Digits')
plt.show()

# Comparar diferentes perplexity values
fig, axes = plt.subplots(1, 3, figsize=(15, 5))
perplexities = [5, 30, 50]

for idx, perp in enumerate(perplexities):
    tsne = TSNE(n_components=2, perplexity=perp, random_state=42)
    X_embedded = tsne.fit_transform(X[:500])
    
    axes[idx].scatter(X_embedded[:, 0], X_embedded[:, 1], 
                     c=y[:500], cmap='tab10', s=30)
    axes[idx].set_title(f'Perplexity = {perp}')

plt.tight_layout()
plt.show()
```

### Ejemplo 6: Gaussian Mixture Model

```python
from sklearn.mixture import GaussianMixture

# GMM
gmm = GaussianMixture(n_components=4, random_state=42)
y_gmm = gmm.fit_predict(X)

# Probabilidades (soft clustering)
proba = gmm.predict_proba(X)
print(f"Sample probabilities:\n{proba[:5]}")

# Visualizar
plt.scatter(X[:, 0], X[:, 1], c=y_gmm, s=50, cmap='viridis')
plt.title('Gaussian Mixture Model')
plt.show()

# BIC/AIC para selección de modelo
n_components_range = range(1, 11)
bic_scores = []
aic_scores = []

for n_components in n_components_range:
    gmm = GaussianMixture(n_components=n_components, random_state=42)
    gmm.fit(X)
    bic_scores.append(gmm.bic(X))
    aic_scores.append(gmm.aic(X))

plt.plot(n_components_range, bic_scores, label='BIC', marker='o')
plt.plot(n_components_range, aic_scores, label='AIC', marker='s')
plt.xlabel('Number of components')
plt.ylabel('Information Criterion')
plt.legend()
plt.title('Model Selection: BIC and AIC')
plt.show()
```

### Ejemplo 7: Autoencoder Simple

```python
import torch
import torch.nn as nn

class Autoencoder(nn.Module):
    def __init__(self, input_dim, encoding_dim):
        super().__init__()
        
        # Encoder
        self.encoder = nn.Sequential(
            nn.Linear(input_dim, 128),
            nn.ReLU(),
            nn.Linear(128, encoding_dim)
        )
        
        # Decoder
        self.decoder = nn.Sequential(
            nn.Linear(encoding_dim, 128),
            nn.ReLU(),
            nn.Linear(128, input_dim)
        )
    
    def forward(self, x):
        encoded = self.encoder(x)
        decoded = self.decoder(encoded)
        return decoded

# Crear y entrenar
autoencoder = Autoencoder(input_dim=64, encoding_dim=10)
criterion = nn.MSELoss()
optimizer = torch.optim.Adam(autoencoder.parameters(), lr=0.001)

# Training loop (simplificado)
X_tensor = torch.FloatTensor(X)
for epoch in range(50):
    output = autoencoder(X_tensor)
    loss = criterion(output, X_tensor)
    
    optimizer.zero_grad()
    loss.backward()
    optimizer.step()
    
    if epoch % 10 == 0:
        print(f'Epoch [{epoch}/50], Loss: {loss.item():.4f}')

# Obtener encoding
with torch.no_grad():
    encoded = autoencoder.encoder(X_tensor)
    
# Visualizar encoding 2D
if encoded.shape[1] >= 2:
    plt.scatter(encoded[:, 0], encoded[:, 1], c=y, cmap='tab10')
    plt.title('Autoencoder Latent Space')
    plt.show()
```

## Comparación de Algoritmos

```python
import pandas as pd
from sklearn.metrics import silhouette_score, davies_bouldin_score

# Comparar algoritmos
algorithms = {
    'K-Means': KMeans(n_clusters=4, random_state=42),
    'Hierarchical': AgglomerativeClustering(n_clusters=4),
    'DBSCAN': DBSCAN(eps=0.5, min_samples=5),
    'GMM': GaussianMixture(n_components=4, random_state=42)
}

results = []

for name, algorithm in algorithms.items():
    labels = algorithm.fit_predict(X_scaled)
    
    # Métricas (si hay al menos 2 clusters)
    if len(set(labels)) > 1:
        sil_score = silhouette_score(X_scaled, labels)
        db_score = davies_bouldin_score(X_scaled, labels)
    else:
        sil_score = db_score = np.nan
    
    results.append({
        'Algorithm': name,
        'N Clusters': len(set(labels)) - (1 if -1 in labels else 0),
        'Silhouette': sil_score,
        'Davies-Bouldin': db_score
    })

results_df = pd.DataFrame(results)
print(results_df)
```

## Mejores Prácticas

### 1. Escalar Datos

```python
# ✅ Bueno
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)
kmeans = KMeans(n_clusters=3)
kmeans.fit(X_scaled)

# ❌ Malo - olvidar escalar
kmeans = KMeans(n_clusters=3)
kmeans.fit(X)  # Features con diferentes escalas
```

### 2. Validar Número de Clusters

```python
# ✅ Bueno - usar elbow method y silhouette
from sklearn.metrics import silhouette_score

for k in range(2, 11):
    kmeans = KMeans(n_clusters=k)
    labels = kmeans.fit_predict(X)
    print(f"k={k}, Silhouette: {silhouette_score(X, labels):.3f}")
```

### 3. Comparar Múltiples Algoritmos

```python
# ✅ Bueno - probar varios algoritmos
algorithms = [KMeans(3), DBSCAN(), AgglomerativeClustering(3)]
for alg in algorithms:
    labels = alg.fit_predict(X)
    # Evaluar y comparar
```

## Proyectos Sugeridos

1. **Customer Segmentation**
   - K-means en datos de clientes
   - RFM analysis
   - Perfiles de segmentos

2. **Anomaly Detection System**
   - DBSCAN o Isolation Forest
   - Detección de fraude
   - Monitoring en tiempo real

3. **Image Compression con PCA**
   - Reducir dimensionalidad de imágenes
   - Trade-off calidad vs tamaño
   - Reconstrucción

4. **Topic Modeling**
   - LDA o NMF
   - Clustering de documentos
   - Visualización con t-SNE

5. **Recommendation System**
   - Collaborative filtering
   - User clustering
   - Matrix factorization

## Checklist de Aprendizaje

- [ ] Implementar K-means desde cero
- [ ] Entender elbow method y silhouette
- [ ] Comparar hierarchical linkages
- [ ] Dominar parámetros de DBSCAN
- [ ] Aplicar PCA para visualización
- [ ] Usar t-SNE correctamente
- [ ] Implementar autoencoder básico
- [ ] Evaluar calidad de clustering
- [ ] Manejar diferentes escalas de features
- [ ] Completar proyecto de segmentación

## Próximos Pasos

1. Aprender **Anomaly Detection** avanzado
2. Estudiar **Dimensionality Reduction** avanzada (UMAP)
3. Explorar **Topic Modeling** (LDA, NMF)
4. Profundizar en **Autoencoders** y VAE
5. Aplicar en proyectos de **Recommender Systems**

---
**Tiempo estimado**: 3-4 semanas
