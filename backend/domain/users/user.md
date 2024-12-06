# User

- **Aggregate Root**: `User`
- **Namespace**: `AgendaManager.Domain.Users`
- **Tipo**: Entidad de Dominio Sellada (sealed)
- **Herencia**: `AggregateRoot`

## Descripción General

El agregado root `User` representa a un usuario en el sistema de gestión de agenda. Cada usuario tiene información de identidad básica, credenciales de acceso, roles asociados y un estado de actividad y confirmación de email. Esta clase es fundamental para gestionar las cuentas de los usuarios, su autenticación y autorización dentro del sistema.

### Responsabilidades

- **Gestión de Identidad**:

  - Autenticar y autorizar el acceso del usuario.
  - Mantener la información de identidad única del usuario (`UserId`).
  - Almacenar y validar las credenciales del usuario (hash de contraseña).
  - Actualizar y validar el `Token`.
  - Permitir la actualización del hash de la contraseña.
  - Gestionar el estado de confirmación del email.
  - Almacenar y validar la contraseña del usuario.

- **Gestión de Tokens**:

  - Gestionar tokens de recuperación de contraseña.
  - Gestionar tokens de confirmación de email.

- **Gestión de Estado**:

  - Manejar el estado de confirmación del email (`IsEmailConfirmed`).
  - Controlar el estado de actividad del usuario (`IsActive`).

- **Roles**:

  - Añadir y remover roles asociados al usuario.
  - Proveer una colección de roles asignados al usuario.

- **Profile**:

  - Gestionar el perfil del usuario (`UserProfile`).

- **Eventos de Dominio**:

  - Disparar eventos de dominio cuando se realicen cambios significativos en el usuario.

- **IsAssignableResource**

  - Determinar si el usuario puede ser asignado como recurso.

  ## Propiedades

| Propiedad              | Tipo                      | Descripción                                         |
| ---------------------- | ------------------------- | --------------------------------------------------- |
| `UserId`               | `UserId`                  | Identificador único del usuario.                    |
| `PasswordHash`         | `PasswordHash`            | Hash de la contraseña del usuario.                  |
| `Email`                | `EmailAddress`            | Dirección de correo electrónico única del usuario.  |
| `IsEmailConfirmed`     | `bool`                    | Estado de confirmación del email del usuario.       |
| `IsActive`             | `bool`                    | Estado de actividad del usuario.                    |
| `IsAssignableResource` | `bool`                    | Indica si el usuario puede ser asignado a recursos. |
| `RefreshToken`         | `Token?`                  | Token de refresco de autorización del usuario.      |
| `UserRoles`            | `IReadOnlyList<UserRole>` | Lista de roles asociados al usuario.                |
| `Tokens`               | `IReadOnlyList<Token>`    | Lista de tokens asociados al usuario.               |

## Invariantes

- `Id` no puede ser `null` en ningún momento
- `Email` no puede ser `null` en ningún momento y debe ser un formato de email válido
- `PasswordHash` no puede ser `null` o vacío en ningún momento
- `IsEmailConfirmed`, `IsActive` y `IsAssignableResource` debe ser un valor booleano (`true` o `false`) consistente
- `UserRoles` debe ser una colección de UserRole que puede ser vacía pero nunca `null`
- `UserTokens` debe ser una colección de tokens que puede ser vacía pero nunca `null`

## Reglas de negocio

- **Unicidad de Identidad**:

  - `Id` debe ser único en toda la aplicación.
  - `Email` debe ser único en toda la aplicación.

- **Confirmación de Email**:

  - Un usuario puede confirmar su email solo si no está ya confirmado (`IsEmailConfirmed` pasa de `false` a `true`).

- **Estado de Actividad**:

  - Un usuario con permisos puede cambiar su estado de actividad (`IsActive`) y este cambio **debe ser registrado**.

- **Roles**:

  - Un usuario puede tener múltiples roles, pero no roles duplicados.
  - Los roles asociados a un usuario deben permitir la correcta autorización dentro del sistema.

