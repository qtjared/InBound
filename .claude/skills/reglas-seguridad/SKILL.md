---
name: reglas-seguridad
description: Usa esta skill cuando escribas o modifiques reglas de seguridad de Firestore, la Cloud Function puente de acceso a R2, implementes control de acceso por rol, o trabajes cualquier cosa relacionada con permisos de field/super_admin/viewer.
---

# Reglas de seguridad y roles

Referencia completa: `docs/04-reglas-seguridad.md`. Este resumen cubre lo esencial para no
romper el control de acceso al programar.

## Roles confirmados

- `field` (trabajador, solo mobile): crea sus propios `records`, los puede editar solo bajo
  una condición **todavía sin definir** (ver `docs/08-pendientes.md`) — mientras tanto, no
  implementar ninguna escritura de edición. No tiene acceso al dashboard web.
- `super_admin` (dashboard web): acceso total — plantillas, todos los registros, gestión de
  cuentas, exportaciones, reportes con marca.
- `viewer` (dashboard web): lectura de todo **y exportar/descargar**, pero no puede editar
  nada (ni plantillas ni registros ni cuentas).

**Fail-safe obligatorio:** cualquier cuenta administrativa sin rol definido en el código se
trata como `viewer` (el más restrictivo), nunca como `super_admin`.

## Reglas que no deben romperse al escribir Firestore Rules

- Un `field` solo lee/escribe `records` donde `userId` sea el suyo — nunca los de otro
  usuario (salvo que la visibilidad por equipo se confirme — hoy no está confirmada).
- Nadie borra documentos de `records` — ni `field` ni `super_admin`. No implementar ningún
  `delete` sobre esta colección.
- `templates/draft` y `templates/.../reportConfig` solo los lee/escribe `super_admin` — un
  `field` nunca los necesita, y `viewer` solo los lee, no escribe.
- Las fotos viven en R2, no en Firebase Storage — no hay reglas declarativas como las de
  Firestore. El control de acceso se aplica a mano dentro de la Cloud Function puente que
  emite las URLs firmadas (ver `docs/02-arquitectura-stack.md` y `docs/04-reglas-seguridad.md`).

## Control de gasto (antes de producción)

Firestore no tiene tope de gasto automático. Antes de que existan usuarios reales:
configurar alertas de presupuesto en Google Cloud Billing, revisar que todo `onSnapshot` se
cierre correctamente, y probar las reglas con el emulador de Firestore — no solo revisarlas
manualmente. Detalle en `docs/04-reglas-seguridad.md`.
