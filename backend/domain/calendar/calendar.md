# Clase: Calendar

- **Aggregate Root**: `Calendar`
- **Namespace**: `AgendaManager.Domain.Calendars`
- **Tipo**: Entidad de Dominio Sellada (sealed)
- **Herencia**: `AggregateRoot`

## Descripción

Representa un calendario que actúa como contenedor lógico para appointments, resources y services.

Este agregado no mantiene referencias directas a sus entidades relacionadas para mantener la consistencia y simplicidad del modelo.

El calendario gestiona distintas configuraciones que afectan al comportamiento de diferentes situaciones:

### Responsabilidades

- **Creación de un nuevo calendario**:

  - La creación de un nuevo calendario implica la creación de un nuevo `CalendarSettings` asociado.
  - El `CalendarSettings` se crea con la configuración predeterminada.
    - Ver [CalendarSettings](./entities/calendar-configuration.md#Configuraciones-con-Opciones-Predefinidas) para más detalles.
  - En la creación de un calendario, se establece el `IanaTimeZone`, ver `CalendarManager`.

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

- **Gestión de settings**:

  - Gestionar la asociación con `CalendarSettings`
  - Emitir eventos de dominio relacionados con la configuración del calendario.

## Propiedades

| Propiedad     | Tipo                             | Descripción                                  |
| ------------- | -------------------------------- | -------------------------------------------- |
| `Id`          | `CalendarId`                     | Identificador único del calendario.          |
| `SettingsId`  | `CalendarSettingsId`             | Identificador único de la configuración.     |
| `settings`    | `CalendarSettings`               | Configuraciones del calendario.              |
| `Name`        | `string`                         | Nombre del calendario.                       |
| `Description` | `string`                         | Descripción del calendario.                  |
| `IsActive`    | `bool`                           | Indica si el calendario está activo.         |
| `Holidays`    | `IReadOnlyList<CalendarHoliday>` | Lista de vacaciones asociadas al calendario. |

## Invariantes

- `Id` no puede ser `null` en ningún momento.
- `Name` no puede ser nulo y debe tener entre 1 y 50 caracteres.
- `Description` no puede ser nulo y debe tener entre 1 y 500 caracteres.
- `IsActive` debe ser `true` o `false`.
- `Holidays` no puede ser nulo.
- `CalendarSettings` no puede ser nulo y debe ser un `CalendarSettings` asociado **one-to-one**.

## Reglas de Negocio

- **Unicidad de Identificador**:

  - `Id` debe ser único en toda la aplicación.
  - `Name` debe ser único en toda la aplicación.

- **Gestión de Holidays**:

  - Un holiday debe pertenecer a un único calendario
  - No puede haber holidays duplicados (misma fecha) en un calendario
  - Los holidays se eliminan en cascada al eliminar el calendario

- **Estado del CalendarSettings**:

  - Un calendar siempre debe tener un `CalendarSettings` asociados con las configuraciones predeterminadas.
  - El campo `IanaTimeZone` del `CalendarSettings` es el único que puede ser variable en la creación del calendario.

- **Nombre y Descripción**:

  - El nombre del calendario debe ser único en toda la aplicación y debe tener entre 1 y 50 caracteres.
  - La descripción del calendario debe tener entre 1 y 500 caracteres.

- **Eliminación de Calendario**:

  - No se puede eliminar un calendario si tiene alguna cita (`Appointment`) asociada.
  - No se puede eliminar un calendario si tiene algún recurso (`Resource`) asignado.
  - No se puede eliminar un calendario si tiene algún servicio (`Service`) asignado.
  - Deberá eliminar los días festivos asociados al calendario al eliminar el calendario.
  - Deberá eliminar el settings asociada al calendario al eliminar el calendario.

## Métodos

### Constructor

```csharp
private Calendar(CalendarId id, CalendarSettings settings, string name, string description, bool isActive)
```

- **Descripción**: Constructor privado para crear una nueva instancia de `Calendar`.
- **Parámetros**:
  - `id`: Identificador único del calendario.
  - `settings`: Configuraciones del calendario.
  - `name`: Nombre del calendario.
  - `description`: Descripción del calendario.
  - `isActive`: Indica si el calendario está activo.

### Activate

```csharp
public void Activate()
```

- **Descripción**: Activa el calendario.
- **Eventos**:
  - `CalendarActivatedDomainEvent(Id)`
  - **Descripción**: Lanza el eventos al activar el calendario.
  - **Parámetros**:
    - `Id`: El identificador del calendario.

### Deactivate

```csharp
public void Deactivate()
```

- **Descripción**: Desactiva el calendario.
- **Eventos**:
  - `CalendarDeactivatedDomainEvent(Id)`
  - **Descripción**: Lanza el eventos al desactivar el calendario.
  - **Parámetros**:
    - `Id`: El identificador del calendario.

### AddHoliday

```csharp
public void AddHoliday(CalendarHoliday holiday)
```

- **Descripción**: Agrega una nueva vacación al calendario.
- **Parámetros**:
  - `holiday`: La vacación a agregar.
- **Eventos**:
  - `CalendarHolidayAddedDomainEvent(Id, holiday.Id)`
  - **Descripción**: Lanza el evento al agregar una nueva vacación al calendario.
  - **Parámetros**:
    - `Id`: El identificador del calendario.
    - `holiday.Id`: El identificador de la vacación agregada.

### RemoveHoliday

```csharp
public void RemoveHoliday(CalendarHoliday holiday)
```

- **Descripción**: Elimina una vacación del calendario.
- **Parámetros**:
  - `holiday`: La vacación a eliminar.
- **Eventos**:
  - `CalendarHolidayRemovedDomainEvent(Id, holiday.Id)`
  - **Descripción**: Lanza el evento al eliminar una vacación del calendario.
  - **Parámetros**:
    - `Id`: El identificador del calendario.
    - `holiday.Id`: El identificador de la vacación eliminada.

### UpdateSettings

```csharp
public bool UpdateSettings(CalendarSettings settings)
```

- **Descripción**: Actualiza la configuración del calendario.
- **Parámetros**:
  - `settings`: Configuraciones del calendario.
- **Eventos**:
  - `CalendarSettingsUpdatedDomainEvent(Id)`
  - **Descripción**: Lanza el evento al actualizar la configuración del calendario.
  - **Parámetros**:
    - `Id`: Identificador único del calendario.
- **Retorno**: `true` si la configuración se actualizó, `false` en caso contrario.

### Create

```csharp
internal static Calendar Create(CalendarId id, string name, string description, bool active = true)
```

- **Descripción**: Crea un nuevo calendario con los valores proporcionados.
- **Parámetros**:
  - `id`: Identificador único del calendario.
  - `name`: Nombre del calendario.
  - `description`: Descripción del calendario.
  - `active`: Indica si el calendario está activo.
- **Eventos**:
  - `CalendarCreatedDomainEvent(Id)`
  - **Descripción**: Lanza el evento al crear un nuevo calendario.
  - **Parámetros**:
    - `Id`: Identificador único del calendario.
- **Retorno**: El calendario creado.

### Update

```csharp
internal void Update(string name, string description)
```

- **Descripción**: Actualiza el nombre y la descripción del calendario.
- **Parámetros**:
  - `name`: Nombre del calendario.
  - `description`: Descripción del calendario.
- **Eventos**:
  - `CalendarUpdatedDomainEvent(Id)`
  - **Descripción**: Lanza el evento al actualizar el calendario.
  - **Parámetros**:
    - `Id`: Identificador único del calendario.

### GuardAgainstInvalidName

```csharp
private static void GuardAgainstInvalidName(string name)
```

- **Descripción**: Valida que el nombre del calendario no sea nulo y no exceda los 50 caracteres.
- **Parámetros**:
  - `name`: Nombre del calendario.
- **Excepciones**: Lanza la excepción `CalendarDomainException` si el nombre del calendario es nulo o excede los 50 caracteres.

### GuardAgainstInvalidDescription

```csharp
private static void GuardAgainstInvalidDescription(string description)
```

- **Descripción**: Valida que la descripción del calendario no sea nula y no exceda los 500 caracteres.
- **Parámetros**:
  - `description`: Descripción del calendario.
- **Excepciones**: Lanza la excepción `CalendarDomainException` si la descripción del calendario es nula o excede los 500 caracteres.

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

### Directas

- **Entidades Base**:
  - `AggregateRoot`: Base class que designa esta entidad como raíz de agregado, proporcionando control transaccional y consistencia del agregado
    - Hereda capacidades de auditoría (`AuditableEntity`)
    - Hereda gestión de eventos de dominio (`Entity`)

### Entidades

- `CalendarHoliday`: La clase `Calendar` tiene una lista de la clase `CalendarHoliday`.
- `CalendarSettings`: La clase `Calendar` tiene una referencia a la clase `CalendarSettings`.

### Servicios

- `ICalendarRepository`: Interfaz que define los métodos para interactuar con el repositorio de calendarios.

### Managers

- `CalendarManager`: Clase que proporciona métodos para interactuar con el calendario.

### Policies

- `IHasAppointmentsInCalendarPolicy`: Interfaz que define la política para verificar si un calendario tiene citas.
- `IHasResourcesInCalendarPolicy`: Interfaz que define la política para verificar si un calendario tiene recursos.
- `IHasServicesInCalendarPolicy`: Interfaz que define la política para verificar si un calendario tiene servicios.

### Value Objects

- `CalendarId`: Identificador único del calendario.
- `CalendarSettingsId`: Identificador único de la configuración del calendario.

## Errores

### NotFound

- **Identifier**: `CalendarNotFound` Se lanza cuando se intenta acceder a un calendario que no existe.
  - **Code**: `CalendarErrors.CalendarNotFound`
  - **Description**: The calendar was not found.

### Validation

- **Identifier**: `NameAlreadyExists` Se lanza cuando un nombre de calendario ya existe.
  - **Code**: `Name`
  - **Description**: The calendar name already exists.

### Conflict

- **Identifier**: `CannotDeleteCalendarWithAppointments` Se lanza cuando se intenta eliminar un calendario que tiene citas.

  - **Code**: `CalendarErrors.CannotDeleteCalendarWithAppointments`
  - **Description**: The calendar cannot be deleted because it has appointments.

- **Identifier**: `CannotDeleteCalendarWithServices` Se lanza cuando se intenta eliminar un calendario que tiene servicios.

  - **Code**: `CalendarErrors.CannotDeleteCalendarWithServices`
  - **Description**: The calendar cannot be deleted because it has services.

- **Identifier**: `CannotDeleteCalendarWithResources` Se lanza cuando se intenta eliminar un calendario que tiene recursos.
  - **Code**: `CalendarErrors.CannotDeleteCalendarWithResources`
  - **Description**: The calendar cannot be deleted because it has resources.

## Comentarios adicionales

- Los métodos `Create` y `Update` son `internal` para forzar el uso del `CalendarManager`
- Este diseño asegura que las validaciones de negocio y base de datos se realicen consistentemente
- La gestión de holidays se hace a través de la propia entidad para mantener la consistencia del agregado
