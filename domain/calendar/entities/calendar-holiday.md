# CalendarHoliday

**Entity**: `CalendarHoliday`

## Descripción General

`CalendarHoliday` representa un día festivo asociado a un calendario. Permite definir períodos festivos que pueden ser recurrentes o puntuales, aplicables a días específicos de la semana dentro del período establecido.

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

- **Solapamiento con Appointments**:
  - El solapamiento con appointments debe ser verificado antes de la creación o modificación de un holiday
  - La decisión de solapamiento debe ser tomada por quien utiliza el sistema
  - La opción de cómo manejar el solapamiento debe ser configurable en `CalendarSettings`
  - Las posibles estrategias de manejo del solapamiento son:
    - `RejectIfOverlapping`: Rechaza la creación/modificación del holiday si existen citas solapadas
    - `CancelOverlapping`: Cancela automáticamente las citas que se solapan y crea/modifica el holiday
    - `AllowOverlapping`: Permite la creación/modificación del holiday manteniendo las citas existentes
  - El sistema debe notificar apropiadamente:
    - El número de citas afectadas antes de aplicar la estrategia
    - Las citas que han sido canceladas si se usa `CancelOverlapping`
    - Las citas que quedan solapadas si se usa `AllowOverlapping`

## Invariantes

- `Id` no puede ser `null` en ningún momento
- `Name` no puede ser `null` ni vacío en ningún momento y debe tener una longitud máxima de 50 caracteres
- `Description` puede ser `null` o vacío, pero si está presente, no debe exceder los 500 caracteres
- `Weekdays` debe de tener un valor distinto a `None`

## Reglas de negocio

- **Unicidad de Identificador**:
  - `Id` no puede ser `null` en ningún momento y debe ser único en toda la aplicación.

- **Nombre y Descripción**:
  - El nombre del holiday debe ser único en toda la aplicación y debe tener entre 1 y 50 caracteres.
  - La descripción del holiday debe tener entre 1 y 500 caracteres.

- **Reglas Temporales**:
  - El período debe tener una fecha de inicio válida
  - Si el período es recurrente, debe tener una fecha de fin
  - Los días de semana seleccionados deben ser coherentes con el período
  - No pueden existir dos holidays que se solapen en el mismo calendario para los mismos días de la semana

- **Gestión de Solapamientos**:
  - La creación o modificación de un holiday debe respetar la estrategia de solapamiento configurada en `CalendarSettings`
  - Las estrategias de solapamiento son mutuamente excluyentes y deben aplicarse de manera consistente:
    - Si la estrategia es `RejectIfOverlapping`, cualquier solapamiento debe resultar en un rechazo
    - Si la estrategia es `CancelOverlapping`, todas las citas solapadas deben ser canceladas antes de crear/modificar el holiday
    - Si la estrategia es `AllowOverlapping`, se debe permitir el solapamiento pero registrar y notificar la situación
  - La modificación de un holiday existente debe reevaluar los solapamientos si se modifica el período o los días de la semana

## Propiedades

| Nombre         | Tipo                | Descripción                                               |
| -------------- | ------------------- | --------------------------------------------------------- |
| `Id`           | `CalendarHolidayId` | Identificador único del holiday                           |
| `CalendarId`   | `CalendarId`        | Identificador del calendario asociado                     |
| `Calendar`     | `Calendar`          | Referencia al calendario asociado                         |
| `SettingsId`   | `CalendarSettingsId`| Identificador de la configuración del calendario asociado |
| `Settings`     | `CalendarSettings`  | Configuración del calendario asociado                     |
| `Period`       | `Period`            | Periodo del holiday                                       |
| `Weekdays`     | `Weekdays`          | Días de la semana del holiday                             |
| `Name`         | `string`            | Nombre del holiday                                        |
| `Description`  | `string`            | Descripción del holiday                                   |

## Relaciones

- Un `CalendarHoliday` pertenece a un único `Calendar`
- La relación con `Calendar` es obligatoria y se mantiene a través de `CalendarId`
- La relación con `CalendarSettings` es obligatoria y se mantiene a través de `SettingsId`
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

```csharp
public void Update(Period period, WeekDays weekDays, string name, string description)
```

- Actualiza los valores del holiday
- `period`: Período del holiday
- `weekdays`: Días de la semana del holiday
- `name`: Nombre del holiday
- `description`: Descripción del holiday
- Lanza el evento `CalendarHolidayUpdatedDomainEvent`

```csharp
private static void GuardAgainstInvalidName(string name)
```

- Valida que el nombre del holiday no sea nulo o vacío
- `name`: Nombre del holiday
- Lanza `CalendarHolidayDomainException` si el nombre no cumple con los requisitos

```csharp
private static void GuardAgainstInvalidDescription(string description)
```

- Valida que la descripción del holiday no exceda los 500 caracteres
- `description`: Descripción del holiday
- Lanza `CalendarHolidayDomainException` si la descripción no cumple con los requisitos

## Estado y Transiciones

## Dependencias

- **Entities**:
  - [Calendar](../calendar.md): Referencia al calendario asociado
- **Services**:
  - [ICalendarRepository](../interfaces/i-calendar-holiday-repository.md): Servicio para gestionar calendarios
- **Managers**:
  - [CalendarManager](../managers/calendar-manager.md): Gestiona la lógica de negocio relacionada con los calendarios
- **Policies**:
- **Value Objects**:
  - [CalendarHolidayId](../value-objects/calendar-holiday-id.md): Identificador único del holiday
  - [CalendarId](../value-objects/calendar-id.md): Identificador único del calendario
  - [Period](../../common/value-objects/period/period.md): Representa un periodo de tiempo
  - [Weekdays](../../common/weekdays/weekdays.md): Representa los días de la semana

## Eventos de Dominio

- `CalendarHolidayCreatedDomainEvent(Id)`: Evento de dominio disparado cuando se crea un nuevo `CalendarHoliday`.
- `CalendarHolidayUpdatedDomainEvent(Id)`: Evento de dominio disparado cuando se actualiza un `CalendarHoliday`.

## Interceptores EF Core

## Comentarios adicionales

Para crear o actualizar un nuevo `CalendarHoliday`, require de `CalendarManager` para la creación de un nuevo `CalendarHoliday`.

## Ejemplos de Uso

```csharp
// Crear un nuevo CalendarHoliday
var calendarHoliday = Create(
    calendarHolidayId: CalendarHolidayId.Create(),
    calendarId: CalendarId.From(Guid.Parse("00000000-0000-0000-0000-000000000000")),
    period: Period.From(DateTimeOffset.Now, DateTimeOffset.Now.AddDays(1)),
    weekDays: WeekDays.Weekdays,
    name: "New name",
    description: "New Description");
```
