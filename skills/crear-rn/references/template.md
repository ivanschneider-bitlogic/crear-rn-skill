# Plantilla de Release Notes

Esta es la estructura EXACTA que debe seguir el Markdown final. Usar los mismos encabezados, emojis y orden. Reemplazar los placeholders `<...>` con los datos recolectados. Cuando una sección indique "si no aplica, eliminar/indicar", respetar esa regla (no dejar tablas vacías con placeholders sin completar).

**Importante — compatibilidad con Google Docs**: el output final se pega como texto plano en Google Docs, que interpreta Markdown al pegar (convierte `#`/`##` en heading styles y tablas con fila separadora `|---|` en tablas reales). Por eso:
- El título va como `# ` (Heading 1) y cada sección como `## ` (Heading 2), incluyendo el emoji dentro del heading.
- TODAS las tablas, incluida "Paquetes de software" y "Arquitectura", deben ser tablas Markdown válidas (con fila de encabezado + fila separadora `|---|---|`), nunca líneas sueltas tipo `Campo | Valor` sin separador — Google Docs no las reconoce como tabla si falta la fila separadora.

```markdown
# Release Notes - <Nombre del Proyecto> <Versión>

Estado del Release: CREADO / DEV COMPLETE / QA COMPLETE / PRODUCTIVO

## 📑 Introducción
Estas notas corresponden al release <Versión> del proyecto <Nombre del Proyecto>. Incluye información sobre su alcance, requerimientos, funcionalidad provista, pruebas realizadas, problemas conocidos y detalles de los componentes entregables.

## 📋 Pre-Requisitos para el despliegue
| Id | Descripción |
|----|-------------|
| 1  | <pre-requisito> |
| 2  | <pre-requisito> |

(Si no hay pre-requisitos o dependencias, eliminar la tabla e indicarlo explícitamente.)

## 🧩 Deployment Tasks (DT)
| Id | Descripción |
|----|-------------|
| 1  | <Enlace a la DT en Jira Siglo> |
| 2  | <Enlace a la DT en Jira Siglo> |

(Si el proyecto no tiene estas dependencias, eliminar esta sección.)

## 🚀 Alcance
| Descripción | Link |
|-------------|------|
| <issue funcional de Linear> | <URL del issue en Linear> |
| <issue funcional de Linear> | <URL del issue en Linear> |

## 🏗️ Arquitectura
(Si no requiere cambios respecto al release anterior, usar la nota: "El Alcance funcional y técnico del presente release Note no requieren cambios y/o actualizaciones en la arquitectura con respecto al Release note anterior, siendo aun valido el documento:" antes de la tabla.)

| Documento | Versión |
|-----------|---------|
| <link o nota de "no requiere cambios"> | <versión> |

## 💾 Paquetes de software

(Duplicar esta tabla por cada componente de software incluido en el release, una tabla por componente.)

| Campo | <nombre-del-servicio> |
|-------|------------------------|
| Aplicación | <nombre-del-servicio> |
| Versión | <nombre-del-servicio>: <número> |
| Versión Anterior | <nombre-del-servicio>: <número> |
| DT Despliegue | <Enlace a la DT con la que se despliega el servicio> |
| Repositorio | <URL del repositorio> |
| Documentación | <Enlace a la documentación / Swagger UI> |

## ✅ Variables de Entorno
| Servicio | Nombre de la variable | Valor |
|----------|-----------------------|-------|
| <servicio> | <NOMBRE_VARIABLE> | <valor> |

(Si no hay cambios, eliminar la tabla e indicarlo.)

## ⏱️📊 Reporte de pruebas
|  | Resultado Esperado | Resultado Release |
|--|---------------------|--------------------|
| Casos de prueba ejecutados | 100% | |
| Exitosos vs. fallados | | |
| Defectos/Vul. Bloqueantes | 0 | |
| Defectos/Vul. High | | |
| Resultado Final | ACEPTADO | |

## 🧐 Reporte detallado
<URL del test run / reporte de pruebas>

## ✅ Problemas Resueltos
| Type | Key | Resumen | Persona asignada | Prioridad | Estado |
|------|-----|---------|-------------------|-----------|--------|
| 🐞 | <ID-####> | <Resumen del defecto> | <Responsable> | <Prioridad> | <Estado> |

## 🐛 Problemas Conocidos
(Si no hay problemas conocidos, indicarlo explícitamente. De lo contrario, listar en tabla con el mismo formato que la sección anterior.)

## ✍️ Versionado del Documento
| Revisión | Responsable | Fecha | Descripción |
|----------|-------------|-------|-------------|
| 1.1 | @<Responsable> | <dd mmm aaaa> | DEV COMPLETE |
| 1.2 | @<Responsable> | <dd mmm aaaa> | Actualización de pruebas |
| 1.3 | @<Responsable> | <dd mmm aaaa> | QA COMPLETE |
```
