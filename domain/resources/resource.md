# Resource

- **Aggregate Root**: `Resource`
- **Namespace**: `AgendaManager.Domain.Resources`
- **Tipo**: Entidad de Dominio Sellada (sealed)
- **Herencia**: `AggregateRoot`

## Descripción General

El agregado root `Resource` representa un recurso en el sistema de gestión de agenda. Define la naturaleza y características base de los recursos que pueden ser gestionados en el sistema. Cada tipo de recurso puede representar tanto recursos humanos (que requieren un rol) como recursos físicos (sin rol asociado).

### Responsabilidades

- **Gestión del Ciclo de Vida**:
  - Mantener la integridad de sus datos básicos (nombre, descripción, etc.)
  - Coordinar su creación, actualización y eliminación
  - Garantizar que su eliminación solo ocurra cuando no tenga citas asociadas

- **Control de Propiedades**:
  - Solo permitir la modificación de nombre y descripción una vez creado el recurso
  - Mantener inmutables las demás propiedades tras la creación (TypeId, UserId, CalendarId)

- **Coordinación con Calendario**:
  - Mantener la relación con el calendario asignado
  - Asegurar que siempre tenga un calendario válido asociado

- **Gestión de Programaciones**:
  - Mantener la colección de programaciones (ResourceSchedule)
  - Servir como punto de entrada para la gestión de disponibilidad

- **Control de Estado**:
  - Gestionar el estado activo/inactivo del recurso
  - Permitir la desactivación temporal sin eliminar el recurso

## Invariantes del Agregado

- **Unicidad de Identificador**:
  - `Id` no puede ser `null` en ningún momento
  - `UserId` puede ser `null` o no, pero si no es `null` debe ser un valor válido
  - `CalendarId` no puede ser `null` en ningún momento
  - `TypeId` no puede ser `null` en ningún momento
  - `Name` no puede ser `null` o vacío en ningún momento, no debe exceder los 50 caracteres
  - `Description` no puede ser `null` o vacío en ningún momento, no debe exceder los 500 caracteres
  - `DeactivationReason` puede ser null, pero cuando se proporciona no debe estar vacío y no debe exceder los 250 caracteres
  - `ColorScheme` no puede ser `null` en ningún momento

- **Integridad Referencial**:
  - El tipo de recurso asignado debe existir y estar activo
  - Si se requiere un usuario (basado en el tipo de recurso), debe existir y tener el rol correcto
  - El calendario asociado debe existir y estar activo

## Reglas de negocio

- **Proceso de Creación de Recurso**:
  - El proceso de creación de un recurso implica la asignación de un tipo de recurso al recurso
  - Si el tipo de recurso tiene un RoleId asignado, requiere la asignación de un usuario con ese rol específico
  - Si el tipo de recurso no tiene RoleId asignado, el UserId debe ser null
  - Una vez creado, estas asignaciones son inmutables

- **Proceso de Actualización de Recurso**:
  - Solo se permite la modificación del nombre y descripción del recurso
  - No se permite modificar el tipo de recurso, usuario o calendario asignado

- **Proceso de Eliminación de Recurso**:
  - Solo se puede eliminar un recurso si no tiene citas asignadas, independientemente del estado de las mismas
  - La eliminación es física (se elimina completamente de la base de datos)
  - Antes de la eliminación, el sistema debe verificar la ausencia total de citas asociadas

- **Proceso de Desactivación/Activación**:
  - Un recurso puede ser desactivado en cualquier momento
  - Al desactivar, se debe proporcionar un motivo que quedará registrado
  - Al reactivar, se limpia el motivo de desactivación
  - La desactivación afecta a las citas existentes de la siguiente manera:
    - Citas `Completed`: No se ven afectadas
    - Citas `InProgress`: Pueden completarse normalmente
    - Citas `Pending`, `Accepted` y `Waiting`: Pasan a estado `RequiresRescheduling`
  - La desactivación emite un evento de dominio que permitirá:
    - La gestión de las citas afectadas
    - El proceso de notificación a las partes interesadas
  - Un recurso desactivado no puede aceptar nuevas citas
  - Un recurso puede ser reactivado si no ha sido eliminado

