---
name: crear-rn
description: >
  Genera Release Notes para cualquier proyecto siguiendo una plantilla estándar de
  equipo. Pregunta sección por sección (versión, estado, pre-requisitos, DT, alcance,
  arquitectura, paquetes de software, variables de entorno, pruebas, problemas
  resueltos/conocidos, versionado) y autocompleta lo que puede obtener de Linear
  (milestones y bugs), Google Drive (doc de arquitectura y RN previo), AWS ECS
  (última versión desplegada en QA) y repos de configuración locales (variables de
  entorno nuevas en los últimos 40 días). Crea el documento final directamente como
  Google Doc en la carpeta de Drive del release correspondiente. La primera vez que
  se usa en un proyecto hace onboarding para registrar sus componentes.
  Trigger: cuando el usuario diga "crear-rn", "generar release note", "armar RN",
  "release notes de <proyecto>", o pida crear/armar el release note de un proyecto.
license: Apache-2.0
metadata:
  author: schneider
  version: "1.0"
---

# crear-rn

Genera el contenido de un Release Note siguiendo siempre la misma plantilla del equipo. Guía la conversación sección por sección, autocompletando lo que se pueda desde Linear/AWS/git y pidiendo confirmación antes de avanzar. El Markdown final se muestra primero en el chat para revisión y, una vez confirmado, se crea automáticamente como Google Doc en la carpeta de Drive correspondiente al release y proyecto. No se escribe ningún archivo en disco local.

## Archivos de referencia

Antes de generar el output final, leer:
- `references/template.md` — estructura EXACTA del documento final (encabezados, emojis, tablas). El output final debe calcar este esqueleto. Genérico, no depende del proyecto.
- `references/known-components.md` — datos fijos (aplicación, repo, documentación, DT) de cada componente conocido del proyecto, más el documento de arquitectura por defecto y los clusters/entornos QA. Específico de cada equipo/proyecto — no viene incluido al instalar la skill, se genera la primera vez (ver "Onboarding" abajo).
- `references/known-components.template.md` — plantilla vacía que explica el formato esperado de `known-components.md`. Usarla como referencia al hacer el onboarding, nunca como fuente de datos reales.

No dupliques ese contenido dentro de este archivo: siempre leelos en el momento de necesitarlos.

## Onboarding (primera vez en un proyecto nuevo)

Si `references/known-components.md` no existe todavía:
1. Avisarle al usuario que es la primera vez que se corre la skill para este proyecto y que hace falta configurar los componentes una vez.
2. Preguntarle, componente por componente, los mismos campos que pide `references/known-components.template.md` (Aplicación, service, si se resuelve por ECS/orquestador o no, DT Despliegue, Repositorio, Documentación), más el documento de arquitectura por defecto (link + versión) y los clusters/entornos QA a consultar.
3. Escribir la respuesta a `references/known-components.md` con el mismo formato del template.
4. Continuar con el flujo normal (paso 1 en adelante), ya usando ese archivo recién creado.

Si el archivo ya existe, saltar este paso directamente.

## Reglas generales

- Preguntar de a una sección por vez (o un grupo pequeño y lógico de campos dentro de la misma sección), nunca todo junto.
- Cuando un dato se autocompleta (Linear, AWS, git), mostrárselo al usuario y pedir confirmación antes de darlo por bueno. Si el usuario no dice nada en contra ante una propuesta explícita (ej. arquitectura por defecto), se sigue con la propuesta.
- Si una sección "no aplica" (sin pre-requisitos, sin DT, sin variables de entorno nuevas, sin problemas conocidos), respetar lo que indica `template.md` para ese caso (omitir tabla e indicarlo explícitamente) en vez de dejar placeholders sin completar.
- Cualquier campo del template que no se haya preguntado ni autocompletado durante la ejecución (porque el usuario no llegó a esa parte del flujo, la saltó, o quedó fuera de lo cubierto en "Flujo paso a paso") debe dejarse tal cual figura en `references/template.md` (el placeholder literal, ej. `<Responsable>`, `<dd mmm aaaa>`). Nunca inventar un valor ni improvisar un formato distinto para rellenarlo.
- El Markdown final se muestra siempre primero en el chat para revisión; el documento en Drive (ver sección "Creación del documento en Drive") solo se crea después de que el usuario confirma el contenido y la carpeta destino.

