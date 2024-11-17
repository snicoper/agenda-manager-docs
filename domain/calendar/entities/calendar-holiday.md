# CalendarHoliday

- **Aggregate Root**: `CalendarHoliday`
- **Namespace**: `AgendaManager.Domain.Calendars.Entities`
- **Tipo**: Entidad de Dominio Sellada (sealed)
- **Herencia**: `AggregateRoot`

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

## Propiedades

| Nombre         | Tipo                | Acceso           | Descripción                                               |
| -------------- | ------------------- | ---------------------------------------------------------------------------- |
| `Id`           | `CalendarHolidayId` | get              | Identificador único del holiday                           |
| `CalendarId`   | `CalendarId`        | get/private set  | Identificador del calendario asociado                     |
| `Calendar`     | `Calendar`          | get/private set  | Referencia al calendario asociado                         |
| `SettingsId`   | `CalendarSettingsId`| get/private set  | Identificador de la configuración del calendario asociado |
| `Settings`     | `CalendarSettings`  | get/private set  | Configuración del calendario asociado                     |
| `Period`       | `Period`            | get/private set  | Periodo del holiday                                       |
| `Weekdays`     | `Weekdays`          | get/private set  | Días de la semana del holiday                             |
| `Name`         | `string`            | get/private set  | Nombre del holiday                                        |
| `Description`  | `string`            | get/private set  | Descripción del holiday                                   |

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

## Métodos

### Create

```csharp
public static CalendarHoliday Create(
    CalendarHolidayId calendarHolidayId,
    CalendarId calendarId,
    Period period,
    WeekDays weekDays,
    string name,
    string description)
```

- **Descripción**: Crea una nueva instancia de `CalendarHoliday` con los valores proporcionados
- **Parámetros**:
  - `id`: Identificador único del holiday
  - `calendarId`: Identificador del calendario asociado
  - `name`: Nombre del holiday
  - `period`: Período del holiday
  - `weekdays`: Días de la semana del holiday
  - `name`: Nombre del holiday
  - `description`: Descripción del holiday
- **Eventos**: `CalendarHolidayCreatedDomainEvent`: Evento de creación del holiday
- **Retorno**: `CalendarHoliday` con los valores proporcionados.

### Update

```csharp
public void Update(Period period, WeekDays weekDays, string name, string description)
```

- **Descripción**: Actualiza los valores del holiday
- **Parámetros**:
  - `period`: Período del holiday
  - `weekdays`: Días de la semana del holiday
  - `name`: Nombre del holiday
  - `description`: Descripción del holiday
- **Eventos**: `CalendarHolidayUpdatedDomainEvent`: Evento de actualización del holiday

### GuardAgainstInvalidName

```csharp
private static void GuardAgainstInvalidName(string name)
```

- **Descripción**: Valida que el nombre del holiday no sea nulo o vacío
- **Parámetros**:
  - `name`: Nombre del holiday
- **Eventos**: `CalendarHolidayDomainException`: Evento de excepción de Dominio

### GuardAgainstInvalidDescription

```csharp
private static void GuardAgainstInvalidDescription(string description)
```

- **Descripción**: Valida que la descripción del holiday no exceda los 500 caracteres
- **Parámetros**:
  - `description`: Descripción del holiday
- **Excepciones**: `CalendarHolidayDomainException`: Evento de excepción de dominio

## Estado y Transiciones

- **No Aplica**

## Dependencias

### Directas

- **Entidades Base**:
  - `AuditableEntity`: Base class que proporciona capacidades de auditoría
    - Hereda gestión de eventos de dominio (`Entity`)

### Entidades

- **Calendar**: Referencia al calendario asociado

### Servicios

- `ICalendarRepository`: Servicio para gestionar calendarios

### Managers

- `CalendarManager`: Gestiona la lógica de negocio relacionada con los calendarios

### Políticas

### Value Objects

- `CalendarHolidayId`: Identificador único del holiday
- `CalendarId`: Identificador único del calendario
- `Period`: Representa un periodo de tiempo
- `Weekdays`: Días de la semana del holiday
ys.md): Representa los días de la semana

## Comentarios adicionales

Para crear o actualizar un nuevo `CalendarHoliday`, require de `CalendarManager` para la creación de un nuevo `CalendarHoliday`.

## Ejemplos de Uso

```csharp
// Crear un nuevo CalendarHoliday
var calendarHoliday = CalendarHoliday.Create(
    calendarHolidayId: CalendarHolidayId.Create(),
    calendarId: CalendarId.From(Guid.Parse("00000000-0000-0000-0000-000000000000")),
    period: Period.From(DateTimeOffset.Now, DateTimeOffset.Now.AddDays(1)),
    weekDays: WeekDays.Weekdays,
    name: "New name",
    description: "New Description");
```