## Propiedades

| Propiedad           | Tipo                      | Acceso          | Descripción                                               |
|---------------------|---------------------------|-----------------|-----------------------------------------------------------|
| Id                  | `ResourceId`              | get             | Identificador único del recurso.                          |
| UserId              | `UserId?`                 | get/private set | Identificador único del usuario propietario del recurso.  |
| User                | `User?`                   | get/private set | Usuario propietario del recurso.                          |
| CalendarId          | `CalendarId`              | get/private set | Identificador único del calendario asociado al recurso.   |
| Calendar            | `Calendar`                | get/private set | Calendario asociado al recurso.                           |
| TypeId              | `ResourceTypeId`          | get/private set | Identificador único del tipo de recurso.                  |
| Name                | `string`                  | get/private set | Nombre del recurso.                                       |
| Description         | `string`                  | get/private set | Descripción del recurso.                                  |
| IsActive            | `bool`                    | get/private set | Indica si el recurso está activo o no.                    |
| DeactivationReason  | `string?`                 | get/private set | Último motivo por el que se desactivó el recurso.         |
| ColorScheme         | `string`                  | get/private set | Esquema de color del recurso.                             |
| Schedules           | `List<ResourceSchedule>`  | get/private set | Programaciones asociadas al recurso.                      |

## Métodos

### AddSchedule

```csharp
public void AddSchedule(ResourceSchedule schedule)
```

- **Descripción**: Agrega una nueva programación al recurso.
- **Parámetros**:
  - `schedule`: La programación a agregar.
- **Excepciones**:
- **Eventos**:
  - `ResourceScheduleCreatedDomainEvent`: Evento que se dispara cuando se agrega una nueva programación al recurso.

### RemoveSchedule

```csharp
public void RemoveSchedule(ResourceSchedule schedule)
```

- **Descripción**: Elimina una programación del recurso.
- **Parámetros**:
  - `schedule`: La programación a eliminar.
- **Excepciones**:
- **Eventos**:
  - `ResourceScheduleRemovedDomainEvent`: Evento que se dispara cuando se elimina una programación del recurso.

### Activate

```csharp
public void Activate()
```

- **Descripción**: Activa el recurso.
- **Parámetros**: Ninguno.
- **Excepciones**:
- **Eventos**:
  - `ResourceActivatedDomainEvent`: Evento que se dispara cuando se activa el recurso.

### Deactivate

```csharp
public void Deactivate(string reason)
```

- **Descripción**: Desactiva el recurso.
- **Parámetros**:
  - `reason`: Motivo por el que se desactiva el recurso.
- **Excepciones**:
- **Eventos**:
  - `ResourceDeactivatedDomainEvent`: Evento que se dispara cuando se desactiva el recurso.

### UpdateSchedule

```csharp
public Result UpdateSchedule(ResourceScheduleId scheduleId)
```

- **Descripción**: Actualiza una programación del recurso.
- **Parámetros**:
  - `scheduleId`: Identificador único de la programación a actualizar.
- **Excepciones**:
- **Eventos**:
  - `ResourceScheduleUpdatedDomainEvent`: Evento que se dispara cuando se actualiza una programación del recurso.
**Retorno**: `Result` Resultado de la operación.

### Create

```csharp
internal static Resource Create(
    ResourceId id,
    UserId? userId,
    CalendarId calendarId,
    ResourceTypeId typeId,
    string name,
    string description,
    ColorScheme colorScheme,
    bool isActive)
```

