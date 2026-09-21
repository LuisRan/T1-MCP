# Tarea 1: MCP y servidor de sistema de archivos (investigación e implementación)

| | |
|---|---|
| **Nombre** | Rangel Mata José Luis |
| **Boleta** | 2023630577 |
| **Grupo** | 7CV4 |
| **Asignatura** | Desarrollo de aplicaciones móviles nativas |
| **Profesor** | Gabriel Hurtado Avilés |
| **Fecha de entrega** | 21 de septiembre de 2026 |

## 1. Resumen de la actividad

TODO: escribe aquí 3 o 4 líneas con tus palabras: qué investigaste y qué implementaste.

## 2. Índice de documentos (`docs/`)

Investigación, un archivo `.md` por punto (en elaboración):

- `docs/01-evolucion-modelos.md`
- `docs/02-aislamiento.md`
- `docs/03-mcp-vs-api.md`
- `docs/04-arquitectura-mcp.md`
- `docs/05-servidor-filesystem.md`
- `docs/06-seguridad.md`
- `docs/07-casos-de-uso.md`

## 3. Tabla comparativa MCP vs API

TODO: tabla con al menos estos criterios: quién decide qué se invoca, cómo se descubren las capacidades, acoplamiento del cliente al servicio, formato de los mensajes, autenticación y consentimiento, y reutilización entre aplicaciones. Incluye la aclaración de que MCP no sustituye a las APIs.

## 4. Instalación paso a paso

### Entorno utilizado

| Componente | Versión |
|---|---|
| Sistema operativo | macOS 26.6.2 (build 25G83), Apple Silicon |
| Node.js | v26.8.1 |
| npx / npm | 11.19.0 |
| Cliente MCP | Claude Desktop 2.2553.1 |
| Servidor MCP | `@modelcontextprotocol/server-filesystem` 2026.8.31 |
| Transporte | stdio (el servidor corre como proceso hijo del cliente) |

### Justificación del cliente

TODO: por qué elegiste Claude Desktop (con tus palabras).

### Pasos

1. **Verificar Node.js.** Si no está instalado, instalar la versión LTS desde nodejs.org o con `brew install node`.

```bash
   node -v
   npx -v
```

2. **Crear el directorio de trabajo** dedicado a esta tarea (no la raíz del disco ni la carpeta de usuario) y algunos archivos de prueba. Además, un archivo **fuera** del directorio para la prueba de seguridad:

```bash
   mkdir -p /Users/luis/Documents/mcp
   echo "Hola, este es un archivo de prueba" > /Users/luis/Documents/mcp/prueba.txt
   echo "Notas de la tarea de MCP" > /Users/luis/Documents/mcp/notas.md
   echo "contenido fuera del limite" > /Users/luis/Documents/fuera.txt
```

3. **Editar la configuración del cliente.** En Claude Desktop: Ajustes → Desarrollador → *Editar configuración*. Se abre `~/Library/Application Support/Claude/claude_desktop_config.json`. Agregar la clave `mcpServers` al mismo nivel que las demás claves existentes (en mi instalación el archivo ya contenía una sección `preferences`, por lo que hay que separar ambas con una coma). En un archivo nuevo o vacío basta con el fragmento de abajo. En una máquina distinta hay que cambiar la ruta final por el directorio propio.

4. **Validar que el JSON sea correcto:**

```bash
   python3 -m json.tool "$HOME/Library/Application Support/Claude/claude_desktop_config.json" > /dev/null && echo "JSON valido"
```

5. **Reiniciar Claude Desktop por completo** (Cmd+Q y volver a abrir).

6. **Verificar que el servidor corre:** Ajustes → Desarrollador debe mostrar `filesystem` con el estado *En ejecución*.

7. **Usar el servidor:** en un chat en modo **Chat** (no Cowork), menú `+` → Conectores, y comprobar que `filesystem` está activado.

