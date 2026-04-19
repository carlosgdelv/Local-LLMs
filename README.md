# Local-LLMs

## 🛠️ Paso 0: Preparar el terreno (Dependencias)
Antes de nada, asegúrate de tener las herramientas básicas. Copia y pega esto:
```bash
sudo apt update && sudo apt install -y curl wget gpg ca-certificates
```

## 🛠️ Paso 1: Instalación de Ollama (El Motor)
Instala Ollama con un solo comando:
```bash
curl -fsSL https://ollama.com/install.sh | sh
```
Descarga los modelos que hemos analizado:
Para razonamiento general (Mossad-level logic):
```bash
ollama pull llama3.1:8b
```
Para programar (Tu experto en código):
```bash
ollama pull deepseek-coder-v2:16b-lite-instruct-q4_K_M
```

## 🌐 Paso 2: Interfaz Visual (Open WebUI)

¡OJO! Antes de correr el comando que tienes, necesitas Docker.


Para que se vea como ChatGPT, usaremos Docker. Es la forma más limpia de no "ensuciar" tu sistema.
Levanta el contenedor de Open WebUI:
```bash
docker run -d -p 3000:8080 --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data --name open-webui ghcr.io/open-webui/open-webui:main
```
Acceso: Abre tu navegador en `http://localhost:3000`. Crea tu cuenta local (recuerda: todo se queda en tu PC) y selecciona el modelo en el desplegable superior. El primer usuario que se registre será el Administrador.

## 💻 Paso 3: El Taller (VSCodium + Continue)
Cómo instalarlo (en Ubuntu/Debian)
Abre tu terminal y pega esto:

```bash
# 1. Añadir clave y repositorio
wget -qO - https://download.vscodium.com/debs/pub.gpg | gpg --dearmor | sudo dd of=/usr/share/keyrings/vscodium-archive-keyring.gpg
echo 'deb [ signed-by=/usr/share/keyrings/vscodium-archive-keyring.gpg ] https://download.vscodium.com/debs vscodium main' | sudo tee /etc/apt/sources.list.d/vscodium.list

# 2. Instalar
sudo apt update && sudo apt install -y codium
```
2. Instalar la extensión "Continue"
Una vez abras VSCodium:

En la barra lateral izquierda, haz clic en el icono de Extensiones (parecen 4 cuadraditos).

Busca: `Continue`.

Dale a Install. Aparecerá un icono de una letra "C" en la barra lateral.

3. Configurar Continue para que "hable" con Ollama
Ahora vamos a decirle a VSCodium que use el DeepSeek que ya tienes en Ollama.

Haz clic en el icono de Continue (la "C").

Abajo del todo verás un icono de un engranaje (Settings). Haz clic.

Se abrirá un archivo llamado `config.json`. Tienes que dejar la sección de `models` así (borra lo que haya y pega esto):

```bash
{
  "models": [
    {
      "title": "DeepSeek Coder 16B",
      "provider": "ollama",
      "model": "deepseek-coder-v2:16b-lite-instruct-q4_K_M"
    },
    {
      "title": "Llama 3.1 8B",
      "provider": "ollama",
      "model": "llama3.1:8b"
    }
  ],
  "tabAutocompleteModel": {
    "title": "Tab Autocomplete",
    "provider": "ollama",
    "model": "deepseek-coder-v2:16b-lite-instruct-q4_K_M"
  },
  "embeddingsProvider": {
    "provider": "ollama",
    "model": "llama3.1:8b"
  }
}
```
4. ¿Cómo se usa esto en el día a día?
Para crear un archivo: Abres un archivo (ej. `firewall_rules.sh`).

Para que la IA te ayude: Seleccionas un trozo de código y pulsas `Ctrl + L`. Se abre un chat a la derecha. Le dices: "Optimiza este script para OPNsense" y lo hace ahí mismo.

Autocompletado: Mientras escribes, la IA te sugerirá la siguiente línea (como el autocompletado de los móviles, pero inteligente). Pulsas `Tab` y listo.

Como estás en Linux, el problema es que el servicio de Ollama viene bloqueado por defecto para conexiones externas (como la de Docker, donde corre Open WebUI).

Sigue estos pasos exactos en tu terminal para solucionarlo:

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

2. Reiniciar el servicio
Para que Linux aplique los cambios, ejecuta estos dos comandos:

```bash
sudo systemctl daemon-reload
sudo systemctl restart ollama
```

3. Configurar la URL en Open WebUI
Ahora vuelve a tu navegador (la pantalla de tu captura) y haz lo siguiente:

En el apartado de API Ollama, asegúrate de que la URL sea:
`http://172.17.0.1:11434` o `http://host.docker.internal:11434`

Haz clic en el icono de refrescar (el círculo con la flecha) que está al lado de la URL.

Si se pone en verde, ¡ya está! El error de "Fallo al obtener los modelos" desaparecerá.

4. Cómo "meter" la IA (Descargar Llama 3)
Una vez que el círculo esté verde:

Verás un campo de texto que dice "Pull a model from Ollama.com" o un icono de una flecha hacia abajo.

Escribe: `llama3`

Dale al botón de descargar (el icono de la flecha o el "+" ).

Espera a que llegue al 100%. Cuando termine, ya podrás ir al chat principal y seleccionar Llama 3 en el menú desplegable de arriba.
