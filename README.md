# Local-LLMs

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
Para que se vea como ChatGPT, usaremos Docker. Es la forma más limpia de no "ensuciar" tu sistema.
Levanta el contenedor de Open WebUI:
```bash
docker run -d -p 3000:8080 --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data --name open-webui ghcr.io/open-webui/open-webui:main
```
Acceso: Abre tu navegador en `http://localhost:3000`. Crea tu cuenta local (recuerda: todo se queda en tu PC) y selecciona el modelo en el desplegable superior.

## 💻 Paso 3: Integración en VSCodium (Continue.dev)
Cómo instalarlo (en Ubuntu/Debian)
Abre tu terminal y pega esto:
```bash
# Añadir la clave GPG y el repositorio
wget -qO - https://gitlab.com/paulcarroty/vscodium-deb-rpm-repo/raw/master/pub.gpg | gpg --dearmor | sudo dd of=/usr/share/keyrings/vscodium-archive-keyring.gpg
echo 'deb [ signed-by=/usr/share/keyrings/vscodium-archive-keyring.gpg ] https://download.vscodium.com/debs vscodium main' | sudo tee /etc/apt/sources.list.d/vscodium.list

# Instalar
sudo apt update && sudo apt install codium
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
  }
}
```
4. ¿Cómo se usa esto en el día a día?
Para crear un archivo: Abres un archivo (ej. `firewall_rules.sh`).

Para que la IA te ayude: Seleccionas un trozo de código y pulsas `Ctrl + L`. Se abre un chat a la derecha. Le dices: "Optimiza este script para OPNsense" y lo hace ahí mismo.

Autocompletado: Mientras escribes, la IA te sugerirá la siguiente línea (como el autocompletado de los móviles, pero inteligente). Pulsas `Tab` y listo.