8. **Verificar el catálogo de herramientas** preguntando en el chat qué herramientas del servidor `filesystem` están disponibles.

### Configuración usada

Fragmento que se agrega a `claude_desktop_config.json` (también en `config/claude_desktop_config.mcp.json`). No contiene credenciales:

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/Users/luis/Documents/mcp"
      ]
    }
  }
}
```

- `command` y `args`: el cliente ejecuta `npx`, que descarga y lanza el servidor.
- El **último argumento** es el directorio permitido. Cualquier ruta fuera de él queda bloqueada por el servidor.

## 5. Evidencias

**Servidor en ejecución** en Ajustes → Desarrollador:

![Servidor filesystem en ejecución](img/01-servidor-en-ejecucion.png)

**Conector `filesystem` activado** en el chat:

![Conector filesystem activo](img/02-conector-filesystem-activo.png)

**Catálogo de herramientas.** El chat reportó 14 herramientas del servidor: `create_directory`, `directory_tree`, `edit_file`, `get_file_info`, `list_allowed_directories`, `list_directory`, `list_directory_with_sizes`, `move_file`, `read_file`, `read_media_file`, `read_multiple_files`, `read_text_file`, `search_files` y `write_file`.

![Herramientas del servidor filesystem](img/03-herramientas-filesystem.png)

### Operaciones realizadas

Prompts usados y resultado:

| # | Operación | Prompt | Resultado |
|---|---|---|---|
| 1 | Listar | "Lista el contenido de mi directorio autorizado." | Correcto |
| 2 | Leer | "Lee el archivo prueba.txt y dime qué contiene." | Correcto |
| 3 | Crear y escribir | "Crea un archivo tareas.txt con tres líneas sobre desarrollo de apps móviles." | Correcto (`tareas.txt`, 3 líneas) |
| 4 | Modificar | "Agrega una línea al final de notas.md." | Correcto (herramienta `edit_file`) |
| 5 | Buscar | "Busca los archivos cuyo nombre contenga 'prueba'." | Correcto (solo `prueba.txt`) |
| 6 | Límite de seguridad | "Lee el archivo /Users/luis/Documents/fuera.txt." | Bloqueado |

**Verificación independiente desde la terminal.** Comprueba que las operaciones ocurrieron de verdad en el disco: `tareas.txt` existe con sus tres líneas y `notas.md` tiene la línea agregada.

![Verificación en la terminal](img/05-verificacion-terminal.png)

TODO: agregar aquí las capturas de cada operación (listar, leer, crear, modificar, buscar) y la del resumen de pruebas.

## 6. Prueba del límite de seguridad

Se pidió al modelo leer `/Users/luis/Documents/fuera.txt`, un archivo que está en `Documents`, un nivel arriba del único directorio autorizado (`/Users/luis/Documents/mcp`).

**Respuesta obtenida:** el servidor rechazó la lectura con un error de acceso denegado (`Access denied - path outside allowed directories`, según lo reportado por el chat). TODO: verificar el texto exacto en la captura y citarlo tal cual.

TODO: insertar la captura `img/06-limite-seguridad.png`.

**Mecanismo que impidió la operación:** no fue que el modelo se negara, sino que el **servidor validó la ruta**. Al arrancar recibió el directorio permitido como argumento y, en cada operación, comprueba que la ruta solicitada quede dentro de esa lista antes de tocar el disco. Como `fuera.txt` no está dentro, devolvió error sin acceder al archivo. Sin este límite, el modelo podría leer o escribir en cualquier ruta donde mi usuario tenga permisos (llaves SSH, documentos personales, archivos `.env`).

## 7. Conclusiones personales

TODO: escríbelas tú, con tus palabras.

## 8. Referencias (APA)

TODO: agregar las fuentes consultadas, incluyendo la especificación de MCP con su **versión y fecha de consulta**, y el paquete `@modelcontextprotocol/server-filesystem` (versión 2026.8.31).
