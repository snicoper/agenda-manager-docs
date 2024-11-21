# Role

- **Entity**: `Role`
- **Namespace**: `AgendaManager.Domain.Authorization`
- **Tipo**: Entidad de Dominio Sellada (sealed)
- **Herencia**: `AggregateRoot`

## ToDo List

- [ ] Verificar roles en `src/Domain/Authorization/Constants/SystemPermissions.cs`

## Descripción General

Un `Role` es una entidad que representa un rol de usuario en el sistema. Los roles son una agrupación de permisos que determinan qué acciones puede realizar un usuario dentro del sistema.

### Responsabilidades

- **Estado**:

  - Por defecto el sistema debe tener roles predefinidos, como "Administrator", "Employee", "Customer", etc y estos roles deben ser marcados `IsEditable` como `false` para que no puedan ser modificados por los usuarios.

- **Eventos de Dominio**:

  - Disparar eventos de dominio cuando se realicen cambios significativos en el role.

- **Validación**:

  - Asegurarse de que `Name` y `Description` no exceden la longitud permitida.

  ## Propiedades

| Propiedad     | Tipo               | Descripción                         |
| ------------- | ------------------ | ----------------------------------- |
| `Id`          | `RoleId`           | Identificador único del rol.        |
| `Name`        | `string`           | Nombre del rol.                     |
| `Description` | `string`           | Descripción del rol.                |
| `IsEditable`  | `bool`             | Indica si el rol es editable.       |
| `Permissions` | `List<Permission>` | Lista de permisos asociados al rol. |

## Invariantes

- `Id` no puede ser `null` en ningún momento.
- `Name` no puede ser `null` en ningún momento ni exceder la longitud permitida.
- `Description` no puede ser `null` en ningún momento ni exceder la longitud permitida.
- `Editable` debe ser un valor booleano (`true` o `false`) consistente.

## Reglas de Negocio

- **Unicidad de Identidad**:

  - El `RoleId` debe ser único en toda la aplicación y no pude exceder los 100 caracteres.
  - El `Name` debe ser único en toda la aplicación y no pude exceder los 500 caracteres.
  - El `Description` debe ser único en toda la aplicación.

- **Estado IsEditable**:

  - Un rol **NO** puede cambiar su estado de editable (`IsEditable`).
  - Un role `IsEditable` con estado a `false` nunca puede ser eliminado.
  - Todos los roles creados por personal autorizado deben tener `IsEditable` con estado a `true`.

- **Permisos**:

  - Un rol puede tener múltiples permisos, pero no permisos duplicados.
  - Los permisos asociados a un rol deben permitir acciones específicas dentro del sistema.

- **Actualización de Name y Description**:

  - Un rol puede cambiar su nombre y descripción, pero solo si el rol es editable.

- **Integridad**:
  - Un rol no puede ser eliminado si tiene usuarios asignados.

## Roles Predefinidos

El sistema define cuatro roles inmutables (`Editable = false`) que no pueden ser modificados:

- **Administrator**:

  - Acceso total al sistema
  - Gestión de usuarios, roles y configuración del sistema
  - Asignado típicamente a administradores del sistema

- **Employee**

  - Personal interno de la empresa
  - Acceso a funciones operativas básicas
  - No implica capacidad de ser asignado como recurso

- **Customer**:

  - Acceso limitado al sistema
  - Destinatarios de los servicios
  - Pueden ver sus propias citas e historial

- **AssignableStaff**:
  - Pueden ser asignados como recursos de tipo `Staff`
  - Independiente del rol Employee
  - Permite tanto empleados internos como colaboradores externos
  - Requerido para la asignación de recursos en la programación

> **Nota**: Estos roles son predefinidos por el sistema y no pueden ser modificados ni eliminados.

## Métodos

### Constructor

```csharp
internal Role(RoleId roleId, string name, string description, bool isEditable = false)
```

- **Descripción**: Crea una nueva instancia de `Role`.
  - `roleId`: Identificador único del rol.
  - `name`: Nombre del rol.
  - `description`: Descripción del rol.
  - `isEditable`: Indica si el rol es editable.
- **Eventos**:
  - `RoleCreatedDomainEvent(Id)`:
    - **Descripción**: Se lanza cuando se crea un nuevo rol.
    - **Parámetros**:
      - `Id`: Identificador del rol creado.

