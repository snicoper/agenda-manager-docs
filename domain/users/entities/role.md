# Role

**Entity**: Role

## Descripción General

Un `Role` es una entidad que representa un rol de usuario en el sistema. Los roles son una agrupación de permisos que determinan qué acciones puede realizar un usuario dentro del sistema.

### Responsabilidades

- **Estado**:
  - Por defecto el sistema debe tener roles predefinidos, como "Administrator", "Employee", "Customer", etc y estos roles deben ser marcados `Editable` como `false` para que no puedan ser modificados por los usuarios.

- **Eventos de Dominio**:
  - Disparar eventos de dominio cuando se realicen cambios significativos en el role.

- **Validación**:
  - Asegurarse de que `Name` y `Description` no exceden la longitud permitida.

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

## Invariantes

- `Id` no puede ser `null` en ningún momento.
- `Name` no puede ser `null` en ningún momento ni exceder la longitud permitida.
- `Description` no puede ser `null` en ningún momento ni exceder la longitud permitida.
- `Editable` debe ser un valor booleano (`true` o `false`) consistente.

## Reglas de Negocio

- **Unicidad de Identidad**:
  - El `RoleId` debe ser único en toda la aplicación.
  - El `Name` debe ser único en toda la aplicación.
  - El `Description` debe ser único en toda la aplicación.

- **Estado Editable**:
  - Un rol **NO** puede cambiar su estado de editable (`Editable`).
  - Un role `Editable` con estado a `false` nunca puede ser eliminado.
  - Todos los roles creados por personal autorizado deben tener `Editable` con estado a `true`.

- **Permisos**:
  - Un rol puede tener múltiples permisos, pero no permisos duplicados.
  - Los permisos asociados a un rol deben permitir acciones específicas dentro del sistema.

- **Actualización de Name y Description**:
  - Un rol puede cambiar su nombre y descripción, pero solo si el rol es editable.

- **Integridad**:
  - Un rol no puede ser eliminado si tiene usuarios asignados.

## Propiedades

| Propiedad     | Tipo                | Descripción                           |
|---------------|---------------------|---------------------------------------|
| `Id`          | `RoleId`            | Identificador único del rol.          |
| `Name`        | `string`            | Nombre del rol.                       |
| `Description` | `string`            | Descripción del rol.                  |
| `Editable`    | `bool`              | Indica si el rol es editable.         |
| `Permissions` | `List<Permission>`  | Lista de permisos asociados al rol.   |

## Relaciones

- Un `Role` tiene un `RoleId` que es su identificador único.
- Un `Role` puede tener muchos `Permission`.

## Métodos

```csharp
internal void UpdateRole(string name, string description)
```

- Actualiza el nombre y la descripción del rol.
- Lanza el evento `RoleUpdatedDomainEvent`.
- Return `void`.

```csharp
internal Result AddPermission(Permission permission)
```

- Agrega un permiso al rol.
- Lanza el evento `RolePermissionAddedDomainEvent`.
- Return `void`.

```csharp
internal Result RemovePermission(Permission permission)
```

- Elimina un permiso del rol.
- Lanza el evento `RolePermissionRemovedDomainEvent`.
- Return `void`.

```csharp
private static void GuardAgainstInvalidName(string name)
```

- Valida que el nombre del rol no sea nulo y no exceda los 100 caracteres.
- Lanza la excepción `RoleDomainException` si el nombre del rol es nulo o excede los 256 caracteres o no tiene un sufijo permitido.
- Return `void`.

```csharp
private static void GuardAgainstInvalidDescription(string description)
```

- Valida que la descripción del rol no sea nula y no exceda los 500 caracteres.
- Lanza la excepción `RoleDomainException` si la descripción del rol es nula o excede los 256 caracteres.

## Estado y Transiciones

La clase `Role` tiene los siguientes estados y transiciones:

- **Estados Iniciales**:
  - **Estado Inicial**: Al crear un nuevo rol, el estado inicial es:
    - `Editable`: `false`, por defecto el rol no es editable.

- **Estados Posibles**
  - **Editable**:
    - `true`: El rol es editable.
    - `false`: El rol no es editable y no puede ser eliminado.

- **Transiciones Permitidas**:
  - **Transición de `Editable`**:
    - Un rol no puede cambiar su estado de editable (`Editable`) en ningún momento.

## Dependencias

- **Entidades**:
  - `Permission`: Un `Role` puede tener muchos `Permission`.
- **Servicios**:
  - `IRoleRepository`: Interfaz para acceder a los datos de los roles.
- **Managers**:
  - `RoleManager`: Responsable de la creación, actualización y eliminación de roles.
  - `AuthorizationManager`: Gestiona la autorización de usuarios.
- **Value Objects**:
  - `RoleId`: Identificador único del rol.

## Eventos de Dominio

- `RoleCreatedDomainEvent(Id)`: Se lanza cuando se crea un nuevo rol.
- `RoleUpdatedDomainEvent(Id)`: Se lanza cuando se actualiza un rol.
- `RolePermissionAddedDomainEvent(Id, permission.Id)`: Se lanza cuando se agrega un permiso a un rol.
- `RolePermissionRemovedDomainEvent(Id, permission.Id)`: Se lanza cuando se elimina un permiso de un rol.

## Comentarios adicionales

## Ejemplos

```csharp
// Crear un nuevo rol.
var role = new Role(RoleId.Create(), "Admin", "Administrator role");
