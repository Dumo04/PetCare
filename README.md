# PetCare | Frontend

Aplicación web para conectar propietarios de mascotas con proveedores de servicios de cuidado. Este componente presenta el recorrido **encontrar, comparar y reservar**, con interfaces diferenciadas para propietarios y proveedores.

**Rama:** `frontend` · **Estado:** documentación inicial de análisis y diseño · **Actualización:** 4 de octubre de 2026.

> Este README define el trabajo previsto. La rama contiene documentación; todavía no incluye una aplicación ejecutable ni acredita funcionalidades implementadas o pruebas aprobadas.

[README del backend](https://github.com/Dumo04/PetCare/blob/backend/README.md) · [Repositorio](https://github.com/Dumo04/PetCare) · [Tablero de Jira](https://santiagosworkspace-32817046.atlassian.net/jira/software/projects/PET/boards/1)

## Contexto académico y fuente

Proyecto de la línea B, **PetCare**, para Análisis y Diseño de Sistemas, Programa de Ingeniería de Software, Facultad de Ingeniería, Corporación Universitaria Iberoamericana. Docente: Tatiana Cabrera.

La fuente es el documento del equipo **“Actividad 1 - Identificar el proyecto tecnológico a trabajar”**, septiembre de 2026, en la versión PDF suministrada el 4 de octubre de 2026. Este README toma el alcance del numeral 6.2 (p. 28), la planificación del 6.3 (p. 29), los requisitos del 7.1 y 7.2 (pp. 30-31), las historias del 7.3 (pp. 32-33) y las decisiones de interfaz del 5.3 (pp. 26-27).

Los identificadores HU y RF corresponden a esta versión del PDF; no deben confundirse con la numeración de borradores anteriores de Jira. Las estructuras y pautas técnicas marcadas como propuestas se añaden para orientar la implementación y pueden ajustarse por el equipo.

## Problema y objetivo

La información sobre servicios está dispersa y suele requerir conversaciones individuales para conocer precios, horarios y especies atendidas. PetCare busca centralizar esa información y permitir que el propietario solicite una cita y consulte su respuesta, mientras el proveedor organiza las solicitudes en una agenda.

El frontend debe hacer visible la información necesaria para decidir, mantener un flujo sencillo en móvil y comunicar de forma clara los estados de las reservas.

## Tecnologías

| Tecnología definida en el PDF | Uso previsto |
| --- | --- |
| HTML | Estructura semántica de las páginas y formularios. |
| CSS | Presentación y adaptación a escritorio, tableta y móvil. |
| JavaScript | Interacciones, validaciones de interfaz y comunicación con el backend. |

El backend previsto utiliza Node.js, Express y MySQL. No se presupone el uso de React, Angular u otro framework de interfaz.

## Roles y pantallas

| Pantalla prevista | Usuario | Contenido y comportamiento |
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

| Historia del PDF | Requisitos | Resultado esperado en la interfaz | Prioridad |
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

## Integración prevista con el backend

El contrato de API se propone en el [README del backend](https://github.com/Dumo04/PetCare/blob/backend/README.md). Las rutas aún no están implementadas.

- Consumir datos de autenticación, catálogo, detalle, disponibilidad, solicitudes y agenda.
- Representar estados de carga, lista vacía, error y resultado exitoso.
- Conservar los datos pertinentes del formulario ante un error y permitir corregirlo sin repetir todo el recorrido.
- Reconsultar la disponibilidad después de un conflicto y actualizar las solicitudes tras una respuesta del proveedor.
- Centralizar la URL de la API como configuración pública. Nunca incluir contraseñas de MySQL, claves privadas ni credenciales en el frontend.
- Acordar con backend el mecanismo de sesión, el formato de errores y la zona horaria antes de integrar. Las validaciones del navegador no sustituyen las validaciones y permisos del servidor.

## Organización propuesta de archivos

La siguiente estructura es una propuesta, no una lista de archivos ya creados:

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

## Consulta y ejecución

Para consultar esta documentación en una copia local:

```bash
git clone --branch frontend --single-branch https://github.com/Dumo04/PetCare.git
cd PetCare
```

**No hay comando de ejecución disponible todavía:** faltan las páginas, estilos y scripts. Cuando el equipo incorpore el código, debe documentar el servidor local elegido, la URL de la API, los pasos de inicio y los datos ficticios de prueba. No se requiere instalar dependencias para leer este README.

## Calidad y validación prevista

Los siguientes puntos son metas del PDF y actividades pendientes, no resultados comprobados:

| Requisito | Validación prevista |
| --- | --- |
| RNF-01: búsqueda inferior a 3 segundos | Medir el recorrido de consulta y presentación con backend integrado y registrar las condiciones. |
| RNF-03: diseño adaptable | Revisar catálogo, formularios y agenda en escritorio, tableta y móvil. |
| RNF-06: reservar en menos de 3 minutos | Medir el recorrido con usuarios sin experiencia previa. |
| RNF-07: compatibilidad | Probar en las dos versiones más recientes, al momento de la evaluación, de los navegadores principales acordados. |
| RNF-08: documentación y versionado | Mantener README, cambios y evidencias asociados a la historia trabajada. |

RNF-02 se resuelve principalmente en backend; RNF-04 requiere evaluar el servicio completo y RNF-05 exige tratar los datos personales según lo establecido en el documento. Para el repositorio y las pruebas públicas se usarán datos ficticios, sin publicar respuestas individuales de encuestas.

Como pautas de interfaz propuestas: asociar etiquetas a los campos, permitir navegación por teclado, mantener el foco visible y acompañar los colores de estado con texto.

Pruebas funcionales pendientes:

- [ ] Registro válido y correo duplicado; ingreso y panel por rol.
- [ ] Publicación completa y bloqueo cuando falte precio o especie.
- [ ] Filtro por especie con coincidencias y sin resultados.
- [ ] Detalle con servicios y proveedor sin servicios publicados.
- [ ] Solicitud válida y conflicto de horario con recuperación del formulario.
- [ ] Agenda ordenada y agenda vacía.
- [ ] Confirmación, rechazo con motivo y consulta del estado actualizado.

El PDF indica que aún no se aplicaron pruebas de usabilidad al prototipo. Prevé cinco participantes en el Sprint 3 para observar tiempo, finalización y dudas durante la tarea de encontrar proveedor y solicitar cita.

## Equipo y planificación

| Integrante | Rol y responsabilidad según el PDF |
| --- | --- |
| Ariana Calderón Fuentes | Product Owner y desarrollo; contacto con usuarios; HU-01 en Sprint 1 (8 puntos). |
| Heidy Viviana Cárdenas Soler | Desarrollo; HU-02 en Sprint 1 (5 puntos). |
| Laura Valentina Ñustes Contento | Desarrollo; HU-03 en Sprint 1 (5 puntos). |
| Santiago Duque Mora | Scrum Master y desarrollo; Jira, repositorio y entorno; apoyo a las tres historias. |

El Sprint 1 está planificado en el documento del **5 al 18 de octubre de 2026**, con 18 puntos. La disponibilidad indicada es de 64 horas del equipo para esas dos semanas; los puntos son una estimación relativa, no una conversión fija a horas. Las historias incluyen trabajo de frontend y backend; no asignan a una persona una sola capa de forma exclusiva.

## Trabajo con ramas

- `main`: presentación y enlaces a la documentación.
- `frontend`: este README y el desarrollo futuro de la interfaz.
- `backend`: README independiente del servidor y la base de datos.

Los dos documentos se llaman `README.md` y están en la raíz de sus respectivas ramas. Para esta entrega deben permanecer diferenciados. Antes de integrar ramas, acordar cómo conservar ambos documentos sin sobrescribir el contenido de una capa con el de la otra. Relacionar cada cambio con la historia del PDF y la tarjeta de Jira correspondiente, comprobando primero su equivalencia.

## Recursos

- [Jira del proyecto](https://santiagosworkspace-32817046.atlassian.net/jira/software/projects/PET/boards/1).
- [Diseños de PetCare en Figma, referenciados en el PDF](https://www.figma.com/design/EUB7PbmZvgfWeAtE1QbvYk/PetCare).
- Documento académico del equipo citado en la sección de fuente. No se publican aquí la encuesta ni las respuestas individuales.

Pendiente de acordar: versiones de herramientas, contrato definitivo de API, política de sesiones, alojamiento y licencia. Este README no declara un despliegue ni una aplicación terminada.
