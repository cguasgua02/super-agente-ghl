# Super Agente v2: n8n + Go High Level + RAG

Este repositorio contiene el workflow de n8n para un "Super Agente" conversacional multicanal integrado con Go High Level (GHL). El agente utiliza un sistema de **Retrieval-Augmented Generation (RAG)** para responder preguntas basándose en una base de conocimiento propia, y es capaz de gestionar entradas de texto, audio e imágenes.

---

## 🚀 Sección 0: Setup del Entorno n8n (VPS Recomendado)

Para un agente de esta complejidad, que necesita ejecutarse 24/7, gestionar webhooks y potencialmente manejar archivos, se desaconseja ejecutarlo en un PC local o en n8n Cloud (debido a limitaciones de acceso al sistema de archivos o de ejecución).

La mejor solución es un **VPS (Virtual Private Server)** auto-hospedado (self-hosted).

### ¿Por qué un VPS?
* **Ejecución 24/7:** Tu agente siempre estará en línea para recibir webhooks.
* **Control Total:** Tienes control sobre las versiones, los recursos y el sistema de archivos (necesario para nodos como `Read/Write Files`).
* **Rendimiento:** Puedes asignar recursos específicos (RAM, CPU) para que tu agente funcione con fluidez, especialmente durante la ingesta de documentos RAG.

### Paso 1: Adquirir un VPS

Puedes usar cualquier proveedor de VPS (DigitalOcean, Vultr, AWS), pero una opción popular por su excelente relación costo/beneficio es **Contabo**.

