Al final la he dejado así:

```markdown
# Permisos de Usuario

**Entity**: Permission

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
- Lanza `PermissionDomainException` si el nombre no cumple con las reglas de negocio.
- Return `void`.

## Estado y Transiciones

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
// Crear un nuevo permiso
var permissionId = new PermissionId(Guid.NewGuid());
var permission = new Permission(permissionId, "CreateUser");

// Esto lanzará una excepción
var invalidPermission = new Permission(
    new PermissionId(Guid.NewGuid()),
    string.Empty);
```
```
