# Kubernetes - Container Orchestration

## Descripción

Kubernetes (K8s) orquesta containers Docker a escala: auto-scaling, load balancing, self-healing, rolling updates. Essential para ML en producción: manejar múltiples modelos, escalar según demanda, alta disponibilidad. Conceptos: Pods (grupo containers), Deployments (manage replicas), Services (networking), Ingress (routing externo). ML use: model serving con auto-scaling, batch jobs, distributed training. Managed K8s: GKE, EKS, AKS. Helm para package management. Kubeflow para ML workflows. Alternatives: Docker Swarm, Nomad.

## Conceptos

**Pod**: Unidad deployment mínima
**Deployment**: Manage replicas
**Service**: Load balancing interno
**ConfigMap/Secrets**: Configuration
**HPA**: Horizontal Pod Autoscaling

## Ejemplo Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ml-api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: ml-api
  template:
    metadata:
      labels:
        app: ml-api
    spec:
      containers:
      - name: api
        image: ml-api:v1
        ports:
        - containerPort: 8000
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
---
apiVersion: v1
kind: Service
metadata:
  name: ml-api-service
spec:
  selector:
    app: ml-api
  ports:
  - port: 80
    targetPort: 8000
  type: LoadBalancer
```

## Comandos

```bash
kubectl apply -f deployment.yaml
kubectl get pods
kubectl scale deployment ml-api --replicas=5
kubectl logs <pod-name>
```

---
**Tiempo**: 3-4 semanas
