# Agenda manager docs

## Herramientas

<https://mermaid.js.org/syntax/classDiagram.html>

## Descripción del proyecto

Crear una Agenda para gestión de citas.

Se necesita gestionar los `Appointment` de los `Clients` (Role) en un `Calendar`.

Se debe de contar con unos `Resource` y tipos de `Resource` para poder gestionar los recursos de la agenda.

Los `Resource` pueden ser `Users` o generados de manera dinámica.

Los `Resource` deberán tener `Schedule` con el tiempo de disponibilidad en un rango de `Start` y `End` y los días de la semana que se aplican (Monday, Tuesday, Wednesday, Thursday, Friday, Saturday, Sunday).

También los `Resource` podrán tener `Schedule` con un `Exception` que se aplica a un rango de `Start` y `End` y los días de la semana que se aplican (Monday, Tuesday, Wednesday, Thursday, Friday, Saturday, Sunday).

Para los diferentes tipos de `Schedule`, se deberá tener un `ScheduleType` que indica si es un evento `Event` o un evento `Exception`.

Las `ScheduleType` de tipo `Exception` tendrán precedencia a los de tipo `Event`.

Se deberá poder añadir `CalendarEvents` indicando los `Resource` y el tiempo necesario para crear un `Appointment`.

Los `Appointment` deberán tener seguimiento del estado de la cita tipo (Pending, Accepted, Rejected, Canceled, Completed) y se vera su estado en `AppointmentStatus`.
