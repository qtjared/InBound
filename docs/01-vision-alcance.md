# Visión y alcance

## Problema que resuelve

Un equipo de 50 a 100 personas (con posibilidad de crecer a 200–400) realiza trabajo de campo
en zonas sin cobertura de internet. Necesitan capturar información estructurada (formularios)
y fotografías durante su jornada, y esa información debe llegar a una sola persona
administradora, quien la trabaja después en su computadora en formato Excel.

## A quién sirve

- **Usuarios de campo (50–400):** capturan formularios y fotos desde su teléfono
  (Android o iPhone), sin necesidad de conexión en el momento de la captura.
- **Administrador (el cliente, 1 persona o un equipo pequeño):** diseña los formularios,
  publica cambios, y descarga los datos capturados en Excel para trabajarlos.

## Alcance confirmado de la versión 1

Esto quedó definido en la conversación de descubrimiento con el cliente y **no debe
cambiarse sin confirmación explícita**:

1. **Captura offline** — la app funciona sin conexión; los datos se guardan localmente en
   el teléfono y se sincronizan solos al recuperar señal.
2. **Formularios diseñados por el cliente** — el cliente diseña y publica las plantillas de
   formulario desde un panel de administración, sin depender del desarrollador para cada
   cambio.
3. **Volumen esperado** — ~5 formularios por persona al día. Con 100 usuarios: 250–500
   formularios diarios (7,500–15,000 al mes). Con 400 usuarios, el volumen escala
   proporcionalmente (ver `02-arquitectura-stack.md` para el análisis de capacidad).
4. **Fotografías obligatorias, máximo 3 por formulario** — los 5 formularios diarios por
   persona incluyen fotografías, con un tope confirmado de **3 fotos por formulario**, con
   compresión automática (ver `02-arquitectura-stack.md`). La firma digital queda **fuera de
   la v1** (se descartó para no subir tiempo/costo), pero **sí se considera para una fase
   futura** — no es un "no" definitivo, es un "todavía no".
5. **Trazabilidad de fecha** — cada registro conserva su fecha real de captura (cuando se
   llenó el formulario), distinta de su fecha de sincronización (cuando llegó al servidor).
   Esto es crítico: un formulario llenado el lunes por alguien que sincroniza el jueves debe
   mostrar "lunes" como fecha de captura.
6. **Retención y vista de historial** — ningún registro se elimina nunca. Los registros de
   los últimos 3 días se muestran en una vista "activa" en el panel admin. Los registros con
   más de 3 días se mueven (lógicamente, no físicamente) a una sección "Historial de
   Registros" — siguen existiendo en la base de datos, solo se ocultan de la vista principal.
7. **Exportación a Excel bajo demanda** — el administrador descarga, cuando lo necesite, un
   archivo (Excel o ZIP con Excel + fotos) con una pestaña por plantilla y una fila por
   respuesta. No hay generación automática diaria — es bajo demanda.
8. **Compartir por WhatsApp** — función aparte para compartir un registro individual desde
   la app.
9. **No hay integración con Google Drive / Google Sheets** — se descartó explícitamente en
   favor de exportación directa a Excel/ZIP descargable.
10. **Un "caso" agrupa varios formularios distintos** — varios usuarios de campo pueden
    trabajar sobre el mismo caso, cada uno llenando un tipo de formulario diferente (no es
    "cada usuario ve solo lo suyo sin ninguna relación entre registros"). Por defecto, un
    usuario de campo **no** puede ver los formularios que llenaron sus compañeros sobre el
    mismo caso — verlos generaría más lecturas (costo) sin que sea indispensable. Se
    considera, como posible mejora, permitir que un usuario vea los formularios de sus
    compañeros **solo si pertenecen al mismo equipo** — ver nota de "Equipos" en
    `03-modelo-de-datos.md`, marcada como propuesta, no confirmada para la v1.
11. **Edición de registros ya sincronizados** — sí estará permitida para el usuario de
    campo que capturó el registro, pero **bajo una condición que todavía no está definida**
    (por ejemplo: hasta que el usuario confirme que el registro está correcto, o hasta que
    termine su turno a una hora fija como las 6 pm). Ver pendiente abajo.
12. **Roles múltiples en el panel admin** — confirmado que habrá más de un rol de
    administrador con permisos distintos (por ejemplo uno que solo puede ver, otro que puede
    editar). La lista exacta de roles y qué puede hacer cada uno todavía no está definida —
    ver pendiente abajo.
13. **Super admin y dashboard web** — el rol superior mencionado en el punto 12 (el que ve
    y controla todo) se llama **"super admin"** — es el mismo rol pendiente de enumerar, solo
    con nombre confirmado, no un rol nuevo adicional. Este rol usa un **dashboard web**
    conectado directamente a la base de datos de Firebase para ver los reportes de todos los
    trabajadores y exportarlos a Excel. **Esto no es una aplicación nueva** — es exactamente
    el panel de administración (`apps/admin-web`) ya definido en `02-arquitectura-stack.md`;
    el super admin es quien lo usa con acceso completo.
