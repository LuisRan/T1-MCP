# 4. Arquitectura de MCP

> **Versión de la especificación consultada:** Model Context Protocol **2026-07-28**, consultada el 21 de septiembre de 2026 (<https://modelcontextprotocol.io/specification/2026-07-28>). La especificación se actualiza con frecuencia; este documento está escrito sobre esa revisión y menciona explícitamente lo que cambió respecto a la anterior (2025-11-25).

## Modelo host / cliente / servidor

MCP separa tres roles:

| Rol | Qué es | En mi instalación |
|---|---|---|
| **Host** | La aplicación con la que interactúa la persona y donde vive el modelo. Decide qué servidores se conectan y gestiona permisos. | **Claude Desktop** 2.2553.1 |
| **Cliente MCP** | El componente dentro del host que mantiene la conexión con **un** servidor. Un host con varios servidores tiene varios clientes. | El componente interno de Claude Desktop que se conecta al servidor `filesystem`. |
| **Servidor MCP** | Un programa que expone capacidades (herramientas, recursos, prompts) a través del protocolo. | El proceso de Node.js que ejecuta `npx @modelcontextprotocol/server-filesystem` (versión 2026.8.31). |

Es importante no confundir cliente con servidor, y recordar que **el modelo nunca accede directamente al disco**: genera una petición de herramienta, el host la manda al servidor y este ejecuta la operación y devuelve el resultado.

```
Persona ──> Host (Claude Desktop, con el modelo)
                 │
                 └─ Cliente MCP ──(stdio, JSON-RPC 2.0)──> Servidor MCP (Node.js) ──> Disco
                                                            (solo /Users/luis/Documents/mcp)
```

## Primitivas que expone un servidor

Un servidor puede ofrecer tres tipos de capacidades. Es un error mencionar solo las herramientas:

| Primitiva | Quién la controla | Para qué sirve | Ejemplo |
|---|---|---|---|
| **Herramientas (tools)** | El **modelo** decide cuándo invocarlas. | Realizar acciones o cálculos, que pueden tener efectos. | `write_file`, `search_files` en el servidor de archivos. |
| **Recursos (resources)** | La **aplicación** decide cuáles incluir como contexto. | Exponer datos de solo lectura identificados por URI. | El contenido de un archivo, un registro de base de datos, un esquema. |
| **Plantillas de prompt (prompts)** | La **persona usuaria** las elige explícitamente. | Ofrecer flujos o mensajes predefinidos y parametrizables. | Una plantilla "revisar este código" con argumentos. |

Cada tipo se descubre con su propia llamada de listado: `tools/list`, `resources/list` y `prompts/list`. En la versión 2026-07-28 estas respuestas incluyen pistas de caché (`ttlMs` y `cacheScope`) para que los clientes puedan guardar el catálogo.

## Primitivas del lado del cliente

Además de lo que ofrece el servidor, el cliente puede ofrecer capacidades al servidor:

- **Roots (raíces):** el cliente le indica al servidor qué directorios o ubicaciones son relevantes o están permitidos.
- **Elicitation:** el servidor puede pedirle a la persona información adicional o una confirmación a mitad de una operación (por ejemplo, "¿confirmas que quieres borrar esto?").
- **Sampling:** el servidor puede pedirle al cliente que use el modelo para generar una respuesta.

**Cambios en 2026-07-28** (Model Context Protocol, 2026):

- **Roots, Sampling y Logging quedaron marcados como obsoletos (deprecated).** Siguen funcionando durante al menos doce meses, pero las nuevas implementaciones no deberían adoptarlos.
- Las peticiones del servidor al cliente como `elicitation/create`, `sampling/createMessage` y `roots/list` se rediseñaron con **Multi Round-Trip Requests (MRTR)**: en lugar de mantener un canal abierto, el servidor responde que necesita información (`input_required`) y el cliente repite la llamada con las respuestas.

## Transportes

El transporte define cómo viajan los mensajes JSON-RPC entre cliente y servidor:

| Transporte | Cuándo se usa | Cómo funciona |
|---|---|---|
| **stdio** | Servidores **locales**. | El cliente lanza el servidor como **proceso hijo** y se comunican por entrada y salida estándar. Es el que uso en esta práctica (por eso la configuración tiene `command` y `args`). |
| **Streamable HTTP** | Servidores **remotos**. | El servidor corre en otra máquina y recibe peticiones HTTP. En 2026-07-28 las peticiones incluyen los encabezados `Mcp-Method` y `Mcp-Name` para que los gateways puedan enrutar y autorizar sin leer el cuerpo del mensaje. |

El antiguo transporte HTTP+SSE está oficialmente obsoleto, con un periodo de transición de un año.

## Qué cambió con la especificación 2026-07-28

- El protocolo pasó de ser bidireccional y con estado a un modelo **sin estado** de petición/respuesta: se retiraron el intercambio `initialize`/`initialized` y el encabezado `Mcp-Session-Id`.
- Cada petición es autodescriptiva: lleva su versión de protocolo y capacidades del cliente en `_meta`. La llamada `server/discover` es opcional.
- Se formalizó un marco de **extensiones** (por ejemplo Tasks, MCP Apps) y se reforzó la autorización.

Aun así, el concepto central no cambia: un servidor publica su catálogo y el cliente/modelo lo descubre y usa.

## Referencias

Model Context Protocol. (2026). *Specification* (Versión 2026-07-28). Recuperado el 21 de septiembre de 2026, de https://modelcontextprotocol.io/specification/2026-07-28

Model Context Protocol. (2026, 28 de julio). *The 2026-07-28 specification* [Entrada de blog]. https://blog.modelcontextprotocol.io/posts/2026-07-28/

JSON-RPC Working Group. (2013). *JSON-RPC 2.0 specification*. https://www.jsonrpc.org/specification
