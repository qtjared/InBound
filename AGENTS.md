# AGENTS.md — App de registro de trabajo de campo

App móvil offline-first (React Native/Expo) + panel web (React) + Firebase (Firestore,
Auth, Cloud Functions) + Cloudflare R2 (fotos), para que trabajadores de campo capturen
formularios con fotos sin internet, y un super admin los vea, diseñe plantillas y genere
reportes con marca desde un dashboard web.

## Regla de oro: no inventar

La documentación completa vive en `/docs`. Si una decisión de producto, técnica o de
nombres **no está en `/docs`, no se inventa** — se agrega como pregunta abierta en
`docs/08-pendientes.md` (en la tabla que corresponda) y se avisa explícitamente en la
respuesta, en vez de decidirla en silencio y seguir construyendo.

Antes de tocar código: revisa `docs/09-plan-implementacion.md` para saber en qué fase está
el proyecto, y `docs/08-pendientes.md` para confirmar que lo que vas a construir no depende
de un pendiente sin cerrar.

## Dónde está cada cosa

| Necesitas saber... | Documento |
|---|---|
| Qué hace la app, qué está confirmado, qué queda fuera de alcance | `docs/01-vision-alcance.md` |
| Stack, estructura del monorepo, por qué Firebase, por qué R2 para fotos y no Firebase Storage | `docs/02-arquitectura-stack.md` |
| Colecciones de Firestore, campos exactos, estructura de fotos en R2 | `docs/03-modelo-de-datos.md` |
| Reglas de seguridad de Firestore, roles, control de gasto | `docs/04-reglas-seguridad.md` |
| Qué datos personales se manejan y qué no se debe asumir sin revisión legal | `docs/06-legal-privacidad.md` |
| Términos del proyecto y convenciones de nombres de código | `docs/07-glosario-convenciones.md` |
| Lista completa de decisiones abiertas | `docs/08-pendientes.md` |
| En qué orden se construye el proyecto, fase por fase | `docs/09-plan-implementacion.md` |

## Convenciones rápidas de código

- Colecciones de Firestore: inglés, plural, `camelCase` (`records`, `templates`, `users`).
- Campos de documentos: `camelCase`, siempre en inglés (`capturedAt`, no `fechaCaptura`).
- Componentes de React/React Native: `PascalCase`.
- Interfaz visible al usuario: español. Código, nombres de variables, comentarios: inglés.
- Detalle completo y razones: `docs/07-glosario-convenciones.md`.

## Estructura del repositorio

```
/apps
  /mobile        (React Native / Expo — app de campo)
  /admin-web     (React — dashboard, super_admin / viewer)
  /functions     (Cloud Functions — exportación, reportes, lógica de servidor)
/packages
  /shared-types  (interfaces de Template, Record, User)
/docs            (toda la documentación de producto y técnica — fuente de verdad)
```

Monorepo porque un solo desarrollador y un modelo de datos compartido entre `mobile` y
`admin-web` no justifican repos separados — razón completa en
`docs/02-arquitectura-stack.md`.

## Qué NO hacer nunca, sin excepción

- No borrar físicamente documentos de `records` — nunca se eliminan (ver
  `docs/03-modelo-de-datos.md`).
- No implementar edición de un `record` ya sincronizado sin que la condición esté cerrada en
  `docs/01-vision-alcance.md` (sigue pendiente).
- No subir una fotografía sin comprimir antes (resize 1920px + JPEG 80% — ver
  `docs/02-arquitectura-stack.md`).
- No hablar con el bucket de R2 directamente desde mobile o admin-web — siempre a través de
  la Cloud Function que valida el token de Firebase Auth y emite la URL firmada.
- No tratar a una cuenta administrativa como `super_admin` por defecto — el fail-safe es
  tratarla como `viewer` (el más restrictivo) hasta confirmar su rol real.