14. **Reportes con plantilla de marca, diseñada dentro de la app (confirmado)** — además de
    la exportación genérica a Excel/ZIP ya definida (punto 7), el objetivo es que los
    reportes generados a partir de los formularios se vean organizados y profesionales (logo
    de la empresa, secciones claras, espacio para fotos), para que los superiores los lean
    con facilidad y se note si un trabajador llenó bien sus datos. **Decisión confirmada:**
    el reporte **se diseña desde cero dentro del dashboard web** (no se reconstruye el PDF
    que el cliente ya tiene como archivo base) — el PDF existente se usa solo como
    *referencia visual* (mismo logo, colores, orden de secciones), no como plantilla técnica
    a replicar. Esto evita la fragilidad de encajar datos variables (texto largo, número
    variable de fotos) en coordenadas fijas de un documento ya hecho. El super admin
    controla desde el dashboard: logo, qué campos del formulario aparecen en el reporte, en
    qué orden/sección, y dónde va el bloque de fotos — ver `02-arquitectura-stack.md` para
    el enfoque técnico.
15. **Campos obligatorios por formulario** — el sistema ya soporta marcar cada campo de una
    plantilla como obligatorio u opcional de forma individual (`required: true/false`, ver
    `03-modelo-de-datos.md`) — esto no es un vacío de diseño. Lo que falta es que, al diseñar
    cada plantilla real, se defina cuáles de sus campos específicos deben marcarse como
    obligatorios. Es una decisión de contenido, no de arquitectura.
16. **Cuentas individuales para cada usuario (confirmado)** — todos los usuarios (de campo y
    administrativos) tienen su propia cuenta — esto ya estaba en el modelo de datos
    (`03-modelo-de-datos.md`, colección `users` + Firebase Auth), ahora confirmado
    explícitamente como requisito. Además, queda confirmado el primer rol administrativo
    menor concreto: **viewer** — una cuenta que puede entrar al dashboard, ver reportes,
    registros e historial, y **también exportar/descargar** (Excel genérico y reporte con
    marca) — lo único que no puede hacer es editar (no modifica plantillas, no modifica
    registros, no gestiona cuentas). Sigue abierto si existen más roles administrativos
    intermedios además de `super_admin` y `viewer` — ver pendientes abajo.
17. **La diferenciación de roles es del lado del dashboard, no del lado móvil (confirmado)**
    — del lado de la app móvil hay un solo tipo de usuario: el **trabajador** (rol `field`
    ya definido), quien llena los formularios en campo. No se necesita más de un rol en
    mobile. La distinción de permisos (quién ve todo, quién solo visualiza, etc.) vive
    exclusivamente en el dashboard web, entre `super_admin` y `viewer` — a los que el
    cliente se refiere en general como **"supervisor"**. Esto significa que el campo
    `role` único en la colección `users` (ver `03-modelo-de-datos.md`) es suficiente — se
    evaluó separarlo en un campo para mobile y otro para web, pero no hace falta porque
    mobile no tiene variación de rol.

## Explícitamente fuera de alcance en la v1

- Firma digital dentro de los formularios — **no descartada permanentemente**, se
  considera para una fase futura, ver punto 4 arriba.
- Cualquier integración con Google Drive o Google Sheets como almacenamiento de datos.
- Generación automática y periódica de reportes (todo es bajo demanda).
- Cualquier sistema de pagos o cobro dentro de la app — no es un requisito de este
  proyecto (ver `05-plan-negocio-presupuesto.md`).

## Decisiones pendientes de validar

Estas ya no dependen de una plática con un cliente externo en el sentido comercial
tradicional (ver `05-plan-negocio-presupuesto.md` sobre la naturaleza de este proyecto),
pero siguen siendo decisiones de producto que deben cerrarse antes de construir la parte
correspondiente:

- **Regla exacta de edición de registros sincronizados** — confirmado que se permite editar
  (punto 11 arriba), pero falta definir la condición: ¿hasta que el usuario confirme
  manualmente que el registro está bien, hasta una hora fija de corte (ej. fin de turno a
  las 6 pm), o alguna combinación? Esto determina si se necesita un campo adicional en
  `records` (ver `03-modelo-de-datos.md`).
- **Lista exacta de roles de administrador** — confirmado `super_admin` (acceso total) y
  `viewer` (solo lectura del dashboard, punto 16 arriba). Falta confirmar si existen roles
  intermedios adicionales (por ejemplo uno que pueda editar plantillas pero no gestionar
  cuentas) o si solo son estos dos.
- **Mecánica del "caso"** — quién crea un caso, cómo se asignan los usuarios de campo a un
  caso, y si un caso tiene un conjunto fijo de plantillas asociadas o cualquier plantilla se
  puede usar en cualquier caso. Ver nota en `03-modelo-de-datos.md`.
- **Equipos y visibilidad compartida** — si finalmente se implementa que un usuario vea los
  formularios de sus compañeros de equipo sobre el mismo caso, o se queda en "cada quien ve
  solo lo suyo" para la v1.
- **Diseño visual exacto del reporte** — ya no depende de recibir el PDF existente como
  archivo (punto 14, decisión tomada: se diseña dentro de la app), pero sí falta definir el
  detalle visual concreto (qué campos van en qué sección, colores, disposición del bloque de
  fotos) cuando se construya el diseñador de reportes.
- Tiempo de vida real del proyecto en producción (para dimensionar el crecimiento de
  almacenamiento a 6–12 meses, ya que las fotos nunca se borran).
- Requisitos legales sobre los datos personales capturados (ver `06-legal-privacidad.md`).
