# Tarea: Servidor MCP de sistema de archivos

| | |
|---|---|
| **Nombre** | Rangel Mata José Luis |
| **Boleta** | 2023630577 |
| **Grupo** | 7CV4 |
| **Asignatura** | Desarrollo de aplicaciones móviles nativas |
| **Profesor** | Gabriel Hurtado Avilés |
| **Fecha de entrega** | 21 de septiembre de 2026 |

## 1. Resumen de la actividad

(Escribe aquí 3 o 4 líneas con tus palabras: qué investigaste y qué implementaste.)

## 2. Índice de documentos (`docs/`)

- `docs/01-evolucion-modelos.md`
- `docs/02-aislamiento.md`
- `docs/03-mcp-vs-api.md`
- `docs/04-arquitectura-mcp.md`
- `docs/05-servidor-filesystem.md`
- `docs/06-seguridad.md`
- `docs/07-casos-de-uso.md`

## 3. Tabla comparativa MCP vs API

(Pendiente: la armamos juntos.)

## 4. Instalación paso a paso

**Entorno utilizado**

- Sistema operativo: macOS (indica la versión exacta)
- Node.js: v26.8.1
- npx: 11.19.0
- Cliente MCP: Claude Desktop (indica la versión)
- Servidor: `@modelcontextprotocol/server-filesystem` (indica la versión)

**Justificación del cliente:** (pendiente, con tus palabras)

**Pasos:** (pendiente: directorio de trabajo, edición de la config, reinicio, verificación)

**Configuración usada** (fragmento que se agrega al `claude_desktop_config.json`, también en `config/`):

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

## 5. Evidencias

(Aquí enlazas las capturas de `img/` con `![descripción](img/archivo.png)`.)

## 6. Prueba del límite de seguridad

(Pendiente: captura del error y explicación del mecanismo.)

## 7. Conclusiones personales

(Pendiente: tuyas.)

## 8. Referencias (APA)

(Pendiente: incluye la versión de la especificación MCP consultada.)
