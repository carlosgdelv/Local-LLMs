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

#### Optimizaciones
1. Forzar CPU en Modo Rendimiento (Máxima Frecuencia)

```bash
# 1. Instalar la herramienta de control de frecuencia
sudo apt update && sudo apt install -y cpufrequtils

# 2. Configurar el gobernador en rendimiento máximo permanente
echo "GOVERNOR=\"performance\"" | sudo tee /etc/default/cpufrequtils

# 3. Reiniciar el servicio para aplicar los cambios
sudo systemctl restart cpufrequtils
```
3. Optimización de la Memoria RAM (Evitar el uso de disco)

```bash
# 1. Cambiar el valor de intercambio (swappiness) a 1 temporalmente
sudo sysctl vm.swappiness=1

# 2. Hacerlo permanente para que no se borre al reiniciar:
# Abre el archivo de configuración del sistema:
sudo nano /etc/sysctl.conf

# 3. Baja hasta el final del archivo con las flechas y añade esta línea:
vm.swappiness=1

# 4. Guarda con Ctrl+O (luego Enter) y sal con Ctrl+X.
```


2: Configuración de la BIOS (MSI Click BIOS 5)

Pantalla: Overclocking \ Advanced CPU Configuration
`Intel C-State` ──> Cambiar a: `[Disabled]` (Evita que los núcleos se vayan a dormir).

`C1E Support` ──> Cambiar a: `[Disabled]` (Si te aparece disponible tras desactivar el C-State).

`Long Duration Maintained(s)` ──> Cambiar el valor a: `128` (Estira el tiempo que la CPU puede mantener el modo Turbo sin caparse).

`CPU Lite Load` ──> Cambiar de Mode 16 a: `Mode 12` (Baja el voltaje excesivo de fábrica para reducir la temperatura de la CPU y evitar el ahogamiento térmico).

Botón Principal (Esquina superior izquierda de la BIOS)
`XMP (Extreme Memory Profile)` ──> Hacer clic en el botón `Profile 1` para ponerlo en Rojo/Activado. (Pasa tu RAM DDR5 de los 4800 MHz capados de fábrica a los 6000 MHz reales de tu hardware).












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

Version 1.0 sin compilación. Instala el motor de inferencia con el script oficial:

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

### 🛑 Paso 0: Purgar Ollama (Liberar el puerto 11434)

Si tienes Ollama corriendo como servicio de Systemd, se quedará con el puerto. Hay que tumbarlo y desactivarlo:
 ```bash
# 1. Detener y deshabilitar el servicio del sistema
sudo systemctl stop ollama
sudo systemctl disable ollama

# 2. Eliminar el archivo del servicio y recargar el demonio de Systemd
sudo rm -f /etc/systemd/system/ollama.service
sudo systemctl daemon-reload

# 3. Eliminar el ejecutable de Ollama
sudo rm -f $(which ollama)

# 4. Borrar los pesos de modelos y metadatos residuales
sudo rm -rf /usr/share/ollama
sudo rm -rf ~/.ollama

# 5. Eliminar el usuario y grupo operativo de Ollama
sudo userdel ollama
sudo groupdel ollama
```
### 🛠️ Paso 1: Instalar las Herramientas de Compilación en Ubuntu
Necesitamos los compiladores de C/C++ optimizados de GNU y la herramienta CMake:
 ```bash
sudo apt update
sudo apt install -y build-essential cmake git curl wget
```

### 🏗️ Paso 2: Clonar y Compilar llama.cpp de Forma Nativa
 Clonar el repositorio oficial
 
 ```bash
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp

# 2. Crear un directorio de compilación limpio
mkdir build && cd build

# 3. Configurar CMake inyectando las banderas de optimización nativa y OpenMP para multihilo
cmake .. -DCMAKE_BUILD_TYPE=Release \
         -DCMAKE_C_FLAGS="-march=native -O3" \
         -DCMAKE_CXX_FLAGS="-march=native -O3" \
         -DLLAMA_OPENMP=ON

# 4. Compilar usando todos los núcleos de tu procesador (tardará un par de minutos)
cmake --build . --config Release --parallel $(nproc)
```
Al terminar, dentro de la carpeta llama.cpp/build/bin/ tendrás el Santo Grial: el binario ejecutable llama-server optimizado al milímetro para tu silicio.

### 📂 Paso 3: Descargar el Modelo en Formato GGUF Manualmente

Volvemos a la raíz o a tu carpeta de Documentos
```bash
cd ~/Documentos
mkdir -p modelos && cd modelos

1. Razonamiento General:

```bash
cd ~/modelos
```

```bash
wget https://huggingface.co/bartowski/Qwen2.5-7B-Instruct-GGUF/resolve/main/Qwen2.5-7B-Instruct-Q4_K_M.gguf
```
o

 ```bash
