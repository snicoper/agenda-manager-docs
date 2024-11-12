# Clase: Calendar

**AggregateRoot**: `Calendar`

## Descripción

Representa un calendario que actúa como contenedor lógico para appointments y resources, etc.

Este agregado no mantiene referencias directas a sus entidades relacionadas para mantener la consistencia y simplicidad del modelo.

Las excepciones son la relación de `Calendar` con `CalendarSettings` y `CalendarHoliday`, que es una relación de uno a muchos y se utiliza para representar los días festivos asociados al calendario.

### Responsabilidades

- **Creación de un nuevo calendario**:
  - La creación de un nuevo calendario implica la creación de un nuevo `CalendarSettings` asociado.
  - El `CalendarSettings` se ha de pasar como argumento en el constructor del `Calendar`.

- **Eventos de Dominio**:
  - Disparar eventos de dominio cuando se realicen cambios significativos en el calendario.

- **Validación**:
  - Asegurarse de que el `Name` y `Description` cumplen con las restricciones de longitud.

- **Gestión de Estado**:
  - Gestionar el estado activo o inactivo del calendario (`IsActive`).

- **Gestión de Holidays**:
  - Mantener la colección de días festivos
  - Asegurar la consistencia al añadir o eliminar holidays
  - Emitir eventos de dominio relacionados con holidays

## Invariantes

- `Id` no puede ser `null` en ningún momento.
- `SettingsId` no puede ser `null` en ningún momento.
- `Name` no puede ser nulo y debe tener entre 1 y 50 caracteres.
- `Description` no puede ser nulo y debe tener entre 1 y 500 caracteres.

## Reglas de Negocio

- **Unicidad de Identificador**:
  - `Id` debe ser único en toda la aplicación.
  - `SettingsId` debe ser único en toda la aplicación.

- **Gestión de Holidays**:
  - Un holiday debe pertenecer a un único calendario
  - No puede haber holidays duplicados (misma fecha) en un calendario
  - Los holidays se eliminan en cascada al eliminar el calendario

- **Estado del CalendarioSettings**:
  - Un calendar siempre debe tener un `CalendarSettings` asociado

- **Nombre y Descripción**:
  - El nombre del calendario debe ser único en toda la aplicación y debe tener entre 1 y 50 caracteres.
  - La descripción del calendario debe tener entre 1 y 500 caracteres.

- **Eliminación de Calendario**:
  - No se puede eliminar un `Calendar` si tiene alguna cita (`Appointment`) asociada.
  - No se puede eliminar un `Calendar` si tiene algún día festivo (`CalendarHoliday`) asociado.
  - No se puede eliminar un `Calendar` si tiene algún recurso (`Resource`) asignado.
  - No se puede eliminar un `Calendar` si tiene algún servicio (`Service`) asignado.
  - No se puede eliminar un `Calendar` si tiene algún horario de recurso (`ResourceSchedule`) asignado.

## Propiedades

| Propiedad     | Tipo                    | Descripción                                             |
|---------------|-------------------------|---------------------------------------------------------|
| `Id`          | `CalendarId`            | Identificador único del calendario.                     |
| `SettingsId`  | `CalendarSettingsId`    | Identificador único del `CalendarSettings` asociado.    |
| `IsActive`    | `bool`                  | Indica si el calendario está activo.                    |
| `Name`        | `string`                | Nombre del calendario.                                  |
| `Description` | `string`                | Descripción del calendario.                             |
| `Holidays`    | `List<CalendarHoliday>` | Lista de vacaciones asociadas al calendario.            |

## Métodos

```csharp
public void ChangeActiveStatus(bool isActive)
```

- Cambia el estado activo del calendario.
- `isActive`: Indica si el calendario está activo.
- Lanza el evento `CalendarUpdatedDomainEvent`.

```csharp
public void AddHoliday(CalendarHoliday calendarHoliday)
```

- Agrega una nueva vacación al calendario.
- `calendarHoliday`: La vacación a agregar.
- Lanza el evento `CalendarUpdatedDomainEvent`.

```csharp
public void RemoveHoliday(CalendarHoliday calendarHoliday)
```

- Elimina una vacación del calendario.
- `calendarHoliday`: La vacación a eliminar.
- Lanza el evento `CalendarUpdatedDomainEvent`.

```csharp
internal static Calendar Create(
    CalendarId id,
    CalendarSettings settings,
    string name,
    string description,
    bool active = true)
```

- Crea un nuevo calendario con los valores proporcionados.
- `id`: Identificador único del calendario.
- `settings`: Configuración del calendario.
- `name`: Nombre del calendario.
- `description`: Descripción del calendario.
- `active`: Indica si el calendario está activo.
- Lanza el evento `CalendarCreatedDomainEvent`.
- Devuelve el calendario creado.

```csharp
internal void Update(string name, string description)
```

- Actualiza el nombre y la descripción del calendario.
- `name`: Nombre del calendario.
- `description`: Descripción del calendario.
- Lanza el evento `CalendarUpdatedDomainEvent`.

```csharp
private static void GuardAgainstInvalidName(string name)
```

- Valida que el nombre del calendario no sea nulo y no exceda los 50 caracteres.
- `name`: Nombre del calendario.
- Lanza la excepción `CalendarDomainException` si el nombre del calendario es nulo o excede los 50 caracteres.

