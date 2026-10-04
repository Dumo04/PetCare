# PetCare | Backend

Componente de servidor de PetCare, la plataforma web que conecta propietarios de mascotas y proveedores de servicios de cuidado. Su propósito es gestionar usuarios y roles, publicaciones, disponibilidad y solicitudes de reserva con información consistente para ambas partes.

**Componente:** Backend · **Rama:** `backend`

[README del frontend](https://github.com/Dumo04/PetCare/blob/frontend/README.md) · [Repositorio](https://github.com/Dumo04/PetCare) · [Tablero de Jira](https://santiagosworkspace-32817046.atlassian.net/jira/software/projects/PET/boards/1)

## Objetivo y tecnologías

Centralizar las reglas que permiten encontrar servicios y solicitar una cita, evitando que la disponibilidad o el estado de una reserva dependan solamente de lo que muestre el navegador.

| Tecnología | Responsabilidad definida |
| --- | --- |
| Node.js | Entorno de ejecución del servidor. |
| Express | Organización de las rutas y solicitudes HTTP. |
| MySQL | Persistencia relacional de usuarios, servicios, horarios y reservas. |

El cliente definido utiliza HTML, CSS y JavaScript. Las versiones y dependencias se registran en la configuración del componente; el mecanismo de autenticación debe mantener un contrato común con el cliente.

## Módulos y trazabilidad

| Módulo | Historias y requisitos | Responsabilidad del backend |
| --- | --- | --- |
| Autenticación y roles | HU-01; RF-01 | Registro, unicidad de correo, autenticación y autorización de propietarios y proveedores. |
| Publicación de servicios | HU-02; RF-02 | Validar y guardar descripción, precio, horario y especies atendidas del proveedor autenticado. |
| Catálogo y detalle | HU-03, HU-04; RF-03, RF-04, RF-10 | Consultar proveedores por especie, entregar sus servicios y devolver una colección vacía cuando no haya coincidencias. |
| Solicitud y disponibilidad | HU-05; RF-05, RF-06 | Validar servicio, mascota, fecha, hora y disponibilidad antes de registrar una solicitud pendiente. |
| Agenda | HU-06; RF-07 | Entregar al proveedor sus solicitudes ordenadas por fecha y hora. |
| Respuesta del proveedor | HU-07; RF-08 | Confirmar o rechazar solicitudes propias; exigir motivo de rechazo y actualizar la disponibilidad. |
| Consulta de estado | HU-08; RF-09 | Permitir que las partes autorizadas consulten pendiente, confirmada o rechazada y el motivo pertinente. |

HU-01 a HU-07 tienen prioridad **Debe**; HU-08 tiene prioridad **Debería**. RF-09 es de prioridad media y RF-10 de prioridad baja; los demás RF son de prioridad alta.

## Reglas de negocio del MVP

1. El sistema reconoce los roles propietario y proveedor. Las funciones disponibles y el acceso a los datos dependen del rol autenticado.
2. Un proveedor publica servicios con descripción, precio, horario y especies atendidas. No se guarda una publicación incompleta.
3. El filtro del catálogo utiliza la especie atendida. Un proveedor sin servicios publicados no habilita una reserva.
4. Una solicitud incluye el servicio, el nombre y la especie de la mascota, la fecha y la hora. Se valida la disponibilidad antes de guardarla.
5. Una solicitud nueva queda en estado `pendiente` y aparece en la agenda del proveedor.
6. El proveedor correspondiente puede confirmar o rechazar la solicitud. El rechazo exige un motivo.
7. Confirmar bloquea el horario para otras solicitudes; rechazar permite que vuelva a estar disponible, conforme a las reglas de disponibilidad.
8. Las partes autorizadas consultan el mismo estado persistido. El propietario no confirma su propia solicitud ni responde en nombre del proveedor.

### Estados y consistencia de disponibilidad

```text
pendiente -> confirmada
pendiente -> rechazada (con motivo)
```

El MVP contempla las transiciones de confirmación y rechazo. La cancelación y la reprogramación quedan fuera del alcance.

La política de disponibilidad debe especificar el tratamiento de solicitudes pendientes, la capacidad por franja, la duración de cada servicio y la zona horaria. El servidor valida la disponibilidad al solicitar, impide confirmaciones incompatibles y comprueba los cambios dentro de una operación atómica. Rechazar una solicitud nunca debe liberar una franja ocupada por otra reserva confirmada.

## Modelo conceptual de datos

El modelo conceptual organiza las entidades y relaciones necesarias para el MVP.

| Entidad | Información principal | Relación definida |
| --- | --- | --- |
| Usuario | Identificador, nombre, correo único, resumen de contraseña y rol. | Un usuario puede ser propietario o proveedor. |
| Proveedor | Usuario asociado y descripción pública. | Un proveedor publica varios servicios. |
| Servicio | Proveedor, descripción, precio y condiciones de atención. | Pertenece a un proveedor y atiende una o varias especies. |
| Especie | Identificador y nombre. | Catálogo compartido para filtrar y validar. |
| ServicioEspecie | Servicio y especie. | Relación entre servicios y especies admitidas. |
| Horario | Servicio o recurso, fecha/franja y capacidad según la política acordada. | Define la disponibilidad consultable. |
| Reserva | Propietario, servicio, nombre y especie de mascota, fecha, hora, estado y motivo de rechazo. | Vincula al propietario con el proveedor del servicio. |

Los datos básicos de mascota se registran para la solicitud. El MVP no incluye un perfil clínico ni historial de vacunación. La ficha y el filtro del proveedor se derivan de las especies y servicios que tenga publicados.

## Contrato HTTP de referencia

Base de referencia: `/api`. Las siguientes rutas describen el contrato de integración. Cualquier cambio en rutas o formatos debe actualizarse de forma coordinada con el frontend.

| Método y ruta | Acción | Acceso definido |
| --- | --- | --- |
| `POST /api/auth/register` | Registrar cuenta con rol. | Sin sesión. |
| `POST /api/auth/login` | Autenticar usuario. | Sin sesión. |
| `POST /api/auth/logout` | Finalizar sesión, según el mecanismo elegido. | Usuario autenticado. |
| `GET /api/especies` | Listar especies para selección. | Según la política de acceso al catálogo. |
| `POST /api/servicios` | Publicar un servicio. | Proveedor autenticado. |
| `GET /api/proveedores?especie=roedores` | Consultar catálogo filtrado. | Según la política de acceso al catálogo. |
| `GET /api/proveedores/:id` | Consultar ficha y servicios. | Mismo criterio del catálogo. |
| `GET /api/servicios/:id/disponibilidad?fecha=AAAA-MM-DD` | Consultar horarios disponibles. | Propietario. |
| `POST /api/reservas` | Solicitar reserva. | Propietario autenticado. |
| `GET /api/mis-solicitudes` | Consultar solicitudes propias y sus estados. | Propietario autenticado. |
| `GET /api/agenda` | Consultar solicitudes recibidas en orden cronológico. | Proveedor autenticado. |
| `PATCH /api/reservas/:id/estado` | Confirmar o rechazar con motivo. | Proveedor de la reserva. |

El propietario y el proveedor de una operación se obtienen de la identidad autenticada y de las relaciones guardadas, no de un identificador enviado por el cliente sin verificar.

Convención de respuestas: `201` para creación, `200` para consulta o actualización, `400` para datos inválidos, `401` para falta de autenticación, `403` para permisos insuficientes, `404` para recurso inexistente y `409` para correo duplicado o conflicto de disponibilidad. Un catálogo sin coincidencias devuelve una lista vacía para que frontend presente el mensaje de RF-10.

El contrato define un error uniforme con `codigo`, `mensaje` y, cuando aplique, `campos`, sin exponer contraseñas, consultas SQL ni detalles internos. El contrato se validará entre ambos componentes antes de la integración.

## Calidad, seguridad y privacidad

| Requisito | Trabajo definido |
| --- | --- |
| RNF-01: búsqueda inferior a 3 segundos | Medir consultas e integración con frontend; definir volumen de datos y condiciones de prueba. |
| RNF-02: función de resumen con sal | Guardar contraseñas como hash con sal mediante una función apropiada para contraseñas; nunca en texto plano ni devolverlas al cliente. |
| RNF-04: disponibilidad del 99 % | Definir alojamiento y periodo de medición, observar disponibilidad y registrar incidentes.  |
| RNF-05: tratamiento de datos personales | Incorporar las medidas requeridas por el proyecto para el tratamiento conforme a la Ley 1581 de 2012; validar política, finalidad y acceso antes de usar datos reales. |
| RNF-08: documentación y versionado | Documentar API, modelo, configuración, migraciones y pruebas en el repositorio. |

RNF-03, RNF-06 y RNF-07 se verifican principalmente en la interfaz, con apoyo del backend para completar el recorrido.

Pautas técnicas: validar entradas en servidor, usar consultas parametrizadas, comprobar la propiedad de cada recurso, proteger sesiones y acordar orígenes permitidos para la interfaz. El mecanismo de sesión y sus medidas de protección deben mantenerse coherentes con el contrato del frontend.

Las credenciales de base de datos y los secretos se configurarán fuera del código. El archivo `.env.example` debe contener solo nombres y valores ficticios; los archivos con secretos reales no se publicarán. Las pruebas y ejemplos públicos utilizarán información sintética y no respuestas individuales de encuestas.

## Organización de referencia

```text
README.md
package.json
.env.example
src/
  app.js
  server.js
  config/
  routes/
  controllers/
  services/
  repositories/
  middlewares/
database/
  migrations/
  seeds/
tests/
```

La estructura separa rutas, reglas de negocio y acceso a MySQL para mantener responsabilidades identificables.

## Acceso al componente

Para consultar esta rama localmente:

```bash
git clone --branch backend --single-branch https://github.com/Dumo04/PetCare.git
cd PetCare
```

## Criterios de aceptación

- Registro válido, correo duplicado, credenciales incorrectas y permisos por rol (HU-01).
- Publicación completa y rechazo cuando falte precio o especie (HU-02).
- Filtro por especie con coincidencias y lista vacía (HU-03).
- Ficha con servicios y proveedor sin servicios publicados (HU-04).
- Reserva válida en estado pendiente y rechazo de un horario ocupado (HU-05).
- Agenda propia ordenada y agenda sin solicitudes (HU-06).
- Confirmación por el proveedor correcto, rechazo con motivo y acceso denegado a terceros (HU-07).
- Consulta de estados propios y motivo de rechazo (HU-08).
- Solicitudes o confirmaciones simultáneas sin sobrepasar la capacidad acordada.
- Búsqueda inferior a tres segundos y validación del recorrido junto con frontend.

La validación incluye pruebas por módulo y pruebas de integración del recorrido completo.

## Equipo y metodología

| Integrante | Rol |
| --- | --- |
| Ariana Calderón Fuentes | Product Owner y equipo de desarrollo. |
| Santiago Duque Mora | Scrum Master y equipo de desarrollo. |
| Laura Valentina Ñustes Contento | Equipo de desarrollo. |
| Heidy Viviana Cárdenas Soler | Equipo de desarrollo. |

El equipo utiliza Scrum y gestiona el backlog, las prioridades y el seguimiento de las historias en Jira. Las historias comprenden el trabajo de interfaz y servidor necesario para completar cada funcionalidad. La planificación de iteraciones y las asignaciones se mantienen en el tablero del proyecto.

## Ramas y colaboración

- `main`: presentación del proyecto y enlaces.
- `frontend`: documentación y código del cliente.
- `backend`: documentación y código del servidor.

Cada componente mantiene su documentación en el archivo `README.md` de su rama. Los cambios se relacionan con las historias de usuario y las tareas de Jira. La integración deberá conservar la documentación específica de ambas capas.

## Alcance excluido y recursos

Fuera del MVP: perfil de mascota con historial de vacunación, filtros por ciudad y precio, notificaciones automáticas, historial de reservas y reportes, pagos, geolocalización, reseñas, mensajería y aplicación móvil nativa.

- [Jira del proyecto](https://santiagosworkspace-32817046.atlassian.net/jira/software/projects/PET/boards/1).
- [Diseños de PetCare en Figma](https://www.figma.com/design/EUB7PbmZvgfWeAtE1QbvYk/PetCare).

## Mantenimiento de la documentación

Actualizar este README cuando cambien el alcance, la arquitectura, el contrato de integración o los procedimientos de configuración. Mantener la planificación temporal y el estado de las tareas en Jira, y registrar los cambios técnicos junto con el código correspondiente.
