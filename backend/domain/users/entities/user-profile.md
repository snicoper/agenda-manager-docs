# User

- **Aggregate Root**: `UserProfile`
- **Namespace**: `AgendaManager.Domain.Entities`
- **Tipo**: Entidad de Dominio Sellada (sealed)
- **Herencia**: `AuditableEntity`

## ToDo List

## Descripción General

El agregado `UserProfile` representa la información de perfil de un usuario en el sistema de gestión de agenda. Contiene información personal, de contacto y documentos de identidad del usuario, manteniendo la integridad y unicidad de estos datos.

### Responsabilidades

- **Gestión de Identidad**
  - Mantener información personal básica (nombre, apellidos)
  - Gestionar documentos de identidad
  - Asegurar la unicidad de documentos identificativos

- **Gestión de Contacto**
  - Mantener información de contacto (teléfono, dirección)
  - Asegurar la unicidad de datos de contacto

- **Validación de Datos**
  - Validar formato y longitud de campos
  - Asegurar integridad de datos obligatorios
  - Verificar unicidad de datos críticos

- **Eventos de Dominio**
  - Disparar eventos de dominio cuando se realicen cambios significativos en el perfil
  - Notificar la creación de nuevos perfiles
  - Asegurar que solo se emiten eventos cuando hay cambios reales

## Propiedades

| Nombre             | Tipo                | Descripción                                                 |
| ------------------ | ------------------- | ----------------------------------------------------------- |
| `Id`               | `UserProfileId`     | Identificador único del perfil de usuario.                  |
| `UserId`           | `UserId`            | Identificador único del usuario al que pertenece el perfil. |
| `User`             | `User`              | Referencia al usuario al que pertenece el perfil.           |
| `FirstName`        | `string`            | Nombre del usuario.                                         |
| `LastName`         | `LastName`          | Apellidos del usuario.                                      |
| `PhoneNumber`      | `PhoneNumber?`      | Número de teléfono del usuario.                             |
| `Address`          | `Address?`          | Dirección del usuario.                                      |
| `IdentityDocument` | `IdentityDocument?` | Documento de identidad del usuario.                         |

## Invariantes

- `Id` no puede ser `null` en ningún momento
- `FirstName` no puede superar los 100 caracteres
- `LastName` no puede superar los 100 caracteres
- `PhoneNumber` si no es `null` debe ser único en la base de datos
- `Address` si no es `null` debe ser único en la base de datos
- `UserId` no puede ser `null`
- Un User debe tener exactamente un UserProfile asociado
- Un UserProfile debe tener exactamente un User asociado
- Un UserProfile debe eliminarse cuando se elimina su User (integridad referencial)
- La existencia de UserProfile está vinculada a la existencia de su User

## Reglas de negocio

- **Unicidad de Datos de Contacto**:
  - Si se proporciona un `PhoneNumber`, debe ser único en toda la aplicación
  - Si se proporciona un `IdentityDocument`, debe ser único por tipo y país

- **Formato y Validación**:
  - `FirstName` y `LastName` no pueden contener números ni caracteres especiales
  - `FirstName` y `LastName` deben tener longitud entre 2 y 100 caracteres
  - `FirstName` y `LastName` se almacenan con la primera letra en mayúscula (normalización)

- **Actualización de Datos**:
  - Los cambios en el perfil solo generan eventos si hay modificaciones reales
  - La actualización de documentos de identidad debe mantener un historial auditable
  - No se pueden tener dos documentos del mismo tipo activos simultáneamente

- **Vinculación con User**:
  - Un UserProfile solo puede estar vinculado a un User
  - La relación User-UserProfile es inmutable una vez establecida
  - Si se desactiva el User asociado, el perfil debe marcarse como inactivo

- **Ciclo de Vida**:
  - Un UserProfile debe crearse automáticamente al crear un User
  - Un UserProfile no puede existir sin un User asociado
  - Un UserProfile no puede transferirse entre Users
  - Un UserProfile debe eliminarse automáticamente cuando se elimina su User asociado

- **Ciclo de Vida y Eliminación**:
  - La existencia del UserProfile está ligada al ciclo de vida del User
  - La eliminación se realiza en cascada cuando se elimina el User
  - No se permite la eliminación directa del UserProfile

## Métodos

### Constructor

```csharp
private UserProfile(
        UserProfileId userProfileId,
        UserId userId,
        string firstName,
        string lastName,
        PhoneNumber? phoneNumber,
        Address? address,
        IdentityDocument? identityDocument)
```

- **Descripción**: Constructor principal para crear una nueva instancia de `UserProfile`.
- **Parámetros**:
  - `userProfileId`: Identificador único del perfil de usuario.
  - `userId`: Identificador único del usuario al que pertenece el perfil.
  - `firstName`: Nombre del usuario.
  - `lastName`: Apellidos del usuario.
  - `phoneNumber`: Número de teléfono del usuario.
  - `address`: Dirección del usuario.
  - `identityDocument`: Documento de identidad del usuario.