```csharp
private static void GuardAgainstInvalidDescription(string description)
```

- Valida que la descripción del calendario no sea nula y no exceda los 500 caracteres.
- `description`: Descripción del calendario.
- Lanza la excepción `CalendarDomainException` si la descripción del calendario es nula o excede los 500 caracteres.

```csharp
public void AddHoliday(CalendarHoliday calendarHoliday)
```

- Agrega un día festivo al calendario.
- `calendarHoliday`: Día festivo a agregar.
- Lanza el evento `CalendarHolidayAddedDomainEvent`.

```csharp
public void RemoveHoliday(CalendarHoliday calendarHoliday)
```

- Elimina un día festivo del calendario.
- `calendarHoliday`: Día festivo a eliminar.
- Lanza el evento `CalendarHolidayRemovedDomainEvent`.

## Estado y Transiciones

La clase `Calendar` tiene diferentes estados que dependen de ciertas condiciones y acciones realizadas dentro del sistema.

- **IsActive**:
  - **`true`**: El calendario está activo se pueden mostrar y crear citas dentro de él.
  - **`false`**: El calendario está inactivo y no se pueden mostrar ni crear citas dentro de él.

- **Transiciones Permitidas**:
  - **Transiciones de `IsActive`**:
  - De **`true` a `false`**: Puede ocurrir en cualquier momento cuando el personal autorizado decide desactivar al calendario.
    - **Condición**: Solo personal autorizado puede cambiar el estado a inactivo.
  - De **`false` a `true`**: Puede ocurrir en cualquier momento cuando el personal autorizado decide reactivar al calendario.
  - **Condición**: Solo personal autorizado puede cambiar el estado a activo.

## Dependencias

- **Entities**:
  - [CalendarHoliday](./calendar-holiday.md): La clase `Calendar` tiene una lista de la clase `CalendarHoliday`.
  - [CalendarSettings](./entities/calendar-settings.md): La clase `Calendar` tiene una relación con `CalendarSettings` a través de `CalendarId`.
  - [Appointment](../appoitments/appointment.md): La clase `Appointment` tiene una relación con `Calendar` a través de `CalendarId`.
  - [Resource](../resources/resource.md): La clase `Resource` tiene una relación con `Calendar` a través de `CalendarId`.
  - [Service](../services/service.md): La clase `Service` tiene una relación con `Calendar` a través de `CalendarId`.
  - [ResourceSchedule](../resources/resource-schedule.md): La clase `ResourceSchedule` tiene una relación con `Calendar` a través de `CalendarId`.
  - [CalendarHoliday](./calendar-holiday.md): La clase `Calendar` tiene una lista de la clase `CalendarHoliday`.
- **Services**:
  - [ICalendarRepository](./interfaces/i-calendar-repository.md): Repositorio para interactuar con la base de datos.
- **Managers**:
  - [CalendarManager](./managers/calendar-manager.md): Servicio para gestionar calendarios.
- **Policies**:
- **Value Objects**:
  - [CalendarId](./value-objects/calendar-id.md): Identificador único del calendario.
  - [CalendarSettingsId](./value-objects/calendar-settings-id.md): Identificador único de la configuración del calendario.

## Eventos de Dominio

- `CalendarCreatedDomainEvent(id)`: Evento de dominio que se lanza cuando se crea un nuevo calendario.
- `CalendarUpdatedDomainEvent(Id)`: Evento de dominio que se lanza cuando se actualiza un calendario.
- `CalendarActiveStatusChangedDomainEvent(Id, IsActive)`: Evento de dominio que se lanza cuando se cambia el estado activo de un calendario.
- `CalendarHolidayAddedDomainEvent(Id, calendarHoliday.Id)`: Evento de dominio que se lanza cuando se agrega un día festivo a un calendario.
- `CalendarHolidayRemovedDomainEvent(Id, calendarHoliday.Id)`: Evento de dominio que se lanza cuando se elimina un día festivo de un calendario.

## Interceptores EF Core

## Comentarios adicionales

- Los métodos `Create` y `Update` son `internal` para forzar el uso del `CalendarManager`
- Este diseño asegura que las validaciones de negocio y base de datos se realicen consistentemente
- La gestión de holidays se hace a través de la propia entidad para mantener la consistencia del agregado

## Ejemplos

```csharp
var settings = CalendarSettings.Create(/** **/);

// Crear un nuevo calendario a partir de un identificador único.
var calendar = Calendar.Create(
    CalendarId.From(Guid.Parse("7f2c1a3e-9b4d-4c8f-a45d-e8f1d2c3b4a5")),
    settings,
    "Mi Calendario",
    "Este es mi calendario");

// Crear un nuevo calendario con un identificador único aleatorio.
var calendar = Calendar.Create(
    CalendarId.Create(),
    settings,
    "Mi Calendario",
    "Este es mi calendario");

// Añadir un holiday
var holiday = CalendarHoliday.Create(
    CalendarHolidayId.Create(),
    calendar.Id,
    "Navidad",
    new DateTime(2024, 12, 25));

calendar.AddHoliday(holiday);

// Remover un holiday
calendar.RemoveHoliday(holiday);
```
