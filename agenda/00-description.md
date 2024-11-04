# Notas

## Herramientas

<https://mermaid.js.org/syntax/classDiagram.html>

## Descripción del proyecto

Crear una Agenda para gestión de citas.

Se necesita gestionar los `Appointment` de los `Clients` (Role) en un `Calendar`.

Se debe de contar con unos `Resource` y tipos de recursos `ResourceType` para poder gestionar los recursos de la agenda.

Los `Resource` con `especialidad` pueden ser `Users` o generados de manera dinámica.

`ResourceType` puede ser `Box` o `Equipment` y uno de ellos debe ser de tipo `Person` (inmutable).

Los `Resource` deberán tener `CalendarEvent` con el tiempo de disponibilidad en un rango de `Start` y `End`.

También los `Resource` podrán tener `CalendarEvent` con un `Unavailable` que se aplica a un rango de `Start` y `End` y los días de la semana que se aplican (Monday, Tuesday, Wednesday, Thursday, Friday, Saturday, Sunday).

Para los diferentes tipos de `CalendarEvent`, se deberá tener un `ScheduleType` que indica si es un evento `Available` o un evento `Unavailable`.

Las `ScheduleType` de tipo `Unavailable` tendrán precedencia a los de tipo `Available`.

Se deberá poder añadir `CalendarEvents` indicando los `Resource` y el tiempo necesario para crear un `Appointment`.

Los `Appointment` deberán tener seguimiento del estado de la cita tipo (Pending, Accepted, Rejected, Canceled, Completed) y se vera su estado en `AppointmentStatus`.

`CalendarHoliday` Determinas días festivos que no se podrán programar citas, precedencia mas alta que los `ResourceSchedule`.

## Apuntes y notas que voy recopilando

`CalendarEvent`

* **CalendarEvent**
  * **Ortodoncia**
  * **Recursos**
    * **Ortodoncia** (Seleccionar en base a disponibilidad de los recursos y horarios de una lista de resources categorizados como ortodoncistas)
    * **Anestesista** (Seleccionar en base a disponibilidad de los recursos y horarios de una lista de resources categorizados como anestesistas)
    * **Box** (Seleccionar en base a disponibilidad de los recursos y horarios de una lista de resources categorizados como box)
    * **Instrumental (opcional)** (Seleccionar en base a disponibilidad de los recursos y horarios de una lista de resources categorizados como instrumental)
  * **Duración** 1 hora

Por lo tanto y desde el punto de vista de administración (back office se podría llamar también?)

* Crear recursos (users, boxes, instrumentals, etc), asignar al recurso la especialidad que se le asigna.
* Los de tipo users, [mostrar lista de usuarios en el sistema (quiza filtrando por roles)]
* Los de tipo boxes, instrumental, etc, (a esto se le ha de dar una vuelta)
* Schedule de tipo Available para asignar a los recursos de tipo users, la disponibilidad de horarios para cada día de la semana entre un rango de fechas.
* Schedule de tipo Unavailable, simplemente una Exception a un recurso, por ejemplo que se va de vacaciones, etc.
* CalendarEvent, asignar un conjunto de especialidades necesarias y recursos necesarios (estos mas bien de tipo box o instrumental) y una duración.

De esta manera, para crear un Appointment de tipo CalendarEvent, por ejemplo extracción, ya el sistema sabe que recursos necesita y la duración.

**Back office - Administración:**

1. **Resource:**
    * **Users:**
        * **Filtro por roles:**  Permite a los administradores filtrar por roles (Doctor, Anestesista, Higienista,
etc.) para gestionar usuarios específicos.
        * **Asigna especialidades:**  Cada usuario puede tener una o varias especialidades asociadas.
    * **Boxes:**
        * **Descripción:** Nombre del box, ubicación, capacidad, etc.
        * **Tipo:**  Podría tener categorías como "Cirugía", "Ortodoncia", "General", etc.
        * **Disponibilidad:**  Estado actual (Disponible, Ocupado), con posibilidad de programar la disponibilidad
