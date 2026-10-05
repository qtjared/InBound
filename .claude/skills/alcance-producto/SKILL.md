---
name: alcance-producto
description: Usa esta skill cuando implementes una feature de cara al usuario y necesites saber si está confirmada, fuera de alcance, o pendiente de decidir — por ejemplo firma digital, equipos, casos, o campos obligatorios.
---

# Alcance del producto

Referencia completa: `docs/01-vision-alcance.md`.

## Explícitamente fuera de la v1 (no implementar)

- Firma digital en los formularios — no se descarta para el futuro, pero no entra ahora.
- Integración con Google Drive/Sheets como almacenamiento.
- Generación automática/periódica de reportes — todo es bajo demanda.
- Cualquier sistema de pagos o cobro dentro de la app.

## Confirmado, pero con un detalle sin cerrar (revisar antes de construir)

- Edición de un `record` ya sincronizado: **sí se permite**, pero la condición exacta
  (confirmación manual, hora de corte, u otra) no está definida — no implementar la
  escritura hasta que `docs/08-pendientes.md` lo marque como resuelto.
- "Caso" (agrupa formularios distintos de varios usuarios sobre la misma situación): el
  concepto existe, pero su mecánica de creación/asignación no está definida — no
  implementar la colección `cases` todavía.
- Visibilidad por equipo (`teamId`): propuesta, no confirmada para v1 — por defecto, un
  usuario de campo solo ve sus propios registros.

## Confirmado y sin ambigüedad

- 3 fotos máximo por formulario, con compresión obligatoria.
- Registros nunca se eliminan — después de 3 días pasan a "Historial", solo se ocultan.
- Exportación a Excel/ZIP y reporte con marca son ambas bajo demanda, no automáticas.
- Campos obligatorios (`required`) se definen por campo, dentro de cada plantilla — no es
  una decisión global del sistema.
