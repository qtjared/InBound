# Pendientes abiertos — checklist único

Este archivo consolida **todas** las decisiones que faltan por cerrar en el proyecto, sin
importar de qué documento vienen. El detalle completo de cada pendiente vive en su documento
de origen (enlazado abajo) — aquí solo está el resumen y el estado, para que se pueda ver
todo de un vistazo sin tener que buscar en los 8 archivos.

**Regla de uso:** cuando se cierre un pendiente, se actualiza esta lista (marcar como
resuelto o eliminar la fila) **y** se actualiza el documento de origen en el mismo momento —
nunca uno sin el otro, para que no se desincronicen.

## Producto / alcance

| Pendiente | Dónde se detalla | Bloquea construir |
|---|---|---|
| Condición exacta para editar un registro ya sincronizado (confirmación manual, hora de corte, u otra) | `01-vision-alcance.md` (punto 11), `03-modelo-de-datos.md` (`editableUntil`), `04-reglas-seguridad.md` | Sí — la regla de escritura de `records` no se puede implementar sin esto |
| Mecánica del "caso": quién lo crea, cómo se asignan usuarios, si tiene plantillas fijas asociadas | `01-vision-alcance.md`, `03-modelo-de-datos.md` (colección `cases`) | Sí — la colección `cases` y el campo `caseId` en `records` no se implementan hasta cerrar esto |
| Si la visibilidad por equipo (`teamId`) se construye en la v1 o se pospone | `01-vision-alcance.md`, `04-reglas-seguridad.md` | No bloquea nada más — es una mejora aislada, se puede decidir después |
| Si existen roles administrativos intermedios además de `super_admin` y `viewer` | `01-vision-alcance.md` (punto 16), `03-modelo-de-datos.md`, `04-reglas-seguridad.md` | No bloquea el MVP — con `super_admin`/`viewer` ya se puede construir |

## Infraestructura técnica

| Pendiente | Dónde se detalla | Bloquea construir |
|---|---|---|
| Implementación exacta de la Cloud Function puente entre Firebase Auth y R2 (validación de token + emisión de URL firmada) | `02-arquitectura-stack.md`, `03-modelo-de-datos.md` | Sí — bloquea subir/ver fotos, se construye en Fase 1 |

## Diseño del reporte con marca

| Pendiente | Dónde se detalla | Bloquea construir |
|---|---|---|
| Detalle visual exacto del reporte (colores, disposición del bloque de fotos, estructura de `fieldLayout`) | `01-vision-alcance.md` (punto 14), `03-modelo-de-datos.md` (`reportConfig`) | Sí — pero solo bloquea el diseñador de reportes, no el resto del proyecto (ver plan de fases) |

## Negocio / operación

| Pendiente | Dónde se detalla | Bloquea construir |
|---|---|---|
| Quién cubre el costo operativo mensual de Firebase en producción (desarrollador o cliente) | `05-plan-negocio-presupuesto.md` | No — es una decisión administrativa, no técnica |

## Legal

| Pendiente | Dónde se detalla | Bloquea construir |
|---|---|---|
| Si el cliente ya tiene un aviso de privacidad para su personal de campo | `06-legal-privacidad.md` | Bloquea **lanzar a producción con usuarios reales**, no bloquea desarrollar |
| Responsable vs. encargado del tratamiento de datos (quién es cuál entre cliente y desarrollador) | `06-legal-privacidad.md` | Bloquea producción, no desarrollo |
| Si la retención permanente de fotos choca con alguna obligación legal del giro del cliente | `06-legal-privacidad.md` | Bloquea producción, no desarrollo |

## Resueltos recientemente (referencia, no acción)

Se deja registro breve de lo ya cerrado para no tener que ir a confirmar en cada documento:

- Viewer sí puede exportar/descargar — confirmado.
- Mobile tiene un solo rol (`field`); no se divide el campo `role` por plataforma — confirmado.
- Reporte con marca se diseña dentro de la app, no se replica el PDF existente del cliente — confirmado.
- 3 fotos máximo por formulario, con compresión obligatoria — confirmado.
- Firma digital fuera de v1, pero abierta a una fase futura — confirmado.
- Fotografías se almacenan en Cloudflare R2, no en Firebase Storage (egreso $0 vs. costo
  creciente) — confirmado.
