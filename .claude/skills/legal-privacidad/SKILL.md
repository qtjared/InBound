---
name: legal-privacidad
description: Usa esta skill cuando implementes algo relacionado con datos personales del personal de campo, retención o borrado de información, exportaciones que incluyan datos identificables, o antes de cualquier lanzamiento a producción con usuarios reales.
---

# Legal y privacidad

Referencia completa: `docs/06-legal-privacidad.md`. Esta skill no es asesoría legal — es un
recordatorio de qué no se debe asumir resuelto sin confirmación.

## Qué no asumir como seguro sin confirmación

- Que la retención permanente de fotos y registros es legalmente aceptable para el giro del
  cliente — no está confirmado.
- Que existe un aviso de privacidad vigente para el personal de campo — no está confirmado.
- Quién es "responsable" y quién "encargado" del tratamiento de datos entre el cliente y el
  desarrollador — no está definido en el contrato.

## Antes de lanzar a producción con usuarios reales (no antes de programar)

No lanzar sin al menos un aviso de privacidad básico revisado por el cliente, y sin que el
contrato de desarrollo deje explícito quién es responsable de qué ante una fuga de datos.
Esto no bloquea el desarrollo ni las pruebas — bloquea el lanzamiento real.

Cualquier avance nuevo en estos temas se documenta en `docs/06-legal-privacidad.md` y se
refleja en `docs/08-pendientes.md`.
