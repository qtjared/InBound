# Documentación del proyecto — App de captura de trabajo de campo offline

Este directorio contiene la definición completa del proyecto. Está escrito para que cualquier
persona o agente que trabaje en el desarrollo tenga el contexto necesario **sin necesidad de
inventar o suponer decisiones que no están aquí**.

## Cómo usar esta documentación

- Si una decisión de producto, técnica o de negocio **no está escrita en estos archivos**,
  no se debe asumir. Se debe preguntar al responsable del proyecto antes de implementar.
- Estos documentos son la fuente de verdad. El código debe ajustarse a lo aquí definido, no
  al revés. Si el código y la documentación entran en conflicto, se corrige el código o se
  actualiza la documentación explícitamente — nunca se ignora en silencio.
- Cada archivo cubre un área específica. Empieza por este índice para saber a cuál ir.

## Índice de documentos

| Archivo | Contenido |
|---|---|
| `01-vision-alcance.md` | Qué problema resuelve la app, para quién, y qué queda fuera del alcance de la v1 |
| `02-arquitectura-stack.md` | Stack tecnológico, estructura de repositorio (monorepo), componentes del sistema, decisiones técnicas y sus razones |
| `03-modelo-de-datos.md` | Estructura de datos en Firestore, colecciones, campos, tipos, convenciones de nombres |
| `04-reglas-seguridad.md` | Reglas de seguridad de Firestore/Storage, control de acceso, control de gasto |
| `05-plan-negocio-presupuesto.md` | Costos estimados de desarrollo, costos operativos, modelo de precios |
| `06-legal-privacidad.md` | Manejo de datos personales, retención, responsabilidades, términos pendientes de definir con el cliente |
| `07-glosario-convenciones.md` | Glosario de términos del proyecto y convenciones de código/nombres a seguir |
| `08-pendientes.md` | Checklist único de todas las decisiones abiertas en el proyecto, consolidadas de todos los documentos |
| `09-plan-implementacion.md` | Orden de construcción por fases, con un entregable demostrable al final de cada una |

**Antes de construir cualquier cosa, revisa `09-plan-implementacion.md` para saber en qué
fase estás, y `08-pendientes.md` para ver si lo que vas a construir depende de un pendiente
sin cerrar.**

## Estado del proyecto

- **Etapa:** definición técnica, antes de la primera plática formal con el cliente.
- **Desarrollador:** una sola persona (autor de esta documentación), sin equipo adicional confirmado aún.
- **Cliente:** gran parte del alcance ya se ha ido confirmando en conversaciones de
  descubrimiento. Todo lo marcado como "confirmado" en cada documento ya no necesita
  revalidarse; todo lo marcado como "propuesto" o "pendiente" sí — ver el listado completo y
  centralizado en `08-pendientes.md`, en vez de buscarlo documento por documento.
