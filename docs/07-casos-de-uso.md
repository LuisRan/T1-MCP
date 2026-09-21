# 7. Casos de uso: herramientas actuales que implementan MCP

Estas son herramientas actuales que funcionan como **clientes MCP** (host con cliente integrado) y para qué lo usan. En todas, el mecanismo es el mismo que en mi práctica: un servidor publica un catálogo de herramientas, el cliente lo descubre y el modelo decide cuáles invocar.

## 1. Claude Desktop

Aplicación de escritorio de Anthropic con un asistente conversacional. Permite configurar servidores MCP locales mediante un archivo JSON (`claude_desktop_config.json`), que es lo que utilicé en esta práctica.

- **Para qué usa MCP:** conectar al asistente con archivos locales, bases de datos, servicios de terceros y otras fuentes, sin copiar y pegar información en el chat.
- **Ejemplo:** con el servidor `filesystem`, pedirle que liste, lea, cree, modifique y busque archivos de un directorio autorizado.

## 2. Claude Code

Herramienta de programación agéntica de Anthropic que trabaja desde la terminal y sobre un repositorio.

- **Para qué usa MCP:** además de sus herramientas propias para leer y editar código, puede conectarse a servidores MCP para ampliar lo que sabe hacer (por ejemplo, consultar un gestor de incidencias, una base de datos o documentación).
- **Ejemplo:** editar varios archivos de un proyecto y consultar, mediante un servidor MCP, la información de una tarea o un servicio externo.

## 3. Visual Studio Code (modo agente de GitHub Copilot)

Editor de código ampliamente utilizado. En su modo agente, el asistente puede usar herramientas, incluidas las que provienen de servidores MCP configurados en el editor.

- **Para qué usa MCP:** que el agente realice tareas de varios pasos sobre el proyecto abierto, usando herramientas externas como repositorios, bases de datos o APIs.

## 4. Cursor y Zed

Editores orientados a programación asistida por IA. Ambos permiten configurar servidores MCP para que sus agentes usen herramientas adicionales.

- **Para qué usan MCP:** ampliar el contexto y las acciones del agente más allá del código abierto en el editor (documentación, incidencias, bases de datos, navegador, etc.), con una integración estándar en vez de una específica por servicio.

## 5. Google Antigravity

**Google Antigravity es un entorno de desarrollo agéntico (IDE)**, construido sobre un editor tipo VS Code, en el que el agente planifica, escribe y valida código. Soporta MCP: los servidores se administran desde el panel del agente (*Manage MCP Servers* → *View raw config*) y se declaran en un archivo de configuración `mcp_config.json`.

- **Para qué usa MCP:** conectar al agente con servicios externos (por ejemplo, el servidor MCP de GitHub o servidores de Google Cloud) para consultar y actuar sobre ellos desde el mismo entorno.
- **Relevancia para desarrollo móvil:** la documentación oficial de Flutter incluye una guía para usar Antigravity en el desarrollo de apps Flutter, y existen servidores MCP orientados a desarrollo móvil que se pueden conectar a estos entornos.

### Nota de precisión sobre Qwen

**Qwen es una familia de modelos, no una plataforma de desarrollo.** Si se quisiera incluir en esta lista, habría que referirse a la herramienta concreta que la utiliza (por ejemplo, un asistente de línea de comandos basado en Qwen) y no a "Qwen" como si fuera el entorno. Por eso no la incluí como caso de uso.

## Cómo editan repositorios completos sin subir los archivos manualmente

Estas herramientas no "suben" archivos al modelo. El proceso es el siguiente:

1. **El host corre en mi equipo** y tiene acceso al proyecto abierto (el directorio de trabajo).
2. **Ofrece al modelo herramientas** para explorarlo y modificarlo: listar directorios, buscar por nombre o contenido, leer archivos, editar o crear archivos, y en algunos casos ejecutar comandos o pruebas. Algunas vienen integradas y otras llegan mediante **servidores MCP**, como el de sistema de archivos.
3. **El modelo decide qué herramienta usar en cada paso.** Por ejemplo: busca los archivos relevantes, lee algunos, propone y aplica una edición, y vuelve a leer para verificar.
4. **Cada herramienta la ejecuta un proceso local**, no el modelo. El resultado vuelve al modelo como texto.
5. **La persona revisa y aprueba** los cambios (por ejemplo, mediante un *diff* o una confirmación) y puede revertirlos con control de versiones.

El resultado es que el agente trabaja sobre el repositorio completo leyendo solo lo que necesita, sin que yo copie y pegue archivos, y con el alcance limitado por los permisos y directorios configurados.

## Relación con mi práctica

Lo que hice con Claude Desktop y el servidor `filesystem` es una versión mínima de lo que hacen estas herramientas: un cliente MCP, un servidor local por stdio, herramientas descubiertas en tiempo de ejecución y un directorio autorizado como límite.

## Referencias

Flutter. (s. f.). *Antigravity* [Documentación]. Recuperado el 21 de septiembre de 2026, de https://docs.flutter.dev/ai/antigravity

GitHub. (s. f.). *GitHub MCP server: Installation guides* [Repositorio]. GitHub. Recuperado el 21 de septiembre de 2026, de https://github.com/github/github-mcp-server

Google Cloud. (s. f.). *Use MCP servers* [Documentación]. Recuperado el 21 de septiembre de 2026, de https://docs.cloud.google.com/data-cloud-extension/antigravity/use-mcp-servers

Model Context Protocol. (2026). *Specification* (Versión 2026-07-28). Recuperado el 21 de septiembre de 2026, de https://modelcontextprotocol.io/specification/2026-07-28
