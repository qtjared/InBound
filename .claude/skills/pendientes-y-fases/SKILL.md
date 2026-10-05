---
name: pendientes-y-fases
description: Usa esta skill antes de implementar cualquier feature nueva, cuando no esté claro si algo ya está decidido, o cuando surja una decisión de producto/técnica que no está en /docs. Cubre el orden de construcción por fases y el manejo de decisiones abiertas.
---

# Pendientes y fases de construcción

## Antes de empezar cualquier feature

1. Revisa `docs/09-plan-implementacion.md` para confirmar en qué fase está el proyecto. No
   construyas algo de una fase posterior (por ejemplo el generador de reportes con marca,
   Fase 5) si las fases anteriores no están terminadas — el orden existe para no construir
   sobre una base que no funciona.
2. Revisa `docs/08-pendientes.md` — si lo que vas a construir depende de una fila marcada
   "Sí" en la columna "¿Bloquea construir?", detente y repórtalo en vez de avanzar con un
   supuesto.

## Cuando encuentres una decisión no documentada

Si al implementar algo te encuentras con una decisión que no está resuelta en ningún
documento de `/docs` (un nombre de campo, un comportamiento, una regla de negocio):

1. **No la inventes ni la decidas en silencio**, aunque parezca obvia.
2. Agrégala como fila nueva en la tabla correspondiente de `docs/08-pendientes.md`, con una
   descripción breve y marcando si bloquea o no seguir construyendo.
3. Repórtalo explícitamente en tu respuesta — no sigas construyendo esa parte asumiendo una
   respuesta.

## Cuando se cierre un pendiente

Si en la conversación se confirma la respuesta a algo que estaba en `docs/08-pendientes.md`:

1. Actualiza el documento de origen correspondiente (el que tenía el detalle completo del
   pendiente).
2. Actualiza `docs/08-pendientes.md` — mueve el ítem a "Resueltos recientemente" o
   elimínalo, nunca lo dejes en ambos estados a la vez.

Estos dos documentos se actualizan siempre juntos — nunca uno sin el otro.
