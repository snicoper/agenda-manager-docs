# Permisos de Usuario

- **Entity**: `Permission`
- **Namespace**: `AgendaManager.Domain.Users.Entities`
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

| Nombre | Tipo             | Acceso          | Descripción                     |
| ------ | ---------------- | ----------------|-------------------------------- |
| `Id`   | `PermissionId`   | get             | Identificador único del permiso |
| `Name` | `string`         | get/private set | Nombre del permiso              |

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

## Métodos

### GuardAgainstInvalidName

```csharp
private static void GuardAgainstInvalidName(string name)
```

- **Descripción**:  Valida el nombre del permiso.
  - `name`: Nombre del permiso.
- **Excepciones**: `PermissionDomainException` si el nombre no cumple con las reglas de negocio

## Estado y Transiciones

La entidad Permission es inmutable y no tiene transiciones de estado después de su creación. Una vez creado un permiso:
    - No puede modificarse su nombre
    - No puede eliminarse
    - No puede modificarse su Id

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

## Comentarios adicionales

- **No Aplica**

## Ejemplos de Uso
