# Componentes conocidos — <Nombre del Proyecto>

Plantilla vacía. Cuando la skill `crear-rn` no encuentra un `known-components.md` junto a este archivo, hace onboarding: pregunta estos mismos datos y genera `known-components.md` con el mismo formato que este archivo, reemplazando `known-components.md` como fuente de verdad para las próximas ejecuciones.

Datos fijos por componente. Al armar la sección "💾 Paquetes de software", copiar tal cual Aplicación / Repositorio / Documentación / DT Despliegue. La Versión y Versión Anterior NO se copian de acá: la Versión se resuelve en vivo contra el orquestador de contenedores (o se pregunta al usuario si no aplica), y la Versión Anterior se resuelve leyendo el RN de la versión previa en Google Drive (ver SKILL.md, paso 8).

Campo `service` = nombre del servicio/imagen a buscar en el cluster/entorno de despliegue. Campo `ecs` = `si`/`no` (si es `no`, no se busca en el orquestador, se le pide el dato al usuario directamente — por ejemplo apps mobile con versión Android/iOS).

---

### <Nombre del componente>
- Aplicación: <Nombre del componente>
- service: <nombre-del-servicio-o-imagen>
- ecs: si | no
- DT Despliegue: <link o "pedir al usuario">
- Repositorio: <URL del repositorio>
- Documentación: <URL de documentación / Swagger>

(Repetir un bloque `###` por cada componente del proyecto.)

---

## Documento de arquitectura por defecto

- Link: <URL del documento de arquitectura>
- Versión: <versión>
- Usar como default en la sección "🏗️ Arquitectura" con la nota de "no requiere cambios", salvo que el usuario indique lo contrario para ese release.

## Clusters / entornos QA a consultar

- <cluster-o-entorno-qa-1>
- <cluster-o-entorno-qa-2>
