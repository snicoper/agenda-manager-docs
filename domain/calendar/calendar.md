# Clase: Calendar

## Descripción

Un `Calendar` es una entidad que agrupa `Appointment`s y otros elementos relacionados dentro de una misma categoría.

### Responsabilidades

- **Gestión de Citas**:
  - Almacenar y gestionar una colección de `Appointment`s, `Resource`s, etc.

- **Eventos de Dominio**:
  - Disparar eventos de dominio cuando se crean, actualizan o eliminan citas (`CalendarCreatedDomainEvent`, `CalendarUpdatedDomainEvent`, `CalendarDeletedDomainEvent`, etc.).

- **Validación**:
  - Asegurarse de que el `Name` y `Description` cumplen con las restricciones de longitud.

## Propiedades

| Propiedad     | Tipo         | Descripción                           |
|---------------|--------------|---------------------------------------|
| `Id`          | `CalendarId` | Identificador único del calendario.   |
| `Name`        | `string`     | Nombre del calendario.                |
| `Description` | `string`     | Descripción del calendario.           |

## Métodos

`Calendar` no dispone de métodos públicos accesibles fuera de su contexto de dominio.

### Constructor (Instancia un nuevo calendario)

Crea un nuevo `Calendar` con los parámetros proporcionados.

- **Parámetros**:
  - **id** `CalendarId`: El identificador del calendario.
  - **name** `string`: El nombre del calendario.
  - **description** `string`: La descripción del calendario.
- **Eventos**:
  - `CalendarCreatedDomainEvent`: Se lanza cuando se crea un nuevo calendario.
- **Excepciones**:
  - `CalendarDomainException` si el nombre o la descripción no cumplen con los requisitos de longitud.

### Update (Actualiza un calendario existente)

Actualiza el nombre y la descripción del calendario.

- **Parámetros**:
  - **name** `string`: El nuevo nombre del calendario.
  - **description** `string`: La nueva descripción del calendario.
- **Eventos**:
  - `CalendarUpdatedDomainEvent`: Se lanza cuando se actualiza un calendario.
- **Excepciones**:
  - `CalendarDomainException` si el nombre o la descripción no cumplen con los requisitos de longitud.

## Invariantes

- `Id` no puede ser nulo.
- `Name` no puede ser nulo y debe tener entre 1 y 50 caracteres.
- `Description` no puede ser nulo y debe tener entre 1 y 500 caracteres.

## Reglas de Negocio

- **Unicidad de Identificador**:
  - El identificador del calendario debe ser único.

- **Longitud de Nombre y Descripción**:
  - El nombre del calendario debe ser único en toda la aplicación.
  - La descripción del calendario debe tener entre 1 y 500 caracteres.

- **Eliminación de Calendario**:
  - No se puede eliminar un `Calendar` si tiene alguna cita (`Appointment`) asociada.
  - No se puede eliminar un `Calendar` si tiene algún día festivo (`CalendarHoliday`) asociado.
  - No se puede eliminar un `Calendar` si tiene algún recurso (`Resource`) asignado.
  - No se puede eliminar un `Calendar` si tiene algún servicio (`Service`) asignado.
  - No se puede eliminar un `Calendar` si tiene algún horario de recurso (`ResourceSchedule`) asignado.

## Estado y Transiciones

### Estados Iniciales

- **Estado Inicial**:
  - Un `Calendar` debe tener un identificador único, un nombre y una descripción.

### Estados Posibles

- **Creador**:
  - Estado en que se encuentra un `Calendar` recién creado.

### Transiciones Permitidas

- **De Creación a Actualización**:
  - Un `Calendar` puede ser actualizado después de su creación.

### Condiciones para Transiciones

- Las transiciones deben cumplir con las reglas de negocio establecidas, especialmente en relación a la validez de `Name` y `Description`.

## Dependencias

- **Entidades**:
  - `Appointment`
  - `CalendarHoliday`
  - `Resource`
  - `Service`
  - `ResourceSchedule`
- **Services**:
  - `ICalendarRepository`: Repositorio para interactuar con la base de datos.
- **Managers**:
  - `CalendarManager`: Servicio para gestionar calendarios.
- **Policies**:
- **Value Objects**:
  - `CalendarId`: Identificador único del calendario.

## Ejemplos

La entidad `Calendar` no puede ser instanciada fuera de su contexto de dominio. Para crear un nuevo calendario, se debe utilizar el método `CreateCalendarAsync` del servicio `CalendarManager`.

```csharp
// Crear un nuevo calendario a partir de un identificador único.
var calendar = new Calendar(
    CalendarId.From(Guid.Parse("7f2c1a3e-9b4d-4c8f-a45d-e8f1d2c3b4a5")),
    "Mi Calendario",
    "Este es mi calendario");

// Crear un nuevo calendario con un identificador único aleatorio.
var calendar = new Calendar(
    CalendarId.Create(),
    "Mi Calendario",
    "Este es mi calendario");
