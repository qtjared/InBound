---
name: arquitectura-stack
description: Usa esta skill cuando elijas una librería, decidas dónde vive una pieza de lógica (mobile, admin-web o Cloud Functions), o tengas dudas sobre el stack técnico del proyecto.
---

# Arquitectura y stack técnico

Referencia completa: `docs/02-arquitectura-stack.md`.

## Stack confirmado (no proponer alternativas sin documentarlo primero)

- App móvil: **React Native con Expo**.
- Backend: **Firebase** (Firestore con persistencia offline, Auth, Cloud Functions) —
  elegido sobre Supabase específicamente por el soporte offline nativo del SDK, que es el
  requisito central del proyecto. No proponer Supabase ni otro backend sin que el documento
  se actualice primero.
- Fotografías: **Cloudflare R2** (compatible con API S3), **no Firebase Storage** —
  confirmado por el egreso $0 de R2 frente al costo creciente de Firebase Storage, decisivo
  porque las fotos nunca se borran y se exportan seguido. Acceso solo vía una Cloud Function
  que valida el token de Firebase Auth y emite una URL firmada — nunca acceso directo a R2
  desde el cliente.
- Panel admin: **React** (web), conectado directo a Firestore — es la misma app para
  `super_admin` y `viewer`, no hay dos dashboards distintos.
- Generación de Excel: Cloud Functions + `exceljs`.
- Generación de reporte con marca: página HTML → PDF vía Cloud Functions (ej. Puppeteer) —
  diseñado dentro de la app, **no** replicando un PDF externo del cliente como archivo base.

## Reglas no negociables

- **Compresión de fotos obligatoria antes de subir:** resize a máx. 1920px + JPEG calidad
  ~80%, ejecutado en el cliente (teléfono) antes de guardar localmente. Nunca subir una foto
  sin procesar — es el mayor riesgo de costo del proyecto porque las fotos nunca se borran.
- Las fotos suben a R2, nunca a Firestore ni a Firebase Storage — solo a través de la Cloud
  Function puente (ver arriba).
- Estructura de repo: **monorepo** (`apps/mobile`, `apps/admin-web`, `apps/functions`,
  `packages/shared-types`) con npm/yarn/pnpm workspaces — no introducir Nx/Turborepo salvo
  que el build se vuelva lento.
- Si en el futuro se ofrece el proyecto a otro cliente: no se implementa multi-tenant
  (`tenantId` compartido) a menos que se documente explícitamente — el camino confirmado es
  un proyecto de Firebase separado por cliente, mismo código.
