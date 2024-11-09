# Clase: NombreDeLaEntidad

## Descripción

Un `Calendar` es una agrupación de `Appointment`s que pertenecen a una misma categoría.

### Responsabilidades

- **Gestión de Citas**:
  - Almacenar y gestionar una colección de `Appointment`s, `Resources`s, etc.

- **Eventos de Dominio**:
  - Disparar eventos de dominio cuando se crean, actualizan o eliminan citas (`CalendarCreatedDomainEvent`, `CalendarUpdatedDomainEvent`, `CalendarDeletedDomainEvent`, etc.).

- **Validación**:
  - Asegurarse de que el `Name` y `Description` no exceda la longitud permitida.

## Propiedades

| Propiedad           | Tipo                      | Descripción                                                |
|---------------------|---------------------------|------------------------------------------------------------|
| `Id`                | `CalendarId`              | Identificador único del calendario.                        |
| `Name`              | `string`                  | Nombre del calendario.                                     |
| `Description`       | `string`                  | Descripción del calendario.                                |

## Asociaciones

- `Calendar` no dispone de asociaciones directas.

## Métodos

`Calendar` no dispone de métodos públicos.

## Invariantes

- `Id` no pueden ser nulo.
- `Name` no puede ser nulo y debe cumplir una longitud de entre 1 y 50 caracteres.
- `Description` no puede ser nulo y debe tener entre 1 y 500 caracteres.

## Reglas de negocio

- **Unicidad de Identificador**:
  - El identificador del calendario debe ser único.

- **Longitud de Nombre y Descripción**:
  - El nombre del calendario debe ser único en toda la aplicación.

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

### Transiciones Permitidas

### Condiciones para Transiciones

## Dependencias

- **Entidades**:
- **Servicios**:
- **Value Objects**:
  - `CalendarId`: Identificador único del calendario.

## Ejemplos

La entidad `Calendar` no pueder ser instanciada fuera de nu contexto de dominio. Para crear un nuevo calendario, se debe utilizar el `CreateCalendarAcync` del service model `CalendarModel`.

TODO: Agregar enlace de referencia a la documentación de `CalendarManager`.
