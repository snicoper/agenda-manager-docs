# Service

- **Aggregate Root**: `Service`
- **Namespace**: `AgendaManager.Domain.Services`
- **Tipo**: Entidad de Dominio Sellada (sealed)
- **Herencia**: `AggregateRoot`

## Descripción General

`Service` representa un servicio que puede ser agendado en la aplicación. Define los recursos necesarios y la duración base para su realización. La gestión del ciclo de vida del servicio (creación, edición y eliminación) se realiza a través del `ServiceManager`.

### Responsabilidades

- Requisitos necesarios para la creación del servicio
- Establecer la duración base que se requiere para su realización
- Pertenecer a un calendario específico

## Propiedades

| Nombre        | Tipo             | Acceso          | Descripción                                       |
| ------------- | ---------------- | ----------------|-------------------------------------------------- |
| `Id`          | `ServiceId`      | get             | Identificador único del servicio                  |
| `CalendarId`  | `CalendarId`     | get/private set | Identificador del calendario asociado al servicio |
| `Name`        | `string`         | get/private set | Nombre del servicio                               |
| `Description` | `string`         | get/private set | Descripción del servicio                          |
| `ColorScheme` | `ColorScheme`    | get/private set | ColorScheme del servicio                          |
| `Duration`    | `Duration`       | get/private set | Duración del servicio                             |
| `IsActive`    | `bool`           | get/private set | Indica si el servicio está activo                 |

## Invariantes

- `ID` no puede ser `null` en ningún momento
- `CalendarId` no puede ser `null` en ningún momento y debe ser un ID válido
- `Name` no puede ser nulo o vacío y debe tener un máximo de 50 caracteres
- `Description` no puede ser nulo o vacío y debe tener un máximo de 500 caracteres
- `ColorScheme` no puede ser nulo
- `Duration` no puede ser nulo y debe ser un tiempo mayor a cero
- `IsActive` debe tener un valor y por defecto es `true`

## Reglas de negocio

- **Unicidad de Identidad**:
  - `Id` debe ser único en toda la aplicación.
  - `Name` debe ser único dentro del calendario asociado

- **Gestión de Creación**:
  - Al crear un servicio, se emitirá un evento de dominio para notificar la creación del servicio
    - El `CalendarId` debe ser un ID válido
    - El `Duration` no puede ser nulo y debe ser un tiempo mayor a cero
    - El `Name` debe ser único dentro del calendario asociado y no ser nulo o vacío y debe tener un máximo de 50 caracteres
    - El `Description` puede ser nulo o vacío y debe tener un máximo de 500 caracteres
    - El `Id` debe ser único en toda la aplicación
    - El `ColorScheme` no puede ser nulo
    - El `IsActive` debe tener un valor booleano válido por defecto `true`

- **Gestión de Modificaciones**:
  - Al modificar un servicio, se emitirá un evento de dominio para notificar cambios en la programación
  - La edición solo permite modificar: `Duration`, `Name`, `Description` y `ColorScheme`
    - La `Duration` no puede ser nulo y debe ser un tiempo mayor a cero
    - El `Name` debe ser único dentro del calendario asociado
    - La `Description` puede ser nulo o vacío y debe tener un máximo de 500 caracteres
    - El `ColorScheme` no puede ser nulo

- **Gestión de Eliminación**:
  - Al eliminar un servicio, se emitirá un evento de dominio para notificar cambios en la programación
  - Restricciones:
    - No debe tener citas asociadas

## Métodos

### Delete

- **No Aplica**

### AddResourceType

```csharp
public void AddResourceType(ResourceType resourceType)
```

- **Descripción**: Agrega un tipo de recurso al servicio.
- **Parámetros**:
  - `resourceType`: Tipo de recurso a agregar.
- **Eventos**: `ResourceTypeAddedToServiceDomainEvent` emitido cuando se agrega un tipo de recurso al servicio.

### RemoveResourceType

```csharp
public void RemoveResourceType(ResourceType resourceType)
```

- **Descripción**: Elimina un tipo de recurso del servicio.
- **Parámetros**:
  - `resourceType`: Tipo de recurso a eliminar.
- **Eventos**: `ResourceTypeRemovedFromServiceDomainEvent` emitido cuando se elimina un tipo de recurso del servicio.

### Create

```csharp
public static Service Create(
   ServiceId serviceId,
   CalendarId calendarId,
   Duration duration,
   string name,
   string description,
   ColorScheme colorScheme,
   bool isActive = true)
```

- **Descripción**: Crea una nueva instancia de `Service` con los valores proporcionados.
- **Parámetros**:
  - `serviceId`: Identificador único del servicio.
  - `calendarId`: Identificador del calendario asociado al servicio.
  - `duration`: Duración del servicio.
  - `name`: Nombre del servicio.
  - `description`: Descripción del servicio.
  - `colorScheme`: ColorScheme del servicio.
- **Eventos**: `ServiceCreatedDomainEvent` emitido cuando se crea el servicio.
- **Retorno**: Una nueva instancia de `Service` con los valores proporcionados.

### Update

```csharp
internal bool Update(
   Duration duration,
   string name,
   string description,
   ColorScheme colorScheme)
```

- **Descripción**: Actualiza los detalles del servicio.
- **Parámetros**:
  - `duration`: Duración del servicio.
  - `name`: Nombre del servicio.
  - `description`: Descripción del servicio.
  - `colorScheme`: ColorScheme del servicio.
- **Eventos**: `ServiceUpdatedDomainEvent` emitido cuando se actualiza el servicio.
- **Retorno**: `true` si la actualización fue exitosa, `false` en caso contrario.

## Estado y Transiciones

- **No Aplica**

## Dependencias

- **Entidades Base**:
  - `AggregateRoot`: Base class que designa esta entidad como raíz de agregado, proporcionando control transaccional y consistencia del agregado
    - Hereda capacidades de auditoría (`AuditableEntity`)
    - Hereda gestión de eventos de dominio (`Entity`)

### Directas

- `Calendar`: Calendario asociado al servicio

### Interfaces

- `IServiceRepository`: Interfaz para acceder a los servicios

### Managers

- `ServiceManager`: Gestor de servicios

### Policies

- **No Aplica**

### Value Objects

- `ServiceId`: Identificador único del servicio
- `CalendarId`: Identificador del calendario asociado al servicio
- `ColorScheme`: ColorScheme del servicio

## Comentarios adicionales

- **No Aplica**

## Ejemplos de Uso

- **No Aplica**
