# Modelo de datos

Este documento define la estructura exacta de datos en Firestore y Storage. Cualquier
colección, campo o convención que no esté aquí **no debe inventarse** — se agrega a este
archivo primero, se discute, y luego se implementa.

## Convenciones generales

- Nombres de colecciones: en inglés, plural, `camelCase` cuando aplique (`records`,
  `templates`, `users`).
- Nombres de campos dentro de documentos: `camelCase` (`capturedAt`, `templateId`).
- IDs de documentos: generados automáticamente por Firestore (no IDs manuales), excepto
  donde se indique lo contrario.
- Fechas: se guardan como `Timestamp` de Firestore, nunca como string libre.
- Todo campo de referencia a otra colección se nombra `<nombreColeccionSingular>Id`
  (ejemplo: `templateId`, `userId`).

## Colección `users`

Un documento por usuario (de campo o administrador).

| Campo | Tipo | Descripción |
|---|---|---|
| `name` | string | Nombre del usuario |
| `role` | string | `"field"` \| `"super_admin"` \| `"viewer"` — ver "Roles" abajo |
| `teamId` | string \| null | Referencia a la colección `teams`. Ver nota de equipos abajo — campo preparado desde ahora aunque la funcionalidad de visibilidad compartida no esté activa en v1 |
| `phone` | string \| null | Opcional, para identificarlo en WhatsApp |
| `createdAt` | Timestamp | Fecha de alta |
| `active` | boolean | Si el usuario puede seguir capturando (para dar de baja sin borrar historial) |

### Roles (confirmados: `field`, `super_admin`, `viewer` — campo único, decisión cerrada)

Se evaluó dividir `role` en dos campos separados (uno para permisos de mobile, otro para
permisos del dashboard web), pero **se descartó**: confirmado que la app móvil solo tiene un
tipo de usuario (el trabajador, rol `field`) — toda la variación de permisos vive del lado
del dashboard web (`super_admin` / `viewer`, a los que el cliente se refiere en general como
"supervisor"). Con un solo tipo de usuario por plataforma, un campo único `role` es
suficiente y más simple — no se necesita la separación.

- `"field"` — usuario de campo (trabajador). Único rol del lado móvil. Confirmado, sin
  cambios.
- `"super_admin"` — **confirmado**: rol superior con acceso total (plantillas, todos los
  registros, exportaciones, generación de reportes con marca, gestión de cuentas). Usa el
  dashboard web (`apps/admin-web`), que es la misma aplicación de panel de administración ya
  definida — no hay una app de dashboard separada.
- `"viewer"` — **confirmado**: acceso al dashboard para visualizar (reportes, registros,
  historial) **y exportar/descargar** (Excel genérico y reporte con marca). No puede editar
  plantillas, no puede modificar registros, no puede gestionar cuentas de otros usuarios.
- Roles administrativos intermedios adicionales (por ejemplo uno que pueda editar
  plantillas pero no gestionar cuentas) — **no confirmados todavía**, no implementar hasta
  que se definan explícitamente (ver `04-reglas-seguridad.md`).

### Colección `teams` (propuesta, no confirmada para v1)

Agrupa usuarios de campo que trabajan juntos. Existe como propuesta para permitir que un
usuario vea los formularios de sus compañeros de equipo sobre un mismo caso — **no está
confirmado que esta visibilidad compartida se construya en la v1**; por defecto, un usuario
de campo solo ve sus propios registros (ver regla de visibilidad en `04-reglas-seguridad.md`).

| Campo | Tipo | Descripción |
|---|---|---|
| `name` | string | Nombre del equipo |
| `createdAt` | Timestamp | — |

## Colección `cases` (nuevo — mecánica exacta pendiente de definir)

Un "caso" agrupa varios registros de formulario distintos que distintos usuarios de campo
llenan sobre la misma situación de trabajo (ejemplo: varios tipos de formulario capturados
por personas distintas sobre el mismo sitio o evento). Esto es un concepto nuevo introducido
después de una conversación de aclaración — **la mecánica de cómo se crea un caso, quién lo
crea, y cómo se asignan usuarios a él todavía no está definida**. Estructura mínima
propuesta, sujeta a cambio:

| Campo | Tipo | Descripción |
|---|---|---|
| `name` \| `label` | string | Identificador visible del caso — **por definir el criterio** (¿número consecutivo? ¿nombre de sitio?) |
| `createdBy` | string | `userId` de quien lo creó — **por definir si lo crea un admin o un usuario de campo** |
| `createdAt` | Timestamp | — |
| `status` | string | Por definir (¿`"open"` / `"closed"`?) |