- **Profile**:

  - El perfil del usuario (`UserProfile`) debe ser gestionado de manera adecuada.
  - Al crear un usuario, el perfil debe ser creado y asociado al usuario.

- **Tokens de Actualización**:

  - `Token` debe ser único y gestionado de manera segura, asegurando que solo un token activo esté asociado con el usuario en cualquier momento.

## Métodos

### Constructor

```csharp
internal User(
    UserId userId,
    EmailAddress email,
    PasswordHash passwordHash,
    string? firstName,
    string? lastName,
    bool isActive = true,
    bool isAssignableResource = false,
    bool emailConfirmed = false)
```

- **Descripción**: Constructor principal del usuario.
- **Parámetros**:
  - `userId`: Identificador único del usuario.
  - `email`: Dirección de correo electrónico única del usuario.
  - `passwordHash`: Hash de la contraseña del usuario.
  - `firstName`: Nombre del usuario, puede ser `null`.
  - `lastName`: Apellido del usuario, puede ser `null`.
  - `isActive`: Estado de actividad del usuario.
  - `isAssignableResource`: Indica si el usuario puede ser asignado a recursos.
  - `emailConfirmed`: Estado de confirmación del email del usuario.
- **Eventos**:
  - `UserCreatedDomainEvent(userId)`:
    - **Descripción**: Se dispara cuando se crea un nuevo usuario.
    - **Parámetros**:
      - `userId`: Identificador único del usuario.

### UpdatePassword

```csharp
public Result UpdatePassword(PasswordHash newPasswordHash)
```

- **Descripción**: Actualiza la contraseña del usuario.
- **Parámetros**:
  - `newPasswordHash`: Hash de la nueva contraseña del usuario.
- **Eventos**:
  - `UserPasswordUpdatedDomainEvent(Id)`:
    - **Descripción**: Se dispara cuando se actualiza la contraseña del usuario.
    - **Parámetros**:
      - `userId`: Identificador único del usuario.
- **Retorno**: `Result`: Un resultado que indica si la actualización fue exitosa o no.

### UpdateEmail

```csharp
public async Task<Result> UpdateEmail(
    EmailAddress newEmail,
    IEmailUniquenessChecker emailUniquenessChecker,
    CancellationToken cancellationToken)
```

- **Descripción**: Actualiza la dirección de correo electrónico del usuario.
- **Parámetros**:
  - `newEmail`: Nueva dirección de correo electrónico del usuario.
  - `emailUniquenessChecker`: Verificador de unicidad de correo electrónico.
  - `cancellationToken`: Token de cancelación para la operación.
    **Eventos**:
  - `UserEmailUpdatedDomainEvent(Id)`: - **Descripción**: Se dispara cuando se actualiza la dirección de correo electrónico del usuario. - **Parámetros**: - `userId`: Identificador único del usuario.
    **Retorno**: `Result` Un resultado que indica si la actualización fue exitosa o no.

### UpdateRefreshToken

```csharp
public void UpdateRefreshToken(Token token)
```

- **Descripción**: Actualiza el token de refresco del usuario.
- **Parámetros**:
  - `token`: Token de refresco del usuario.
- **Eventos**:
  - `UserRefreshTokenUpdatedDomainEvent(Id)`:
    - **Descripción**: Se dispara cuando se actualiza el token de refresco del usuario.
    - **Parámetros**:
      - `userId`: Identificador único del usuario.

### ConfirmEmail

```csharp
public void ConfirmEmail()
```

- **Descripción**: Marca el correo electrónico del usuario como confirmado.
- **Eventos**:
  - `UserEmailConfirmedDomainEvent(Id)`:
    - **Descripción**: Se dispara cuando se confirma el correo electrónico del usuario.
    - **Parámetros**:
      - `userId`: Identificador único del usuario.

### Activate

```csharp
public void Activate()
```

