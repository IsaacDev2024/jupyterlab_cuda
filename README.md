# JupyterLab + CUDA Docker

Entorno de JupyterLab con soporte CUDA para usar la GPU desde el navegador.

**GPU requerida:** NVIDIA (driver >= 525)
**CUDA:** 12.6 | **Python:** 3.11 | **JupyterLab:** 4.x

---

## Requisitos previos

### Windows
1. Instalar [Docker Desktop](https://www.docker.com/products/docker-desktop/)
2. Habilitar **WSL2** como backend (Settings > General > Use WSL 2)
3. En Settings > Resources > WSL Integration, activar tu distro WSL2
4. Instalar drivers NVIDIA para Windows (los mismos drivers de escritorio funcionan con WSL2)

> El soporte de GPU en Docker Desktop para Windows requiere WSL2. No funciona con Hyper-V backend.

### Linux
1. Instalar Docker Engine: https://docs.docker.com/engine/install/
2. Instalar **NVIDIA Container Toolkit**:
   ```bash
   curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
   curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
     sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
     sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
   sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit
   sudo nvidia-ctk runtime configure --runtime=docker
   sudo systemctl restart docker
   ```

### macOS
> **Las GPUs NVIDIA no son compatibles con macOS.** Apple Silicon (M1/M2/M3) usa Metal, no CUDA.
> El contenedor puede correr sin GPU (solo CPU) eliminando la seccion `deploy` del `docker-compose.yml`.

---

## Instalacion y uso

### 1. Clonar el repositorio
```bash
git clone <url-del-repo>
cd jupiter_cuda
```

### 2. Construir la imagen
```bash
docker compose build
```
La primera vez descarga la imagen base de CUDA (~5 GB) e instala los paquetes. Puede tardar varios minutos.

### 3. Iniciar JupyterLab
```bash
docker compose up
```

Para correr en segundo plano:
```bash
docker compose up -d
```

### 4. Acceder desde el navegador
Abrir: **http://localhost:8888**

No se requiere token ni contrasena.

---

## Detener el contenedor

```bash
docker compose down
```

---

## Estructura del proyecto

```
jupiter_cuda/
├── Dockerfile           # Imagen CUDA + Python + JupyterLab
├── docker-compose.yml   # Configuracion del servicio con GPU
├── requirements.txt     # Paquetes Python (PyTorch, numpy, etc.)
└── notebooks/           # Directorio montado en el contenedor
```

Los notebooks guardados en `notebooks/` persisten aunque el contenedor se detenga.

---

## Verificar que la GPU esta disponible

Dentro de JupyterLab, crear un notebook y ejecutar:

```python
import torch
print(torch.cuda.is_available())       # True si la GPU esta activa
print(torch.cuda.get_device_name(0))   # Nombre de la GPU
```

---

## Paquetes incluidos

| Categoria      | Paquetes                                      |
|----------------|-----------------------------------------------|
| Deep Learning  | PyTorch, torchvision, torchaudio (CUDA 12.6)  |
| Data Science   | NumPy, Pandas, Scikit-learn, SciPy            |
| Visualizacion  | Matplotlib, Seaborn, Pillow                   |
| Utilidades     | tqdm, PyYAML, requests, ipywidgets            |

Para agregar paquetes, editá `requirements.txt` y reconstrui la imagen con `docker compose build`.