wget https://huggingface.co/bartowski/microsoft_Phi-4-mini-instruct-GGUF/resolve/main/microsoft_Phi-4-mini-instruct-Q4_K_M.gguf
```

2. Motor de Embeddings (Obligatorio para RAG):

 ```bash
nomic-ai/nomic-embed-text-v1.5
```
Modelo de Reclasificación
 ```bash
BAAI/bge-reranker-v2-m3
```

### 🚀 Paso 4: Lanzar el Servidor Nativo de Inferencia

# Ejecuta esto desde la carpeta donde compilaste el binario
```bash
cd ~/llama.cpp/build/bin

# Lanzar el servidor
./llama-server -m ~/modelos/microsoft_Phi-4-mini-instruct-Q4_K_M.gguf \
               -c 16384 \
               -t 6 \
               --host 0.0.0.0 \
               --port 11434 \
               --embedding
```
o
```bash
./llama-server -m ~/modelos/Qwen2.5-7B-Instruct-Q4_K_M.gguf \
               -c 16384 \
               -t 6 \
               --host 0.0.0.0 \
               --port 11434 \
               --embedding
```


Deja esta terminal abierta. Para verificar que responde, en otra terminal:

```bash
curl http://localhost:11434/v1/models
```
Comando de apagado limpio
```bash
pkill llama-server
```

### ⚙️ Paso 5: Lanzar la Interfaz (Open WebUI en Modo OpenAI API)

```bash
sudo docker stop open-webui
sudo docker rm open-webui

sudo docker pull ghcr.io/open-webui/open-webui:main

sudo docker run -d -p 3000:8080 \
  --add-host=host.docker.internal:host-gateway \
  -e OPENAI_API_BASE_URL=http://host.docker.internal:11434/v1 \
  -e OPENAI_API_KEY=not_needed \
  -e WEBUI_TIMEOUT=36000 \
  -e GUNICORN_TIMEOUT=36000 \
  -e AIOHTTP_CLIENT_TIMEOUT=36000 \
  -e AIOHTTP_CLIENT_TIMEOUT_OPENAI_MODEL_LIST=36000 \
  -v open-webui:/app/backend/data \
  --name open-webui \
  ghcr.io/open-webui/open-webui:main
```
### ✅ Paso 6: Configuración Inicial en Pantalla
- Abre tu navegador e ingresa a http://localhost:3000 y crea tu cuenta de Administrador.
- Ve a Ajustes > Conexiones, desactiva Ollama por completo.
- En el apartado OpenAI API, pon la URL: http://host.docker.internal:11434/v1 y haz clic en Refrescar (círculo verde = OK).
- Ve a Ajustes > Documentos, y en Embedding Engine selecciona "WebUI (Transformers)" para procesar los embeddings del RAG de forma local interna y ahorrar RAM
- Clic en Refrescar → círculo verde = conexión OK




##  📂 Paso 5: Monitoreo de Hardware:


Monitoreo de Hardware: Para ver el rendimiento de tus 20 núcleos durante la inferencia:

```bash
sudo apt install -y nvtop && nvtop
```

```bash
btop
```
Análisis por un posible Throttling Térmico

```bash
sudo apt update
sudo apt install lm-sensors
```

```bash
# Pon esto en una terminal siempre que uses llama-server
watch -n 1 "sensors | grep 'Package' && grep MHz /proc/cpuinfo | sort -t: -k2 -rn | head -3"
```
Te muestra simultáneamente la temperatura Y los 3 núcleos más rápidos. Si la temperatura está por debajo de 80°C y hay núcleos a 4000+ MHz → todo perfecto. Si la temperatura supera 85°C y todos están a 800 → throttling.




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



##  🚀 Despliegue del Motor de Extracción Documental (Docling)



 Instalación de dependencias del sistema

 Actualiza los repositorios e instala el gestor de paquetes de Python y las herramientas de entornos virtuales necesarias en Ubuntu.
```bash
sudo apt update && sudo apt install -y python3-pip python3-venv pstree psmisc
```

El problema es que el paquete pstree no existe con ese nombre en Ubuntu (en realidad viene dentro de otro paquete llamado psmisc que ya habías puesto en la lista).

```bash
sudo apt install -y python3-pip python3-venv psmisc
```

Instalación de dependencias del sistema:

```bash
sudo apt install python3.12-venv
```
Creación del entorno virtual soberano:
```bash
python3 -m venv docling_env
```
Activar el entorno
```bash
source docling_env/bin/activate
```
Instalar Docling-Serve

```bash
pip install docling-serve
```
Entrar a mi sesion  de Docling una vez instalado
```bash
source /home/carlos/docling_env/bin/activate
```
```bash
# Primero matamos el proceso actual si sigue vivo
fuser -k 5001/tcp
```
Lanzamiento del servido

```bash
docling-serve run --host 0.0.0.0 --port 5001 --timeout-keep-alive 18000
```
Arranca el microservicio en el puerto 5001, permitiendo que tu Asistente CAST procese documentos de forma local y privada.

Obtén tu IP local, en una nueva termianl:
```bash
hostname -I | awk '{print $1}'
```
Docling Server URL: `http://xxx:5001`