- **Descripción**: Activa el usuario.
- **Eventos**:
  - `UserActivatedDomainEvent(Id)`:
    - **Descripción**: Se dispara cuando se activa el usuario.
    - **Parámetros**:
      - `userId`: Identificador único del usuario.

### Deactivate

```csharp
public void Deactivate()
```

- **Descripción**: Desactiva el usuario.
- **Eventos**:
  - `UserDeactivatedDomainEvent(Id)`:
    - **Descripción**: Se dispara cuando se desactiva el usuario.
    - **Parámetros**:
      - `userId`: Identificador único del usuario.

## CreateUserToken

```csharp
public Result<UserToken> CreateUserToken(UserTokenType type)
```

- **Descripción**: Crea un nuevo token de usuario.
- **Parámetros**:
  - `type`: Tipo de token.
- **Eventos**:
  - `UserTokenCreatedDomainEvent(userToken)`:
    - **Descripción**: Se dispara cuando se crea un nuevo token de usuario.
    - **Parámetros**:
      - `userToken`: Token de usuario creado.
- **Retorno**: `Result<UserToken>`: Resultado que contiene el nuevo token de usuario.

### ConsumeUserToken

```csharp
public Result ConsumeUserToken(UserTokenId userTokenId, string tokenValue)
```

- **Descripción**: Consume un token de usuario.
- **Parámetros**:
  - `userTokenId`: Identificador del token de usuario.
  - `tokenValue`: Valor del token.
- **Eventos**:
  - `UserTokenConsumedDomainEvent(userToken.Id)`:
    - **Descripción**: Se dispara cuando se consume un token de usuario.
    - **Parámetros**:
      - `userTokenId`: Identificador del token de usuario consumido.
- **Retorno**: `Result`: Resultado que indica si el token fue consumido o no.

### AddUserToken

```csharp
public void AddUserToken(Token token)
```

- **Descripción**: Agrega un token de usuario.
- **Parámetros**:
  - `token`: Token de usuario.
- **Eventos**:
  - `UserTokenAddedDomainEvent(Id, userToken.Id)`:
    - **Descripción**: Se dispara cuando se agrega un token de usuario.
    - **Parámetros**:
      - `userId`: Identificador del usuario.
      - `userTokenId`: Identificador del token de usuario agregado.

### RemoveUserToken

```csharp
public void RemoveUserToken(UserToken userToken)
```

- **Descripción**: Elimina un token de usuario.
- **Parámetros**:
  - `userToken`: Token de usuario a eliminar.
- **Eventos**:
  - `UserTokenRemovedDomainEvent(Id, userToken.Id)`:
    - **Descripción**: Se dispara cuando se elimina un token de usuario.
    - **Parámetros**:
      - `userId`: Identificador del usuario.
      - `userTokenId`: Identificador del token de usuario eliminado.

### HasRole

```csharp
public bool HasRole(UserRole userRole)
```

- **Descripción**: Verifica si el usuario tiene un rol específico.
- **Parámetros**:
  - `userRole`: UserRole a verificar.
- **Retorno**: `bool`: `true` si el usuario tiene el rol, `false` de lo contrario.

### AddRole

```csharp
public void AddRole(Role role)
```

- **Descripción**: Agrega un rol al usuario.
- **Parámetros**:
  - `role`: Rol a agregar al usuario.
- **Eventos**:
  - `UserRoleAddedDomainEvent(userRole)`: Se dispara cuando se agrega un rol al usuario.
    - **Parámetros**:
      - `userRole`: Rol agregado al usuario.

### RemoveRole

```csharp
public void RemoveRole(Role role)
```

- **Descripción**: Elimina un rol del usuario.
- **Parámetros**:
  - `role`: Rol a eliminar del usuario.
- **Eventos**:
  - `UserRoleRemovedDomainEvent(userRole)`:
    - **Descripción**: Se dispara cuando se elimina un rol del usuario.
    - **Parámetros**:
      - `userRole`: Rol eliminado del usuario.

### AddProfile

```csharp
internal void AddProfile(UserProfile profile)
```

