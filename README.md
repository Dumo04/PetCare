# PetCare | Frontend

Aplicación web para conectar propietarios de mascotas con proveedores de servicios de cuidado. Este componente presenta el recorrido **encontrar, comparar y reservar**, con interfaces diferenciadas para propietarios y proveedores.

**Componente:** Frontend · **Rama:** `frontend`

[README del backend](https://github.com/Dumo04/PetCare/blob/backend/README.md) · [Repositorio](https://github.com/Dumo04/PetCare) · [Tablero de Jira](https://santiagosworkspace-32817046.atlassian.net/jira/software/projects/PET/boards/1)

## Problema y objetivo

La información sobre servicios está dispersa y suele requerir conversaciones individuales para conocer precios, horarios y especies atendidas. PetCare busca centralizar esa información y permitir que el propietario solicite una cita y consulte su respuesta, mientras el proveedor organiza las solicitudes en una agenda.

El frontend debe hacer visible la información necesaria para decidir, mantener un flujo sencillo en móvil y comunicar de forma clara los estados de las reservas.

## Tecnologías

| Tecnología | Uso definido |
| --- | --- |
| HTML | Estructura semántica de las páginas y formularios. |
| CSS | Presentación y adaptación a escritorio, tableta y móvil. |
| JavaScript | Interacciones, validaciones de interfaz y comunicación con el backend. |

El backend definido utiliza Node.js, Express y MySQL. La interfaz utiliza tecnologías web estándar.

## Roles y pantallas

| Pantalla definida | Usuario | Contenido y comportamiento |
| --- | --- | --- |
| Registro e inicio de sesión | Ambos roles | Datos de acceso, selección del rol al registrarse y acceso al panel correspondiente. |
| Publicación de servicios | Proveedor | Descripción, precio, horario de atención y especies atendidas. |
| Catálogo | Propietario | Proveedores, filtro por especie siempre visible y tarjetas con precio y especies atendidas. |
| Detalle del proveedor | Propietario | Descripción, especies y servicios con precio y horario; sin reserva si no hay servicios publicados. |
| Solicitud de reserva | Propietario | Servicio, nombre y especie de la mascota, fecha y hora disponible. |
| Mis solicitudes | Propietario | Estado pendiente, confirmada o rechazada; motivo cuando exista rechazo. |
| Agenda | Proveedor | Solicitudes ordenadas por fecha y hora, servicio y datos de la mascota. |
| Respuesta a la solicitud | Proveedor | Acciones confirmar o rechazar; motivo obligatorio para rechazar. |

## Funcionalidades y trazabilidad

| Historia de usuario | Requisitos | Resultado esperado en la interfaz | Prioridad |
| --- | --- | --- | --- |
| HU-01: registro e inicio de sesión | RF-01 | Redirigir al panel del rol; informar correo duplicado sin borrar los demás campos válidos. | Debe |
| HU-02: publicar servicios | RF-02 | Validar campos obligatorios y presentar el servicio publicado. | Debe |
| HU-03: filtrar catálogo por especie | RF-03, RF-10 | Mostrar solo coincidencias; informar ausencia de resultados y conservar el filtro editable. | Debe |
| HU-04: consultar detalle | RF-04 | Presentar descripción, especies, precio y horario; deshabilitar reserva sin servicios. | Debe |
| HU-05: solicitar reserva | RF-05, RF-06 | Enviar los datos y mostrar estado pendiente; volver a elegir horario si ya está ocupado. | Debe |
| HU-06: consultar agenda | RF-07 | Mostrar solicitudes ordenadas y un mensaje cuando la agenda esté vacía. | Debe |
| HU-07: confirmar o rechazar | RF-08 | Pedir motivo para rechazo y reflejar la respuesta guardada por el servidor. | Debe |
| HU-08: consultar estado | RF-09 | Mostrar el estado actualizado y el motivo de rechazo cuando corresponda. | Debería |

### Recorridos principales

**Propietario:** registro o ingreso → catálogo → filtro por especie → ficha del proveedor → servicio y horario → datos básicos de la mascota → solicitud pendiente → consulta de la respuesta.

**Proveedor:** registro o ingreso → publicación de servicios → agenda → revisión de solicitud → confirmación o rechazo con motivo → estado actualizado para ambas partes.

Los únicos estados de reserva definidos para este MVP son `pendiente`, `confirmada` y `rechazada`. La interfaz muestra el estado devuelto por el backend; no confirma una cita solo por un cambio visual local.

## Límites del MVP

Quedan para versiones posteriores: perfil de mascota con historial de vacunación, filtros por ciudad o rango de precio, notificaciones automáticas, historial de reservas y reportes, pagos en línea, geolocalización, reseñas, mensajería y aplicación móvil nativa.

Registrar nombre y especie de la mascota en una solicitud no equivale a construir un módulo de historia clínica. La vista **Mis solicitudes** permite seguir el estado actual y no amplía el alcance a reportes históricos.

## Integración definida con el backend

La integración se describe en el [README del backend](https://github.com/Dumo04/PetCare/blob/backend/README.md).

- Consumir datos de autenticación, catálogo, detalle, disponibilidad, solicitudes y agenda.
- Representar estados de carga, lista vacía, error y resultado exitoso.
- Conservar los datos pertinentes del formulario ante un error y permitir corregirlo sin repetir todo el recorrido.
- Reconsultar la disponibilidad después de un conflicto y actualizar las solicitudes tras una respuesta del proveedor.
- Centralizar la URL de la API como configuración pública. Nunca incluir contraseñas de MySQL, claves privadas ni credenciales en el frontend.
- Acordar con backend el mecanismo de sesión, el formato de errores y la zona horaria antes de integrar. Las validaciones del navegador no sustituyen las validaciones y permisos del servidor.

## Organización de referencia

Estructura de referencia para organizar el código del componente:

```text
README.md
index.html
pages/
  registro.html
  servicios.html
  proveedor.html
  reserva.html
  solicitudes.html
  agenda.html
assets/
  css/
  js/
    api.js
    auth.js
    catalogo.js
    reservas.js
  img/
```

## Acceso al componente

Para consultar esta documentación en una copia local:

```bash
git clone --branch frontend --single-branch https://github.com/Dumo04/PetCare.git
cd PetCare
```

## Criterios de calidad y validación

La validación del componente se realizará mediante los siguientes criterios de calidad:

| Requisito | Validación definida |
| --- | --- |
| RNF-01: búsqueda inferior a 3 segundos | Medir el recorrido de consulta y presentación con backend integrado y registrar las condiciones. |
| RNF-03: diseño adaptable | Revisar catálogo, formularios y agenda en escritorio, tableta y móvil. |
| RNF-06: reservar en menos de 3 minutos | Medir el recorrido con usuarios sin experiencia previa. |
| RNF-07: compatibilidad | Probar en las dos versiones más recientes, al momento de la evaluación, de los navegadores principales acordados. |
| RNF-08: documentación y versionado | Mantener README, cambios y evidencias asociados a la historia trabajada. |

RNF-02 se resuelve principalmente en backend; RNF-04 requiere evaluar el servicio completo y RNF-05 exige tratar los datos personales conforme a la especificación de requisitos. Para el repositorio y las pruebas públicas se usarán datos ficticios, sin publicar respuestas individuales de encuestas.

Pautas de interfaz: asociar etiquetas a los campos, permitir navegación por teclado, mantener el foco visible y acompañar los colores de estado con texto.

Casos de aceptación:

- Registro válido y correo duplicado; ingreso y panel por rol.
- Publicación completa y bloqueo cuando falte precio o especie.
- Filtro por especie con coincidencias y sin resultados.
- Detalle con servicios y proveedor sin servicios publicados.
- Solicitud válida y conflicto de horario con recuperación del formulario.
- Agenda ordenada y agenda vacía.
- Confirmación, rechazo con motivo y consulta del estado actualizado.

La evaluación de usabilidad contempla sesiones con cinco participantes y registra el tiempo empleado, la finalización de la tarea y las dudas observadas durante la búsqueda de un proveedor y la solicitud de una cita.

## Equipo y metodología

| Integrante | Rol |
| --- | --- |
| Ariana Calderón Fuentes | Product Owner y equipo de desarrollo. |
| Santiago Duque Mora | Scrum Master y equipo de desarrollo. |
| Laura Valentina Ñustes Contento | Equipo de desarrollo. |
| Heidy Viviana Cárdenas Soler | Equipo de desarrollo. |

El equipo utiliza Scrum y gestiona el backlog, las prioridades y el seguimiento de las historias en Jira. Las historias comprenden el trabajo de interfaz y servidor necesario para completar cada funcionalidad. La planificación de iteraciones y las asignaciones se mantienen en el tablero del proyecto.

## Trabajo con ramas

- `main`: presentación y enlaces a la documentación.
- `frontend`: documentación y código de la interfaz.
- `backend`: README independiente del servidor y la base de datos.

Cada componente mantiene su documentación en el archivo `README.md` de su rama. Los cambios se relacionan con las historias de usuario y las tareas de Jira. La integración deberá conservar la documentación específica de ambas capas.

## Recursos

- [Jira del proyecto](https://santiagosworkspace-32817046.atlassian.net/jira/software/projects/PET/boards/1).
- [Diseños de PetCare en Figma](https://www.figma.com/design/EUB7PbmZvgfWeAtE1QbvYk/PetCare).

## Mantenimiento de la documentación

Actualizar este README cuando cambien el alcance, la arquitectura, el contrato de integración o los procedimientos de configuración. Mantener la planificación temporal y el estado de las tareas en Jira, y registrar los cambios técnicos junto con el código correspondiente.
