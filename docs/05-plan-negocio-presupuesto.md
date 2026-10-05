# Plan de negocio y presupuesto preliminar

## Naturaleza real de este proyecto (importante — leer antes que nada)

Este proyecto **no se está desarrollando con fines de lucro**. Es un desarrollo para un
familiar del desarrollador; no hay intención de cobrar un monto significativo por el
trabajo. Si el cliente decide dar algo a cambio, se acepta, pero **no es el objetivo ni una
condición del proyecto**. Esto cambia el enfoque de este documento frente a un plan de
negocio tradicional:

- No es prioridad cerrar un presupuesto grande ni negociar condiciones comerciales.
- **Sí es prioridad** construir el proyecto con la misma calidad, disciplina técnica y
  documentación que se usaría en un proyecto comercial — la falta de fin de lucro no baja el
  estándar de lo que se construye.
- No hay sistema de pagos ni cobro dentro de la aplicación — no es un requisito de este
  proyecto y no debe implementarse.

## Costo de desarrollo — contexto, no objetivo de negociación

Se hizo una estimación de referencia en una conversación de descubrimiento previa
($220,000–$400,000 MXN para el alcance completo con 100 usuarios). Ese número se conserva
aquí únicamente como **referencia de la magnitud de esfuerzo que implica el proyecto** — útil
para dimensionar tiempo y prioridades — **no como un precio a cobrar ni a negociar con el
cliente**.

## Costo operativo (esto sí importa, aunque no haya fin de lucro)

Aunque no se busque ganar dinero con el desarrollo, el costo operativo de Firebase
(almacenamiento de fotos, lecturas/escrituras) es real y alguien lo paga — probablemente el
desarrollador o el cliente, a definir. Vale la pena tenerlo claro para no llevarse una
sorpresa:

- El rubro que más crece con el tiempo son las fotografías, porque nunca se eliminan (ver
  `01-vision-alcance.md` y `03-modelo-de-datos.md`).
- Se mitiga con la compresión obligatoria de imágenes definida en `02-arquitectura-stack.md`.
- **Pendiente de definir:** quién cubre el costo operativo mensual de Firebase una vez el
  proyecto esté en producción (el desarrollador, como parte de la ayuda, o el cliente).

## Qué NO aplica a este proyecto (a diferencia de un proyecto comercial típico)

- No hay modelo de pago por hitos ni facturación — no se está cobrando por el desarrollo.
- No hay necesidad de definir términos de contrato comercial formal en el sentido de
  compra-venta de un servicio.
- No es prioritario cerrar un presupuesto antes de empezar a construir.

## Qué sí sigue aplicando igual que en un proyecto comercial

- Los temas legales de `06-legal-privacidad.md` (protección de datos personales, aviso de
  privacidad) siguen siendo relevantes — que no haya fin de lucro no exime del manejo
  responsable de los datos personales del personal de campo del cliente.
- El estándar de calidad técnica, seguridad y documentación de todo el proyecto se mantiene
  igual que si fuera un proyecto pagado — esa es una decisión explícita del desarrollador,
  no un efecto secundario de que sea "gratis".
