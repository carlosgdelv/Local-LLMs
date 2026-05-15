# Local-LLMs MVP: IA Soberana Privada con RAG

Guía técnica para el despliegue de una infraestructura de Inteligencia Artificial local, privada y optimizada para tareas administrativas sin dependencia de GPU externa.

## 🖥️ Personal PC Specifications


- **Processor (CPU)**  
  Intel Core i7 (14th Gen) with 20 cores (8 Performance + 12 Efficient), 3.4 GHz base and 5.6 GHz max turbo.

- **Memory (RAM)**  
  32GB DDR5 (2×16GB) at 6000 MHz with CL32 latency in dual-channel configuration.

- **Storage**  
  2TB SSD using PCIe 4.0 NVMe interface with Gen 4x4 for high-speed data transfer.

- **Power Supply (PSU)**  
  750W unit with 80 Plus Bronze certification for efficient energy usage.

- **Cooling**  
  240mm liquid cooler with dual fans.

- **Motherboard**  
  ATX board with B760 chipset, supports DDR5 and PCIe 4.0.



## 🛠️ Paso 1: Preparación del Sistema (Dependencias)
Actualiza el sistema e instala las herramientas de monitoreo y red necesarias:

```bash
sudo apt update && sudo apt install -y docker.io
```
Para que no tengas que escribir sudo cada vez que uses Docker (y que Open WebUI funcione bien), ejecuta esto:

```bash
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
```
IMPORTANTE: Después de este comando, cierra la terminal y ábrela de nuevo (o reinicia sesión) para que el cambio de grupo surta efecto.

## 🛠️ Paso 2: Instalación de Ollama (El Motor)

Instala el motor de inferencia con el script oficial:

```bash
curl -fsSL https://ollama.com/install.sh | sh
```
Descarga de Modelos Estratégicos

Ejecuta estos comandos para descargar los modelos optimizados para ejecución en CPU:

1. Razonamiento General (Llama 3.1 8B):
```bash
ollama pull qwen2.5:32b
```
2. Motor de Embeddings (Obligatorio para RAG):
 ```bash
ollama pull qwen3-embedding:8b
```
3. El "Motor" de búsqueda (Embeddings) - ¡OBLIGATORIO PARA RAG!:
Este modelo no chatea, se encarga de procesar los PDFs que subas a la plataforma.
 ```bash
ollama pull mxbai-embed-large
```


## ⚙️ Paso 3: Configuración de Red (Acceso Local/Privado)
Este paso "abre las puertas" de Ollama para que la interfaz web pueda entrar.
1. Crear regla de acceso:
 ```bash
sudo mkdir -p /etc/systemd/system/ollama.service.d/
echo -e "[Service]\nEnvironment=\"OLLAMA_HOST=0.0.0.0\"\nEnvironment=\"OLLAMA_ORIGINS=*\"" | sudo tee /etc/systemd/system/ollama.service.d/override.conf
```
2. Reiniciar el motor:Para que los cambios surtan efecto
```bash
sudo systemctl daemon-reload
sudo systemctl restart ollama
```

## ⚙️ Paso 4: Lanzar la Interfaz (Open WebUI)

Ahora que la red está lista, levantamos la interfaz visual con Docker.

```bash
docker run -d -p 3000:8080 --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data --name open-webui ghcr.io/open-webui/open-webui:main
```
Configuración Inicial:
Accede a `http://localhost:3000` y crea tu cuenta de Administrador.

Ve a Ajustes > Conexiones.

Verifica que la URL sea: `[http://host.docker.internal:11434](http://host.docker.internal:11434)`

Haz clic en el icono de Refrescar. El círculo verde confirma la conexión.





## Paso 5: Activando el RAG (Documentos Administrativos)

Configura el sistema para que pueda "leer" tus archivos. Accede a http://localhost:3000.

Ve a Settings (Ajustes) -> Documents.

En Embedding Model Engine, selecciona ollama.

En Embedding Model, escribe mxbai-embed-large y presiona descargar/guardar.


##  📂 Paso 6: Uso del Sistema

Chat con RAG: En el chat principal, arrastra un PDF. Escribe `#` seguido del nombre del archivo para preguntar sobre él.

Monitoreo de Hardware: Para ver el rendimiento de tus 20 núcleos durante la inferencia:

```bash
sudo apt install -y nvtop && nvtop
```




## 1. Configurar los permisos (OLLAMA_ORIGINS)
Abre una terminal y ejecuta el siguiente comando para editar la configuración del servicio:

```bash
sudo systemctl edit ollama.service
```
Se abrirá un editor de texto (probablemente `nano`). Baja hasta el final y pega estas líneas:

```bash
[Service]
Environment="OLLAMA_HOST=0.0.0.0"
Environment="OLLAMA_ORIGINS=*"
```
OLLAMA_HOST=0.0.0.0 le dice a Ollama que escuche en todas las interfaces de red.

OLLAMA_ORIGINS=* permite que la interfaz web se conecte sin bloqueos de seguridad.

Para guardar en Nano: Presiona `Ctrl + O`, luego `Enter` para confirmar, y `Ctrl + X` para salir.

1. Crea la carpeta y el archivo a la fuerza
Copia y pega este bloque de comandos entero en tu terminal:

```bash
sudo mkdir -p /etc/systemd/system/ollama.service.d/
echo -e "[Service]\nEnvironment=\"OLLAMA_HOST=0.0.0.0\"\nEnvironment=\"OLLAMA_ORIGINS=*\"" | sudo tee /etc/systemd/system/ollama.service.d/override.conf
```

##  Plantilla LLM (Cerebro)
K: 15
Temperatura: 0
top_k: 1
top_p: 0.1
presence_penalty: 0
frecuency_penalty: 0
num_ctx: 16384


##  Plantilla de Embeddings (Libreria)
Tamaño de los Fragmnentos: 2000
Superposición de Fragmentos: 200
Top K: 10


                                                                

Perfil ---- Adm ----- Documentos ---- Motor de Modelo de Incrustación [http://172.17.0.1:11434](http://172.17.0.1:11434) y Ollama --------  mxbai-embed-large:latest -------- http://localhost:11434/ 







