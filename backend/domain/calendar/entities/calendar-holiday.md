# CalendarHoliday

- **Aggregate Root**: `CalendarHoliday`
- **Namespace**: `AgendaManager.Domain.Calendars.Entities`
- **Tipo**: Entidad de Dominio Sellada (sealed)
- **Herencia**: `AuditableEntity`

## ToDo List

- **No Aplica**

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
  - Verificar el formato y longitud de nombre

- **Solapamiento con Appointments**:

  - El solapamiento con appointments debe ser verificado antes de la creación o modificación de un holiday
  - La decisión de solapamiento debe ser tomada por quien utiliza el sistema a través de una configuración
  - La opción de cómo manejar el solapamiento debe ser configurable en `CalendarSettings`
  - La categoría para manejar es `HolidayConflictStrategy`
  - Las posibles estrategias de manejo del solapamiento son:
    - `RejectIfOverlapping`: Rechaza la creación/modificación del holiday si existen citas solapadas
    - `CancelOverlapping`: Cancela automáticamente las citas que se solapan y crea/modifica el holiday
    - `AllowOverlapping`: Permite la creación/modificación del holiday manteniendo las citas existentes
  - El sistema debe notificar apropiadamente:
    - El número de citas afectadas antes de aplicar la estrategia
    - Las citas que han sido canceladas si se usa `CancelOverlapping`
    - Las citas que quedan solapadas si se usa `AllowOverlapping`

## Propiedades

| Nombre        | Tipo                 | Descripción                                               |
| ------------- | -------------------- | --------------------------------------------------------- |
| `Id`          | `CalendarHolidayId`  | Identificador único del holiday                           |
| `CalendarId`  | `CalendarId`         | Identificador del calendario asociado                     |
| `Calendar`    | `Calendar`           | Referencia al calendario asociado                         |
| `SettingsId`  | `CalendarSettingsId` | Identificador de la configuración del calendario asociado |
| `Settings`    | `CalendarSettings`   | Configuración del calendario asociado                     |
| `Period`      | `Period`             | Periodo del holiday                                       |
| `Name`        | `string`             | Nombre del holiday                                        |

## Invariantes

- `Id` no puede ser `null` en ningún momento
- `Name` no puede ser `null` ni vacío en ningún momento y debe tener una longitud máxima de 50 caracteres

## Reglas de negocio

- **Unicidad de Identificador**:

  - `Id` no puede ser `null` en ningún momento y debe ser único en toda la aplicación.

- **Nombre**:

  - El nombre del holiday debe ser único en toda la aplicación y debe tener entre 1 y 50 caracteres.

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
  - La modificación de un holiday existente debe revaluar los solapamientos si se modifica el período o los días de la semana

## Métodos

### Create

```csharp
public static CalendarHoliday Create(
    CalendarHolidayId calendarHolidayId,
    CalendarId calendarId,
    Period period,
    string name)
```

- **Descripción**: Crea una nueva instancia de `CalendarHoliday` con los valores proporcionados
- **Parámetros**:
  - `id`: Identificador único del holiday
  - `calendarId`: Identificador del calendario asociado
  - `name`: Nombre del holiday
  - `period`: Período del holiday
  - `name`: Nombre del holiday
- **Eventos**:
  - `CalendarHolidayCreatedDomainEvent(calendarHolidayId)`:
  - **Descripción**: Evento de creación del holiday
  - **Parámetros**:
    - `calendarHolidayId`: Identificador del holiday
- **Retorno**: `CalendarHoliday` con los valores proporcionados.

### Update

```csharp
public void Update(Period period, string name)
```

- **Descripción**: Actualiza los valores del holiday
- **Parámetros**:
  - `period`: Período del holiday
  - `name`: Nombre del holiday
- **Eventos**:
  - `CalendarHolidayUpdatedDomainEvent`:
  - **Descripción**: Evento de actualización del holiday
  - **Parámetros**:
    - `calendarHolidayId`: Identificador del holiday

### GuardAgainstInvalidName

```csharp
private static void GuardAgainstInvalidName(string name)
```

- **Descripción**: Valida que el nombre del holiday no sea nulo o vacío
- **Parámetros**:
  - `name`: Nombre del holiday
- **Excepciones**: `CalendarHolidayDomainException`: Cuando el nombre es nulo, vacío o o excede los 50 caracteres

## Estado y Transiciones

- **No Aplica**

## Dependencias

### Directas

- **Entidades Base**:
  - `AuditableEntity`: Base class que proporciona capacidades de auditoría
    - Hereda gestión de eventos de dominio (`Entity`)

### Entidades

- `Calendar`: Referencia al calendario asociado

### Servicios

- `ICalendarRepository`: Servicio para gestionar calendarios

### Managers

- `CalendarManager`: Gestiona la lógica de negocio relacionada con los calendarios

### Políticas

### Value Objects

- `CalendarHolidayId`: Identificador único del holiday
- `CalendarId`: Identificador único del calendario
- `Period`: Representa un periodo de tiempo

### Errores

### Conflict

- **Identifier**: `CreateOverlappingReject` Se lanza cuando se intenta crear un holiday que se solapa con citas existentes
  - **Code**: `CalendarHolidayErrors.CreateOverlappingReject`
  - **Description**: Cannot create a holiday that overlaps with existing appointments

- **Identifier**: `HolidaysOverlap` Se lanza cuando un holiday se solapa con otro holiday existente.
  - **Code**: `CalendarHolidayErrors.HolidaysOverlap`
  - **Description**: Cannot create a holiday that overlaps with existing holidays.

### Validation

- **Identifier**: `NameAlreadyExists` Se lanza cuando el nombre del holiday ya existe en el calendario
  - **Code**: `Name`
  - **Description**: A holiday with the same name already exists in the calendar.

### NotFound

- **Identifier**: `CalendarHolidayNotFound` Se lanza cuando no se encuentra un holiday con el identificador proporcionado
  - **Code**: `CalendarHolidayErrors.CalendarHolidayNotFound`
  - **Description**: The requested holiday was not found in the calendar.

## Comentarios adicionales

Para crear o actualizar un nuevo `CalendarHoliday`, require de `CalendarManager` para la creación de un nuevo `CalendarHoliday`.

## Ejemplos de Uso

```csharp
// Crear un nuevo CalendarHoliday
var calendarHoliday = CalendarHoliday.Create(
    calendarHolidayId: CalendarHolidayId.Create(),
    calendarId: CalendarId.From(Guid.Parse("00000000-0000-0000-0000-000000000000")),
    period: Period.From(DateTimeOffset.Now, DateTimeOffset.Now.AddDays(1)),
    name: "New name");
```
