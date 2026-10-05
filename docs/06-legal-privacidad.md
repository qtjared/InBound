# Legal y privacidad

> Aviso: este documento no es asesoría legal. Es un listado de temas que deben revisarse
> con un abogado antes de lanzar el proyecto a producción, especialmente porque se capturan
> datos personales y fotografías de manera masiva. No se debe tratar ningún punto de este
> archivo como resuelto hasta que un profesional legal lo confirme.

## Por qué esto importa para este proyecto en particular

La app captura, como mínimo:
- Nombre del usuario de campo (identifica a la persona).
- Fecha, hora y ubicación implícita del trabajo realizado.
- Fotografías tomadas en campo, que pueden incluir personas, propiedades o información
  sensible según el tipo de trabajo que se documente.

Esto la coloca dentro del tratamiento de datos personales regulado en México por la Ley
Federal de Protección de Datos Personales en Posesión de los Particulares (LFPDPPP). El
cliente, como responsable de los datos capturados por su personal, probablemente necesita
un aviso de privacidad — y el desarrollador, como quien construye el sistema que los
almacena, debe dejar claro en el contrato quién es responsable de qué.

## Temas pendientes de resolver con el cliente (y posiblemente con un abogado)

- **Aviso de privacidad:** ¿el cliente ya tiene uno para su personal de campo? Si no, debe
  crearse antes de que la app capture datos en producción.
- **Responsable vs. encargado del tratamiento de datos:** en términos de la ley, normalmente
  el cliente es el "responsable" (decide qué datos se recaban y para qué) y el
  desarrollador/la infraestructura (Firebase) actúa como "encargado" (solo procesa y
  almacena). Esto debe quedar explícito en el contrato de desarrollo.
- **Consentimiento del personal de campo:** si son empleados del cliente, probablemente
  aplica bajo la relación laboral, pero conviene confirmarlo — no asumir.
- **Fotografías con terceros identificables:** si las fotos de campo pueden incluir
  personas ajenas al personal (por ejemplo, en espacios públicos o propiedades de
  terceros), esto puede requerir tratamiento adicional. Debe preguntarse al cliente qué
  tipo de fotos se van a capturar realmente.
- **Ubicación del almacenamiento de datos:** Firebase almacena en la infraestructura de
  Google Cloud; confirmar en qué región se configura el proyecto y si eso tiene
  implicaciones para el cliente (por ejemplo, si maneja información de dependencias
  gubernamentales o de un sector regulado).
- **Retención indefinida de fotos:** el diseño actual no borra nunca los registros ni las
  fotos (ver `01-vision-alcance.md` y `03-modelo-de-datos.md`). Esto debe evaluarse contra
  cualquier obligación legal de retención máxima o mínima que aplique al giro del cliente —
  no se debe asumir que "guardar todo para siempre" es automáticamente aceptable.
- **Propiedad de los datos:** dejar explícito en el contrato que los datos capturados son
  propiedad del cliente, no del desarrollador, y qué pasa con ellos si termina la relación
  de desarrollo/mantenimiento.
- **Responsabilidad ante una fuga de datos:** definir en el contrato qué corresponde al
  desarrollador (seguridad del código y configuración) y qué corresponde al cliente (uso
  correcto del sistema, gestión de accesos de su personal).

## Qué NO se debe hacer sin resolver lo anterior

- No lanzar la app a producción con usuarios reales sin al menos un aviso de privacidad
  básico revisado por el cliente.
- No firmar el contrato de desarrollo sin que quede explícito quién es responsable y quién
  es encargado del tratamiento de datos.
- No asumir que la retención indefinida de fotos es legalmente segura sin confirmarlo — es
  una decisión de producto que puede necesitar ajuste legal.