Version del Docling

```bash
source /home/carlos/docling_env/bin/activate && pip show docling-serve
```
http://host.docker.internal:5001

Parámetros
```bash
{
  "do_ocr": false,
  "do_table_structure": false,
  "pdf_backend": "docling_parse"
}
```

# Convertir archivo .PDF en archi .md en Docling

🥇 Opción 1 — PDF consolidado directo "(LA MEJOR)"

Descarga el PDF oficial y súbelo directamente a Open WebUI sin pasar por Docling ni conversiones intermedias:
```bash
wget https://www.boe.es/buscar/pdf/2017/BOE-A-2017-12902-consolidado.pdf \
     -O ~/Documentos/LCSP_2017_consolidada.pdf
```
/home/carlos/Documentos/LCSP_2017_consolidada.pdf
Luego en Open WebUI: Espacio de Trabajo → Conocimiento → Nueva Colección → Subir archivo

🥈 Opción 2 — Conversión a Markdown con Docling primero
Conviertes el PDF a Markdown con Docling y subes el .md:

```bash
source /home/carlos/docling_env/bin/activate
```

Solución — Lanza Docling desactivando OCR explícitamente


```bash
docling --to md \
        --no-ocr \
        --pdf-backend docling_parse \
        "LCSP_2017_consolidada.pdf"
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

Motor para la Extracción de Contenido (Convierte PDFs en Markdown estructural): Docling
Parámetros:
{
  "do_ocr": false,
  "do_table_structure": false,
  "pdf_backend": "docling_parse"
}

- do_ocr: Lee texto dentro de imágenes; activado permite procesar escaneos, desactivado ignora fotos y ahorra tiempo.
- do_table_structure: Identifica filas y columnas; activado mantiene los datos de tablas ordenados, desactivado mezcla los números como texto plano.
- detailed_structure: Detecta títulos y apartados; activado crea un índice real para la IA, desactivado trata la ley como un muro de texto.
- ocr_engine (tesseract): Es el software de visión local; garantiza que el reconocimiento de letras sea 100% privado en tu PC.
- pdf_backend (dlce): Es el motor interno de IBM; optimiza la lectura de documentos complejos para que no se salte ninguna página.

### Fragmentación e Incrustación

- Tamaño de Fragmentos (1500): Si lo subes la IA tiene más contexto, si lo bajas las respuestas son más atómicas y precisas.
  
- Superposición (300): Si la subes evitas cortar artículos por la mitad entre trozos, si la bajas ahorras memoria pero pierdes continuidad.
  
- Lote de Incrustación (2): Si lo subes procesas documentos más rápido (usa más RAM), si lo bajas es más lento pero más estable.
  
- Peticiones Concurrentes (4): Si lo subes aprovechas tus 20 núcleos para indexar leyes en segundos, si lo bajas el PC irá más relajado.

Perfil ---- Adm ----- Documentos ---- Motor de Modelo de Incrustación [http://172.17.0.1:11434](http://172.17.0.1:11434) y Ollama --------  qwen3-embedding:8b -------- http://localhost:11434/ 

### Búsqueda y Reclasificación

- Texto Enriquecido: Añade etiquetas de contexto a cada trozo; activada ayuda a la IA a saber siempre en qué ley está, desactivada ahorra tokens.

- Reranker (BGE-M3): Un segundo cerebro que ordena los resultados; es el filtro final que asegura que lo más importante aparezca primero.

- Top K (10): Número de fragmentos iniciales que busca; si lo subes tienes más "materia prima", si lo bajas vas más rápido.

- Top K Reclasificador (7): Fragmentos finales que lee la IA; si lo subes das más info al modelo, si lo bajas evitas que se confunda con ruido.

- Umbral de Relevancia (0.2): Filtro de seguridad; si lo subes la IA solo usa fragmentos perfectos, si lo bajas permite usar "dudas" razonables.

- Ponderación BM25 (0.5): Equilibrio entre significado y palabra exacta; si la subes priorizas el término literal (ej: "Art. 159"), si la bajas priorizas el concepto.






