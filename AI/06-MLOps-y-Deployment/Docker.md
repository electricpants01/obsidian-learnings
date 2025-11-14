# Docker - Containerización

## Descripción

Docker containeriza aplicaciones para deployment consistente. Empaqueta código, dependencias, runtime en containers portables que corren igual en dev/prod. Ventajas vs VMs: lightweight (comparten OS kernel), start rápido (segundos), eficiente recursos. Conceptos: Images (template read-only), Containers (instancia running), Dockerfile (build instructions), Docker Hub (registry). Uso ML: reproducibilidad (freeze dependencies), aislamiento (sin conflictos), escalabilidad (orchestration con Kubernetes), CI/CD pipelines. Comandos críticos: `docker build`, `docker run`, `docker push/pull`. Multi-stage builds reducen tamaño. Docker Compose para multi-container. Essential para MLOps moderno.

## Conceptos

**Image**: Template con app + dependencies
**Container**: Running instance de image
**Dockerfile**: Build script
**Registry**: Docker Hub, ECR, GCR para storage
**Volumes**: Persist data
**Networks**: Container communication

## Ejemplos

### Dockerfile para FastAPI ML App

```dockerfile
# Multi-stage build
FROM python:3.9-slim as builder

WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

FROM python:3.9-slim
WORKDIR /app

COPY --from=builder /usr/local/lib/python3.9/site-packages /usr/local/lib/python3.9/site-packages
COPY . .

EXPOSE 8000
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Docker Compose

```yaml
version: '3.8'
services:
  api:
    build: .
    ports:
      - "8000:8000"
    volumes:
      - ./models:/app/models
    environment:
      - MODEL_PATH=/app/models/model.pkl
  
  redis:
    image: redis:alpine
    ports:
      - "6379:6379"
```

### Comandos Básicos

```bash
# Build image
docker build -t ml-api:v1 .

# Run container
docker run -p 8000:8000 ml-api:v1

# List containers
docker ps

# Stop container
docker stop <container_id>

# Push to registry
docker tag ml-api:v1 username/ml-api:v1
docker push username/ml-api:v1

# Docker Compose
docker-compose up -d
docker-compose down
```

## Mejores Prácticas

```dockerfile
# ✅ Use .dockerignore
# ✅ Multi-stage builds
# ✅ Minimize layers
# ✅ Use official base images
# ✅ Don't run as root
# ❌ No guardar secrets en image
```

---
**Tiempo**: 1-2 semanas