- **Descripción**: Crea una nueva instancia de `Resource`.
- **Parámetros**:
  - `id`: Identificador único del recurso.
  - `userId`: Identificador único del usuario propietario del recurso.
  - `calendarId`: Identificador único del calendario asociado al recurso.
  - `typeId`: Identificador único del tipo de recurso.
  - `name`: Nombre del recurso.
  - `description`: Descripción del recurso.
  - `colorScheme`: Esquema de color del recurso.
  - `isActive`: Indica si el recurso debe ser activado o desactivado.
- **Excepciones**:
- **Eventos**:
  - `ResourceCreatedDomainEvent`: Evento que se dispara cuando se crea un nuevo recurso.
**Retorno**: La instancia de `Resource` creada.

### Update

```csharp
internal void Update(string name, string description, ColorScheme colorScheme)
```

- **Descripción**: Actualiza el nombre, la descripción y el esquema de color del recurso.
- **Parámetros**:
  - `name`: Nuevo nombre del recurso.
  - `description`: Nueva descripción del recurso.
  - `colorScheme`: Nuevo esquema de color del recurso.
- **Excepciones**:
- **Eventos**:
  - `ResourceUpdatedDomainEvent`: Evento que se dispara cuando se actualiza el recurso.

### GuardAgainstInvalidName

```csharp
private static void GuardAgainstInvalidName(string name)
```

- **Descripción**: Valida que el nombre del recurso sea válido.
- **Parámetros**:
  - `name`: Nombre del recurso.
- **Excepciones**:
  - `ResourceDomainException`: Si el nombre del recurso es inválido.
- **Eventos**:

### GuardAgainstInvalidDescription

```csharp
private static void GuardAgainstInvalidDescription(string description)
```

- **Descripción**: Valida que la descripción del recurso sea válida.
- **Parámetros**:
  - `description`: Descripción del recurso.
- **Excepciones**:
  - `ResourceDomainException`: Si la descripción del recurso es inválida.
- **Eventos**:

## Dependencias

### Directas

- **Entidades Base**:
  - `AggregateRoot`: Base class que designa esta entidad como raíz de agregado, proporcionando control transaccional y consistencia del agregado
    - Hereda capacidades de auditoría (`AuditableEntity`)
    - Hereda gestión de eventos de dominio (`Entity`)

- `User`: Representa un usuario en el sistema.
- `Calendar`: Representa un calendario en el sistema.
- `ResourceType`: Representa un tipo de recurso en el sistema.
- `ResourceSchedule`: Representa una programación de un recurso en el sistema.

### Interfaces

- `IResourceRepository`: Define el contrato para la persistencia de `Resource`

### Managers

- `ResourceManager`: Gestiona las operaciones que requieren acceso a la base de datos

### Value Objects

- `ResourceId`: Identificador único del recurso
- `UserId`: Identificador único del usuario propietario del recurso
- `CalendarId`: Identificador único del calendario asociado al recurso
- `ResourceTypeId`: Identificador único del tipo de recurso
- `ColorScheme`: Esquema de color del recurso

## Eventos de Dominio

- `ResourceScheduleCreatedDomainEvent`: Evento que se dispara cuando se agrega una nueva programación al recurso.
- `ResourceScheduleRemovedDomainEvent`: Evento que se dispara cuando se elimina una programación del recurso.
- `ResourceActivatedDomainEvent`: Evento que se dispara cuando se activa el recurso.
- `ResourceDeactivatedDomainEvent`: Evento que se dispara cuando se desactiva el recurso.
- `ResourceCreatedDomainEvent`: Evento que se dispara cuando se crea un nuevo recurso.
- `ResourceUpdatedDomainEvent`: Evento que se dispara cuando se actualiza el recurso.
- `ResourceScheduleUpdatedDomainEvent`: Evento que se dispara cuando se actualiza una programación del recurso.

## Interceptores EF Core

- **No Aplica**

## Comentarios adicionales

- **No Aplica**

## Ejemplos de Uso

- **No Aplica**
