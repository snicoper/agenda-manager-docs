# CalendarHoliday

**Entity**: `CalendarHoliday`

## Descripción General

`CalendarHoliday` representa un día festivo asociado a un calendario. Permite definir períodos festivos que pueden ser recurrentes o puntuales, aplicables a días específicos de la semana dentro del período establecido

### Responsabilidades

- **Gestión Temporal**:
  - Mantener la información del período festivo
  - Gestionar los días de la semana aplicables
  - Validar la coherencia temporal del holiday

- **Eventos de Dominio**:
  - Notificar la creación y modificación de holidays
  - Informar cambios en el período o días aplicables

- **Validación**:
  - Asegurar la validez del período temporal
  - Validar los días de la semana seleccionados
  - Verificar el formato y longitud de nombre y descripción

## Invariantes

- `Id` no puede ser `null` en ningún momento
- `Name` no puede ser `null` ni vacío en ningún momento y debe tener una longitud máxima de 50 caracteres
- `Description` puede ser `null` o vacío, pero si está presente, no debe exceder los 500 caracteres
- `Weekdays` debe de tener un valor distinto a `None`

## Reglas de negocio

- **Unicidad de Identificador**:
  - `Id` no puede ser `null` en ningún momento.

- **Nombre y Descripción**:
  - El nombre del holiday debe ser único en toda la aplicación y debe tener entre 1 y 50 caracteres.
  - La descripción del holiday debe tener entre 1 y 500 caracteres.

- **Reglas Temporales**:
  - El período debe tener una fecha de inicio válida
  - Si el período es recurrente, debe tener una fecha de fin
  - Los días de semana seleccionados deben ser coherentes con el período
  - No pueden existir dos holidays que se solapen en el mismo calendario para los mismos días de la semana

## Propiedades

| Nombre         | Tipo                | Descripción                           |
| -------------- | ------------------- | ------------------------------------- |
| `Id`           | `CalendarHolidayId` | Identificador único del holiday       |
| `CalendarId`   | `CalendarId`        | Identificador del calendario asociado |
| `Calendar`     | `Calendar`          | Referencia al calendario asociado     |
| `Period`       | `Period`            | Periodo del holiday                   |
| `Weekdays`     | `Weekdays`          | Días de la semana del holiday         |
| `Name`         | `string`            | Nombre del holiday                    |
| `Description`  | `string`            | Descripción del holiday               |

## Relaciones

- Un `CalendarHoliday` pertenece a un único `Calendar`
- La relación con `Calendar` es obligatoria y se mantiene a través de `CalendarId`
- La eliminación de un `Calendar` implica la eliminación en cascada de sus `CalendarHoliday`s

## Métodos

```csharp
public static CalendarHoliday Create(
    CalendarHolidayId calendarHolidayId,
    CalendarId calendarId,
    Period period,
    WeekDays weekDays,
    string name,
    string description)
```

- Crea una nueva instancia de `CalendarHoliday` con los valores proporcionados
- `id`: Identificador único del holiday
- `calendarId`: Identificador del calendario asociado
- `name`: Nombre del holiday
- `period`: Período del holiday
- `weekdays`: Días de la semana del holiday
- `name`: Nombre del holiday
- `description`: Descripción del holiday
- Lanza el evento `CalendarHolidayCreatedDomainEvent`
- Return `CalendarHoliday`

## Estado y Transiciones

## Dependencias

- **Entidades**:
  - [Calendar](./calendar.md): Referencia al calendario asociado
- **Services**:
  - [ICalendarRepository](./interfaces/i-calendar-holiday-repository.md): Servicio para gestionar calendarios
- **Managers**:
- **Policies**:
- **Value Objects**:
  - [CalendarHolidayId](./value-objects/calendar-holiday-id.md): Identificador único del holiday
  - [CalendarId](./value-objects/calendar-id.md): Identificador único del calendario
  - [Period](../common/value-objects/period/period.md): Representa un periodo de tiempo
  - [Weekdays](../common/weekdays/weekdays.md): Representa los días de la semana

## Eventos de Dominio

- `CalendarHolidayCreatedDomainEvent(calendarHoliday.Id)`: Evento de dominio disparado cuando se crea un nuevo `CalendarHoliday`.

## Interceptores EF Core

## Comentarios adicionales

## Ejemplos de Uso

```csharp
```
