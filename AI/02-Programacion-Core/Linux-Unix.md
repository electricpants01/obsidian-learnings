# Linux/Unix - Comandos Básicos para AI/ML

## Descripción

El conocimiento de Linux/Unix es esencial para trabajar en AI/ML, ya que la mayoría de servidores, clusters de entrenamiento y plataformas cloud usan sistemas Unix. Dominar la línea de comandos permite automatizar tareas, gestionar recursos eficientemente y trabajar con entornos de producción.

## Conceptos Clave

### 1. **Sistema de Archivos**
- Estructura de directorios (/, /home, /usr, /etc)
- Rutas absolutas vs relativas
- Permisos (rwx)
- Links simbólicos

### 2. **Navegación y Gestión**
- cd, ls, pwd
- mkdir, rm, cp, mv
- find, locate
- Tree structure

### 3. **Manipulación de Archivos**
- cat, less, head, tail
- grep, sed, awk
- sort, uniq, wc
- Redirección (>, >>, |)

### 4. **Procesos**
- ps, top, htop
- kill, killall
- bg, fg, jobs
- nohup, screen, tmux

### 5. **Permisos**
- chmod, chown
- Users y groups
- sudo

### 6. **Networking**
- ssh, scp, rsync
- wget, curl
- ping, netstat

### 7. **Shell Scripting**
- Variables
- Loops y conditionals
- Functions
- Automatización

## Recursos de Aprendizaje

### Cursos Online

1. **Linux Journey** (Gratis)
   - https://linuxjourney.com/
   - Tutorial interactivo completo

2. **The Linux Command Line** (Gratis)
   - http://linuxcommand.org/tlcl.php
   - Libro completo online

3. **OverTheWire - Bandit** (Gratis)
   - https://overthewire.org/wargames/bandit/
   - Juego para aprender comandos

### Videos

1. **freeCodeCamp - Linux for Beginners**
   - https://www.youtube.com/watch?v=sWbUDq4S6Y8

2. **NetworkChuck - Linux**
   - https://www.youtube.com/c/NetworkChuck

### Documentación

1. **Man Pages**
   - `man comando`
   - https://man7.org/linux/man-pages/

2. **TLDR Pages**
   - https://tldr.sh/
   - Ejemplos prácticos

## Comandos Esenciales

### Navegación

```bash
# Directorio actual
pwd

# Listar archivos
ls
ls -l      # Formato largo
ls -la     # Incluir ocultos
ls -lh     # Human readable sizes
ls -ltr    # Ordenar por tiempo, reverso

# Cambiar directorio
cd /path/to/directory
cd ~       # Home directory
cd ..      # Directorio padre
cd -       # Directorio anterior

# Crear directorio
mkdir mydir
mkdir -p path/to/nested/dir  # Crear padres si no existen

# Remover
rm file.txt
rm -r directory/    # Recursivo
rm -rf directory/   # Forzar, sin preguntar
rmdir emptydir/     # Solo directorios vacíos
```

### Archivos

```bash
# Ver contenido
cat file.txt
less file.txt    # Paginado
head file.txt    # Primeras 10 líneas
head -n 20 file.txt
tail file.txt    # Últimas 10 líneas
tail -f log.txt  # Seguir archivo (logs)

# Copiar
cp source.txt dest.txt
cp -r dir1/ dir2/    # Recursivo

# Mover/renombrar
mv old.txt new.txt
mv file.txt /path/to/destination/

# Buscar archivos
find . -name "*.py"
find . -type f -name "model*"
find . -size +100M  # Archivos >100MB
locate filename

# Buscar en contenido
grep "pattern" file.txt
grep -r "import torch" .  # Recursivo
grep -i "error" log.txt  # Case insensitive
grep -n "def" script.py  # Con números de línea
```

### Procesos

```bash
# Ver procesos
ps aux
ps aux | grep python
top      # Interactivo
htop     # Mejor que top

# Matar proceso
kill PID
kill -9 PID  # Force kill
killall python  # Matar todos python

# Background/Foreground
python script.py &  # Ejecutar en background
jobs  # Ver jobs
fg %1  # Traer job 1 al foreground
bg %1  # Enviar job 1 al background

# Mantener proceso corriendo
nohup python train.py &
screen  # Terminal virtual
tmux    # Terminal multiplexer
```