- **Descripción**: Agrega un perfil al usuario.
- **Parámetros**:
  - `profile`: Perfil a agregar al usuario.
- **Eventos**:
  - `UserProfileAddedDomainEvent(userProfileId)`:
  - **Descripción**: Se dispara cuando se agrega un perfil al usuario.
  - **Parámetros**:
    - `userProfileId`: Identificador del perfil agregado al usuario.

### UpdateProfile

```csharp
internal void UpdateProfile(
        string firstName,
        string lastName,
        PhoneNumber? phoneNumber = null,
        Address? address = null,
        IdentityDocument? identityDocument = null)
```

- **Descripción**: Actualiza el perfil del usuario.
- **Parámetros**:
  - `firstName`: Nombre del usuario.
  - `lastName`: Apellido del usuario.
  - `phoneNumber`: Número de teléfono del usuario.
  - `address`: Dirección del usuario.
  - `identityDocument`: Documento de identidad del usuario.
**Eventos**:
  - `UserProfileUpdatedDomainEvent(Id)`:
  - **Descripción**: Se dispara cuando se actualiza el perfil del usuario.
  - **Parámetros**:
  - `Id`: Identificador del usuario.

## Estado y Transiciones

La clase `User` tiene diferentes estados que dependen de ciertas condiciones y acciones realizadas dentro del sistema.

- **Estados Iniciales**:

  - **Estado Inicial**: Al crear un nuevo `User`, los estados iniciales son los siguientes:
    - `IsEmailConfirmed`: `false` (el email del usuario no está confirmado).
    - `IsActive`: `true` (el usuario está activo por defecto).

- **Estados Posibles**:

  - **IsEmailConfirmed**:

    - **`true`**: El email del usuario ha sido confirmado.
    - **`false`**: El email del usuario no ha sido confirmado.

  - **IsActive**:
    - **`true`**: El usuario está activo y puede interactuar con el sistema.
    - **`false`**: El usuario está inactivo y no puede interactuar con el sistema.

- **Transiciones Permitidas**:

  - **Transiciones de `IsEmailConfirmed`**:

    - De **`false` a `true`**: Ocurre cuando el usuario confirma su email a través del enlace de confirmación enviado a su correo electrónico.
      - **Condición**: La confirmación debe ser iniciada por el usuario.

  - **Transiciones de `IsActive`**:
    - De **`true` a `false`**: Puede ocurrir en cualquier momento cuando el personal autorizado decide desactivar al usuario.
      - **Condición**: Solo personal autorizado puede cambiar el estado a inactivo.
    - De **`false` a `true`**: Puede ocurrir en cualquier momento cuando el personal autorizado decide reactivar al usuario.
    - **Condición**: Solo personal autorizado puede cambiar el estado a activo.

- **Condiciones para Transiciones**:
  - **Confirmación de Email**:
    - Solo puede ocurrir una vez después de que el usuario se ha registrado y ha completado el proceso de confirmación de email.

## Dependencias

### Directas

- **Entidades Base**:

  - `AggregateRoot`: Base class que designa esta entidad como raíz de agregado, proporcionando control transaccional y consistencia del agregado
    - Hereda capacidades de auditoría (`AuditableEntity`)
    - Hereda gestión de eventos de dominio (`Entity`)

- `UserRole`: Representa un rol asignado a un usuario.
- `UserProfile`: Representa el perfil de un usuario.

### Interfaces

- `IEmailUniquenessPolicy`: Interfaz para verificar la unicidad del email.
- `IPasswordHasher`: Interfaz para el servicio de hash de contraseñas.
- `IPasswordPolicy`: Interfaz para las políticas de contraseña.
- `IUserRepository`: Interfaz para el repositorio de usuarios.

### Services

- `AuthenticationService`: Gestiona la autenticación del usuario.
- `UserManager`: Gestiona la creación, actualización y eliminación de usuarios.

### Policies

- `EmailUniquenessPolicy`: Verifica la unicidad del email.
- `PasswordPolicy`: Define las políticas de contraseña.

