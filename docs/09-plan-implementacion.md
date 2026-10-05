# Plan de implementación por fases

El objetivo de este documento es definir **en qué orden se construye** el proyecto, no qué
se construye (eso ya está en los documentos anteriores). El criterio para ordenar las fases
es: primero lo que hace que el sistema funcione de punta a punta aunque sea con lo mínimo
(un trabajador captura, se sincroniza, el admin lo ve), y después todo lo que mejora esa
base — nunca al revés, para evitar construir una pieza vistosa (como el reporte con marca)
sobre una base que todavía no funciona.

Siendo un solo desarrollador, cada fase debe dejar algo demostrable y usable, no solo código
a medias.

## Fase 0 — Preparación

- Crear el monorepo con la estructura definida en `02-arquitectura-stack.md`
  (`apps/mobile`, `apps/admin-web`, `apps/functions`, `packages/shared-types`).
- Crear el proyecto de Firebase (Firestore, Auth) para este cliente — **ya no incluye
  Storage**, eso ahora vive en R2 (ver siguiente punto).
- Crear una cuenta de Cloudflare y un bucket de R2 para este cliente, con su API token.
- Configurar alertas de presupuesto en Google Cloud (ver `04-reglas-seguridad.md`) —
  **antes** de que exista cualquier dato real, no después. Revisar también si Cloudflare
  tiene un equivalente de alertas de gasto para R2 y activarlo igual de temprano.
- Definir en `packages/shared-types` los tipos de `users`, `templates` (con su subcolección
  `versions`) y `records`, tal como están en `03-modelo-de-datos.md`.

## Fase 1 — Captura y sincronización (el núcleo del proyecto)

Esta es la fase que de verdad valida la idea completa: sin esto, nada más importa.

- App móvil: login con Firebase Auth, un trabajador puede llenar un formulario **fijo** (sin
  diseñador todavía — la plantilla se crea directo en Firestore a mano, por script o consola,
  para esta fase).
- Captura offline con persistencia local del SDK de Firestore.
- Compresión de fotos obligatoria antes de guardar localmente (resize + JPEG 80%, ver
  `02-arquitectura-stack.md`) — se construye desde esta fase, no se deja para después, porque
  es más fácil evitar fotos sin comprimir desde el principio que corregirlo con datos reales
  ya subidos.
- **Cloud Function puente para R2:** valida el token de Firebase Auth y emite URLs firmadas
  de subida/descarga hacia el bucket de R2 (ver `02-arquitectura-stack.md`). Se construye en
  esta fase porque sin ella no hay forma de subir una foto — no es opcional ni se puede
  posponer a una fase posterior.
- Cola de subida de fotos pendientes en el teléfono: la foto se guarda localmente y se marca
  como pendiente en Firestore (eso sí sincroniza solo); la subida real del archivo a R2 se
  dispara aparte cuando hay conexión, en lotes controlados (ver conversación sobre
  concurrencia) en vez de todas a la vez.
- Sincronización automática al recuperar conexión, con `capturedAt` y `syncedAt` correctos.
- **Entregable demostrable:** un trabajador llena un formulario sin internet, lo sincroniza
  al recuperar señal, y el registro aparece en Firestore con sus fotos comprimidas.

## Fase 2 — Panel admin básico y diseñador de plantillas

- Dashboard web (`apps/admin-web`) con login, rol `super_admin` funcionando.
- Diseñador de plantillas: crear campos, tipos, `required`, `order`, `maxPhotos` — con el
  flujo de borrador → publicar definido en `03-modelo-de-datos.md`.
- Vista de registros activos (últimos 3 días) e historial, de todos los trabajadores.
- **Entregable demostrable:** el `super_admin` diseña una plantilla desde el dashboard sin
  tocar código, la publica, y un trabajador la ve reflejada en su teléfono.

## Fase 3 — Roles y reglas de seguridad

- Implementar las reglas de seguridad de Firestore completas de `04-reglas-seguridad.md`
  (lectura/escritura por rol, cada trabajador solo ve lo suyo).
- Agregar el rol `viewer` (solo lectura + exportar, sin editar).
- Probar las reglas con el emulador de Firestore antes de pasar a la siguiente fase — no
  informalmente, con pruebas reales (ver pendiente de QA abajo).
- **Entregable demostrable:** una cuenta `viewer` puede ver y exportar, pero el dashboard le
  bloquea cualquier intento de editar una plantilla o un registro.

## Fase 4 — Exportación genérica (Excel + fotos)

- Cloud Function que genera el ZIP (Excel multi-pestaña + carpeta de fotos) bajo demanda,
  como está definido en `02-arquitectura-stack.md` y `03-modelo-de-datos.md`.
- **Entregable demostrable:** el `super_admin` descarga un ZIP con los registros de un día,
  organizados por plantilla, con las fotos correctamente vinculadas por ID.

## Fase 5 — Reporte con marca (diseñador + generador)

Se deja para después de que el resto del sistema ya esté funcionando, porque depende de que
ya existan registros reales y plantillas reales sobre las cuales diseñar el reporte — y
porque su diseño visual exacto todavía está pendiente (ver `08-pendientes.md`).

- Diseñador de `reportConfig` en el dashboard: logo, qué campos van en el reporte, orden,
  posición del bloque de fotos.
- Generador HTML → PDF vía Cloud Function.
- **Entregable demostrable:** el `super_admin` genera el PDF de marca de un registro real y
  se ve organizado y profesional, con el logo del cliente.

## Fase 6 — Mejoras opcionales (solo si se confirman)

Estas no se construyen a menos que se resuelvan primero como pendientes (ver
`08-pendientes.md`):

- Colección `cases` y agrupación de formularios por caso.
- Visibilidad compartida por equipo (`teamId`).
- Roles administrativos intermedios adicionales.
- Firma digital (fase futura, fuera de v1 por decisión ya tomada).

## Pendiente de este documento

- **Estrategia de pruebas (QA):** el proyecto no tiene definida todavía una estrategia de
  pruebas, y el escenario offline es justo el más difícil de probar manualmente (requiere
  simular pérdida de conexión, sincronización tardía, y conflictos). Como mínimo se
  recomienda: pruebas del emulador de Firestore para las reglas de seguridad (Fase 3), y
  pruebas manuales estructuradas de offline/sync antes de dar la Fase 1 por cerrada. No se
  define aquí un plan de pruebas completo — se deja como pendiente a desarrollar cuando se
  llegue a esa fase.