### Permisos

```bash
# Ver permisos
ls -l file.txt
# -rw-r--r-- 1 user group size date file.txt
# - tipo | rw- owner | r-- group | r-- others

# Cambiar permisos
chmod 755 script.sh  # rwxr-xr-x
chmod +x script.sh   # Hacer ejecutable
chmod -R 755 dir/    # Recursivo

# Cambiar owner
chown user file.txt
chown user:group file.txt
chown -R user:group dir/

# Sudo
sudo command  # Ejecutar como root
sudo -i       # Shell como root
```

### Networking

```bash
# SSH
ssh user@hostname
ssh -i key.pem user@ip
ssh -L 8888:localhost:8888 user@server  # Port forwarding

# Copiar archivos
scp file.txt user@server:/path/
scp -r directory/ user@server:/path/
rsync -avz source/ dest/  # Sincronizar

# Descargar
wget https://url.com/file.zip
curl -O https://url.com/file.zip
curl -X POST -d "data" https://api.com

# Red
ping google.com
netstat -tuln  # Puertos abiertos
ifconfig       # Configuración de red
```

## Comandos para ML/AI

### Gestión de Datasets

```bash
# Contar líneas en dataset
wc -l dataset.csv

# Ver primeras filas
head -n 10 data.csv

# Dividir archivo grande
split -l 10000 large_file.txt chunk_

# Comprimir/descomprimir
tar -czf archive.tar.gz directory/
tar -xzf archive.tar.gz
gzip file.txt
gunzip file.txt.gz

# Ver tamaño de directorios
du -sh datasets/
du -h --max-depth=1
```

### Monitoreo de Recursos

```bash
# Uso de CPU/RAM
top
htop
free -h  # Memoria

# Uso de disco
df -h
du -sh *

# GPU (NVIDIA)
nvidia-smi
watch -n 1 nvidia-smi  # Actualizar cada segundo

# Procesos Python
ps aux | grep python
pgrep -f train.py
```

### Automatización

```bash
# Cron jobs (tareas programadas)
crontab -e
# Sintaxis: min hour day month weekday command
# */30 * * * * python /path/to/script.py  # Cada 30 min

# At (ejecutar una vez)
echo "python script.py" | at now + 2 hours

# Watch (repetir comando)
watch -n 60 python check_status.py
```

## Shell Scripting Básico

### Script Simple

```bash
#!/bin/bash
# train_model.sh

# Variables
MODEL_NAME="lstm_v1"
DATA_PATH="./data/train.csv"
EPOCHS=100

# Ejecutar entrenamiento
echo "Training $MODEL_NAME..."
python train.py \
    --data $DATA_PATH \
    --model $MODEL_NAME \
    --epochs $EPOCHS

# Verificar éxito
if [ $? -eq 0 ]; then
    echo "Training completed successfully"
else
    echo "Training failed"
    exit 1
fi
```

### Loops

```bash
#!/bin/bash

# For loop
for i in {1..5}; do
    echo "Iteration $i"
    python experiment.py --run $i
done

# While loop
count=1
while [ $count -le 5 ]; do
    echo "Count: $count"
    ((count++))
done

# Loop sobre archivos
for file in data/*.csv; do
    echo "Processing $file"
    python process.py --input $file
done
```

### Conditionals

```bash
#!/bin/bash

# If statement
if [ -f "model.pth" ]; then
    echo "Model exists"
    python evaluate.py --model model.pth
else
    echo "Model not found, training..."
    python train.py
fi

# Comparar números
if [ $EPOCHS -gt 100 ]; then
    echo "Many epochs"
fi

# AND/OR
if [ -f "data.csv" ] && [ -f "labels.csv" ]; then
    echo "Both files exist"
fi
```

### Functions

```bash
#!/bin/bash

# Función
train_model() {
    local model=$1
    local data=$2
    echo "Training $model with $data"
    python train.py --model $model --data $data
}

# Usar función
train_model "lstm" "train.csv"
train_model "gru" "train.csv"
```

