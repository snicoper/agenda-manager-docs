# ResourceSchedule

- **Aggregate Root**: `ResourceSchedule`
- **Namespace**: `AgendaManager.Domain.Resources.Entities`
- **Tipo**: Entidad de Dominio Sellada (sealed)
- **Herencia**: `AuditableEntity`

## Descripción General

`ResourceSchedule` representa una programación de disponibilidad para un recurso en el sistema. Gestiona los períodos de tiempo en los que un recurso está disponible o no disponible para ser asignado a citas.

Cada programación define:

- Un período específico (fecha inicio - fecha fin)
- Los días de la semana aplicables
- El tipo de disponibilidad (Available/Unavailable)
- Estado de activación (activo/inactivo)
- Información descriptiva (nombre y descripción)

Los horarios tipo `Unavailable` tienen precedencia sobre los `Available`, permitiendo establecer excepciones a la disponibilidad regular del recurso.

### Responsabilidades

- **Gestión de la Disponibilidad**:
  - Mantener y gestionar los períodos de disponibilidad del recurso
  - Coordinar la precedencia entre horarios disponibles y no disponibles
  - Validar que los días de la semana especificados sean válidos
  - Controlar el estado de activación del horario

- **Gestión del Ciclo de Vida**:
  - Mantener la integridad de sus datos (período, tipo, días disponibles)
  - Coordinar la creación, actualización y eliminación de horarios
  - Emitir eventos de dominio cuando los cambios para que afectados sean notificados
  - Gestionar las transiciones de estado activo/inactivo

- **Gestión de Cambios**:
  - Emitir eventos de dominio para notificar cambios en la programación
  - Mantener la trazabilidad de las modificaciones realizadas *

- **Estados de Actividad**:
  - **Activo** (`IsActive = true`):
    - Estado inicial por defecto al crear un horario
    - Estado operativo normal
    - Participa en la validación de disponibilidad del recurso
    - Permite la asignación de citas según el tipo (Available/Unavailable)

  - **Inactivo** (`IsActive = false`):
    - No participa en la validación de disponibilidad
    - No permite nuevas asignaciones de citas

## Invariantes del Agregado

- **Unicidad e Integridad**:
  - `Id` no puede ser `null` en ningún momento
  - `ResourceId` no puede ser `null` y debe referenciar a un recurso existente
  - `CalendarId` no puede ser `null` y debe referenciar a un calendario existente
  - `Period` no puede ser `null` y debe ser un período válido
  - `Type` debe ser un valor válido de `ResourceScheduleType`
  - `AvailableDays` no puede ser `null` o `WeekDays.None`
  - `Name` no puede ser `null` o vacío en ningún momento, no debe exceder los 50 caracteres
  - `Description` puede ser null, o si se proporciona, no debe exceder los 500 caracteres
  - `IsActive` debe tener un valor booleano válido

## Reglas de Negocio

- **Gestión de Modificaciones**:
  - La edición solo permite modificar: `AvailableDays`, `Period`, `Name` y `Description`
  - Al modificar un horario, se emitirá un evento de dominio para notificar cambios en la programación

- **Eliminación**:
  - Al eliminar un horario, se emitirá un evento de dominio para notificar cambios en la programación

- **Adición de No Disponibilidad**:
  - Al añadir un horario tipo `Unavailable`, se emitirá un evento de dominio para notificar cambios en la programación

- **Gestión de Estado Activo**:
  - La desactivación de un horario emitirá un evento de dominio para notificar cambios en la programación
  - La activación de un horario no requiere notificación

## Propiedades

| Propiedad           | Tipo                      | Acceso          | Descripción                                               |
|---------------------|---------------------------|-----------------|-----------------------------------------------------------|
| `Id`                | `ResourceScheduleId`      | get             | Identificador único de la programación de recurso.        |
| `ResourceId`        | `ResourceId`              | get/private set | Identificador del recurso asociado a la programación.     |
| `Resource`          | `Resource`                | get/private set | Recurso asociado a la programación.                       |
| `CalendarId`        | `CalendarId`              | get/private set | Identificador del calendario asociado a la programación.  |
| `Calendar`          | `Calendar`                | get/private set | Calendario asociado a la programación.                    |
| `Period`            | `Period`                  | get/private set | Período de disponibilidad del recurso.                    |
| `Type`              | `ResourceScheduleType`    | get/private set | Tipo de programación de recurso.                          |
| `WeekDays`          | `AvailableDays`           | get/private set | Días de la semana en los que el recurso está disponible.  |
| `Name`              | `string`                  | get/private set | Nombre de la programación de recurso.                     |
| `Description`       | `string?`                 | get/private set | Descripción de la programación de recurso.                |
| `IsActive`          | `bool`                    | get/private set | Estado de activación del horario.                         |

## Métodos

### Create

```csharp
public static ResourceSchedule Create(
    ResourceScheduleId resourceScheduleId,
    ResourceId resourceId,
    CalendarId calendarId,
    Period period,
    ResourceScheduleType type,
    WeekDays availableDays,
    string name,
    string? description,
    bool isActive = true)
```

- **Descripción**: Crea una nueva instancia de `ResourceSchedule`.
- **Parámetros**:
  - `resourceScheduleId`: Identificador único de la programación de recurso.
  - `resourceId`: Identificador del recurso asociado a la programación.
  - `calendarId`: Identificador del calendario asociado a la programación.
  - `period`: Período de disponibilidad del recurso.
  - `type`: Tipo de programación de recurso.
  - `availableDays`: Días de la semana en los que el recurso está disponible.
  - `name`: Nombre de la programación de recurso.
  - `description`: Descripción de la programación de recurso.
  - `isActive`: Estado de activación del horario.