## Flujo paso a paso

### 1. Versión del release
Primera pregunta, siempre. Ej: `2.1`.

### 2. Estado del release
Preguntar cuál de: CREADO / DEV COMPLETE / QA COMPLETE / PRODUCTIVO.

### 3. Nombre del proyecto
Si no quedó claro por contexto, preguntarlo (ej. "Portal Estudiantes", "App Mobile", o ambos si el release es conjunto). La introducción se arma sola con nombre + versión, no hace falta preguntarla aparte.

### 4. Pre-requisitos para el despliegue
Preguntar si hay dependencias/pre-requisitos de entorno (versión anterior del producto, versión de Docker, etc.). Si no hay, omitir la tabla e indicarlo explícitamente en el output.

### 5. Deployment Tasks (DT)
Preguntar si hay DTs (tickets de despliegue) asociadas a este release y sus enlaces. Si no aplica, omitir la sección completa.

### 6. Alcance (auto-fetch desde Linear)
1. Usar `mcp__plugin_linear_linear__list_projects` para encontrar el proyecto correspondiente (Portal Estudiantes / App Mobile).
2. Usar `mcp__plugin_linear_linear__list_milestones` sobre ese proyecto y buscar, con matching flexible (case-insensitive, sin exigir el nombre exacto), un milestone cuyo nombre contenga algo como "SIS `<versión>`" junto con "Portal estudiante" o "App Mobile" (el nombre exacto varía, no es literal).
3. Listar los issues/épicas de ese milestone junto con el link de Linear de cada uno (URL del issue) y mostrárselos al usuario como propuesta de la sección Alcance.
4. Si no hay match claro, pedirle al usuario el nombre exacto del milestone o que liste manualmente los issues.

### 7. Arquitectura
Proponer por defecto (desde `references/known-components.md`): nota de "no requiere cambios" + el documento y versión fijos. Preguntar "¿está bien?" — si el usuario no indica lo contrario, usar esa propuesta. Si indica cambios, pedir el/los documento(s) y versión correspondiente.

### 8. Paquetes de software
1. Preguntar al usuario cuáles componentes de `references/known-components.md` van en este release (puede ser una selección múltiple).
2. Resolver la **versión anterior** del release (auto-fetch desde Google Drive), antes de armar los bloques por componente:
   - Calcular el número de versión anterior a partir de la versión de este release (paso 1). Ej: release `4.5` → anterior `4.4`.
   - Buscar en Drive con `mcp__claude_ai_Google_Drive__search_files` un documento cuyo título contenga "RN" + el número de versión anterior + el nombre del proyecto correspondiente (ej. `title contains 'RN' and title contains '4.4'`), acotando por el nombre del proyecto (paso 3) para no traer el documento equivocado si hay varios con el mismo número de versión.
   - Si hay más de un resultado plausible o ninguno, mostrarle la lista (o la ausencia de resultado) al usuario y pedir confirmación de cuál es el documento correcto, o el dato manual si no se encuentra.
   - Leer el documento encontrado con `mcp__claude_ai_Google_Drive__read_file_content` y extraer, de su tabla "💾 Paquetes de software", el valor de **Versión** (no "Versión Anterior") de cada componente — esa es la versión que en el release anterior era la vigente, y por lo tanto la "Versión Anterior" del release actual.
   - Mostrarle al usuario los valores extraídos y pedir confirmación antes de darlos por definitivos.
3. Para cada componente elegido:
   - Copiar tal cual Aplicación / Repositorio / Documentación / DT Despliegue desde `known-components.md`.
   - **Versión**: si `ecs: si`, resolverla en vivo contra AWS (no usar ningún valor fijo):
     ```
     aws ecs describe-services --cluster <cluster-qa> --services <service> \
       --query 'services[0].taskDefinition' --output text
     aws ecs describe-task-definition --task-definition <arn> \
       --query 'taskDefinition.containerDefinitions[0].image' --output text
     ```
     Probar contra los clusters/entornos QA listados en `known-components.md`, matcheando el nombre de servicio por substring. El tag de versión es la parte después del último `:` en el `image`. Formatear como `<service>: <tag>`.
     Si `ecs: no` (ej. una app mobile con versión Android/iOS), pedirle la versión directamente al usuario.
   - **Versión Anterior**: usar el valor extraído del RN previo (punto 2) para ese mismo componente. Si el componente no aparece en el RN previo (ej. es nuevo en este release) o no se pudo resolver el documento anterior, dejarlo vacío e indicarlo explícitamente al usuario.
