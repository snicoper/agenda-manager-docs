# Permisos de Usuario

- **Entity**: `Permission`
- **Namespace**: `AgendaManager.Domain.Authorization.Entities`
- **Tipo**: Entidad de Dominio Sellada (sealed)
- **Herencia**: `AuditableEntity`

## Descripción General

Entidad que representa un permiso en el sistema. Los permisos son unidades fundamentales de control de acceso que definen acciones específicas que pueden realizarse dentro de la aplicación.

Los permisos son inmutables una vez creados y su creación debe realizarse por el sistema.

### Responsabilidades

- Mantener la integridad de los datos del permiso
- Validar las reglas de negocio relacionadas con la creación de permisos
- Notificar la creación de nuevos permisos a través de eventos de dominio
- Encapsular la lógica de validación del nombre del permiso

## Propiedades

| Nombre      | Tipo           | Descripción                                               |
| ----------- | -------------- | --------------------------------------------------------- |
| `Id`        | `PermissionId` | Identificador único del permiso                           |
| `Name`      | `string`       | Nombre del permiso                                        |

## Invariantes

- `Id` no puede ser `null` en ningún momento
- El `Name` del permiso no puede ser nulo o vacío
- El `Name` del permiso no puede exceder los 100 caracteres
- El `Name` del permiso debe tener un sufijo (`:create`, `:read`, `:update`, `:delete`) al final
  - Ejemplos validos:
    - `user:create`
    - `role:read`
    - `permission:update`
    - `profile:delete`

## Reglas de negocio

- **Inmutabilidad de los Permisos**:

  - Los permisos son inmutables una vez creados, no se podrán modificar o eliminar.

- **Unicidad de Identidad**:
  - `Id` debe ser único en toda la aplicación.
  - `Name` no puede repetirse en ningún momento dentro de la aplicación.

### Añadir Nuevos Módulos al Sistema de Permisos

Cuando se añade un nuevo módulo al sistema, se debe considerar:

1. **Evaluación de Permisos Necesarios**:
   - Determinar si el módulo necesita control de permisos
   - Si es funcionalidad del sistema (como UserToken), no debe añadirse al sistema de permisos
   - Para módulos de negocio, evaluar qué operaciones necesitan control (CRUD completo vs permisos específicos)

2. **Estructura de Permisos**:
   - Por defecto, los módulos siguen el patrón CRUD (`create`, `read`, `update`, `delete`)
   - Si el módulo requiere un conjunto diferente de permisos, debe documentarse y justificarse

3. **Nomenclatura**:
   - Los permisos siguen el formato `ModuleName:Action`
   - Ejemplo: `NewModule:read`, `NewModule:create`

4. **Actualización de Documentación**:
   - Añadir el nuevo módulo a esta lista de permisos predefinidos
   - Documentar cualquier comportamiento especial o restricción

### Permisos Predefinidos

- `Appointment` - (`create`, `read`, `update`, `delete`)
- `AppointmentStatus` - (`create`, `read`, `update`, `delete`)
- `AuditRecord` - (`read`)
- `Calendar` - (`create`, `read`, `update`, `delete`)
- `CalendarHoliday` - (`create`, `read`, `update`, `delete`)
- `CalendarConfiguration` - (`create`, `read`, `update`, `delete`)
- `Resource` - (`create`, `read`, `update`, `delete`)
- `ResourceSchedule` - (`create`, `read`, `update`, `delete`)
- `ResourceType` - (`create`, `read`, `update`, `delete`)
- `Service` - (`create`, `read`, `update`, `delete`)
- `User` - (`create`, `read`, `update`, `delete`)
- `Role` - (`create`, `read`, `update`, `delete`)
- `Permission` - (`create`, `read`, `update`, `delete`)

## Métodos

### Constructor

```csharp
internal Permission(PermissionId permissionId, string name)
```

- **Descripción**: Crea una nueva instancia de `Permission`.
  - `permissionId`: Identificador único del permiso.
  - `name`: Nombre del permiso.
- **Eventos**: `PermissionCreatedDomainEvent`: Evento de dominio que se dispara cuando se crea un nuevo permiso.

### GuardAgainstInvalidName

```csharp
private static void GuardAgainstInvalidName(string name)
```

- **Descripción**: Valida el nombre del permiso.
  - `name`: Nombre del permiso.
- **Excepciones**: `PermissionDomainException` si el nombre no cumple con las reglas de negocio

## Estado y Transiciones

La entidad Permission es inmutable y no tiene transiciones de estado después de su creación. Una vez creado un permiso: - No puede modificarse su nombre - No puede eliminarse - No puede modificarse su Id

## Dependencias

### Directas

- **Entidades Base**:
  - `AuditableEntity`: Base class que proporciona capacidades de auditoría
    - Hereda gestión de eventos de dominio (`Entity`)

### Entidades

- **No Aplica**

### Servicios

- `IPermissionRepository`: Repositorio de permisos.

### Managers

- `PermissionManager`: Gestiona la creación de permisos.
- `AuthorizationManager`: Gestiona la autorización de usuarios.

### Policies

- **No Aplica**

### Value Objects

- `PermissionId`: Identificador único del permiso.

## Errores

### NotFound

- **Identifier**: `PermissionNotFound` Se lanza cuando un permiso no existe.
  - **Code**: `PermissionErrors.PermissionNotFound`
  - **Descripción**: The permission was not found.

### Validation

- **Identifier**: `PermissionNameAlreadyExists` Se lanza cuando un permiso con el mismo nombre ya existe.
  - **Code**: `Name`
  - **Descripción**: The permission name already exists.

## Comentarios adicionales

## Ejemplos de Uso

```csharp
// Crear un nuevo permiso
var permission = new Permission(PermissionId.Create(), "user:create");
```
