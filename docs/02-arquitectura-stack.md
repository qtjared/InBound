# Arquitectura técnica y stack

## Decisión de stack (confirmada)

| Componente | Tecnología | Razón |
|---|---|---|
| App móvil | React Native con Expo | Una sola base de código para Android e iPhone; consistente con la experiencia previa del desarrollador en React |
| Persistencia local + sincronización | Firebase Firestore (SDK con persistencia offline) | La persistencia local y la sincronización automática al recuperar conexión vienen incluidas en el SDK — no hay que construir una cola de sincronización manual |
| Fotografías | **Cloudflare R2** (compatible con API S3) | **Confirmado — reemplaza a Firebase Storage.** Egreso $0 siempre (vs. ~$0.09/GB de S3 y ~$0.12/GB de Firebase Storage) y almacenamiento más barato (~$0.015/GB vs. ~$0.026/GB). Decisivo porque las fotos nunca se borran y el `super_admin` va a descargar exportaciones seguido — ver razón completa abajo |
| Autenticación | Firebase Auth | Cuentas para los usuarios de campo y el/los administradores, con roles |
| Emisión de URLs firmadas para R2 | Cloud Function | Verifica el token de Firebase Auth del usuario y genera una URL temporal de subida/descarga hacia R2 — R2 no conoce a Firebase Auth directamente, así que este paso es el puente entre ambos |
| Panel de administración | React (web) | Mismo lenguaje que la app móvil; diseñador de formularios, dashboard del super admin, historial, exportación |
| Generación de Excel (genérico) | Cloud Functions + librería tipo `exceljs` | Genera el archivo multi-pestaña bajo demanda desde el panel; no vive en el cliente |
| Generación de reportes con marca | Cloud Functions — enfoque a decidir una vez se tenga el archivo real (ver nota abajo) | Reproduce el formato de reporte existente del cliente (logo, espacio para fotos) en vez de una tabla genérica |

## Por qué Firebase y no Supabase

Se evaluó Supabase (PostgreSQL + herramientas de backend) como alternativa. La razón para
descartarlo en este proyecto es específica al requisito de **offline-first**:

- Firestore trae persistencia local y sincronización automática integradas en el SDK, tanto
  para Android como iOS.
- Supabase no tiene esto de fábrica. Para lograr un comportamiento offline-first equivalente
  se necesitaría sumar una capa adicional (por ejemplo PowerSync o ElectricSQL), lo cual es
  una pieza más de infraestructura que aprender, configurar y mantener.

Dado que el offline-first es el requisito central del proyecto (no un extra), Firebase reduce
el riesgo técnico y el tiempo de desarrollo frente a Supabase para este caso específico. Esta
decisión debe revisarse solo si el volumen de datos relacional se vuelve muy complejo (no es
el caso previsto aquí).

## Arquitectura híbrida: Firestore para datos, R2 para fotos (confirmado)

**Decisión confirmada:** las fotografías se almacenan en **Cloudflare R2**, no en Firebase
Storage. Todo lo demás (Firestore, Auth, Cloud Functions) se queda igual — esto es un
cambio puntual de dónde viven los archivos de imagen, no una migración de plataforma.

**Por qué:** R2 tiene egreso **$0 siempre**, contra ~$0.09/GB de S3 y ~$0.12/GB de Firebase
Storage (que internamente usa precios de Google Cloud Storage). Este proyecto tiene
exactamente el patrón de uso donde eso importa: las fotos **nunca se borran** (retención
permanente ya confirmada) y el `super_admin` va a **descargar exportaciones seguido** — cada
descarga es tráfico de salida. Con Firebase Storage ese costo crece mes tras mes sin tope;
con R2 no existe. Además, R2 habla la API de S3, así que no se pierde compatibilidad de
herramientas por el cambio — el SDK de AWS S3 funciona igual contra R2 cambiando solo el
endpoint.

**Lo que SÍ cambia por este cambio (costo de la decisión, no gratis):**

- R2 no conoce a Firebase Auth — no hay reglas de seguridad automáticas como las de Firebase
  Storage que verifican el dueño de un documento directamente. Se necesita una **Cloud
  Function que haga de puente**: recibe el token de Firebase Auth del usuario, lo valida,
  verifica que tiene permiso sobre ese `recordId` (misma lógica que ya existe en
  `04-reglas-seguridad.md`), y genera una **URL firmada temporal** de subida o descarga hacia
  R2. La app móvil y el dashboard nunca hablan con R2 directamente sin pasar por esta
  función.
- Esto es una pieza nueva de infraestructura que no existía con Firebase Storage — se
  construye en la Fase 1 (ver `09-plan-implementacion.md`), al mismo tiempo que la captura
  offline, porque sin ella no hay manera de subir una foto.
