# Notas

## Herramientas

<https://mermaid.js.org/syntax/classDiagram.html>

## Descripción del proyecto

Crear una Agenda para gestión de citas.

Se necesita gestionar los `Appointment` de los `Clients` (Role) en un `Calendar`.

Se debe de contar con unos `Resource` y tipos de recursos `ResourceType` para poder gestionar los recursos de la agenda.

Los `Resource` con `especialidad` pueden ser `Users` o generados de manera dinámica.

`ResourceType` puede ser `Box` o `Instrumental` y uno de ellos debe ser de tipo `User` (inmutable).

Los `Resource` deberán tener `Schedule` con el tiempo de disponibilidad en un rango de `Start` y `End` y los días de la semana que se aplican (Monday, Tuesday, Wednesday, Thursday, Friday, Saturday, Sunday).

También los `Resource` podrán tener `Schedule` con un `Exception` que se aplica a un rango de `Start` y `End` y los días de la semana que se aplican (Monday, Tuesday, Wednesday, Thursday, Friday, Saturday, Sunday).

Para los diferentes tipos de `Schedule`, se deberá tener un `ScheduleType` que indica si es un evento `Planned` o un evento `Exception`.

Las `ScheduleType` de tipo `Exception` tendrán precedencia a los de tipo `Planned`.

Se deberá poder añadir `CalendarEvents` indicando los `Resource` y el tiempo necesario para crear un `Appointment`.

Los `Appointment` deberán tener seguimiento del estado de la cita tipo (Pending, Accepted, Rejected, Canceled, Completed) y se vera su estado en `AppointmentStatus`.

## Apuntes y notas que voy recopilando

`Services` o `CalendarEvent` (TODO: Determinar nombre)

* **Servicio**
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
* Schedule de tipo Planned para asignar a los recursos de tipo users, la disponibilidad de horarios para cada día de la semana entre un rango de fechas.
* Schedule de tipo Exception, simplemente una Exception a un recurso, por ejemplo que se va de vacaciones, etc.
* Service, asignar un conjunto de especialidades necesarias y recursos necesarios (estos mas bien de tipo box o instrumental) y una duración.

De esta manera, para crear un Appointment de tipo Service, por ejemplo extracción, ya el sistema sabe que recursos necesita y la duración.

**Back office - Administración:**

1. **Recursos:**
    * **Users:**
        * **Filtro por roles:**  Permite a los administradores filtrar por roles (Doctor, Anestesista, Higienista,
etc.) para gestionar usuarios específicos.
        * **Asigna especialidades:**  Cada usuario puede tener una o varias especialidades asociadas.
    * **Boxes:**
        * **Descripción:** Nombre del box, ubicación, capacidad, etc.
        * **Tipo:**  Podría tener categorías como "Cirugía", "Ortodoncia", "General", etc.
        * **Disponibilidad:**  Estado actual (Disponible, Ocupado), con posibilidad de programar la disponibilidad
a largo plazo.
    * **Instrumentals:**
        * **Descripción:** Nombre del instrumental, tipo, categoría.
        * **Estado:** Disponible, en uso, mantenimiento.
        * **Ubicación:**  Asociado a un box o categoría específica.
2. **Schedule:**
    * **Planned:**
        * **Recurso:**  Usuario (Doctor, Anestesista, etc.)
        * **Disponibilidad:**  Horarios disponibles para cada día de la semana dentro de un rango de fechas.
        * **Notificaciones:** Recordatorios para el usuario y los administradores sobre eventos próximos.
    * **Exception:**
        * **Recurso:** Usuario
        * **Fecha y hora:** Periodo de inactividad o ausencia.
        * **Motivo:** Vacaciones, capacitación, enfermedad, etc.
3. **Service|CalendarEvent:**
    * **Nombre:**  Nombre del servicio (extracción, ortodoncia, limpieza, etc.)
    * **Especialidades necesarias:**  Listado de especialidades requeridas para el servicio.
    * **Recursos:**
        * **Boxes:**  Boxes específicos necesarios para el servicio.
        * **Instrumentals:**  Instrumentales específicos requeridos para el servicio.
    * **Duración:** Tiempo estimado para realizar el servicio.

**Para crear un Appointment:**

1. **Seleccionar servicio:**  El sistema muestra los servicios disponibles.
2. **Elegir fecha y hora:**  Se muestra la disponibilidad de los recursos (usuarios, boxes, instrumentals) para la
fecha y hora elegida.
3. **Confirmar:**  El sistema genera la cita, asignando los recursos necesarios.
4. **Notificaciones:**  Se envían notificaciones al usuario (paciente) y al equipo de atención.

## Definiciones

### Calendar

El calendario representa los eventos que se pueden programar en un calendario, cada calendario tendrá sus propios recursos, horarios, etc.