Cada documento en `records` se relaciona con un caso mediante un campo `caseId` (ver abajo).
**No implementar esta colección ni el campo `caseId` en `records` hasta que la mecánica de
arriba se cierre** — se documenta aquí para que quede registrado el requisito, no como luz
verde para construirlo tal cual.

## Colección `templates`

Un documento por plantilla de formulario. El contenido de los campos **no vive aquí
directamente** — vive versionado en la subcolección `versions` (ver abajo), para que
cambios futuros no rompan respuestas anteriores.

| Campo | Tipo | Descripción |
|---|---|---|
| `name` | string | Nombre visible de la plantilla |
| `currentVersion` | number | Número de la versión activa actualmente (empieza en 1) |
| `status` | string | `"active"` o `"archived"` |
| `createdBy` | string | `userId` del administrador que la creó |
| `createdAt` | Timestamp | — |

### Documento `templates/{templateId}/reportConfig/current` (nuevo — propuesto)

Configuración del diseño del reporte con marca para esta plantilla, editada por el
`super_admin` desde el dashboard (ver `02-arquitectura-stack.md`, componente 5). Separado de
`versions` porque cambiar el diseño visual del reporte no debería requerir una nueva versión
del formulario de captura — son dos cosas distintas (ver "Plantilla" vs. "Plantilla de
reporte" en `07-glosario-convenciones.md`).

| Campo | Tipo | Descripción |
|---|---|---|
| `logoUrl` | string | Referencia al logo de la empresa en Storage |
| `fieldLayout` | array de objetos | Qué `fieldId` de la plantilla aparece en el reporte, en qué sección y en qué orden — estructura exacta por definir cuando se construya el diseñador |
| `photoBlockPosition` | string | Dónde va el bloque de fotos en el reporte — valores exactos por definir |
| `updatedAt` | Timestamp | — |

> Estructura preliminar — los campos exactos de `fieldLayout` y `photoBlockPosition` se
> deben precisar cuando se diseñe la interfaz del diseñador de reportes en el dashboard, no
> antes.
| `updatedAt` | Timestamp | Última vez que se publicó una nueva versión |

### Documento `templates/{templateId}/draft/current` (borrador — propuesto)

Copia de trabajo editable de la plantilla, separada de las versiones publicadas. El admin
edita aquí libremente (agregar, quitar, reordenar campos) sin que nada cambie todavía para
los trabajadores. La acción de **"Publicar"** en el dashboard toma el contenido de este
documento y crea la siguiente versión en `templates/{templateId}/versions/{n}` — recién ahí
la app móvil empieza a descargarla.

| Campo | Tipo | Descripción |
|---|---|---|
| `fields` | array de objetos | Misma estructura que en una versión publicada (ver abajo) |
| `maxPhotos` | number | — |
| `basedOnVersion` | number | Versión publicada sobre la que se partió al crear este borrador, para referencia |
| `updatedAt` | Timestamp | Último cambio guardado en el borrador (autoguardado mientras el admin edita) |

> Esta separación borrador/publicado es una propuesta de diseño para resolver la pregunta de
> cómo editar sin afectar a los trabajadores a medio camino — no viene de una confirmación
> explícita del cliente. Si se prefiere un flujo más simple (cada guardado publica de una
> vez, sin borrador), se debe decidir y actualizar esta sección antes de implementar.

### Subcolección `templates/{templateId}/versions`

Un documento por cada versión publicada de la plantilla. El ID del documento es el número
de versión como string (`"1"`, `"2"`, ...).

| Campo | Tipo | Descripción |
|---|---|---|
| `fields` | array de objetos | Definición de los campos del formulario (ver estructura abajo) |
| `publishedAt` | Timestamp | Cuándo se publicó esta versión |
| `maxPhotos` | number | Tope de fotografías permitidas en este formulario — confirmado en 3 (ver `01-vision-alcance.md`) |

Estructura de cada elemento en `fields` (misma estructura se usa en el borrador):

```
{
  "fieldId": "string — identificador único del campo dentro de la plantilla",
  "label": "string — texto visible al usuario",
  "type": "text | number | select | photo",
  "required": true | false,
  "order": "number — posición del campo en el formulario, define el orden que ve el usuario",
  "options": ["solo si type es 'select'"]
}
```

> Nota: `type: "photo"` es obligatorio en al menos un campo de toda plantilla de la v1,
> porque las fotografías son requisito confirmado (ver `01-vision-alcance.md`), con un
> máximo de 3 fotos por formulario. No se incluye `type: "signature"` en la v1 — queda
> fuera de alcance por ahora, aunque se considera para una fase futura.

## Colección `records`

Un documento por cada respuesta de formulario capturada por un usuario de campo. Esta es la
colección de mayor volumen del sistema.

| Campo | Tipo | Descripción |
|---|---|---|
| `templateId` | string | Referencia a la plantilla usada |
| `templateVersion` | number | Versión exacta de la plantilla con la que se llenó — **nunca se actualiza retroactivamente** aunque la plantilla cambie después |
| `userId` | string | Quién capturó el registro |
| `caseId` | string \| null | Referencia al caso al que pertenece (ver colección `cases` arriba) — **no implementar hasta cerrar la mecánica de casos** |
| `values` | map | Respuestas del formulario, con clave = `fieldId` y valor según el `type` definido en la versión de la plantilla |
| `photoIds` | array de strings | IDs de las fotos asociadas (ver estructura de Storage abajo), máximo 3 elementos |
| `capturedAt` | Timestamp | Fecha y hora real en que el usuario llenó el formulario (se genera en el teléfono, no al sincronizar) |
| `syncedAt` | Timestamp | Fecha y hora en que el registro llegó al servidor |
| `visibility` | string | `"active"` si `capturedAt` tiene 3 días o menos de antigüedad, `"historial"` si tiene más — ver nota de cálculo abajo |
| `editableUntil` | Timestamp \| null | **Pendiente de definir.** Si la regla de edición termina siendo "hasta una hora de corte fija", este campo guardaría ese límite. Si la regla termina siendo "hasta que el usuario confirme", este campo no se usa y se reemplaza por un booleano `confirmedByUser`. No implementar ninguno de los dos hasta que `01-vision-alcance.md` tenga la regla cerrada. |

> **Sobre `visibility`:** este campo puede calcularse en el momento de la consulta
> (comparando `capturedAt` contra la fecha actual) en lugar de guardarse fijo, para evitar
> tener que actualizar documentos solo por el paso del tiempo. La decisión de implementación
> (campo calculado vs. campo almacenado y actualizado por una tarea programada) se deja a
> criterio técnico del desarrollador, pero el comportamiento observable — 3 días activo,
> después a historial, nunca se borra — es fijo y no debe cambiar.

## Estructura de almacenamiento de fotografías (Cloudflare R2)

**Confirmado:** las fotos viven en un bucket de **Cloudflare R2**, no en Firebase Storage
(ver razón completa en `02-arquitectura-stack.md`). La ruta/key de cada fotografía dentro
del bucket sigue la misma lógica que ya estaba definida:

```
records/{recordId}/{photoId}.jpg
```

- `recordId` es el ID del documento en la colección `records` al que pertenece la foto.
- `photoId` es un identificador único por foto (puede ser un UUID generado en el cliente) —
  es el mismo valor que vive en el array `photoIds` de ese `record`.
- Todas las fotos se suben ya comprimidas (ver regla de compresión en
  `02-arquitectura-stack.md`) — nunca se sube una foto sin procesar.
- **Acceso:** ni la app móvil ni el dashboard hablan con R2 directamente. Piden una URL
  firmada temporal a una Cloud Function (ver `02-arquitectura-stack.md`), que valida el
  token de Firebase Auth y el permiso sobre el `recordId` antes de generarla — la misma
  lógica de permisos de `04-reglas-seguridad.md`, aplicada manualmente porque R2 no conoce a
  Firebase Auth.

## Qué NO existe en este modelo (para evitar que se invente)

- No hay colección de "borradores" separada de `records` — un registro sin sincronizar
  simplemente vive en el almacenamiento local del teléfono hasta que se sube.
- No hay borrado físico de `records` en la v1. Ninguna función debe implementar un `delete`
  sobre esta colección salvo que se agregue explícitamente a este documento primero.
- No hay edición libre de `records` sin restricción — está confirmado que un usuario de
  campo puede editar su propio registro ya sincronizado, pero **solo bajo la condición que
  se cierre en `01-vision-alcance.md`** (confirmación manual, hora de corte, u otra). No
  implementar edición sin esa regla definida.
- No hay integración con Google Sheets/Drive como fuente de datos (ver `01-vision-alcance.md`).
- No hay sistema de pagos ni facturación en la app (ver `05-plan-negocio-presupuesto.md`).
