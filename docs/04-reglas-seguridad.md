# Reglas de seguridad y control de acceso

## Principio general

Ningún usuario de campo debe poder leer o modificar datos de otro usuario. Ningún usuario
sin sesión (no autenticado) debe poder leer o escribir nada. El panel de administración es
el único lugar con acceso de lectura amplio a todos los registros.

## Roles

Definidos en `users.role` (ver `03-modelo-de-datos.md`):

- **`field`** (trabajador, único rol del lado de la app móvil): puede crear registros
  propios, editar sus propios registros bajo la condición pendiente de definir (ver sección
  `records` abajo), no puede leer registros de otros usuarios salvo la excepción de equipos
  descrita abajo, no puede modificar plantillas, no tiene acceso al dashboard web.
- **Roles administrativos (confirmados `super_admin` y `viewer`, ambos exclusivos del
  dashboard web; posibles intermedios pendientes):**
  - `"super_admin"` — **confirmado**: es el rol superior, con acceso total (ve y controla
    todo: plantillas, todos los registros de todos los trabajadores, exportaciones,
    generación de reportes con marca, gestión de cuentas de otros usuarios). Usa el
    dashboard web (`apps/admin-web`) conectado directo a Firestore.
  - `"viewer"` — **confirmado**: puede entrar al dashboard, leer plantillas, registros
    (activos e historial) y reportes generados, **y generar/descargar exportaciones**
    (Excel genérico y reporte con marca). No puede crear ni modificar plantillas, no puede
    modificar registros, no puede gestionar cuentas de otros usuarios.
  - Roles administrativos intermedios adicionales (por ejemplo uno que pueda editar
    plantillas pero no gestionar cuentas) — **no confirmados**, no implementar hasta
    definirse explícitamente aquí.
  - Mientras algún rol administrativo no esté definido en el código, debe tratarse por
    defecto como `viewer` (el más restrictivo), nunca como `super_admin` — fail-safe: más
    restrictivo que lo confirmado, nunca menos.

## Visibilidad entre usuarios de campo (equipos — propuesto, no confirmado)

Por defecto, un usuario de campo **solo** puede leer sus propios registros, incluso si
comparte un mismo "caso" con otros usuarios (ver `03-modelo-de-datos.md`) — permitir que
vea los de sus compañeros generaría más lecturas (costo) sin ser indispensable.

Se considera, como mejora futura, permitir que un usuario de campo vea los registros de sus
compañeros **si y solo si pertenecen al mismo `teamId`**. Esto no está confirmado para la
v1. Si se implementa, la regla sería: lectura permitida cuando
`request.auth.uid` pertenece al mismo equipo que el `userId` del registro solicitado — pero
esta regla no debe escribirse en producción hasta que se confirme explícitamente en
`01-vision-alcance.md`.

## Reglas por colección (Firestore)

### `users`

- Lectura: un usuario solo puede leer su propio documento. Los administradores pueden leer
  todos.
- Escritura: solo administradores pueden crear o modificar usuarios (alta/baja de personal
  de campo).

### `templates`, su subcolección `versions`, su borrador y su `reportConfig`

- Lectura de `versions` (formulario): cualquier usuario autenticado (de campo o admin)
  puede leer plantillas con `status: "active"`. Las archivadas solo las leen `super_admin` y
  `viewer`.
- Lectura de `draft` y `reportConfig`: solo roles administrativos (`super_admin`, `viewer`)
  — un usuario de campo nunca necesita verlos.
- Escritura de `versions`, `draft` y `reportConfig`: solo `super_admin`. `viewer` no puede
  escribir en ninguno de los tres, es justamente la diferencia entre ambos roles.

### `records`

- Lectura: un usuario de campo solo puede leer los registros donde `userId` sea el suyo (ver
  excepción de equipos arriba, no confirmada). Los administradores pueden leer todos.
- Creación: un usuario autenticado solo puede crear registros con su propio `userId` — no
  puede crear un registro a nombre de otro usuario.
- Modificación: **confirmado que sí se permite** que un usuario de campo edite su propio
  registro ya sincronizado, pero **bajo una condición todavía sin definir** — candidatos
  discutidos: hasta que el usuario confirme manualmente que el registro está correcto, o
  hasta una hora de corte fija (ejemplo: fin de turno a las 6 pm). **No implementar la regla
  de escritura de modificación hasta que `01-vision-alcance.md` tenga la condición cerrada**
  — mientras tanto, no hay modificación permitida en el código (fail-safe: más restrictivo
  que lo confirmado, nunca menos).
- Borrado: no permitido para nadie. Los administradores tampoco borran registros — ver
  regla de retención en `03-modelo-de-datos.md`.

### Fotos en R2 (`records/{recordId}/*`)

**Confirmado:** las fotos viven en Cloudflare R2, no en Firebase Storage (ver
`02-arquitectura-stack.md`). R2 no conoce a Firebase Auth, así que estas reglas **no se
aplican como reglas declarativas** (no existe un equivalente a Firebase Storage Rules) —
se aplican a mano dentro de la Cloud Function puente, que es el único punto de acceso:

- Subida: la función verifica el token de Firebase Auth del solicitante y que sea el
  dueño del `recordId` correspondiente (cruzando con el documento en Firestore) antes de
  emitir una URL firmada de subida.
- Lectura: misma verificación — el dueño del registro y los administradores
  (`super_admin`/`viewer`) pueden pedir una URL firmada de descarga; nadie más.
- Ni la app móvil ni el dashboard tienen las credenciales de R2 directamente — solo la
  Cloud Function las tiene, y nunca las expone al cliente.

## Control de gasto (obligatorio antes de pasar a producción)

Firestore no tiene un tope de gasto automático como sí lo tiene Supabase por defecto. Esto
significa que un error de código (por ejemplo, una consulta sin filtro que se ejecuta en
bucle, o un listener `onSnapshot` que no se cierra) puede generar una factura alta en un
solo día si no se controla. Antes de que la app tenga usuarios reales, debe existir:

1. **Alertas de presupuesto** configuradas en Google Cloud Billing, con notificación al
   correo del desarrollador y del administrador del proyecto si el gasto diario supera un
   umbral definido.
2. **Revisión obligatoria de cualquier listener en tiempo real** (`onSnapshot`) antes de
   mergear a producción — confirmar que se cierra correctamente cuando el componente que lo
   usa se desmonta.
3. **Reglas de seguridad probadas** con el emulador de Firestore antes de cada despliegue a
   producción, no solo revisadas manualmente.

## Qué falta definir (pendiente, no inventar)

- Lista exacta de roles administrativos y permisos de cada uno (confirmado que son varios —
  ver arriba).
- Condición exacta para permitir la edición de un registro ya sincronizado (confirmación
  manual vs. hora de corte vs. otra) — ver arriba.
- Si la visibilidad por equipo (`teamId`) se implementa en la v1 o se pospone.
- Mecánica de creación y asignación de "casos" (ver `03-modelo-de-datos.md`) — sin esto
  cerrado, tampoco se pueden escribir las reglas de acceso a `cases`.
- Política de baja de un usuario de campo: si se le revoca el acceso, ¿sus registros
  pasados los sigue viendo el admin sin problema? (La respuesta esperada es sí, porque los
  registros nunca se borran, pero debe confirmarse explícitamente.)
