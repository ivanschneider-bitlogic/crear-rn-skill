# crear-rn (plugin)

Skill de Claude Code para generar Release Notes siguiendo siempre la misma plantilla de equipo, autocompletando datos desde Linear, AWS ECS, git y Google Drive, y creando el documento final directamente como Google Doc en la carpeta de Drive correspondiente.

## Instalación

### Opción A — como marketplace local/privado

1. Publicar este directorio en un repo Git (privado o público según corresponda).
2. Desde Claude Code:
   ```
   /plugin marketplace add <owner>/<repo>
   /plugin install crear-rn@<repo>
   ```

### Opción B — copiar la skill directamente

Copiar `skills/crear-rn/` dentro de `~/.claude/skills/crear-rn/`.

## Requisitos

- MCP de Google Drive (`mcp__claude_ai_Google_Drive__*`) — para leer/crear los documentos de Release Notes.
- MCP de Linear (`mcp__plugin_linear_linear__*`) — para autocompletar alcance y problemas resueltos.
- AWS CLI configurado, si algún componente resuelve su versión contra ECS.
- Acceso local (git) a los repos de configuración/infra, si se quiere autocompletar variables de entorno nuevas.

Ninguno de estos es obligatorio: si falta alguno, la skill simplemente le pide el dato al usuario en vez de autocompletarlo.

## Primer uso en un proyecto nuevo

La skill no viene con datos de ningún proyecto específico. La primera vez que se corre para un proyecto:

1. Detecta que no existe `skills/crear-rn/references/known-components.md`.
2. Hace onboarding: pregunta los componentes de software del proyecto (aplicación, repo, documentación, cómo resolver su versión), el documento de arquitectura por defecto y los clusters/entornos QA.
3. Genera `known-components.md` con esos datos (mismo formato que `references/known-components.template.md`) para reusarlo en las próximas ejecuciones.

Ese archivo queda con datos específicos del equipo/proyecto — no se versiona en este repo del plugin, cada instalación genera el suyo.
