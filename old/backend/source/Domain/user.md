# User

```mermaid
classDiagram
    class User {
        +Id UserId
        +UserName string
        +PasswordHash string
        +Email EmailAddress
        +IsEmailConfirmed bool
        +FirstName string?
        +LastName string?
        +Active bool
        +RefreshToken RefreshToken?

        +Create(UserId userId, EmailAddress email, UserName userName, PasswordHash passwordHash, FirstName? firstName, LastName? lastName) User$
        +UpdatePassword(string passwordHash) Result
        +UpdateEmail(EmailAddress) User
        +UpdateRefreshToken(RefreshToken refreshToken) User
        +SetActiveState(bool state) User
    }

    User --|> UserRole : Has
    UserRole --|> User : Belongs
    Role --|> UserRole : Has
    UserRole --|> Role : Belongs
    User --|> UserPermission : Has
    UserPermission --|> User : Belongs
    Permission --|> UserPermission : Has
    UserPermission --|> Permission : Belongs
```

- **UserRole**: Asociación de roles a usuarios.
- **Role**: Representa un rol que puede tener un usuario.
- **UserPermission**: Asociación de permisos a usuarios.
- **Permission**: Representa un permiso específico que puede tener un usuario.

## Reglas de negocio

- La contraseña debe tener al menos 8 caracteres y contener al menos una letra mayúscula, una minúscula, un número y un carácter especial.
- El campo `Active` debe reflejar el estado actual del usuario, permitiendo su activación o desactivación solo por administradores legítimos.
- El correo electrónico debe ser valido y único.

## Domain Events

- `UserCreatedDomainEvent(UserId userId)` Se lanza cuando se crea un nuevo usuario.
- `UserUpdatedDomainEvent(UserId userId)` Se lanza cuando se actualiza la información básica de un usuario.
- `UserPasswordUpdatedDomainEvent(UserId userId)` Se lanza cuando se actualiza la contraseña de un usuario.
- `UserEmailUpdatedDomainEvent(UserId userId)` Se lanza cuando se actualiza el correo electrónico de un usuario.
- `UserEmailConfirmedDomainEvent(UserId userId)` Se lanza cuando se confirma el correo electrónico de un usuario.
- `UserActiveStateChangedDomainEvent(UserId UserId, bool State)` Se lanza cuando cambia el estado activo de un usuario.

## Value objects

- `UserId` Identificador único del usuario.
- `EmailAddress` Dirección de correo electrónico del usuario.
- `RefreshToken` Es un token utilizado para mantener la autenticación del usuario entre sesiones.

### UserId

```mermaid
classDiagram
    class UserId {
        +Value string

        +From(Guid value) UserId
        +Create() UserId
    }
```

Representa el identificador único de un usuario.

### EmailAddress

```mermaid
classDiagram
    class EmailAddress {
        +Value string

        +From(string value) EmailAddress
    }
```

Representa una dirección de correo electrónico válida.

### RefreshToken

```mermaid
classDiagram
    class RefreshToken {
        +Token string
        +ExpiryTime DateTimeOffset

        +Create(string token, DateTimeOffset expiryTime) RefreshToken
    }
```

Es un token utilizado para mantener la autenticación del usuario entre sesiones.

## Documentación Adicional
