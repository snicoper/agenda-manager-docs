# Permisos de Usuario

**Entity**: `Permission`

## Descripción General

Entidad que representa un permiso en el sistema. Los permisos son unidades fundamentales de control de acceso que definen acciones específicas que pueden realizarse dentro de la aplicación.

Los permisos son inmutables una vez creados y su creación debe realizarse por el sistema.

### Responsabilidades

- Mantener la integridad de los datos del permiso
- Validar las reglas de negocio relacionadas con la creación de permisos
- Notificar la creación de nuevos permisos a través de eventos de dominio
- Encapsular la lógica de validación del nombre del permiso

## Invariantes

- `Id` no puede ser `null` en ningún momento
- El `Name` del permiso no puede ser nulo o vacío
- El ``Name`` del permiso no puede exceder los 100 caracteres
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

## Propiedades

| Nombre | Tipo             | Descripción                     |
| ------ | ---------------- | ------------------------------- |
| `Id`   | `PermissionId`   | Identificador único del permiso |
| `Name` | `string`         | Nombre del permiso              |

## Relaciones

- Un `Permission` tiene un `PermissionId` único que identifica el permiso.

## Métodos

```csharp
private static void GuardAgainstInvalidName(string name)
```

- Valida el nombre del permiso.
- `name`: Nombre del permiso.
- Lanza `PermissionDomainException` si el nombre no cumple con las reglas de negocio.
- Return `void`.

## Estado y Transiciones

La entidad Permission es inmutable y no tiene transiciones de estado después de su creación. Una vez creado un permiso:
    - No puede modificarse su nombre
    - No puede eliminarse
    - No puede modificarse su Id

## Dependencias

- **Entidades**:
- **Services**:
  - `IPermissionRepository`: Repositorio de permisos.
- **Managers**:
  - [PermissionManager](./services/permission-manager.md): Gestiona la creación de permisos.
  - [AuthorizationManager](./services/authorization-manager.md): Gestiona la autorización de usuarios.
- **Policies**:
- **Value Objects**:
- [PermissionId](../value-objects/permission-id.md)

## Eventos de Dominio

- `PermissionCreatedDomainEvent(Id)`: Se lanza cuando se crea un nuevo permiso.

## Interceptores EF Core

## Comentarios adicionales

## Ejemplos de Uso

```csharp
// Ejemplos de creación de permisos válidos
var userCreatePermission = new Permission(
    new PermissionId(Guid.NewGuid()),
    "user:create");

var roleReadPermission = new Permission(
    new PermissionId(Guid.NewGuid()),
    "role:read");

// Ejemplos que lanzarán excepciones
var invalidNamePermission = new Permission(
    new PermissionId(Guid.NewGuid()),
    "user"); // Falta el sufijo

var emptyNamePermission = new Permission(
    new PermissionId(Guid.NewGuid()),
    string.Empty); // Nombre vacío

var tooLongNamePermission = new Permission(
    new PermissionId(Guid.NewGuid()),
    new string('a', 101) + ":create"); // Excede longitud máxima
```
