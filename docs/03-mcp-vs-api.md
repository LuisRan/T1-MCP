# 3. MCP frente a una API

> **Versión de la especificación consultada:** Model Context Protocol **2026-07-28**, consultada el 21 de septiembre de 2026 (<https://modelcontextprotocol.io/specification/2026-07-28>).

## Qué es una API

Una **API** (interfaz de programación de aplicaciones) es un **contrato entre programas**. Define qué operaciones ofrece un servicio, con qué parámetros y qué devuelve cada una.

El flujo típico es este:

1. La persona que desarrolla **lee la documentación**.
2. **Decide qué endpoint llamar** y en qué momento.
3. **Arma la petición** (método, URL, parámetros, autenticación).
4. **Escribe el código que interpreta la respuesta.**

La decisión de *qué se llama y cuándo* está **escrita de antemano en el código**. Si el servicio cambia un endpoint, hay que actualizar el cliente.

## Qué es MCP

El **Model Context Protocol (MCP)** es un **protocolo abierto**, basado en **JSON-RPC 2.0**, que estandariza cómo una aplicación con un modelo de lenguaje se conecta a servicios externos.

En MCP, un **servidor publica un catálogo** de capacidades. Para las herramientas, cada una viene con:

- su **nombre**,
- su **descripción** en lenguaje natural,
- el **esquema de sus parámetros** (JSON Schema).

El cliente pide ese catálogo en tiempo de ejecución y se lo presenta al modelo, que **decide cuál invocar** según lo que la persona usuaria pidió. Nadie escribió de antemano "si el usuario pide leer un archivo, llama a `read_text_file`".

Ejemplo simplificado de descubrimiento y uso (mensajes JSON-RPC 2.0; se omiten campos de metadatos):

```json
{ "jsonrpc": "2.0", "id": 1, "method": "tools/list" }
```

```json
{
  "jsonrpc": "2.0", "id": 1,
  "result": {
    "tools": [
      {
        "name": "read_text_file",
        "description": "Lee el contenido de un archivo de texto",
        "inputSchema": {
          "type": "object",
          "properties": { "path": { "type": "string" } },
          "required": ["path"]
        }
      }
    ]
  }
}
```

```json
{
  "jsonrpc": "2.0", "id": 2, "method": "tools/call",
  "params": { "name": "read_text_file", "arguments": { "path": "/Users/luis/Documents/mcp/prueba.txt" } }
}
```

> Nota de versión: la especificación 2026-07-28 volvió el protocolo *sin estado*: retiró el intercambio `initialize` y las sesiones, y cada petición viaja con su versión y capacidades en `_meta`. Existe una llamada opcional, `server/discover`, para conocer las capacidades del servidor de antemano. Aun así, el descubrimiento del catálogo (`tools/list`) y la invocación (`tools/call`) siguen siendo el mecanismo central.

En mi práctica, el chat listó **14 herramientas** del servidor `filesystem` que yo no programé: es ese catálogo descubierto en tiempo de ejecución.

## Tabla comparativa

| Criterio | API tradicional | MCP |
|---|---|---|
| **Quién decide qué se invoca** | La persona desarrolladora, de antemano, en el código del cliente. | El modelo, en tiempo de ejecución, según la petición del usuario y el catálogo disponible. |
| **Cómo se descubren las capacidades** | Leyendo documentación (por ejemplo OpenAPI) fuera del programa; el cliente ya "sabe" los endpoints al escribirse. | El cliente pide el catálogo al servidor (`tools/list`, `resources/list`, `prompts/list`) y recibe nombre, descripción y esquema de cada elemento. |
| **Acoplamiento del cliente al servicio** | Alto: el cliente está escrito para un servicio concreto y se rompe si este cambia. | Bajo: el cliente solo conoce el protocolo; cualquier servidor compatible se conecta sin código específico. |
| **Formato de los mensajes** | Libre: cada API define el suyo (REST/JSON, GraphQL, gRPC, SOAP…). | Estandarizado: JSON-RPC 2.0 sobre un transporte definido (stdio o Streamable HTTP). |
| **Autenticación y consentimiento** | Cada API define la suya (claves, OAuth…); el consentimiento lo resuelve la aplicación que la consume. | El protocolo define un marco de autorización para servidores remotos y los clientes suelen pedir confirmación antes de ejecutar herramientas. En servidores locales por stdio, el alcance lo limita la configuración (por ejemplo, los directorios permitidos). |
| **Reutilización entre aplicaciones** | Baja: cada aplicación escribe su propia integración. | Alta: el mismo servidor sirve a distintos clientes (Claude Desktop, Claude Code, VS Code, Cursor, Zed, Google Antigravity…) sin cambiar el servidor. |

## MCP no sustituye a las APIs

Esto es fundamental y es un error frecuente afirmar lo contrario:

- Un servidor MCP **casi siempre envuelve una API o un recurso que ya existe**. El servidor de sistema de archivos, por ejemplo, envuelve las operaciones de archivos del sistema operativo; otros servidores envuelven APIs REST de servicios como bases de datos, gestores de tareas o repositorios de código.
- MCP es una **capa por encima** que hace ese recurso **descubrible y utilizable por un modelo**.
- Las APIs siguen siendo necesarias: sin ellas, el servidor MCP no tendría nada que envolver.

Una forma de verlo: la API es el *contrato para programas*; MCP es el *contrato para que un modelo descubra y use capacidades*, y usualmente se apoya en el primero.

## MCP es abierto, no propietario

MCP fue publicado por Anthropic en 2024 como **protocolo abierto**. Su especificación es pública y hoy lo adoptan muchos clientes y proveedores distintos, no uno solo. Por eso no es correcto presentarlo como una tecnología propietaria de un único proveedor.

## Respuesta corta para la exposición

> *¿En qué se diferencia MCP de consumir una API?*
> En una API, quien desarrolla decide de antemano qué endpoint se llama y cuándo. En MCP, el servidor publica un catálogo de herramientas y el modelo, en tiempo de ejecución, decide cuál usar según lo que la persona pidió. Además, MCP no reemplaza a las APIs: normalmente las envuelve.

## Referencias

JSON-RPC Working Group. (2013). *JSON-RPC 2.0 specification*. https://www.jsonrpc.org/specification

Model Context Protocol. (2026). *Specification* (Versión 2026-07-28). Recuperado el 21 de septiembre de 2026, de https://modelcontextprotocol.io/specification/2026-07-28

Model Context Protocol. (2026, 28 de julio). *The 2026-07-28 specification* [Entrada de blog]. https://blog.modelcontextprotocol.io/posts/2026-07-28/