### Create

```csharp
public static UserProfile Create(
        UserProfileId userProfileId,
        UserId userId,
        string firstName,
        string lastName,
        PhoneNumber? phoneNumber,
        Address? address,
        IdentityDocument? identityDocument)
```

- **Descripción**: Método de fábrica para crear una nueva instancia de `UserProfile`.
- **Parámetros**:
  - `userProfileId`: Identificador único del perfil de usuario.
  - `userId`: Identificador único del usuario al que pertenece el perfil.
  - `firstName`: Nombre del usuario.
  - `lastName`: Apellidos del usuario.
  - `phoneNumber`: Número de teléfono del usuario.
  - `address`: Dirección del usuario.
  - `identityDocument`: Documento de identidad del usuario.
- **Eventos**:
  - `UserProfileCreatedDomainEvent(userId)`
  - **Descripción**: Se genera cuando se crea un nuevo perfil de usuario.
  - **Parámetros**:
    - `userId`: Identificador único del usuario al que pertenece el perfil.
- **Retorno**: `UserProfile`

### Update

```csharp
internal void Update(
        string firstName,
        string lastName,
        PhoneNumber? phoneNumber,
        Address? address,
        IdentityDocument? identityDocument)
```

- **Descripción**: Método para actualizar los datos del perfil de usuario.
- **Parámetros**:
  - `firstName`: Nombre del usuario.
  - `lastName`: Apellidos del usuario.
  - `phoneNumber`: Número de teléfono del usuario.
  - `address`: Dirección del usuario.
  - `identityDocument`: Documento de identidad del usuario.
- **Eventos**:
  - `UserProfileUpdatedDomainEvent(userId)`
  - **Descripción**: Se genera cuando se actualizan los datos del perfil de usuario.
    - **Parámetros**:
    - `userId`: Identificador único del usuario al que pertenece el perfil.

### HasChanges

```csharp
internal bool HasChanges(
        string firstName,
        string lastName,
        PhoneNumber? phoneNumber,
        Address? address,
        IdentityDocument? identityDocument)
```

- **Descripción**: Método para verificar si hay cambios en los datos del perfil de usuario.
- **Parámetros**:
  - `firstName`: Nombre del usuario.
  - `lastName`: Apellidos del usuario.
  - `phoneNumber`: Número de teléfono del usuario.
  - `address`: Dirección del usuario.
  - `identityDocument`: Documento de identidad del usuario.
**Retorno**: `bool` - `true` si hay cambios, `false` en caso contrario.

## Estado y Transiciones

El UserProfile no tiene estados explícitos, pero su estado está implícitamente ligado al estado del User asociado:

- Si el User está inactivo, el perfil también se considera inactivo
- Si el User está activo, el perfil está accesible y modificable

## Dependencias

### Directas

- **Entidades Base**:
  - `AuditableEntity`: Base class que proporciona capacidades de auditoría
    - Hereda gestión de eventos de dominio (`Entity`)

- `User`: Entidad que representa un usuario.

### Interfaces

- **No Aplica**

### Managers

- `UserProfileManager`: Manager que proporciona funcionalidades para gestionar perfiles de usuario.

### Policies

- **No Aplica**

### Value Objects

- `UserProfileId`: Value Object que representa el identificador único de un perfil de usuario.
- `UserId`: Value Object que representa el identificador único de un usuario.
- `PhoneNumber`: Value Object que representa un número de teléfono.
- `Address`: Value Object que representa una dirección.
- `IdentityDocument`: Value Object que representa un documento de identidad.

## Errores

### Conflict

- **Identifiers**: `IdentityDocumentAlreadyExists` - Cuando se intenta crear un UserProfile con un IdentityDocument que ya existe en la base de datos
  - **Code**: `UserProfile.IdentityDocumentAlreadyExists`
  - **Description**: Identity document already exists.

- **Identifiers**: `PhoneNumberAlreadyExists` - Cuando se intenta crear un UserProfile con un PhoneNumber que ya existe en la base de datos
  - **Code**: `UserProfile.PhoneNumberAlreadyExists`
  - **Description**: Phone number already exists.

## Comentarios adicionales

- La creación del UserProfile está acoplada a la creación del User. No se puede crear un User sin su Profile correspondiente
- Los Value Objects (Address, PhoneNumber, IdentityDocument) aseguran la validación y normalización de sus datos internamente
- La unicidad de PhoneNumber, Address e IdentityDocument se verifica a nivel de infraestructura
- Los eventos de dominio solo se disparan cuando hay cambios reales en los datos
- La eliminación en cascada se implementa a nivel de infraestructura mediante EF Core
- La edición del UserProfile se realiza a través de `UserProfileManager`

## Ejemplos de Uso
