# Git - Control de Versiones

## Descripción

Git es el sistema de control de versiones distribuido más utilizado en el mundo. Es esencial para cualquier desarrollador, permitiendo rastrear cambios en el código, colaborar con otros desarrolladores, y mantener un historial completo del proyecto. Para AI/ML engineers, Git es fundamental para versionar código, experimentos y colaborar en proyectos.

## Conceptos Clave

### 1. **Repositorios**
- Repositorio local vs remoto
- Inicialización (git init)
- Clonación (git clone)
- Working directory, Staging area, Repository

### 2. **Commits**
- Snapshot del código
- Mensajes descriptivos
- Historial de cambios
- SHA hash

### 3. **Branches**
- Feature branches
- Main/master branch
- Branch strategies (Git Flow, GitHub Flow)
- Merge vs Rebase

### 4. **Colaboración**
- Push y pull
- Pull requests
- Code review
- Merge conflicts

### 5. **Remote Repositories**
- GitHub, GitLab, Bitbucket
- Origin
- Upstream
- Fork

## Recursos de Aprendizaje

### Documentación Oficial

1. **Pro Git Book** (Gratis)
   - https://git-scm.com/book/en/v2
   - Libro oficial completo

2. **Git Documentation**
   - https://git-scm.com/doc
   - Referencia completa

3. **GitHub Docs**
   - https://docs.github.com/en
   - Guías de GitHub

### Cursos Online

1. **Git & GitHub Crash Course - freeCodeCamp** (Gratis)
   - https://www.youtube.com/watch?v=RGOj5yH7evk
   - Tutorial completo

2. **Learn Git Branching** (Gratis, Interactivo)
   - https://learngitbranching.js.org/
   - Visualización interactiva

3. **GitHub Learning Lab** (Gratis)
   - https://lab.github.com/
   - Cursos prácticos

### Videos Recomendados

1. **Corey Schafer - Git Tutorial**
   - https://www.youtube.com/playlist?list=PL-osiE80TeTuRUfjRe54Eea17-YfnOOAx

2. **Traversy Media - Git & GitHub Crash Course**
   - https://www.youtube.com/watch?v=SWYqp7iY_Tc

## Instalación

```bash
# Linux (Ubuntu/Debian)
sudo apt-get install git

# macOS (con Homebrew)
brew install git

# Verificar instalación
git --version

# Configuración inicial
git config --global user.name "Tu Nombre"
git config --global user.email "tu@email.com"
git config --global init.defaultBranch main

# Ver configuración
git config --list
```

## Comandos Esenciales

### Básicos

```bash
# Iniciar repositorio
git init

# Clonar repositorio
git clone https://github.com/user/repo.git

# Ver estado
git status

# Añadir archivos al staging
git add file.txt
git add .  # Todos los archivos
git add *.py  # Todos los Python

# Commit
git commit -m "Mensaje descriptivo"
git commit -am "Add y commit en uno"  # Solo tracked files

# Ver historial
git log
git log --oneline
git log --graph --oneline --all

# Ver diferencias
git diff  # Working directory vs staging
git diff --staged  # Staging vs último commit
git diff commit1 commit2
```

### Branches

```bash
# Listar branches
git branch
git branch -a  # Incluir remotos

# Crear branch
git branch feature-name
git checkout -b feature-name  # Crear y cambiar
git switch -c feature-name  # Método moderno

# Cambiar de branch
git checkout feature-name
git switch feature-name  # Método moderno

# Merge
git checkout main
git merge feature-name

# Eliminar branch
git branch -d feature-name  # Solo si merged
git branch -D feature-name  # Forzar
```

### Remote

```bash
# Ver remotes
git remote -v

# Añadir remote
git remote add origin https://github.com/user/repo.git

# Push
git push origin main
git push -u origin feature-name  # Set upstream

# Pull
git pull origin main
git pull  # Si upstream configurado

# Fetch (sin merge)
git fetch origin

# Eliminar branch remoto
git push origin --delete feature-name
```