1.  Ve a [Contabo](https://contabo.com/).
2.  Selecciona un plan de VPS. Para un agente de n8n que manejará IA y múltiples flujos, se recomienda un plan como el **Cloud VPS M** (o superior):
    * **Mínimo recomendado:** 4 vCores, 8GB RAM.
    * **Sistema Operativo:** Selecciona **Ubuntu 22.04**.
3.  Completa la compra y espera a recibir tus credenciales de acceso (IP del servidor y contraseña root) por email.

### Paso 2: Instalar un Panel de Control (EasyPanel)

Para evitar manejar todo por línea de comandos (SSH) y facilitar la instalación de n8n (que corre sobre Docker), usaremos un panel de control. **EasyPanel** es una opción fantástica y gratuita.

1.  Conéctate a tu VPS por SSH usando la IP y contraseña que te dio Contabo.
2.  Una vez dentro, instala EasyPanel ejecutando el script que proporcionan en su [sitio web](https://easypanel.io/). Generalmente es un solo comando.
3.  Sigue las instrucciones en pantalla. EasyPanel se instalará y te dará una URL y credenciales para acceder a su panel de control web.

### Paso 3: Instalar n8n desde EasyPanel

1.  Accede a tu panel de EasyPanel con la URL y credenciales.
2.  Navega a la sección "Create Project" o "Services".
3.  EasyPanel tiene plantillas pre-configuradas. Busca **n8n** en la lista de aplicaciones.
4.  Haz clic en "Deploy" (Desplegar). EasyPanel se encargará automáticamente de configurar Docker, las redes y los volúmenes de datos por ti.
5.  Una vez desplegado, EasyPanel te proporcionará la URL de tu instancia de n8n (ej. `n8n.tu-dominio.com` si configuraste un dominio).

### Paso 4: Configurar n8n

1.  Accede a la URL de tu n8n.
2.  La primera vez que entres, n8n te pedirá que crees tu cuenta de administrador (`owner`).
3.  ¡Listo! Ya tienes una instancia de n8n potente, privada y lista para producción.

---

## 🤖 Características Principales del Agente

* **Multicanal:** Se conecta a GHL y puede gestionar conversaciones de Instagram, WhatsApp y Facebook.
* **Procesamiento Multimedia:**
    * **Texto:** Responde directamente.
    * **Audio:** Transcribe audios (usando OpenAI Whisper) antes de procesar la solicitud.
    * **Imágenes:** Analiza el contenido de las imágenes (usando OpenAI Vision) y lo usa como contexto.
* **RAG (Base de Conocimiento):**
    * Utiliza una base de datos vectorial para buscar información relevante antes de responder.
    * Incluye un flujo de "ingesta" para cargar documentos (Google Docs, PDF, Sheets, Imágenes) desde Google Drive a la base de datos vectorial.
* **Memoria Conversacional:** Guarda el historial de chat para mantener el contexto en conversaciones largas.
* **Respuesta Dinámica:**
    * Envía respuestas de texto largas en múltiples mensajes para simular una escritura natural.
    * Si la consulta original fue por audio, responde con un audio generado por **ElevenLabs**.

## 🛠️ Herramientas y Requisitos Previos

Para que este workflow funcione, necesitarás cuentas y credenciales para los siguientes servicios:

1.  **n8n (Self-Hosted):** Una instancia de n8n corriendo en tu VPS (ver Sección 0).
2.  **Go High Level (GHL):** Una clave API (Bearer Token) para enviar y recibir mensajes.
3.  **OpenAI:** Una clave API para los modelos (`gpt-4o-mini`, `text-embedding-3-large`, Whisper y Vision).
4.  **Google Cloud Platform:** Credenciales OAuth2 para:
    * Google Drive (para leer documentos RAG y subir audios de respuesta).
    * Google Docs (para leer contenido).
    * Google Sheets (para leer contenido).
5.  **ElevenLabs:** Una clave API para la generación de audio.
6.  **Base de Datos Vectorial:**
    * **Este workflow usa:** **MongoDB Atlas**. Necesitarás una URL de conexión para la base de datos vectorial (colección: `embeddings`) y otra para la memoria del chat (colección: `memory`).
    * Ver la sección de alternativas a continuación.

---

### 💡 Alternativas de Bases de Datos Vectoriales (Recomendado)

Aunque este workflow está construido con **MongoDB Atlas**, no es una base de datos vectorial *dedicada*. Para mayor velocidad, escalabilidad y mejores capacidades de búsqueda semántica, te recomiendo considerar estas alternativas (requerirá modificar el workflow):

* **Pinecone:**  La opción líder del mercado. Es una base de datos vectorial gestionada, increíblemente rápida y fácil de integrar. n8n tiene nodos nativos para Pinecone.
* **Supabase (con pgvector):**  Si prefieres una solución "todo en uno" (Base de datos SQL, Auth, Storage, y Vectorial), Supabase es excelente. Usa la extensión `pgvector` sobre PostgreSQL.
* **Qdrant:**  Una base de datos vectorial muy potente y popular. Ofrece una nube gestionada o puedes auto-hospedarla (incluso en EasyPanel junto a n8n). n8n tiene nodos nativos.
* **Weaviate:** Otra base de datos vectorial nativa de código abierto, muy robusta y con excelentes características de búsqueda. También tiene nodos nativos en n8n.

**Para adaptar el workflow:** Deberás reemplazar los nodos `MongoDB Vector Store Inserter` y `MongoDB Vector Search` por los nodos correspondientes de la nueva base de datos (ej. `Pinecone`, `Qdrant`, etc.).

---

## ⚙️ Guía de Instalación y Configuración

Sigue estos pasos para poner en marcha tu agente:

### Paso 1: Configurar Credenciales en n8n

Antes de importar el workflow, ve a la sección **Credentials** en tu instancia de n8n y configura las siguientes credenciales. El workflow importado las buscará por los nombres usados en el JSON (puedes re-asignarlas si las nombras diferente).

* **OpenAI:**
    * `OpenAi account 2` (o el nombre que prefieras).
* **MongoDB:**
    * `MongoDB account NUO` (para el RAG).
    * `MongoDB account DB AgenteSoporte` (para el RAG). *Nota: El JSON original usa varias, puedes consolidarlas.*
    * `MongoDB account ChatMemory` (para la memoria del chat).
* **Google (OAuth2):**
    * `Google Drive account NUO`
    * `Google Docs account`
    * `Google Sheets account`
    * `Google Drive account` (para subir audios).
* **ElevenLabs:**
    * `ElevenLabs Cris account`
* **Go High Level (Header Auth):**
    * Crea una credencial de tipo **Header Auth**.
    * **Name:** `Authorization`
    * **Value:** `Bearer TU_TOKEN_DE_GHL_AQUI` (Tu GHL API Key)
    * Dale un nombre como `GoHighLevel API`.

### Paso 2: Importar el Workflow

1.  Descarga el archivo `workflow.json` de este repositorio.
2.  En n8n, haz clic en **Import** > **From File** y selecciona el archivo `workflow.json`.

### Paso 3: Conectar Nodos y Credenciales

Una vez importado, verás errores en muchos nodos. Ve a cada nodo que tenga un signo de exclamación `(!)` y:

1.  **Nodos de Credenciales (OpenAI, MongoDB, etc.):** Selecciona la credencial correspondiente que creaste en el Paso 1 en el menú desplegable **Credential**.
2.  **Nodos HTTP Request (GHL):** En el campo **Authentication**, selecciona tu credencial `GoHighLevel API`.

### Paso 4: Poblar la Base de Datos RAG (Setup de MongoDB)

El agente no sabrá nada hasta que no cargues documentos en su base de conocimiento. *(Estos pasos son para el setup con MongoDB que incluye este workflow)*.

1.  Ve al nodo `Code URLs` (cerca de la parte superior del canvas).
2.  Edita el código y añade las URLs de las carpetas de Google Drive que contienen tus documentos.
    ```javascript
    const folderUrls = [
      "httpsDE_TU_CARPETA_1",
      "httpsDE_TU_CARPETA_2"
    ];
    ```
3.  **¡Importante!** Asegúrate de que los archivos en Google Drive estén **compartidos** para que n8n (con sus credenciales de Google) pueda acceder a ellos.
4.  Ejecuta manualmente el workflow haciendo clic en **Execute workflow**. Esto activará el flujo de ingesta (RAG System), que leerá tus archivos, los dividirá (`Document Chunker`), creará embeddings (`OpenAI Embeddings Generator2`) y los guardará en tu MongoDB (`MongoDB Vector Store Inserter`).
5.  Puedes verificar el progreso en la ejecución y confirmar que los datos se han añadido a tu base de datos de MongoDB Atlas.

### Paso 5: Conectar Go High Level a n8n

1.  Selecciona el nodo **Webhook1** al inicio del flujo del agente.
2.  Copia la URL del Webhook de **Production** (ya que estás en un VPS).
3.  En tu cuenta de Go High Level, ve a **Settings** > **Workflow** > **Create Workflow**.
4.  Crea un workflow que se dispare en `Customer Replied`.
5.  Añade una acción de **Webhook**.
6.  Pega la URL de tu webhook de n8n en el campo `POST`.
7.  Guarda y activa el workflow en GHL.
8.  Envía un mensaje de prueba desde un contacto en GHL. Debería aparecer en la sección "Executions" de n8n.
9.  **¡Activa tu workflow en n8n!** (El botón de "Active" en la esquina superior derecha).

### Paso 6: Personalizar tu Agente

* **Personalidad:** La personalidad del agente ("Camila" de "Invers") se define en el **System Message** del nodo `AI Agent`. Puedes editar este prompt para cambiar su nombre, tono y directrices.
* **Modelos de IA:** Puedes cambiar los modelos de IA utilizados en los nodos `OpenAI Chat Model` (actualmente `gpt-4o-mini`) y `OpenAI Embeddings Generator2` (actualmente `text-embedding-3-large`).
* **Voz:** La voz del agente se configura en el nodo `Convert text to speech` de ElevenLabs.
