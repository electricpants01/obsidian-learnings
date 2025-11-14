# CI/CD - Continuous Integration/Deployment

## Descripción

CI/CD automatiza testing y deployment de modelos ML. CI: merge code → auto test → build. CD: deploy automático a staging/prod. Benefits: catch errors early, deploy rápido, reproducible, rollback fácil. ML-specific: retrain triggers, model validation gates, A/B testing, monitoring post-deploy. Tools: GitHub Actions, GitLab CI, Jenkins, CircleCI. Workflow típico: commit → test → build Docker → push registry → deploy K8s. Critical: automated testing (unit, integration, model performance), versioning, blue-green deployments.

## Conceptos

**Pipeline**: Sequence automated steps
**Triggers**: Events que inician pipeline
**Artifacts**: Build outputs
**Environments**: dev/staging/prod
**Gates**: Approval/validation steps

## Ejemplo GitHub Actions

```yaml
name: ML Model CI/CD

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Set up Python
      uses: actions/setup-python@v2
      with:
        python-version: 3.9
    
    - name: Install dependencies
      run: |
        pip install -r requirements.txt
        pip install pytest
    
    - name: Run tests
      run: pytest tests/
    
    - name: Train and validate model
      run: python train.py --validate

  build-and-deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
    - uses: actions/checkout@v2
    
    - name: Build Docker image
      run: docker build -t ml-api:${{ github.sha }} .
    
    - name: Push to registry
      run: |
        echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
        docker push ml-api:${{ github.sha }}
    
    - name: Deploy to K8s
      run: |
        kubectl set image deployment/ml-api api=ml-api:${{ github.sha }}
```

## Model Validation

```python
# tests/test_model.py
import pytest
import joblib

def test_model_performance():
    model = joblib.load('models/model.pkl')
    X_test, y_test = load_test_data()
    accuracy = model.score(X_test, y_test)
    assert accuracy > 0.85, f"Model accuracy {accuracy} below threshold"

def test_model_predictions():
    model = joblib.load('models/model.pkl')
    sample_input = [[1.0, 2.0, 3.0]]
    prediction = model.predict(sample_input)
    assert prediction.shape == (1,)
```

---
**Tiempo**: 2-3 semanas
