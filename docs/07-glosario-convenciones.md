# Glosario y convenciones

Este archivo existe para que cualquier agente o desarrollador use los mismos términos y no
invente nombres distintos para el mismo concepto a lo largo del proyecto.

## Glosario del dominio

| Término | Significado exacto en este proyecto |
|---|---|
| **Plantilla** | Un formulario diseñado por el administrador, con campos definidos, usado para **capturar** datos. Equivalente técnico: documento en la colección `templates`. No confundir con "plantilla de reporte" (ver abajo). |
| **Plantilla de reporte** | El formato de marca (PDF o Excel) que el cliente ya usa actualmente para **presentar** los datos capturados — con logo de la empresa y espacio para fotos. Es sobre la *salida*, no sobre la *captura*. Archivo real pendiente de recibir (ver `01-vision-alcance.md`). |
| **Super admin** | Rol superior del sistema, con acceso total: plantillas, todos los registros de todos los trabajadores, exportaciones, generación de reportes con marca y gestión de cuentas. Usa el dashboard web (`apps/admin-web`). |
| **Viewer** | Rol administrativo de solo lectura: puede entrar al dashboard y ver plantillas, registros y reportes, pero no editar nada. Confirmado como el primer rol administrativo menor concreto. |
| **Versión de plantilla** | Una revisión específica y congelada de una plantilla. Cada registro guarda con qué versión fue llenado, para no romperse si la plantilla cambia después. |
| **Registro** | Una respuesta capturada por un usuario de campo a una plantilla específica. Equivalente técnico: documento en la colección `records`. |
| **Usuario de campo** | Persona que captura formularios en el sitio de trabajo, con o sin conexión. Rol `field`. También llamado **"trabajador"** por el cliente — es el único tipo de usuario del lado de la app móvil. |
| **Administrador** | Persona (el cliente o su equipo) que diseña plantillas, publica cambios y exporta datos. Rol `admin`. |
| **Supervisor** | Término general que usa el cliente para referirse a quien usa el dashboard web (`super_admin` o `viewer`), en contraste con el "trabajador" que usa la app móvil. No es un valor de `role` por sí mismo — es la forma en que el cliente nombra a ambos roles administrativos juntos. |
| **Fecha de captura** (`capturedAt`) | Momento real en que el usuario llenó el formulario, aunque el teléfono no tuviera señal en ese momento. |
| **Fecha de sincronización** (`syncedAt`) | Momento en que el registro llegó al servidor. Puede ser mucho después de la fecha de captura. |
| **Vista activa** | Registros con 3 días o menos de antigüedad desde su `capturedAt`, mostrados por defecto en el panel admin. |
| **Historial de Registros** | Registros con más de 3 días de antigüedad. No se eliminan, solo se ocultan de la vista activa por defecto. |
| **Caso** | Agrupa varios registros de formulario distintos, llenados por distintos usuarios de campo, sobre una misma situación de trabajo. Mecánica de creación y asignación **pendiente de definir** (ver `03-modelo-de-datos.md`). |
| **Equipo** | Agrupación de usuarios de campo (`teamId`). Existe como propuesta para permitir que compañeros del mismo equipo vean los registros del otro sobre un mismo caso — **no confirmado para v1**. |
| **Exportación** | Archivo (Excel o ZIP con Excel + fotos) generado bajo demanda desde el panel admin, nunca de forma automática/programada en la v1. |

## Convenciones de nombres (código)

- **Colecciones de Firestore:** inglés, plural, `camelCase` si son compuestas (`users`,
  `templates`, `records`).
- **Campos de documentos:** `camelCase`, siempre en inglés (`capturedAt`, no `fechaCaptura`).
- **Componentes de React / React Native:** `PascalCase` (`TemplateEditor`,
  `RecordHistoryList`).
- **Archivos de configuración y scripts:** `kebab-case`.
- **Nombres de funciones en Cloud Functions:** verbo + sustantivo en `camelCase`
  (`generateExport`, `syncRecord`).

## Idioma

- **Interfaz visible al usuario** (app móvil y panel admin): español, porque los usuarios
  de campo y el administrador trabajan en español.
- **Código, nombres de variables, comentarios técnicos, nombres de colecciones y campos**:
  inglés, por convención estándar de desarrollo — no mezclar idiomas dentro del código.

## Regla general para agentes que trabajen en este proyecto

Si una tarea requiere una decisión que no está cubierta en ninguno de los documentos de esta
carpeta (nombre de un campo nuevo, comportamiento no descrito, texto legal, cifra de
presupuesto), la acción correcta es:

1. Buscar primero si ya existe una convención aplicable en este archivo o en
   `03-modelo-de-datos.md`.
2. Si no existe, **no inventar una respuesta silenciosamente** — señalar explícitamente que
   falta esa definición y proponerla como pregunta abierta, en vez de decidirla por cuenta
   propia y seguir adelante.
