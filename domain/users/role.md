# Clase: Role

## Descripción

Un `Role` es una entidad que representa un rol de usuario en el sistema. Los roles son una agrupación de permisos que determinan qué acciones puede realizar un usuario dentro del sistema.

### Responsabilidades

- **Eventos de Dominio**:
  - Disparar eventos de dominio cuando se crea, actualiza o elimina un rol (`RoleCreatedDomainEvent`, `RoleUpdatedDomainEvent`, `RoleDeletedDomainEvent`, etc.).

## Propiedades

| Propiedad     | Tipo                | Descripción                           |
|---------------|---------------------|---------------------------------------|
| `Id`          | `RoleId`            | Identificador único del rol.          |
| `Name`        | `string`            | Nombre del rol.                       |
| `Description` | `string`            | Descripción del rol.                  |
| `Editable`    | `bool`              | Indica si el rol es editable.         |
| `Users`       | `List<User>`        | Lista de usuarios asociados al rol.   |
| `Permissions` | `List<Permission>`  | Lista de permisos asociados al rol.   |

## Asociaciones

- Un `Role` tiene un `RoleId` que es su identificador único.
- Un `Role` puede tener muchos `User`.
- Un `Role` puede tener muchos `Permission`.

## Métodos

### UpdateEditableState (public)

Actualiza el estado editable del rol.

- **Parámetros**:
  - **editable** `bool`: Indica si el rol es editable.
- **Valor de retorno**:
  - `void`.
- **Eventos**:
  - `RoleEditableStateUpdatedDomainEvent`.
- **Excepciones**:
  - Ninguna.

### UpdateRole (internal)

Actualiza el nombre y la descripción del rol. Accesible desde `RoleManager`.

- **Parámetros**:
  - **name** `string`: El nuevo nombre del rol.
  - **description** `string`: La nueva descripción del rol.
- **Valor de retorno**:
  - `void`.
- **Eventos**:
  - `RoleUpdatedDomainEvent`.
- **Excepciones**:
  - Ninguna.

### AddPermission (internal)

Agrega un permiso al rol. Accesible desde `AuthorizationManager`.

- **Parámetros**:
  - **permission** `Permission`: El permiso a agregar.
- **Valor de retorno**:
  - `Result`: Un resultado que indica si la operación fue exitosa o no.
- **Eventos**:
  - `RolePermissionAddedDomainEvent`.
- **Excepciones**:
  - Ninguna.

### RemovePermission (internal)

Elimina un permiso del rol. Accesible desde `AuthorizationManager`.

- **Parámetros**:
  - **permission** `Permission`: El permiso a eliminar.
- **Valor de retorno**:
  - `Result`: Un resultado que indica si la operación fue exitosa o no.
- **Eventos**:
  - `RolePermissionRemovedDomainEvent`.
- **Excepciones**:
  - Ninguna.

### GuardAgainstInvalidName (private)

Valida que el nombre del rol sea válido.

- **Parámetros**:
  - **name** `string`: El nombre del rol.
- **Valor de retorno**:
  - `void`.
- **Excepciones**:
  - `RoleDomainException` si el nombre del rol no es válido.

### GuardAgainstInvalidDescription (private)

Valida que la descripción del rol sea válida.

- **Parámetros**:
  - **description** `string`: La descripción del rol.
- **Valor de retorno**:
  - `void`.
- **Excepciones**:
  - `RoleDomainException` si la descripción del rol no es válida.

## Invariantes

- `Id` no puede ser nulo en ningún momento.
- `Name` no puede ser nulo en ningún momento.
- `Description` no puede ser nulo en ningún momento.
- `Editable` debe ser un valor booleano.

## Reglas de Negocio

- **Unicidad de Identidad**:
  - El `RoleId` debe ser único en toda la aplicación.
  - El `Name` debe ser único en toda la aplicación.
  - El `Description` debe ser único en toda la aplicación.

- **Estado Editable**:
  - Un rol puede cambiar su estado de editable (`Editable`) y este cambio debe ser registrado.

- **Permisos**:
  - Un rol puede tener múltiples permisos, pero no permisos duplicados.
  - Los permisos asociados a un rol deben permitir acciones específicas dentro del sistema.

- **Actualización de Name y Description**:
  - Un rol puede cambiar su nombre y descripción, pero solo si el rol es editable.

- **Integridad**:
  - Un rol no puede ser eliminado si tiene usuarios asignados.

## Estado y Transiciones

La clase `Role` tiene los siguientes estados y transiciones:

### Estado Inicial

- **Estado Inicial**: Al crear un nuevo rol, el estado inicial es:
  - `Editable`: `false`, por defecto el rol no es editable.

### Estados Posibles

- **Editable**:
  - `true`: El rol es editable.
  - `false`: El rol no es editable.

### Transiciones Permitidas

- **Transición de `Editable`**:
  - Desde `true` a `false`: Cuando un usuario cambia el estado editable del rol.
  - **Condición**: La confirmación debe ser realizada por un usuario con permisos para editar roles.

### Condiciones para Transiciones

- **Condición de Transición de `Editable`**:
  - Puede ocurrir en cualquier momento, pero solo puede ser realizada por personal autorizado. Este cambio debe ser registrado y auditado adecuadamente dentro del sistema para mantener la integridad y seguridad.

## Dependencias

- **Entidades**:
  - `User`: Un `Role` puede tener muchos `User`.
  - `Permission`: Un `Role` puede tener muchos `Permission`.
- **Servicios**:
  - `IRoleRepository`: Interfaz para acceder a los datos de los roles.
- **Managers**:
  - `RoleManager`: Responsable de la creación, actualización y eliminación de roles.
  - `AuthorizationManager`: Gestiona la autorización de usuarios.
- **Value Objects**:
  - `RoleId`: Identificador único del rol.

## Ejemplos

```csharp
// Crear un nuevo rol.
var role = new Role(RoleId.Create(), "Admin", "Administrator role");
