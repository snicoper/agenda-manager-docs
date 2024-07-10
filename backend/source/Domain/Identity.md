# Identity

## Relaciones User, Role y Permission

```mermaid
classDiagram
    class User {
        -_passwordHash string
        +Id UserId
        +UserName string
        +Email EmailAddress
        +IsEmailConfirmed bool
        +FirstName string?
        +LastName string?
        +Active bool
        +RefreshToken RefreshToken?

        +Create(UserId userId, EmailAddress email, UserName userName, PasswordHash passwordHash, FirstName? firstName, LastName? lastName) User$
        +VerifyPassword(string rawPassword) bool
        +UpdatePassword(string rawPassword) Result
        +UpdateEmail(EmailAddress) User
        +UpdateRefreshToken(RefreshToken refreshToken) User
        +SetActiveState(bool state) User
    }

    class Role {
        +Id RoleId
        +Name string

        +Create(RoleId id, string name) Role
    }

    class Permission {
        +Id PermissionId
        +Name string

        +Create(PermissionId id, string name) Permission
    }


   class UserRole {
        +Guid UserId
        +User User
        +Guid RoleId
        +Role Role

        +Create(UserId userId, RoleId roleId) UserRole$
    }

    class UserPermission {
        +Guid UserId
        +User User
        +Guid PermissionId
        +Permission Permission

        +Create(UserId userId, PermissionId permissionId) UserPermission$
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

- `User` `src/Domain/Users/User.cs`
- `Role` `src/Domain/Authorization/Role.cs`
- `Permission` `src/Domain/Authorization/Permission.cs`
- `UserRole` `src/Domain/Authorization/UserRole.cs`
- `UserPermission` `src/Domain/Authorization/UserPermission.cs`

## Authentication

```mermaid
classDiagram
    class IAuthenticationManager {
        <<interface>>
        +LoginAsync(string email, string password) Result~TokenResponse~
        +RefreshTokenAsync(string refreshToken) Result~TokenResponse~
    }

    IAuthenticationManager <|-- AuthenticationManager : implements
    IAuthenticationManager --> Result~TokenResponse~ : Association
```

- `IAuthenticationManager` `src/Application/Common/Interfaces/Users/IAuthenticationManager.cs`
- `AuthenticationManager` `src/Infrastructure/Common/Authentication/AuthenticationManager.cs`

## JwtTokenGenerator

```mermaid
classDiagram
    class IJwtTokenGenerator {
        <<interface>>
        +GenerateAccessTokenAsync(User user) string
        +GenerateRefreshToken() string
    }

    IJwtTokenGenerator <|-- JwtTokenGenerator : implements
```

- `IJwtTokenGenerator` `src/Application/Common/Interfaces/Users/IJwtTokenGenerator.cs`
- `JwtTokenGenerator` `src/Infrastructure/Common/Authentication/JwtTokenGenerator.cs`

## CurrentUserProvider

```mermaid
classDiagram
    class ICurrentUserProvider{
        <<interface>>
        +GetCurrentUser() CurrentUser
    }

    ICurrentUserProvider <|-- CurrentUserProvider : implements
```

- `ICurrentUserProvider` `src/Application/Common/Interfaces/Users/ICurrentUserProvider.cs`
- `CurrentUserProvider` `src/WebApi/Services/CurrentUserProvider.cs`

## IAuthorizationManager

```mermaid
classDiagram
    class IAuthorizationManager {
        <<interface>>
        +HasRole(UserId userId, string role) bool
        +HasPermission(UserId userId, string permissionName) bool
        +AddRoleAsync(UserId userId, RoleId roleId, CancellationToken cancellationToken = default) void
        +RemoveRoleAsync(UserId userId, RoleId roleId, CancellationToken cancellationToken = default) void
        +AddPermissionAsync(UserId userId, PermissionId permissionId, CancellationToken cancellationToken = default) void
        +RemovePermissionAsync(UserId userId, PermissionId permissionId, CancellationToken cancellationToken = default) void
    }

    IAuthorizationManager <|-- AuthorizationManager : implements
```

- `IAuthorizationManager` `src/Domain/Authorization/Persistence/IAuthorizationManager.cs`
- `AuthorizationManager` `src/Infrastructure/Authorization/AuthorizationManager.cs`

```mermaid
classDiagram
    class IPasswordHasher {
        <<Interface>>
        +HashPassword(string password) Result~string~$
        +VerifyPassword(string password, string hashedPassword) bool$
    }

    IPasswordHasher <|-- PasswordHasher : implements
    IPasswordHasher --> Result~string~ : Association
```

- `IPasswordHasher` `src/Domain/Common/Interfaces/IPasswordHasher.cs`
- `PasswordHasher` `src/Infrastructure/Common/Authentication/PasswordHasher.cs`

## Repositories

```mermaid
classDiagram
    class IUserRepository {
        <<interface>>
        +GetQueryable() IQeuryable~User~
        +GetByIdAsync(UserId id, CancellationToken cancellationToken = default) User
        +GetByEmailAsync(EmailAddress email, CancellationToken cancellationToken = default) User
        +GetByUserNameAsync(string userName, CancellationToken cancellationToken = default) User
        +AddAsync(User user, CancellationToken cancellationToken = default) void
        +Update(User user) void
    }

    IUserRepository <|-- UserRepository : implements
```

- `IUsersRepository` `src/Domain/Users/Persistence/IUsersRepository.cs`
- `UserRepository` `src/Infrastructure/Users/UserRepository.cs`