- Dos proveedores distintos (Firebase + Cloudflare) significa dos consolas, dos conjuntos de
  credenciales y dos facturas — operacionalmente un poco más de superficie que tener todo en
  un solo proveedor, a cambio del ahorro de egreso.

## Componentes del sistema

1. **App móvil (campo)** — instalada en los teléfonos de los 50–400 usuarios. Permite:
   - Descargar plantillas publicadas cuando hay conexión.
   - Llenar formularios sin conexión (persistencia local vía Firestore SDK).
   - Tomar y comprimir fotografías antes de guardarlas localmente.
   - Sincronizar automáticamente al recuperar señal.
2. **Backend / nube (Firebase)** — Firestore (datos), Storage (fotos), Auth (usuarios),
   Cloud Functions (lógica de exportación y tareas programadas).
3. **Panel de administración / dashboard del super admin (web)** — usado por el cliente y,
   si se enumeran roles adicionales, por otros perfiles administrativos con permisos
   menores. Es **una sola aplicación**, conectada directo a Firestore — no hay una
   aplicación de dashboard separada. Permite:
   - Diseñar y publicar plantillas de formulario, con versionado.
   - Ver registros activos (últimos 3 días) e historial (todo lo anterior) de todos los
     trabajadores.
   - Exportar a Excel/ZIP bajo demanda (genérico) o generar el reporte con el formato de
     marca del cliente (ver componente 5).
4. **Exportación genérica (Excel + fotos)** — generada por Cloud Functions al solicitarse
   desde el panel; produce un ZIP con el Excel (una pestaña por plantilla) y una carpeta de
   fotos nombradas por el ID del registro al que pertenecen.
5. **Diseñador de reportes con marca, dentro del dashboard (decisión confirmada)** —
   **confirmado:** el reporte se diseña desde cero dentro del dashboard web, no se replica
   el PDF que el cliente ya tiene como archivo. Ese PDF existente (si se comparte en algún
   momento) se usa solo como referencia visual — no como plantilla técnica base. Esto se
   decidió así porque reconstruir un documento con coordenadas fijas es frágil frente a
   datos variables (texto más largo de lo esperado, número variable de fotos entre 1 y 3);
   un diseño propio con motor de layout real (HTML/CSS) se ajusta solo.

   **Enfoque técnico:**
   - El reporte se genera como una página HTML (logo, secciones, bloque de fotos en
     cuadrícula que se adapta a cuántas haya) y se convierte a PDF en el servidor mediante
     Cloud Functions, usando una librería como Puppeteer.
   - El super admin controla desde el dashboard: el logo de la empresa, qué campos de la
     plantilla de formulario aparecen en el reporte y en qué sección/orden, y la posición
     del bloque de fotos. Esta configuración se guarda por plantilla (ver
     `03-modelo-de-datos.md` para dónde vive este dato).
   - Al no depender ya de un archivo externo, este componente **sí se puede empezar a
     diseñar** — lo que falta cerrar es el detalle visual concreto (colores, disposición
     exacta), no un bloqueo técnico.

## Estructura de repositorio: monorepo (confirmado)

El proyecto vive en **un solo repositorio**, no en repositorios separados para la app móvil
y el panel admin. Esta decisión aplica mientras el proyecto sea para **un solo cliente** —
ver la nota de "Multi-cliente" más abajo para qué cambia si eso deja de ser cierto.

**Estructura de carpetas:**

```
/apps
  /mobile        (React Native / Expo — app de campo)
  /admin-web     (React — panel de administración)
  /functions     (Cloud Functions — exportación, lógica de servidor)
/packages
  /shared-types  (interfaces de Template, Record, User — ver 03-modelo-de-datos.md)
```

**Herramienta de workspaces:** npm/yarn/pnpm workspaces es suficiente para este tamaño de
proyecto. No se debe introducir Nx, Turborepo ni herramientas similares salvo que el build
se vuelva lento o el equipo crezca — no adoptarlas por anticipado.

**Por qué monorepo y no repos separados, en este proyecto específico:**

- La app móvil y el panel admin comparten el mismo modelo de datos (`03-modelo-de-datos.md`).
  Con un paquete compartido (`packages/shared-types`), un cambio en la estructura de
  `Template` o `Record` se refleja en ambas apps a la vez, sin duplicar la definición ni
  arriesgarse a que se desincronicen.
- Es un solo desarrollador. La separación de repos da valor cuando hay equipos distintos con
  ciclos de release independientes — no es el caso aquí, y separar solo agregaría overhead
  de coordinar dos historiales de git y dos configuraciones.