a largo plazo.
    * **Equipment:**
        * **Descripción:** Nombre del instrumental, tipo, categoría.
        * **Estado:** Disponible, en uso, mantenimiento.
        * **Ubicación:**  Asociado a un box o categoría específica.
2. **ResourceSchedule:**
    * **Available:**
        * **Recurso:**  Usuario (Doctor, Anestesista, etc.)
        * **Disponibilidad:**  Horarios disponibles para cada día de la semana dentro de un rango de fechas.
        * **Notificaciones:** Recordatorios para el usuario y los administradores sobre eventos próximos.
    * **Unavailable:**
        * **Recurso:** Usuario
        * **Fecha y hora:** Periodo de inactividad o ausencia.
        * **Motivo:** Vacaciones, capacitación, enfermedad, etc.
3. **CalendarEvent:**
    * **Nombre:**  Nombre del servicio (extracción, ortodoncia, limpieza, etc.)
    * **Especialidades necesarias:**  Listado de especialidades requeridas para el servicio.
    * **Recursos:**
        * **Boxes:**  Boxes específicos necesarios para el servicio.
        * **Instrumentals:**  Instrumentales específicos requeridos para el servicio.
    * **Duración:** Tiempo estimado para realizar el servicio.
4. **Appointment:** Cita programada con un servicio específico en un horario determinado.
    * **Estado:**  Estado actual de la cita (Pendiente, Aceptada, Rechazada, Cancelada, Completada).
    * **Notificaciones:**  Recordatorios para el usuario y los administradores sobre eventos próximos.
5. **Calendar:**

**Para crear un Appointment:**

1. **Seleccionar CalendarEvent:**  El sistema muestra los servicios disponibles.
2. **Elegir fecha y hora:**  Se muestra la disponibilidad de los recursos (usuarios, boxes, instrumentals) para la
fecha y hora elegida.
3. **Confirmar:**  El sistema genera la cita, asignando los recursos necesarios.
4. **Notificaciones:**  Se envían notificaciones al usuario (paciente) y al equipo de atención.

## Domain Model

### Calendars

* El `Calendar` representa un conjunto de eventos que se pueden programar en un `Calendar`, cada `Calendar` tendrá sus propios `Resource`, `CalendarEvent`, etc.
* En `Calendar` comparte todos los `User` del sistema.

### CalendarHolidays

* Un `CalendarHoliday` representa un día festivo en un `Calendar`, aplicable a todos los `ResourceSchedules` del `Calendar`.

### Resources

* Un `Resource` representa un recurso que puede ser utilizado en un `CalendarEvent`.
* Los `Resource` pueden ser de diferentes tipos, como `User`, `Box`, `Instrumental`.

### ResourceSchedules

* Un `ResourceSchedule` tiene asociado un `Resource`.
* Un `ResourceSchedule` representa la disponibilidad de un `Resource`.
* Los `ResourceSchedule` tienen dos tipos, `Available` y `Unavailable`.
* Un `ResourceSchedule` tiene asociado un `Period`.
* Un `ResourceSchedule` tienes disponibilidad diaria con `WeekDays`.
* Un `Resource` puede tener uno o varios `ResourceSchedule`.

### CalendarEvents

* Un `CalendarEvent` son **servicios** e indica los `ResourceType` necesarios para realizar el `CalendarEvent`.
* Un `CalendarEvent` tiene asociado un `Calendar`.

### Appointments

* Un `Appointment` representa una cita programada en un `Calendar`.
* Un `Appointment` tiene asociado un `CalendarEvent`.
* Un `Appointment` tiene un `AppointmentStatus` que representa el estado de la cita.

### Users

* Un `User` representa un usuario del sistema que puede ser asignado a un `Resource` en un `Calendar` o un `Customer`.
* El tipo de `User` se define en el `Role`.

## Types

Algunos de los **Enums** o **ValueObjects** que se utilizan en el sistema.

* `ResourceType` **Enum** de los tipos de `Resource`.
* `Period` **ValueObject** que representa un periodo de tiempo.
* `AppointmentStatus` **Enum** de los estados de un `Appointment`.
* `WeekDays` **Enum** de los días de la semana.
