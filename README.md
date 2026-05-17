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
o

 ```bash
ollama pull nomic-embed-text
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

Ejecuta esto en tu terminal para asegurarte de tener la última versión que corrige estos fallos de compatibilidad:

```bash
docker pull ghcr.io/open-webui/open-webui:main
```
Detén y elimina el contenedor actual (no borra tus datos si usas volúmenes):

```bash
docker stop open-webui
docker rm open-webui
```
Verificar versión (El método que no falla)
```bash
docker logs open-webui | grep "v"
```
Como poner para que WebUI permita importar documentacion. Ejemplo de 5 horas

```bash
docker stop open-webui
docker rm open-webui

docker run -d -p 3000:8080 \
  --add-host=host.docker.internal:host-gateway \
  -e WEBUI_TIMEOUT=18000 \
  -e GUNICORN_TIMEOUT=18000 \
  -v open-webui:/app/backend/data \
  --name open-webui \
  ghcr.io/open-webui/open-webui:main
```

##  📂 Paso 5: Monitoreo de Hardware:


Monitoreo de Hardware: Para ver el rendimiento de tus 20 núcleos durante la inferencia:

```bash
sudo apt install -y nvtop && nvtop
```

```bash
btop
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



##  🚀 Despliegue del Motor de Extracción Documental (Docling)



Instalación del motor Docling-Serve:
```bash
pip install docling-serve
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
source docling_env/bin/activate
```
Entrar a mi sesion  de Docling una vez instalado
```bash
source /home/carlos/docling_env/bin/activate
```
Lanzamiento del servidor
```bash
docling-serve run --host 0.0.0.0 --port 5001
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

Necesitamos lanzar Docling con un tiempo de espera igual de largo  que WebUI. Usa este comando en tu terminal de docling_env:
```bash
# Primero matamos el proceso actual si sigue vivo
fuser -k 5001/tcp

# Lanzamos con timeout de 5 horas (18000 segundos)
docling-serve run --host 0.0.0.0 --port 5001 --timeout-keep-alive 18000
```
### Convertir archivo .PDF en archi .md en Docling

```bash
cd Documentos
```
Ejecutar el comando de conversión
```bash
docling --to md "BOE-A-2017-12902-consolidado (1).pdf"
```
Para buscar el archivo, dentro de Documentos
```bash
ls -lh *.md
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
  "do_ocr": true,
  "do_table_structure": true,
  "detailed_structure": true,
  "ocr_engine": "tesseract",
  "pdf_backend": "dlce"
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