- **Eventos**:
  - `ResourceScheduleCreated`: Evento emitido cuando se crea una nueva programación de recurso.
- **Retorno**: Nueva instancia de `ResourceSchedule`.

### Update

```csharp
internal bool Update(Period period, string name, string? description, WeekDays availableDays)
```

- **Descripción**: Actualiza los detalles de la programación de recurso.
- **Parámetros**:
  - `period`: Período de disponibilidad del recurso.
  - `name`: Nombre de la programación de recurso.
  - `description`: Descripción de la programación de recurso.
  - `availableDays`: Días de la semana en los que el recurso está disponible.
**Retorno**: `true` si hubo actualización, `false` en caso contrario.

### Activate

```csharp
internal void Activate()
```

- **Descripción**: Activa la programación de recurso.
- **Eventos**:
  - `ResourceScheduleActivated`: Evento emitido cuando se activa una programación de recurso.

### Deactivate

```csharp
internal void Deactivate()
```

- **Descripción**: Desactiva la programación de recurso.
- **Eventos**:
  - `ResourceScheduleDeactivated`: Evento emitido cuando se desactiva una programación de recurso.

### GuardAgainstInvalidName

```csharp
private void GuardAgainstInvalidName(string name)
```

- **Descripción**: Verifica si el nombre proporcionado es válido.
- **Parámetros**:
  - `name`: Nombre de la programación de recurso.
- **Excepciones**:
  - `ResourceScheduleDomainException`: Se lanza si el nombre proporcionado es inválido.

### GuardAgainstInvalidDescription

```csharp
private void GuardAgainstInvalidDescription(string? description)
```

- **Descripción**: Verifica si la descripción proporcionada es válida.
- **Parámetros**:
  - `description`: Descripción de la programación de recurso.
- **Excepciones**:
  - `ResourceScheduleDomainException`: Se lanza si la descripción proporcionada es inválida.

## Estado y Transiciones

- **No Aplica**

### Estados del ResourceSchedule

## Dependencias

### Directas

- **Entidades Base**:
  - `AggregateRoot`: Base class que designa esta entidad como raíz de agregado, proporcionando control transaccional y consistencia del agregado
    - Hereda capacidades de auditoría (`AuditableEntity`)
    - Hereda gestión de eventos de dominio (`Entity`)

- `Resource`: Entidad de dominio que representa un recurso en el sistema.
- `Calendar`: Entidad de dominio que representa un calendario en el sistema.

### Interfaces

- `IResourceScheduleRepository`: Interfaz que define los métodos para interactuar con la base de datos de las entidades de `ResourceSchedule`.

### Managers

- **No Aplica**

### Policies

- **No Aplica**

### Value Objects

- `ResourceScheduleId`: Identificador único de la programación de recurso.
- `ResourceId`: Identificador del recurso asociado a la programación.
- `CalendarId`: Identificador del calendario asociado a la programación.
- `Period`: Período de disponibilidad del recurso.

## Eventos de Dominio

Los eventos ocurren en el agregado `Resource`.

## Interceptores EF Core

- **No Aplica**

## Comentarios adicionales

### Funcionamiento de Period

El `Period` es un Value Object que encapsula dos aspectos fundamentales de un intervalo de tiempo:

1. **Componente de Fecha**:
   - Representa el rango de fechas en el calendario (ej: 01/01/2024 - 31/12/2024)
   - Permite definir períodos largos de disponibilidad

2. **Componente de Hora**:
   - Define el horario específico dentro de cada día (ej: 08:00 - 14:00)
   - Se aplica a cada día incluido en el rango de fechas

Esta separación permite:

- Definir horarios recurrentes (ej: todos los lunes de 8:00 a 14:00 durante un año)
- Combinar múltiples franjas horarias en un mismo día
- Establecer excepciones mediante horarios tipo `Unavailable`

Ejemplo 1:

```textplain
Period: 2024-01-01 08:00 - 2024-12-31 14:00
AvailableDays: [Monday, Wednesday, Friday]
Type: Available
```

Este Period indica disponibilidad los lunes, miércoles y viernes de 8:00 a 14:00 durante todo el año 2024.

Ejemplo 2:

Si se crea uno genérico que representa el horario habitual de un recurso como:

```textplain
Period: 2000-01-01 08:00 - 2100-12-31 14:00
AvailableDays: [Monday, Wednesday, Friday]
Type: Available
```

```textplain
Period: 2000-01-01 16:00 - 2100-12-31 18:00
AvailableDays: [Monday, Wednesday, Friday]
Type: Available
```

Este Period indica disponibilidad los lunes, miércoles y viernes de 8:00 a 14:00 desde 2000-01-01 hasta 2100-12-31.

Y luego se crea uno mas especifico para un evento como:

```textplain
Period: 2024-11-11 09:00 - 2024-11-11 16:00
AvailableDays: [Monday]
Type: Available
```

Si "concatenamos" los horarios habituales y este evento especial, el resultado es que para el dia 11 de Noviembre de 2024 el recurso está disponible de 8:00 a 18:00 (siempre que no haya otro horario que se solape con un tipo `Unavailable`).

## Ejemplos de Uso
