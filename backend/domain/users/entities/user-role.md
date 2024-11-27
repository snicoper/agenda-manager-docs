# User

- **Aggregate Root**: `UserRole`
- **Namespace**: `AgendaManager.Domain.Entities`
- **Tipo**: Entidad de Dominio Sellada (sealed)
- **Herencia**: `AuditableEntity`

## Descripción General

La entidad `UserRole` representa la relación entre un usuario y un rol en el sistema. Esta relación se utiliza para asignar roles a usuarios específicos.

### Responsabilidades

- **Gestionar la asociación entre usuarios y roles**:

  - Representar de manera única la relación entre un usuario y un rol dentro del sistema, asegurando que cada combinación de `UserId` y `RoleId` sea única.

- **Garantizar consistencia en las asignaciones**:

  - Asegurar que las reglas de negocio relacionadas con las asociaciones usuario-rol (como unicidad y validación) sean respetadas.

- **Facilitar consultas y reportes**:

  - Servir como un enlace que permita a otros componentes del sistema identificar rápidamente los roles asociados a un usuario o los usuarios asignados a un rol específico.

## Propiedades

| Nombre   | Tipo     | Descripción                     |
| -------- | -------- | ------------------------------- |
| `UserId` | `UserId` | Identificador único del usuario |
| `RoleId` | `RoleId` | Identificador único del rol     |

## Invariantes

- `UserId` no puede ser `null` en ningún momento.
- `RoleId` no puede ser `null` en ningún momento.

## Reglas de negocio

- **Unicidad de Identidad**:
  - La combinación de `UserId` y `RoleId` debe ser única en toda la aplicación.

## Métodos

### Create

```csharp
public static UserRole Create(UserId userId, RoleId roleId)
```

- **Descripción**: Crea una nueva instancia de `UserRole`.
- **Parámetros**:
  - `userId`: Identificador único del usuario.
  - `roleId`: Identificador único del rol.
- **Retorna**: Una nueva instancia de `UserRole`.

## Estado y Transiciones

- **No Aplica**

## Dependencias

### Directas

- **Entidades Base**:
  - `AuditableEntity`: Base class que proporciona capacidades de auditoría
    - Hereda gestión de eventos de dominio (`Entity`)

### Interfaces

- **No Aplica**

### Managers

- **No Aplica**

### Policies

- **No Aplica**

### Value Objects

- `UserId`: Value Object que representa el identificador único de un usuario.
- `RoleId`: Value Object que representa el identificador único de un rol.

## Comentarios adicionales

- **No Aplica**

## Ejemplos de Uso

```csharp
// Crear una nueva relación de usuario y rol.
var userRole = UserRole.Create(userId, roleId);
```

## ToDo List

- **No Aplica**
