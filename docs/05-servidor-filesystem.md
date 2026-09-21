# 5. El servidor de sistema de archivos

## "FS" no es parte del protocolo

El servidor de sistema de archivos (*filesystem*, "FS") **no forma parte de la especificación de MCP**. Es **uno de los servidores de referencia** que se publican como ejemplo de cómo implementar el protocolo, entre muchos otros posibles (bases de datos, control de versiones, navegadores, servicios en la nube, etc.).

- Paquete: `@modelcontextprotocol/server-filesystem` (versión 2026.8.31 en mi instalación).
- Se distribuye por npm y se ejecuta con `npx`, como proceso hijo del cliente, usando el transporte **stdio**.
- La especificación define *cómo* se comunican cliente y servidor; qué herramientas ofrece cada servidor es decisión de quien lo implementa.

## Qué herramientas expone

En mi instalación, el chat reportó **14 herramientas**:

| Categoría | Herramientas |
|---|---|
| **Listar** | `list_directory`, `list_directory_with_sizes`, `directory_tree`, `list_allowed_directories` |
| **Leer** | `read_text_file`, `read_file`, `read_multiple_files`, `read_media_file` |
| **Escribir / crear** | `write_file`, `create_directory` |
| **Modificar** | `edit_file` (ediciones basadas en líneas) |
| **Mover / renombrar** | `move_file` |
| **Buscar** | `search_files` (búsqueda recursiva por patrón) |
| **Información** | `get_file_info` (metadatos de un archivo o directorio) |

Notas:

- `read_file` y `read_text_file` aparecen ambas; en la documentación del servidor `read_file` figura como obsoleta a favor de `read_text_file` (conviene verificarlo en la versión instalada).
- `write_file` **crea o sobrescribe por completo** un archivo, por eso es una de las operaciones a vigilar.
- `list_allowed_directories` muestra al modelo cuáles son los límites, lo que resulta útil para la prueba de seguridad.

## Cómo se delimita su alcance: directorios permitidos

Al arrancar, el servidor recibe como argumentos los **directorios permitidos**. En mi configuración:

```json
"args": ["-y", "@modelcontextprotocol/server-filesystem", "/Users/luis/Documents/mcp"]
```

El último argumento (`/Users/luis/Documents/mcp`) es el único directorio autorizado. En **cada** operación, el servidor verifica que la ruta solicitada (normalizada a su forma absoluta) quede dentro de esa lista antes de tocar el disco. Si no es así, devuelve un error de acceso denegado.

Se pueden pasar varios directorios como argumentos adicionales. La documentación del servidor también explica cómo montar directorios en modo solo lectura cuando se ejecuta en Docker, lo que sirve para permitir consultar archivos sin poder modificarlos.

## Por qué existe ese límite

1. **Principio de mínimo privilegio.** El servidor corre con los permisos de mi usuario; sin límite, cualquier ruta que mi usuario pueda leer o escribir quedaría al alcance del modelo.
2. **El límite lo impone el servidor, no el modelo.** No depende de que el modelo "se porte bien": es una validación en código que se ejecuta en cada llamada.
3. **Defensa ante errores e inyección de instrucciones.** Si el modelo comete un error o es manipulado por el contenido de un archivo, el daño posible queda acotado al directorio de trabajo.

## Qué pasaría sin el límite

Sin directorios permitidos, un modelo con acceso al servidor podría:

- Leer **información sensible**: llaves SSH (`~/.ssh`), archivos `.env`, documentos personales, historiales.
- **Modificar o sobrescribir** archivos importantes con `write_file` o `edit_file`.
- **Mover o dejar inutilizable** información del sistema con `move_file`.
- Ser **guiado por una instrucción maliciosa** escondida en un archivo para exfiltrar datos o dañar archivos (ver [`06-seguridad.md`](06-seguridad.md)).

## Qué habilita exactamente este servidor

Respuesta corta para la exposición:

> El servidor de sistema de archivos no le da al modelo acceso al disco; expone un conjunto de herramientas (listar, leer, escribir, crear, mover, buscar) que el servidor mismo ejecuta, y solo dentro de los directorios permitidos. Le permite al modelo trabajar con archivos locales sin que yo tenga que copiar y pegar su contenido, pero con un alcance acotado.

## Evidencia en mi práctica

Con este servidor se ejecutaron las cinco operaciones pedidas (listar, leer, crear, modificar y buscar) y la prueba de que una lectura fuera del directorio queda bloqueada. Las capturas están en el [README](../README.md).

## Referencias

Model Context Protocol. (2026). *@modelcontextprotocol/server-filesystem* (Versión 2026.8.31) [Software]. npm. Recuperado el 21 de septiembre de 2026, de https://www.npmjs.com/package/@modelcontextprotocol/server-filesystem

Model Context Protocol. (s. f.). *Reference servers* [Repositorio]. GitHub. Recuperado el 21 de septiembre de 2026, de https://github.com/modelcontextprotocol/servers