4. Duplicar el bloque de la sección por cada componente incluido, en el mismo orden en que el usuario los mencionó.

### 9. Variables de Entorno (auto-fetch desde git)
Para cada componente incluido en el paso 8 que tenga un repo de configuración/infra asociado:
1. Localizar ese repo en el directorio local de repos del usuario, buscando carpetas cuyo nombre tokenizado se parezca al del servicio (ej. patrón `*-infra`, `*-config`, u otro que use el equipo — preguntarle al usuario el patrón si no es evidente).
2. Dentro del repo, revisar `git log --since="40 days ago" --oneline` sobre los archivos de configuración de QA, y el diff de esos commits (`git log -p --since="40 days ago" -- <ruta-config-qa>`) para detectar líneas agregadas (`+`) que introduzcan nuevas variables de entorno.
3. Listar las variables nuevas encontradas, SIN su valor (columna Valor vacía).
4. Si no se encuentra el repo o no hay variables nuevas, omitir la tabla e indicarlo explícitamente.

### 10. Reporte de pruebas
Preguntar los valores de "Resultado Release" para: casos de prueba ejecutados, exitosos vs. fallados, defectos/vulnerabilidades bloqueantes, defectos/vulnerabilidades high, resultado final.

### 11. Reporte detallado
Preguntar la URL del test run / reporte de pruebas.

### 12. Problemas resueltos (auto-fetch desde Linear)
1. Usar `mcp__plugin_linear_linear__list_issues` sobre el proyecto correspondiente, filtrando por label/tag "Bug" y estado resuelto/done.
2. Mostrar la lista encontrada (Key, resumen, persona asignada, prioridad, estado) al usuario para confirmar cuáles incluir antes de darlas por definitivas.

### 13. Problemas conocidos
Preguntar directamente al usuario. Si no hay, indicarlo explícitamente en el output en vez de dejar una tabla vacía.

### 14. Versionado del documento
Preguntar las filas (revisión, responsable, fecha, descripción) que quiera incluir. Si el usuario prefiere no detallar todo el historial, ofrecer inferir una sola fila con el estado actual (paso 2) y la fecha de hoy.

## Ensamblado final

Con todos los datos recolectados, generar el Markdown completo siguiendo el esqueleto de `references/template.md` (mismos encabezados, emojis, orden de secciones) y mostrarlo íntegro en la respuesta del chat para que el usuario lo revise antes de crear el documento.

## Creación del documento en Drive

Una vez que el usuario confirma el contenido, crear el Google Doc directamente en la carpeta de Drive que corresponde al release (no pedirle al usuario que lo pegue a mano).

1. **Determinar la carpeta destino** a partir de la versión (paso 1) y el proyecto (paso 3):
   - Usar el nombre del proyecto (paso 3) como palabra clave de carpeta.
   - Buscar con `mcp__claude_ai_Google_Drive__search_files`: `title contains 'RN' and title contains '<versión>' and mimeType = 'application/vnd.google-apps.folder'`.
   - De los resultados, quedarse con el/los que además contengan la palabra clave del proyecto en el título (ej. carpeta `RN <versión> (<aaaa mm>) - <Proyecto>`).
   - Si hay exactamente un match, proponérselo al usuario (nombre y link) y pedir confirmación antes de crear el doc ahí.
   - Si hay más de un match o ninguno, mostrarle la lista (o la ausencia de resultados) y pedirle que indique la carpeta correcta (nombre, link o ID) en vez de asumir.
2. **Nombre del documento**: usar el mismo título que la carpeta encontrada (el equipo nombra el doc igual que su carpeta contenedora, ej. `RN SIS 4.4 (2026 06) - Portal/App mobile`). Si la carpeta no sigue ese patrón, preguntarle al usuario qué título usar.
3. **Crear el doc** con `mcp__claude_ai_Google_Drive__create_file`:
   - `title`: el definido en el punto 2.
   - `parentId`: el ID de la carpeta confirmada en el punto 1.
   - `textContent`: el Markdown final ensamblado.
   - `contentMimeType`: `text/markdown`.
4. Devolver al usuario el link del documento creado (`viewUrl` de la respuesta de `create_file`).