### Value Objects

- `UserId`: Identificador único del usuario.
- `UserProfileId`: Identificador único del perfil del usuario.
- `PasswordHash`: Hash de la contraseña del usuario.
- `EmailAddress`: Dirección de correo electrónico del usuario.
- `Token`: Token de actualización del usuario.

## Receptores de Eventos

### Users Application Event Handlers

#### Audit Handlers

- `AuditUserActivatedDomainEventHandler` -> `UserActivatedDomainEvent`

  - Registra el cambio de `IsActive` en la tabla `AuditRecord`
  - Registra: `userId`, `oldValue: false`, `newValue: true`

- `AuditUserDeactivatedDomainEventHandler` -> `UserDeactivatedDomainEvent`
  - Registra el cambio de `IsActive` en la tabla `AuditRecord`
  - Registra: `userId`, `oldValue: true`, `newValue: false`

## Errores

### NotFound

- **Identifier**: `UserNotFound` Se lanza cuando el usuario no se encuentra.
  - **Code**: `UserErrors.UserNotFound`
  - **Description**: The user was not found.

### Conflict

- **Identifier**: `InvalidCredentials` Se lanza cuando las credenciales proporcionadas son inválidas.

  - **Code**: `UserErrors.InvalidCredentials`
  - **Description**: `The provided credentials are invalid.`

- **Identifier**: `UserIsNotActive` Se lanza cuando el usuario no está activo.

  - **Code**: `UserErrors.UserIsNotActive`
  - **Description**: The user is not active.

- **Identifier**: `EmailIsNotConfirmed` Se lanza cuando el email no está confirmado.

  - **Code**: `UserErrors.EmailIsNotConfirmed`
  - **Description**: The email is not confirmed.

- **Identifier**: `RoleAlreadyExists` Se lanza cuando el usuario ya tiene el rol especificado.

  - **Code**: `UserErrors.RoleAlreadyExists`
  - **Description**: The user already has the specified role.

- **Identifier**: `RoleDoesNotExist` Se lanza cuando el usuario no tiene el rol especificado.

  - **Code**: `UserErrors.RoleDoesNotExist`
  - **Description**: The user does not have the specified role.

- **Identifier**: `UserAlreadyConfirmedEmail` Se lanza cuando el usuario ya ha confirmado su email.
  - **Code**: `UserErrors.UserAlreadyConfirmedEmail`
  - **Description**: The user has already confirmed their email.

### Validación

- **Identifier**: `EmailAlreadyExists` Se lanza cuando el email ya existe.

  - **Code**: `UserErrors.EmailAlreadyExists`
  - **Description**: The email already exists.
  - **FieldName**: `Email`

- **Identifier**: `InvalidFormatPassword` Se lanza cuando la contraseña no cumple con la política de contraseña.
  - **Code**: `UserErrors.InvalidFormatPassword`
  - **Description**: Password must be at least 8 characters long and contain at least one uppercase letter, one lowercase letter, one digit and one special character.
  - **FieldName**: `Password`

## Comentarios adicionales

- La entidad `User` no puede ser instanciada fuera de un contexto de dominio. Para crear un nuevo usuario, se debe utilizar el método `CreateUserAsync` del service model `UserManager`.
- La entidad `User` no puede ser modificada después de ser creada. Para actualizar un usuario, se debe utilizar el método `UpdateUserAsync` del service model `UserManager`.
- La creación de un nuevo usuario, creara un nuevo `UserProfile` desde el `UserManager`.

## Ejemplos de Uso

```csharp
// Crear un nuevo usuario
var user = userManager.CreateUserAsync(
  userId: UserId.Create(),
  email: EmailAddress.From("john.doe@example.com"),
  passwordHash: PasswordHash.From("hashedPassword") o PasswordHash.FromRaw("Password123!"),
  firstName: "John",
  lastName: "Doe",
  isActive: true,
  isEmailConfirmed: false,
  cancellationToken: cancellationToken);
```
