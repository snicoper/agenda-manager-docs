# Clase: Calendar

- **Aggregate Root**: `Calendar`
- **Namespace**: `AgendaManager.Domain.Calendars`
- **Tipo**: Entidad de Dominio Sellada (sealed)
- **Herencia**: `AggregateRoot`

## Descripción

Representa un calendario que actúa como contenedor lógico para appointments, resources, etc.

Este agregado no mantiene referencias directas a sus entidades relacionadas para mantener la consistencia y simplicidad del modelo.

Las excepciones son la relación de uno a muchos de `Calendar` -> `CalendarConfiguration` y `Calendar` -> `CalendarHoliday`, que es una relación de uno a muchos y se utiliza para representar los días festivos asociados al calendario.

### Responsabilidades

- **Creación de un nuevo calendario**:
  - La creación de un nuevo calendario implica la creación de un nuevo `CalendarConfiguration` asociado.
  - El `CalendarConfiguration` se crea con la configuración predeterminada.
    - El valor por defecto de `HolidayCreateStrategy` cuando se crea un nuevo calendario debe ser `RejectIfOverlapping`.
    - El valor por defecto de `AppointmentCreationStrategy` cuando se crea un nuevo calendario debe ser `Direct`.
    - El valor por defecto de `AppointmentOverlappingStrategy` cuando se crea un nuevo calendario debe ser `RejectIfOverlapping`.
    - El valor por defecto de `IanaTimeZone` debe ser proporcionado por el usuario.

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

- **Gestión de Configurations**:
  - Gestionar la asociación con `CalendarConfiguration`
  - Cada `CalendarConfiguration` representa una configuración específica del calendario.
  - Emitir eventos de dominio relacionados con `CalendarConfiguration` cuando se realicen cambios.

## Invariantes

- `Id` no puede ser `null` en ningún momento.
- `Name` no puede ser nulo y debe tener entre 1 y 50 caracteres.
- `Description` no puede ser nulo y debe tener entre 1 y 500 caracteres.
- `IsActive` debe ser `true` o `false`.

## Reglas de Negocio

- **Unicidad de Identificador**:
  - `Id` debe ser único en toda la aplicación.
  - `Name` debe ser único en toda la aplicación.

- **Gestión de Holidays**:
  - Un holiday debe pertenecer a un único calendario
  - No puede haber holidays duplicados (misma fecha) en un calendario *
  - Los holidays se eliminan en cascada al eliminar el calendario

- **Estado del Configurations**:
  - Un calendar siempre debe tener una lista de `CalendarConfiguration` asociados

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

| Propiedad        | Tipo                         |Descripción                                            |
|------------------|------------------------------|-------------------------------------------------------|
| `Id`             | `CalendarId`                 | Identificador único del calendario.                   |
| `Name`           | `string`                     | Nombre del calendario.                                |
| `Description`    | `string`                     | Descripción del calendario.                           |
| `IsActive`       | `bool`                       | Indica si el calendario está activo.                  |
| `Holidays`       | `List<CalendarHoliday>`      | Lista de vacaciones asociadas al calendario.          |
| `Configurations` | `List<CalendarConfiguration>`| Lista de configuraciones asociadas al calendario.     |

## Métodos

### Activate

```csharp
public void Activate()
```

- **Descripción**: Activa el calendario.
- **Eventos**: Lanza el evento `CalendarUpdatedDomainEvent`.

### Deactivate

```csharp
public void Deactivate()
```

- **Descripción**: Desactiva el calendario.
- **Eventos**: Lanza el evento `CalendarUpdatedDomainEvent`.

### AddConfiguration

```csharp
public void AddConfiguration(CalendarConfiguration configuration)
```

- **Descripción**: Agrega una nueva configuración al calendario.
- **Parámetros**:
  - `configuration`: La configuración a agregar.
- **Eventos**: Lanza el evento `CalendarConfigurationAddedDomainEvent`.

### RemoveConfiguration

```csharp
public void RemoveConfiguration(CalendarConfiguration configuration)
```

- **Descripción**: Elimina una configuración del calendario.
- **Parámetros**:
  - `configuration`: La configuración a eliminar.
- **Eventos**: Lanza el evento `CalendarConfigurationRemovedDomainEvent`.

### UpdateConfiguration

```csharp
public void UpdateConfiguration(CalendarConfigurationId configurationId, string category, string selectedKey)
```

- **Descripción**: Actualiza una configuración del calendario.
- **Parámetros**:
  - `configurationId`: El ID de la configuración a actualizar.
  - `category`: La categoría de la configuración.
  - `selectedKey`: La clave seleccionada de la configuración.
**Eventos**: Lanza el evento `CalendarConfigurationUpdatedDomainEvent`.

### AddHoliday

```csharp
public void AddHoliday(CalendarHoliday calendarHoliday)
```

- **Descripción**: Agrega una nueva vacación al calendario.
- **Parámetros**:
  - `calendarHoliday`: La vacación a agregar.
- **Eventos**: Lanza el evento `CalendarHolidayAddedDomainEvent`.

### RemoveHoliday

```csharp
public void RemoveHoliday(CalendarHoliday calendarHoliday)
```

- **Descripción**: Elimina una vacación del calendario.
- **Parámetros**:
  - `calendarHoliday`: La vacación a eliminar.
- **Eventos**: Lanza el evento `CalendarHolidayRemovedDomainEvent`.

### Create

```csharp
internal static Calendar Create(
    CalendarId id,
    CalendarSettings settings,
    string name,
    string description,
    bool active = true)
```

- **Descripción**: Crea un nuevo calendario con los valores proporcionados.
- **Parámetros**:
  - `id`: Identificador único del calendario.
  - `settings`: Configuración del calendario.
  - `name`: Nombre del calendario.
  - `description`: Descripción del calendario.
  - `active`: Indica si el calendario está activo.
- **Eventos**: Lanza el evento `CalendarCreatedDomainEvent`.
- **Retorno**: El calendario creado.

### Update

```csharp
internal void Update(string name, string description)
```

- **Descripción**: Actualiza el nombre y la descripción del calendario.
- **Parámetros**:
  - `name`: Nombre del calendario.
  - `description`: Descripción del calendario.
- **Eventos**: Lanza el evento `CalendarUpdatedDomainEvent`.

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
- `CalendarConfiguration`: La clase `Calendar` tiene una lista de la clase `CalendarConfiguration`.

### Servicios

- `ICalendarRepository`: Interfaz que define los métodos para interactuar con el repositorio de calendarios.

### Managers

- `CalendarManager`: Clase que proporciona métodos para interactuar con el calendario.

### Policies

### Value Objects

- `CalendarId`: Identificador único del calendario.

## Comentarios adicionales

- Los métodos `Create` y `Update` son `internal` para forzar el uso del `CalendarManager`
- Este diseño asegura que las validaciones de negocio y base de datos se realicen consistentemente
- La gestión de holidays se hace a través de la propia entidad para mantener la consistencia del agregado
