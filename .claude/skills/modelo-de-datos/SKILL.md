---
name: modelo-de-datos
description: Usa esta skill cuando trabajes con colecciones de Firestore, definas o modifiques campos de templates/records/users, o toques la estructura de Storage para fotos. Cubre convenciones de nombres y qué colecciones existen.
---

# Modelo de datos (Firestore + Storage)

Referencia completa y autoritativa: `docs/03-modelo-de-datos.md`. Esta skill es un resumen
de lo que más se usa al programar — para el detalle exacto de campos, tipos y estructura,
siempre consulta el documento.

## Colecciones existentes (no agregar otras sin documentarlas primero)

- `users` — un documento por usuario. Campo `role`: `field` | `super_admin` | `viewer`.
- `templates` + subcolección `versions` + documento `draft/current` + subcolección
  `reportConfig/current` — plantillas de formulario, versionadas, con borrador separado de
  lo publicado.
- `records` — una respuesta de formulario capturada. **Nunca se borra.** Campo `visibility`
  (`active`/`historial`) según antigüedad de `capturedAt`, no de `syncedAt`.
- `teams` y `cases` — existen en el documento como **propuestas, no confirmadas para v1**.
  No implementar hasta que `docs/08-pendientes.md` las marque como resueltas.

## Reglas que no deben romperse

- `capturedAt` ≠ `syncedAt` — siempre se guardan ambas, nunca se sobreescribe una con otra.
- `templateVersion` en un `record` **nunca se actualiza retroactivamente** aunque la
  plantilla cambie después — así un cambio en la plantilla no rompe respuestas viejas.
- Fotos: máximo 3 por formulario (`maxPhotos`), ya comprimidas antes de subir, en la ruta
  `records/{recordId}/{photoId}.jpg` dentro de un **bucket de Cloudflare R2** (no Firebase
  Storage) — acceso siempre vía la Cloud Function puente que valida Firebase Auth y emite
  URLs firmadas (ver `docs/02-arquitectura-stack.md`).
- IDs de documentos: generados automáticamente por Firestore, no manuales (salvo que el
  documento ya definido diga lo contrario, como `versions/{n}`).

## Convenciones de nombres

Colecciones en inglés plural `camelCase`; campos en inglés `camelCase`; toda referencia a
otra colección se nombra `<coleccionSingular>Id` (`templateId`, `userId`). Detalle completo
en `docs/07-glosario-convenciones.md`.
