# Clase: Role

## Descripción

Un `Role` es una entidad que representa un rol de usuario en el sistema.

Los roles son una agrupación de permisos que determinan qué acciones un usuario puede realizar en el sistema.

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
- Un `Role`

## Métodos

### Constructor (Instancia un nuevo calendario)

- **Parámetros**:
- **Eventos**:
- **Excepciones**:

### Update (Actualiza un calendario existente)

- **Parámetros**:
- **Eventos**:
- **Excepciones**:

## Invariantes

## Reglas de Negocio

## Estado y Transiciones

### Estados Iniciales

- **Estado Inicial**:

### Estados Posibles

### Transiciones Permitidas

### Condiciones para Transiciones

## Dependencias

- **Entidades**:
- **Services**:
- **Managers**:
- **Policies**:
- **Value Objects**:

## Ejemplos