### Deshacer Cambios

```bash
# Descartar cambios en working directory
git checkout -- file.txt
git restore file.txt  # Método moderno

# Unstage archivos
git reset HEAD file.txt
git restore --staged file.txt  # Método moderno

# Revertir commit (crea nuevo commit)
git revert commit-hash

# Reset (cambiar HEAD)
git reset --soft HEAD~1  # Mantiene cambios en staging
git reset --mixed HEAD~1  # Mantiene cambios en working dir
git reset --hard HEAD~1  # Descarta todo

# Modificar último commit
git commit --amend -m "Nuevo mensaje"
git commit --amend --no-edit  # Sin cambiar mensaje
```

## Workflows Comunes

### Feature Branch Workflow

```bash
# 1. Actualizar main
git checkout main
git pull origin main

# 2. Crear feature branch
git checkout -b feature/nueva-funcionalidad

# 3. Trabajar y commit
git add .
git commit -m "Add nueva funcionalidad"

# 4. Push branch
git push -u origin feature/nueva-funcionalidad

# 5. Crear Pull Request en GitHub

# 6. Después de merge, limpiar
git checkout main
git pull origin main
git branch -d feature/nueva-funcionalidad
```

### Resolver Conflicts

```bash
# Cuando hay conflicto al hacer merge
git merge feature-branch
# CONFLICT en archivo.py

# 1. Ver archivos en conflicto
git status

# 2. Editar archivos manualmente
# Buscar <<<<<<< HEAD y resolver

# 3. Añadir archivos resueltos
git add archivo.py

# 4. Completar merge
git commit -m "Resolve merge conflict"
```

## Git para ML/AI Projects

### .gitignore

```gitignore
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
env/
venv/
.venv

# Jupyter Notebook
.ipynb_checkpoints
*.ipynb_checkpoints

# Data files
*.csv
*.h5
*.pkl
*.pickle
data/
datasets/

# Model files
*.pth
*.pt
*.ckpt
*.h5
*.pb
models/
checkpoints/

# Logs
*.log
logs/
runs/
wandb/

# IDE
.vscode/
.idea/
*.swp
*.swo

# OS
.DS_Store
Thumbs.db

# Environment
.env
.env.local
```

### Versionando Experiments

```bash
# Estructura recomendada
project/
├── .git/
├── .gitignore
├── README.md
├── requirements.txt
├── src/
│   ├── models/
│   ├── data/
│   └── utils/
├── notebooks/
├── tests/
└── experiments/
    ├── exp001_baseline/
    │   ├── config.yaml
    │   ├── train.py
    │   └── results.txt
    └── exp002_improved/

# Commit frecuente de código
git add src/
git commit -m "Add data preprocessing pipeline"

# NO versionar datos/modelos grandes
# Usar DVC o Git LFS
```

### Git LFS (Large File Storage)

```bash
# Instalar Git LFS
git lfs install

# Track archivos grandes
git lfs track "*.pth"
git lfs track "*.h5"

# Añadir .gitattributes
git add .gitattributes

# Usar normalmente
git add model.pth
git commit -m "Add trained model"
git push
```

## Mejores Prácticas

### 1. Commits Atómicos

```bash
# ✅ Bueno: un cambio por commit
git add feature.py
git commit -m "Add user authentication feature"

git add tests.py
git commit -m "Add tests for authentication"

# ❌ Malo: múltiples cambios no relacionados
git add .
git commit -m "Various changes"
```

### 2. Mensajes Descriptivos

```bash
# ✅ Bueno
git commit -m "Fix: Correct data preprocessing for null values"
git commit -m "Feature: Add LSTM model for time series"
git commit -m "Refactor: Extract feature engineering to separate module"

# ❌ Malo
git commit -m "Fix bug"
git commit -m "Update"
git commit -m "Changes"
```

### 3. Sincronizar Frecuentemente

```bash
# Antes de empezar a trabajar
git pull origin main

# Al final del día
git push origin feature-branch
```

