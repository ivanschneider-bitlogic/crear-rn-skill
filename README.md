# crear-rn (plugin)

Skill de Claude Code para generar Release Notes siguiendo siempre la misma plantilla de equipo. Autocompleta datos desde Linear, AWS ECS, git y Google Drive, y crea el documento final directamente como Google Doc en la carpeta de Drive correspondiente al release.

## Requisitos

| Requisito | Para qué se usa | Obligatorio |
|---|---|---|
| [Claude Code](https://docs.claude.com/claude-code) | Ejecutar la skill | Sí |
| MCP de Google Drive (`mcp__claude_ai_Google_Drive__*`) | Buscar/leer el RN de la versión anterior y crear el Google Doc final | Sí — sin esto la skill no puede crear el documento |
| MCP de Linear (`mcp__plugin_linear_linear__*`) | Autocompletar la sección "Alcance" (milestone) y "Problemas Resueltos" (bugs) | No — si falta, se pregunta manualmente |
| AWS CLI configurado con acceso al cluster ECS de QA | Resolver en vivo la versión de los componentes que corren en ECS | No — si falta, se pregunta la versión manualmente |
| Git y acceso local a los repos de configuración/infra | Detectar variables de entorno nuevas en las últimas 6 semanas, contrastando contra el RN anterior para filtrar falsos positivos | No — si falta, se omite esa sección |

Ninguno de los tres últimos es bloqueante: si el MCP o la herramienta no está disponible, la skill simplemente le pide el dato al usuario en vez de autocompletarlo.

## Uso con Cursor, Windsurf u otras IAs.

Este repo está pensado para Claude Code (formato de skill), pero la lógica es portable a cualquier editor con soporte MCP: Cursor, Windsurf, etc. `universal/crear-rn-prompt.md` es el mismo flujo en un prompt Markdown plano, sin el formato de skill específico de Claude Code.

Requisitos: los mismos MCP de la tabla de arriba, pero conectados en tu propio cliente (no en Claude Code). Los nombres exactos de las herramientas MCP varían según cómo cada cliente los expone — el prompt universal describe la acción a realizar, no el nombre literal de la función.

### Instalación como regla de proyecto

```bash
git clone https://github.com/ivanschneider-bitlogic/crear-rn-skill.git
```

Copiar `crear-rn-skill/universal/crear-rn-prompt.md` a la carpeta de reglas de tu cliente:

| Cliente | Ruta de reglas |
|---|---|
| Cursor | `.cursor/rules/crear-rn.mdc` |
| Windsurf | `.windsurf/rules/crear-rn.md` |

Si tu cliente usa un único archivo de reglas (ej. `.windsurfrules` o `.cursorrules` en versiones antiguas), pegar el contenido al final de ese archivo en vez de crear uno nuevo.

Verificar en la configuración del cliente que la regla quedó activa para el proyecto.

### Instalación en clientes sin reglas de proyecto

Pegar el contenido de `universal/crear-rn-prompt.md` directamente al inicio del chat (como instrucción de sistema o primer mensaje) antes de pedir el release note.

### Cómo se usa

Con la regla o el prompt cargado, pedir directamente "generá el release note de `<proyecto>`" (o equivalente). Guía la conversación sección por sección igual que la skill de Claude Code.

## Instalación

### Opción A — como plugin vía marketplace (recomendado)

Desde Claude Code:

```
/plugin marketplace add ivanschneider-bitlogic/crear-rn-skill
/plugin install crear-rn@crear-rn-skill
```

Verificar que quedó instalada:

```
/plugin
```

Debería listarse `crear-rn` como plugin activo.

### Opción B — copia manual (sin marketplace)

```bash
git clone https://github.com/ivanschneider-bitlogic/crear-rn-skill.git
cp -r crear-rn-skill/skills/crear-rn ~/.claude/skills/crear-rn
```

Reiniciar Claude Code (o abrir una sesión nueva) para que detecte la skill nueva.

### Cómo se usa

Una vez instalada, en cualquier proyecto:

```
crear-rn
```

o pedirle directamente "generá el release note de <proyecto>". La skill guía la conversación sección por sección.

## `known-components.md`: qué es y cómo se genera

### El problema que resuelve

De cada componente de software de un release (ej. "Portal Estudiantes", "BFF Mobile"), hay datos que **son siempre los mismos, release tras release**: su repositorio, su URL de documentación, el link fijo de su DT de despliegue. No tiene sentido volver a preguntarlos cada vez que se arma un RN nuevo — se cargan una sola vez y se reusan.

Esos datos fijos viven en `skills/crear-rn/references/known-components.md`, un archivo por instalación (no viene incluido en este repo — ver más abajo por qué).

### Cómo se genera (onboarding automático)

Este repo **no trae `known-components.md`** porque esos datos son específicos de cada equipo/proyecto (URLs de repos internos, nombres de servicios, clusters). Lo que sí trae es `known-components.template.md`, la plantilla vacía que describe el formato.

La primera vez que corrés `crear-rn` en un proyecto:

1. La skill busca `known-components.md` y no lo encuentra.
2. Te avisa que es la primera vez y hace onboarding: te pregunta, componente por componente, sus datos fijos — más el documento de arquitectura por defecto y los clusters/entornos QA a consultar.
3. Con tus respuestas, escribe `known-components.md` en esa misma carpeta, siguiendo el formato del template.
4. De ahí en adelante, cada corrida de `crear-rn` lee ese archivo directo — no vuelve a preguntar esos datos.

Ese archivo queda local a tu instalación (con datos reales de tu equipo) y está en `.gitignore`: no se sube a este repo del plugin ni se comparte entre instalaciones.

### Ejemplo concreto

Supongamos un proyecto con dos componentes: un frontend y su BFF. Después del onboarding, `known-components.md` queda así:

```markdown
# Componentes conocidos — Mi Proyecto

### Frontend App
- Aplicación: Frontend App
- service: frontend-app
- ecs: si
- DT Despliegue: https://jira.miempresa.com/browse/PROJ-100
- Repositorio: https://github.com/miempresa/frontend-app
- Documentación: https://frontend-app.miempresa.com/docs

### BFF Frontend App
- Aplicación: BFF Frontend App
- service: frontend-app-bff
- ecs: si
- DT Despliegue: https://jira.miempresa.com/browse/PROJ-101
- Repositorio: https://github.com/miempresa/frontend-app-bff
- Documentación: https://frontend-app-bff.miempresa.com/swagger-ui.html

---

## Documento de arquitectura por defecto

- Link: https://drive.google.com/file/d/xxxxx/view
- Versión: 1.0

## Clusters / entornos QA a consultar

- miproyecto-qa
```

A partir de acá, cuando alguien genere un RN nuevo y elija "Frontend App" en la sección de Paquetes de software, la skill copia directo Aplicación / Repositorio / Documentación / DT Despliegue de este archivo, y solo resuelve en vivo la **Versión** (contra ECS, cluster `miproyecto-qa`) y la **Versión Anterior** (leyendo el RN previo en Drive). No vuelve a preguntar los datos fijos.

Si más adelante se agrega un componente nuevo al proyecto, simplemente se edita `known-components.md` a mano (agregando un bloque `###` más) — no hace falta rehacer el onboarding completo.