## Pipelines y Redirección

```bash
# Output a archivo
ls -l > filelist.txt
echo "Log entry" >> log.txt  # Append

# Pipe
cat data.csv | grep "positive" | wc -l
ps aux | grep python | awk '{print $2}'

# Stderr
python script.py 2> errors.log
python script.py > output.log 2>&1  # Stdout y stderr

# tee (ver y guardar)
python train.py | tee train.log
```

## Configuración de Entorno

### .bashrc / .zshrc

```bash
# ~/.bashrc

# Alias útiles
alias ll='ls -lah'
alias gs='git status'
alias gp='git pull'
alias ..='cd ..'

# Variables de entorno
export PATH=$PATH:/path/to/custom/bin
export PYTHONPATH=$PYTHONPATH:/path/to/modules

# Conda
export PATH="$HOME/miniconda3/bin:$PATH"

# CUDA
export CUDA_HOME=/usr/local/cuda
export PATH=$CUDA_HOME/bin:$PATH
export LD_LIBRARY_PATH=$CUDA_HOME/lib64:$LD_LIBRARY_PATH
```

### SSH Config

```bash
# ~/.ssh/config

Host gpu-server
    HostName 192.168.1.100
    User myuser
    Port 22
    IdentityFile ~/.ssh/id_rsa

Host aws
    HostName ec2-instance.compute.amazonaws.com
    User ubuntu
    IdentityFile ~/.ssh/aws-key.pem
```

## Mejores Prácticas

### 1. Usar Tab Completion

```bash
# Presionar Tab para autocompletar
cd /home/us[Tab]
python train[Tab]
```

### 2. Historial de Comandos

```bash
# Ver historial
history

# Buscar en historial
Ctrl+R  # Luego escribir

# Ejecutar comando anterior
!!
!123  # Comando número 123

# Último argumento
cd /long/path/to/dir
ls !$  # usa /long/path/to/dir
```

### 3. Múltiples Comandos

```bash
# Secuencial (;)
mkdir project; cd project; git init

# Solo si anterior exitoso (&&)
python preprocess.py && python train.py

# Si anterior falla (||)
python train.py || echo "Training failed"
```

## Recursos Adicionales

### Cheat Sheets

1. **Linux Command Cheat Sheet**
   - https://www.linuxtrainingacademy.com/linux-commands-cheat-sheet/

2. **TLDR Pages**
   - https://github.com/tldr-pages/tldr

### Herramientas Útiles

1. **tmux** - Terminal multiplexer
2. **htop** - Process monitor
3. **ncdu** - Disk usage analyzer
4. **ag/ripgrep** - Búsqueda rápida

### Comandos Modernos

```bash
# Alternativas modernas a comandos clásicos
bat    # Mejor que cat
exa    # Mejor que ls
fd     # Mejor que find
rg     # ripgrep, mejor que grep
htop   # Mejor que top
```

## Proyectos Sugeridos

1. **Automatizar Pipeline de ML**
   - Script bash que ejecute todo el pipeline
   - Logs y error handling

2. **Monitoreo de Experimentos**
   - Script que verifique status de entrenamientos
   - Notificaciones cuando terminen

3. **Configurar Servidor GPU**
   - SSH, environment setup
   - Instalar CUDA, drivers
   - Configurar Jupyter remoto

## Checklist de Aprendizaje

- [ ] Navegar sistema de archivos
- [ ] Manipular archivos y directorios
- [ ] Usar grep y pipes
- [ ] Gestionar procesos
- [ ] Entender permisos
- [ ] SSH a servidores remotos
- [ ] Escribir shell scripts básicos
- [ ] Usar tmux/screen
- [ ] Monitorear recursos (CPU/GPU)
- [ ] Automatizar tareas con cron

## Próximos Pasos

1. Profundizar en **Bash scripting avanzado**
2. Aprender **Docker** para containerización
3. Configurar **servidores remotos** para ML
4. Explorar **Vim** para edición en terminal

---
**Tiempo estimado de estudio**: 1-2 semanas dedicando 1 hora diaria