### 4. Revisar Antes de Commit

```bash
# Ver qué se va a commitear
git diff --staged

# Ver status
git status
```

## Comandos Avanzados

### Stash

```bash
# Guardar cambios temporalmente
git stash
git stash save "WIP: feature incomplete"

# Ver stashes
git stash list

# Aplicar stash
git stash apply
git stash pop  # Apply y eliminar

# Eliminar stash
git stash drop stash@{0}
git stash clear  # Todos
```

### Cherry-pick

```bash
# Aplicar commit específico a branch actual
git cherry-pick commit-hash
```

### Rebase

```bash
# Rebase feature sobre main
git checkout feature-branch
git rebase main

# Interactive rebase (últimos 3 commits)
git rebase -i HEAD~3
```

### Bisect

```bash
# Encontrar commit que introdujo bug
git bisect start
git bisect bad  # Commit actual es malo
git bisect good commit-hash  # Commit bueno conocido

# Git hace binary search
# En cada paso, marcar:
git bisect good  # o
git bisect bad

# Al encontrar
git bisect reset
```

## GitHub Específico

### Pull Requests

```bash
# 1. Fork repositorio en GitHub

# 2. Clonar tu fork
git clone https://github.com/tu-usuario/repo.git

# 3. Añadir upstream
git remote add upstream https://github.com/original/repo.git

# 4. Crear branch
git checkout -b fix-bug

# 5. Hacer cambios y commit
git add .
git commit -m "Fix: Correct calculation bug"

# 6. Push a tu fork
git push origin fix-bug

# 7. Crear PR en GitHub web interface

# 8. Sincronizar con upstream
git fetch upstream
git checkout main
git merge upstream/main
```

### GitHub Actions (CI/CD)

```yaml
# .github/workflows/tests.yml
name: Tests

on: [push, pull_request]

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
      - name: Run tests
        run: |
          pytest tests/
```

## Proyectos Sugeridos

1. **Contribuir a Open Source**
   - Fork proyecto en GitHub
   - Crear feature/fix
   - Hacer Pull Request

2. **ML Experiment Tracking**
   - Versionar experimentos
   - Documentar resultados en commits
   - Usar branches por experimento

3. **Collaborative Project**
   - Trabajar con equipo
   - Code reviews
   - Resolver conflicts

4. **Personal Portfolio**
   - GitHub Pages
   - Documentar proyectos
   - README completos

## Recursos Adicionales

### Cheat Sheets

1. **GitHub Git Cheat Sheet**
   - https://education.github.com/git-cheat-sheet-education.pdf

2. **Atlassian Git Cheat Sheet**
   - https://www.atlassian.com/git/tutorials/atlassian-git-cheatsheet

### Herramientas GUI

1. **GitKraken** - Cliente visual potente
2. **GitHub Desktop** - Cliente simple de GitHub
3. **Sourcetree** - Cliente de Atlassian
4. **VS Code Git** - Integrado en VS Code

### Comunidades

1. **Stack Overflow - Git Tag**
   - https://stackoverflow.com/questions/tagged/git

2. **r/git**
   - https://www.reddit.com/r/git/

## Checklist de Aprendizaje

- [ ] Configurar Git localmente
- [ ] Crear repositorio y hacer commits
- [ ] Trabajar con branches
- [ ] Push/pull a repositorio remoto
- [ ] Resolver merge conflicts
- [ ] Hacer Pull Request
- [ ] Usar .gitignore correctamente
- [ ] Entender Git workflow
- [ ] Contribuir a proyecto open source
- [ ] Usar Git en proyecto ML

## Próximos Pasos

Después de dominar Git:
1. Aprender **GitHub Actions** para CI/CD
2. Explorar **DVC** para versionado de datos
3. Usar **pre-commit hooks** para calidad de código
4. Configurar **branch protection** en repos

---
**Tiempo estimado de estudio**: 1-2 semanas dedicando 1-2 horas diarias