### UpdateRole

```csharp
internal void UpdateRole(string name, string description)
```

- **Descripción**: Actualiza el nombre y la descripción del rol.
  - `name`: Nombre del rol.
  - `description`: Descripción del rol.
- **Eventos**:
  - `RoleUpdatedDomainEvent(Id)`:
  - **Descripción**: Se lanza cuando se actualiza un rol.
  - **Parámetros**:
    - `Id`: Identificador del rol actualizado.

### AddPermission

```csharp
internal Result AddPermission(Permission permission)
```

- **Descripción**: Agrega un permiso al rol.
  - `permission`: Permiso a agregar.
- **Eventos**:
  - `RolePermissionAddedDomainEvent(Id, permission.Id)`
  - **Descripción**: Se lanza cuando se agrega
  - **Parámetros**:
    - `Id`: Identificador del rol.
    - `permission.Id`: Identificador del permiso agregado.
- **Retorno** `Result`
  - `Success` si el permiso se agregó correctamente.
  - `Failure` si el permiso ya existe en el rol.

### RemovePermission

```csharp
internal Result RemovePermission(Permission permission)
```

- **Descripción**: Elimina un permiso del rol.
- `permission`: Permiso a eliminar.
- **Eventos**:
  - `RolePermissionRemovedDomainEvent(Id, permission.Id)`
  - **Descripción**: Se lanza cuando se elimina un permiso del rol.
  - **Parámetros**:
    - `Id`: Identificador del rol.
    - `permission.Id`: Identificador del permiso eliminado.
- **Retorno** `Result`
  - `Success` si el permiso se eliminó correctamente.
  - `Failure` si el permiso no se encontró en el rol.

### HasPermission

```csharp
internal bool HasPermission(PermissionId permissionId)
```

- **Descripción**: Verifica si el rol tiene un permiso específico.
- **Parámetros**
  - `permissionId`: Identificador del permiso a verificar.
- **Retorno** `bool`

### GuardAgainstInvalidName

```csharp
private static void GuardAgainstInvalidName(string name)
```

- Valida que el nombre del rol no sea nulo y no exceda los 100 caracteres.
- **Parámetros**
  - `name`: Nombre del rol.
- **Excepciones**:
  - `RoleDomainException` si el nombre del rol es nulo o excede los 100 caracteres.
- **Retorno**: `void`.

### GuardAgainstInvalidDescription

```csharp
private static void GuardAgainstInvalidDescription(string description)
```

- Valida que la descripción del rol no sea nula y no exceda los 500 caracteres.
- **Parámetros**
  - `description`: Descripción del rol.
- **Excepciones**:
  - `RoleDomainException` si la descripción del rol es nula o excede los 500 caracteres.
- **Retorno**: `void`.

## Estado y Transiciones

La clase `Role` tiene los siguientes estados y transiciones:

- **Estados Iniciales**:

  - **Estado Inicial**: Al crear un nuevo rol, el estado inicial es:
    - `IsEditable`: `false`, por defecto el rol no es editable.

- **Estados Posibles**

  - **Editable**:
    - `true`: El rol es editable.
    - `false`: El rol no es editable y no puede ser eliminado.

- **Transiciones Permitidas**:
  - **Transición de `IsEditable`**:
    - Un rol no puede cambiar su estado de editable (`IsEditable`) en ningún momento.

## Dependencias

### Directas

- **Entidades Base**:
  - `AggregateRoot`: Base class que designa esta entidad como raíz de agregado, proporcionando control transaccional y consistencia del agregado
    - Hereda capacidades de auditoría (`AuditableEntity`)
    - Hereda gestión de eventos de dominio (`Entity`)

### Entidades

- `Permission`: Representa un permiso asociado a un rol.

### Servicios

- `IRoleRepository`: Interfaz para acceder a los datos de los roles.
- `RoleManager`: Responsable de la creación, actualización y eliminación de roles.
- `AuthorizationManager`: Gestiona la autorización de usuarios.

### Value Objects

- `RoleId`: Identificador único del rol.
- `PermissionId`: Identificador único del permiso.

## Comentarios adicionales

- **No Aplica**

## Ejemplos

```csharp
// Crear un nuevo rol.
var role = new Role(RoleId.Create(), "Admin", "Administrator role");
```
