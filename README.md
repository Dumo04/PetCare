# PetCare | Backend

Componente de servidor de PetCare, la plataforma web que conecta propietarios de mascotas y proveedores de servicios de cuidado. Su propósito es gestionar usuarios y roles, publicaciones, disponibilidad y solicitudes de reserva con información consistente para ambas partes.

**Rama:** `backend` · **Estado:** documentación inicial de análisis y diseño · **Actualización:** 4 de octubre de 2026.

> Este README define el trabajo previsto. La rama contiene documentación; no hay servidor, API, base de datos ni pruebas implementadas todavía. Los modelos, rutas y estructuras propuestos requieren acuerdo del equipo antes de programarse.

[README del frontend](https://github.com/Dumo04/PetCare/blob/frontend/README.md) · [Repositorio](https://github.com/Dumo04/PetCare) · [Tablero de Jira](https://santiagosworkspace-32817046.atlassian.net/jira/software/projects/PET/boards/1)

## Contexto académico y fuente

Proyecto de la línea B, **PetCare**, para Análisis y Diseño de Sistemas, Programa de Ingeniería de Software, Facultad de Ingeniería, Corporación Universitaria Iberoamericana. Docente: Tatiana Cabrera.

La fuente es **“Actividad 1 - Identificar el proyecto tecnológico a trabajar”**, documento del equipo de septiembre de 2026, en la versión PDF suministrada el 4 de octubre de 2026. Se toman el alcance y las tecnologías del numeral 6.2 (p. 28), la planificación del 6.3 (p. 29) y los requisitos e historias de los numerales 7.1 a 7.3 (pp. 30-33).

La numeración HU y RF de este README corresponde al PDF actual. Puede diferir del borrador inicial de Jira y debe conciliarse antes de asociar cambios a tarjetas. Las decisiones técnicas añadidas se identifican como propuestas; no se presentan como decisiones ya aprobadas en el documento.

## Objetivo y tecnologías

Centralizar las reglas que permiten encontrar servicios y solicitar una cita, evitando que la disponibilidad o el estado de una reserva dependan solamente de lo que muestre el navegador.

| Tecnología definida en el PDF | Responsabilidad prevista |
| --- | --- |
| Node.js | Entorno de ejecución del servidor. |
| Express | Organización de las rutas y solicitudes HTTP. |
| MySQL | Persistencia relacional de usuarios, servicios, horarios y reservas. |

El cliente previsto utiliza HTML, CSS y JavaScript. Las versiones, bibliotecas de acceso a datos y mecanismo de autenticación están pendientes de definición.

## Módulos y trazabilidad

| Módulo | Historias y requisitos del PDF | Responsabilidad del backend |
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
7. Confirmar bloquea el horario para otras solicitudes; rechazar permite que vuelva a estar disponible, según los criterios del documento.
8. Las partes autorizadas consultan el mismo estado persistido. El propietario no confirma su propia solicitud ni responde en nombre del proveedor.

### Estados y decisión pendiente sobre disponibilidad

```text
pendiente -> confirmada
pendiente -> rechazada (con motivo)
```

El PDF no incorpora cancelación, reprogramación ni estados adicionales en los criterios del MVP.

**Punto por acordar:** el PDF pide verificar disponibilidad al solicitar y bloquear al confirmar, pero no precisa si una solicitud pendiente retiene temporalmente el cupo. Antes de implementar se debe definir esa política, la capacidad por franja, la duración de cada servicio y la zona horaria. Cualquiera que sea el acuerdo, el servidor deberá impedir confirmaciones incompatibles y comprobar la disponibilidad dentro de una operación atómica. Rechazar una solicitud nunca debe liberar una franja ocupada por otra reserva confirmada.

## Modelo de datos propuesto

Este modelo orienta el diseño relacional; no es un esquema SQL ya creado.

| Entidad | Información principal | Relación prevista |
| --- | --- | --- |
| Usuario | Identificador, nombre, correo único, resumen de contraseña y rol. | Un usuario puede ser propietario o proveedor. |
| Proveedor | Usuario asociado y descripción pública. | Un proveedor publica varios servicios. |
| Servicio | Proveedor, descripción, precio y condiciones de atención. | Pertenece a un proveedor y atiende una o varias especies. |
| Especie | Identificador y nombre. | Catálogo compartido para filtrar y validar. |
| ServicioEspecie | Servicio y especie. | Relación entre servicios y especies admitidas. |
| Horario | Servicio o recurso, fecha/franja y capacidad según la política acordada. | Define la disponibilidad consultable. |
| Reserva | Propietario, servicio, nombre y especie de mascota, fecha, hora, estado y motivo de rechazo. | Vincula al propietario con el proveedor del servicio. |

Los datos básicos de mascota se registran para la solicitud. El MVP no incluye un perfil clínico ni historial de vacunación. La ficha y el filtro del proveedor se derivan de las especies y servicios que tenga publicados.

## Contrato HTTP propuesto

**No implementado.** Base propuesta: `/api`. Los nombres y formatos definitivos deben coordinarse con frontend.

| Método y ruta propuesta | Acción | Acceso previsto |
| --- | --- | --- |
| `POST /api/auth/register` | Registrar cuenta con rol. | Sin sesión. |
| `POST /api/auth/login` | Autenticar usuario. | Sin sesión. |
| `POST /api/auth/logout` | Finalizar sesión, según el mecanismo elegido. | Usuario autenticado. |
| `GET /api/especies` | Listar especies para selección. | Según política de catálogo por acordar. |
| `POST /api/servicios` | Publicar un servicio. | Proveedor autenticado. |
| `GET /api/proveedores?especie=roedores` | Consultar catálogo filtrado. | Propietario; acceso público por acordar. |
| `GET /api/proveedores/:id` | Consultar ficha y servicios. | Mismo criterio del catálogo. |
| `GET /api/servicios/:id/disponibilidad?fecha=AAAA-MM-DD` | Consultar horarios disponibles. | Propietario. |
| `POST /api/reservas` | Solicitar reserva. | Propietario autenticado. |
| `GET /api/mis-solicitudes` | Consultar solicitudes propias y sus estados. | Propietario autenticado. |
| `GET /api/agenda` | Consultar solicitudes recibidas en orden cronológico. | Proveedor autenticado. |
| `PATCH /api/reservas/:id/estado` | Confirmar o rechazar con motivo. | Proveedor de la reserva. |

El propietario y el proveedor de una operación se obtienen de la identidad autenticada y de las relaciones guardadas, no de un identificador enviado por el cliente sin verificar.

Propuesta de respuestas: `201` para creación, `200` para consulta o actualización, `400` para datos inválidos, `401` para falta de autenticación, `403` para permisos insuficientes, `404` para recurso inexistente y `409` para correo duplicado o conflicto de disponibilidad. Un catálogo sin coincidencias devuelve una lista vacía para que frontend presente el mensaje de RF-10.

Se propone un error uniforme con `codigo`, `mensaje` y, cuando aplique, `campos`, sin exponer contraseñas, consultas SQL ni detalles internos. Acordar el contrato antes de integrar; estos códigos y rutas no constituyen evidencia de una API funcionando.

## Calidad, seguridad y privacidad

| Requisito del PDF | Trabajo previsto |
| --- | --- |
| RNF-01: búsqueda inferior a 3 segundos | Medir consultas e integración con frontend; definir volumen de datos y condiciones de prueba. |
| RNF-02: función de resumen con sal | Guardar contraseñas como hash con sal mediante una función apropiada para contraseñas; nunca en texto plano ni devolverlas al cliente. |
| RNF-04: disponibilidad del 99 % | Definir alojamiento y periodo de medición, observar disponibilidad y registrar incidentes. Es una meta, no un nivel ya obtenido. |
| RNF-05: tratamiento de datos personales | Incorporar las medidas requeridas por el proyecto para el tratamiento conforme a la Ley 1581 de 2012; validar política, finalidad y acceso antes de usar datos reales. |
| RNF-08: documentación y versionado | Documentar API, modelo, configuración, migraciones y pruebas en el repositorio. |

RNF-03, RNF-06 y RNF-07 se verifican principalmente en la interfaz, con apoyo del backend para completar el recorrido.

Pautas técnicas propuestas: validar entradas en servidor, usar consultas parametrizadas, comprobar la propiedad de cada recurso, proteger sesiones y acordar orígenes permitidos para la interfaz. El mecanismo concreto de sesión y sus medidas de protección quedan pendientes de decisión conjunta.

Las credenciales de base de datos y los secretos se configurarán fuera del código. Un futuro `.env.example` contendrá solo nombres y valores ficticios; los archivos con secretos reales no se publicarán. Las pruebas y ejemplos públicos utilizarán información sintética y no respuestas individuales de encuestas.

## Organización propuesta de archivos

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

Salvo `README.md`, estos archivos y carpetas todavía no están creados. Se propone separar las rutas, reglas de negocio y acceso a MySQL para mantener cada responsabilidad identificable.

## Consulta y ejecución

Para consultar esta rama localmente:

```bash
git clone --branch backend --single-branch https://github.com/Dumo04/PetCare.git
cd PetCare
```

**No hay comandos de instalación, migración o arranque disponibles todavía.** Primero deben incorporarse el código, `package.json`, el esquema de datos y la configuración de ejemplo. Luego se documentarán las versiones de Node.js y MySQL, las dependencias, las variables requeridas, el puerto, la creación de la base y los comandos reales verificados. No se necesita un servidor para leer esta documentación.

## Pruebas previstas

- [ ] Registro válido, correo duplicado, credenciales incorrectas y permisos por rol (HU-01).
- [ ] Publicación completa y rechazo cuando falte precio o especie (HU-02).
- [ ] Filtro por especie con coincidencias y lista vacía (HU-03).
- [ ] Ficha con servicios y proveedor sin servicios publicados (HU-04).
- [ ] Reserva válida en estado pendiente y rechazo de un horario ocupado (HU-05).
- [ ] Agenda propia ordenada y agenda sin solicitudes (HU-06).
- [ ] Confirmación por el proveedor correcto, rechazo con motivo y acceso denegado a terceros (HU-07).
- [ ] Consulta de estados propios y motivo de rechazo (HU-08).
- [ ] Solicitudes o confirmaciones simultáneas sin sobrepasar la capacidad acordada.
- [ ] Búsqueda inferior a tres segundos y validación del recorrido junto con frontend.

Ninguna casilla está marcada porque no existen resultados de ejecución en esta entrega documental.

## Equipo y planificación

| Integrante | Rol y responsabilidad según el PDF |
| --- | --- |
| Ariana Calderón Fuentes | Product Owner y desarrollo; HU-01, registro e ingreso (8 puntos). |
| Heidy Viviana Cárdenas Soler | Desarrollo; HU-02, publicación de servicios (5 puntos). |
| Laura Valentina Ñustes Contento | Desarrollo; HU-03, catálogo por especie (5 puntos). |
| Santiago Duque Mora | Scrum Master y desarrollo; Jira, repositorio y entorno; acompañamiento de las historias. |

El Sprint 1 está planificado del **5 al 18 de octubre de 2026** con 18 puntos. El documento indica 64 horas disponibles entre los cuatro integrantes durante esas dos semanas; esa disponibilidad no establece una conversión fija entre puntos y horas. Las responsabilidades de las historias abarcan ambas capas y no fijan una división exclusiva del equipo entre frontend y backend.

## Ramas y colaboración

- `main`: presentación del proyecto y enlaces.
- `frontend`: documentación y futuro código del cliente.
- `backend`: este README y futuro código del servidor.

Cada rama de componente tiene su propio `README.md` en la raíz. Mantenerlos separados para la entrega; acordar una estructura de integración antes de fusionar ramas para evitar sobrescribir uno con el otro. Asociar los cambios a las historias del PDF y a sus tarjetas equivalentes en Jira.

## Alcance excluido y recursos

Fuera del MVP: perfil de mascota con historial de vacunación, filtros por ciudad y precio, notificaciones automáticas, historial de reservas y reportes, pagos, geolocalización, reseñas, mensajería y aplicación móvil nativa. No se proponen endpoints para esas funciones en esta entrega.

- [Jira del proyecto](https://santiagosworkspace-32817046.atlassian.net/jira/software/projects/PET/boards/1).
- [Diseños de PetCare en Figma, referenciados por el PDF](https://www.figma.com/design/EUB7PbmZvgfWeAtE1QbvYk/PetCare).
- Documento académico del equipo citado en la sección de fuente; las respuestas individuales de investigación no forman parte del repositorio público.

Pendientes antes de implementar: versiones, sesiones, contrato definitivo de API, política de cupos pendientes, duración y zona horaria, esquema SQL, alojamiento y licencia.