- Las reglas de seguridad de Firestore y el código de Cloud Functions también viven en el
  mismo repo, así el modelo de datos, las reglas que lo protegen y el código que las usa
  quedan en un solo lugar — coherente con la regla de `07-glosario-convenciones.md` de no
  fragmentar la fuente de verdad del proyecto.

**Qué implica en la práctica:**

- Un solo `git clone` trae todo el proyecto (móvil, web, funciones, tipos compartidos).
- Un cambio que afecte a ambas apps (por ejemplo, un campo nuevo en el modelo de `Record`)
  se hace en un solo commit, no en dos pull requests coordinados en repos distintos.
- El control de acceso a nivel repositorio es todo-o-nada: cualquiera con acceso al repo ve
  tanto el código móvil como el del panel admin. Mientras el proyecto sea de un solo
  desarrollador, esto no es un problema.

**Nota sobre ofrecer la app a otro cliente en el futuro (no aplica todavía):** el código
seguiría viviendo en este mismo monorepo. Lo que cambiaría entre clientes no es el
repositorio ni el código de negocio, sino el **despliegue**: cada cliente tendría su propio
proyecto de Firebase (su propia base de Firestore, su propio Storage, sus propias reglas),
desplegado desde el mismo código con una configuración distinta (variables de entorno:
nombre del cliente, credenciales del proyecto de Firebase, colores/textos del panel). Por
eso, desde ahora, ningún dato específico del cliente actual debe quedar escrito directamente
en el código — debe vivir en configuración, aunque hoy solo haya un cliente. No se
implementa nada de multi-tenant (por ejemplo, un campo `tenantId` compartiendo una sola base
de datos entre clientes) a menos que este documento se actualice explícitamente para
reflejar ese cambio de alcance.

## Capacidad y escalabilidad

Cálculo de referencia con el volumen esperado:

- 400 usuarios × 5 formularios/día = 2,000 escrituras/día.
- El plan gratuito de Firestore (Spark) incluye 50,000 lecturas, 20,000 escrituras y 20,000
  eliminaciones al día, más 1 GiB de almacenamiento — el volumen de formularios está muy por
  debajo de estos límites incluso en el escenario de 400 usuarios.
- Al superar el plan gratuito, Firestore no requiere cambio de arquitectura: se pasa al plan
  de pago por uso (Blaze), con tarifas bajas por cada bloque adicional de lecturas/escrituras.
- **El costo que más crece con el tiempo no son los registros, son las fotografías**, porque
  ningún registro se elimina nunca (ver `01-vision-alcance.md`, punto de retención). Este
  costo se controla en el origen: comprimiendo antes de subir (ver abajo).

## Compresión de imágenes (obligatoria, no opcional)

Regla de implementación fija para todo el proyecto:

- **Redimensionar** cada fotografía a un máximo de 1920px en el lado más largo antes de
  subirla.
- **Comprimir en JPEG a calidad ~80%** (ni al mínimo ni al máximo).
- Esto se ejecuta **en el teléfono, antes de guardar la foto localmente**, usando una
  librería como `expo-image-manipulator`. No se sube la foto original sin comprimir en
  ningún caso.
- Efecto esperado: una foto de 3–5 MB queda en ~200–400 KB, sin pérdida de calidad
  perceptible en pantalla ni en impresión estándar, y con ahorro significativo en el consumo
  de datos móviles del usuario de campo al sincronizar.

## Riesgos técnicos identificados y cómo se mitigan

| Riesgo | Mitigación |
|---|---|
| Fotos sin comprimir inflan el almacenamiento indefinidamente (nada se borra) | Compresión obligatoria en cliente antes de subir (ver arriba) |
| Firestore no tiene tope de gasto por defecto — un bug de consulta en bucle puede generar una factura alta en un solo día | Configurar alertas de presupuesto en Google Cloud desde el día uno del proyecto en producción |
| Reglas de seguridad mal configuradas permiten que un usuario lea o escriba datos de otros | Reglas de seguridad explícitas por rol (ver `04-reglas-seguridad.md`); cada usuario de campo solo puede escribir sus propios registros |
| Listeners de Firestore (`onSnapshot`) mal cerrados generan lecturas de más | Code review obligatorio de cualquier listener antes de mergear a producción |
| Cambios en una plantilla rompen la legibilidad de respuestas anteriores | Versionado de plantillas: cada respuesta guarda el número de versión con la que fue llenada (ver `03-modelo-de-datos.md`) |
| R2 no valida permisos por sí solo (a diferencia de Firebase Storage, no conoce a Firebase Auth) — una Cloud Function mal escrita podría emitir una URL firmada sin verificar bien al dueño del registro | La Cloud Function puente (ver arriba) replica exactamente la misma lógica de permisos de `04-reglas-seguridad.md` — nunca se emite una URL firmada sin antes validar el token y el `userId` dueño del `recordId` |
