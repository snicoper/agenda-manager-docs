He creado la documentación para `Calendar`, si por favor me la revisas y si ves algo que falte, te lo agradezco.

---

# Clase: Calendar

## Descripción

Representa un calendario que actúa como contenedor lógico para appointments y resources, etc.

Este agregado no mantiene referencias directas a sus entidades relacionadas para mantener la consistencia y simplicidad del modelo.

La única excepción es la relación de `Calendar` con `CalendarHoliday`, que es una relación de uno a muchos y se utiliza para representar los días festivos asociados al calendario.

### Responsabilidades

- **Eventos de Dominio**:
  - Disparar eventos de dominio cuando se realicen cambios significativos en el calendario.

- **Validación**:
  - Asegurarse de que el `Name` y `Description` cumplen con las restricciones de longitud.

- **Gestión de Estado**:
  - Gestionar el estado activo o inactivo del calendario (`IsActive`).

## Propiedades

| Propiedad     | Tipo         | Descripción                                             |
|---------------|--------------|---------------------------------------------------------|
| `Id`          | `CalendarId` | Identificador único del calendario.                     |
| `IsActive`    | `bool`       | Indica si el calendario está activo.                    |
| `Name`        | `string`     | Nombre del calendario.                                  |
| `Description` | `string`     | Descripción del calendario.                             |
| `Holidays`    | `List<CalendarHoliday>` | Lista de vacaciones asociadas al calendario. |

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
internal static Calendar Create(CalendarId id, string name, string description, bool active = true)
```

- Crea un nuevo calendario con los valores proporcionados.
- `id`: Identificador único del calendario.
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

## Invariantes

- `Id` no puede ser `null` en ningún momento.
- `Name` no puede ser nulo y debe tener entre 1 y 50 caracteres.
- `Description` no puede ser nulo y debe tener entre 1 y 500 caracteres.

## Reglas de Negocio

- **Unicidad de Identificador**:
  - `Id` debe ser único en toda la aplicación.

- **Longitud de Nombre y Descripción**:
  - El nombre del calendario debe ser único en toda la aplicación y debe tener entre 1 y 50 caracteres.
  - La descripción del calendario debe tener entre 1 y 500 caracteres.

- **Eliminación de Calendario**:
  - No se puede eliminar un `Calendar` si tiene alguna cita (`Appointment`) asociada.
  - No se puede eliminar un `Calendar` si tiene algún día festivo (`CalendarHoliday`) asociado.
  - No se puede eliminar un `Calendar` si tiene algún recurso (`Resource`) asignado.
  - No se puede eliminar un `Calendar` si tiene algún servicio (`Service`) asignado.
  - No se puede eliminar un `Calendar` si tiene algún horario de recurso (`ResourceSchedule`) asignado.

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

- **Entidades**:
  - [Appointment](../appoitments/appointment.md): La clase `Appointment` tiene una relación con `Calendar` a través de `CalendarId`.
  - [CalendarHoliday](./calendar-holiday.md): La clase `Calendar` tiene una lista de la clase `CalendarHoliday`.
  - [Resource](../resources/resource.md): La clase `Resource` tiene una relación con `Calendar` a través de `CalendarId`.
  - [Service](../services/service.md): La clase `Service` tiene una relación con `Calendar` a través de `CalendarId`.
  - [ResourceSchedule](../resources/resource-schedule.md): La clase `ResourceSchedule` tiene una relación con `Calendar` a través de `CalendarId`.
- **Services**:
  - [ICalendarRepository](./interfaces/i-calendar-repository.md): Repositorio para interactuar con la base de datos.
- **Managers**:
  - [CalendarManager](./managers/calendar-manager.md): Servicio para gestionar calendarios.
- **Policies**:
- **Value Objects**:
  - [CalendarId](./value-objects/calendar-id.md): Identificador único del calendario.

## Eventos de Dominio

- `CalendarCreatedDomainEvent(id)`: Evento de dominio que se lanza cuando se crea un nuevo calendario.
- `CalendarUpdatedDomainEvent(Id)`: Evento de dominio que se lanza cuando se actualiza un calendario.
- `CalendarActiveStatusChangedDomainEvent(Id, IsActive)`: Evento de dominio que se lanza cuando se cambia el estado activo de un calendario.
- `CalendarHolidayAddedDomainEvent(Id, calendarHoliday.Id)`: Evento de dominio que se lanza cuando se agrega un día festivo a un calendario.
- `CalendarHolidayRemovedDomainEvent(Id, calendarHoliday.Id)`: Evento de dominio que se lanza cuando se elimina un día festivo de un calendario.

## Interceptores EF Core

## Comentarios adicionales

La clase `Calendar` no se puede instanciar directamente fuera de su contexto de dominio. Para crear un nuevo calendario, se debe utilizar el método `CreateCalendarAsync` del servicio `CalendarManager`.

## Ejemplos

La entidad `Calendar` no puede ser instanciada fuera de su contexto de dominio. Para crear un nuevo calendario, se debe utilizar el método `CreateCalendarAsync` del servicio `CalendarManager`.

La entidad `Calendar` no puede ser actualizada directamente. Para actualizar un calendario, se debe utilizar el método `UpdateCalendarAsync` del servicio `CalendarManager`.

```csharp
// Crear un nuevo calendario a partir de un identificador único.
var calendar = Calendar.Create(
    CalendarId.From(Guid.Parse("7f2c1a3e-9b4d-4c8f-a45d-e8f1d2c3b4a5")),
    "Mi Calendario",
    "Este es mi calendario");

// Crear un nuevo calendario con un identificador único aleatorio.
var calendar = Calendar.Create(
    CalendarId.Create(),
    "Mi Calendario",
    "Este es mi calendario");

---
