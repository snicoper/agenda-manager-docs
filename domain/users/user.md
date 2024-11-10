# Clase: User

## Descripción

La clase `User` representa a un usuario en el sistema de gestión de agenda. Cada usuario tiene información de identidad básica, credenciales de acceso, roles asociados y un estado de actividad y confirmación de email. Esta clase es fundamental para gestionar las cuentas de los usuarios, su autenticación y autorización dentro del sistema.

### Responsabilidades

- **Gestión de Identidad**:
  - Mantener la información de identidad única del usuario (`UserId`).
  - Almacenar y validar las credenciales del usuario (hash de contraseña).

- **Gestión de Información Personal**:
  - Almacenar y validar la dirección de correo electrónico.
  - Gestionar los datos opcionales como `FirstName` y `LastName`.

- **Gestión de Estado**:
  - Manejar el estado de confirmación del email (`IsEmailConfirmed`).
  - Controlar el estado de actividad del usuario (`Active`).

- **Roles y Permisos**:
  - Añadir y remover roles asociados al usuario.
  - Proveer una colección de roles asignados al usuario.

- **Eventos de Dominio**:
  - Disparar eventos de dominio cuando se crean, actualizan o eliminan datos del usuario.

- **Validación**:
  - Asegurarse de que `FirstName` y `LastName` no exceden la longitud permitida.
  - Validar la unicidad de la dirección de correo electrónico.

- **Actualización de Información**:
  - Actualizar y validar el `Token`.
  - Permitir la actualización del hash de la contraseña.
  - Actualizar la dirección de correo electrónico del usuario de manera segura.

## Propiedades

| Propiedad           | Tipo                      | Descripción                                                |
|---------------------|---------------------------|------------------------------------------------------------|
| `UserId`            | `UserId`                  | Identificador único del usuario.                           |
| `PasswordHash`      | `PasswordHash`            | Hash de la contraseña del usuario.                         |
| `Email`             | `EmailAddress`            | Dirección de correo electrónico única del usuario.         |
| `IsEmailConfirmed`  | `bool`                    | Estado de confirmación del email del usuario.              |
| `FirstName`         | `string?`                 | Nombre del usuario, puede ser `null`.                      |
| `LastName`          | `string?`                 | Apellido del usuario, puede ser `null`.                    |
| `Active`            | `bool`                    | Estado de actividad del usuario.                           |
| `RefreshToken`      | `Token?`           | Token de actualización del usuario, puede ser `null`.      |
| `Roles`             | `IReadOnlyCollection<Role>` | Lista de roles asociados al usuario.                     |

## Asociaciones

- Un `User` tiene un `UserId` único que identifica al usuario.
- Un `User` tiene una dirección de correo electrónico única que identifica al usuario.
- Un `User` tiene un `PasswordHash` que se utiliza para autenticar al usuario.
- Un `User` puede tener un `Token` que le permite acceder a los recursos del sistema.
- Un `User` puede tener 0 o varios `Role` que le permiten acceder a los recursos del sistema.

## Métodos

```csharp
public Result UpdatePassword(PasswordHash newPasswordHash)
```

- Actualiza el hash de la contraseña del usuario.
- Lanza el evento `UserPasswordUpdatedDomainEvent`.
- Return `Result` Un resultado que indica si la operación fue exitosa o no

```csharp
public async Task<Result> UpdateEmail(
    EmailAddress newEmail,
    IEmailUniquenessChecker emailUniquenessChecker,
    CancellationToken cancellationToken)
```

- Actualiza la dirección de correo electrónico del usuario.
- Lanza el evento `UserEmailUpdatedDomainEvent`.
- Return `Result` Un resultado que indica si la operación fue exitosa o no.

```csharp
public void UpdateRefreshToken(Token refreshToken)
```

- Actualiza el `Token` del usuario.
- Lanza el evento `UserRefreshTokenUpdatedDomainEvent`.

```csharp
public void SetEmailConfirmed()
```

- Marca el email del usuario como confirmado.
- Lanza el evento `UserEmailConfirmedDomainEvent`.

```csharp
public void UpdateActiveState(bool newState)
```

- Actualiza el estado de actividad del usuario.
- Lanza el evento `UserActiveStateUpdatedDomainEvent`.

```csharp
internal void UpdateUser(string? firstName, string? lastName)
```

- Actualiza la información del usuario como `FirstName` y `LastName`.
- Lanza el evento `UserUpdatedDomainEvent`.

```csharp
internal Result AddRole(Role role)
```

- Agrega un rol al usuario.
- Lanza el evento `UserRoleAddedDomainEvent`.

```csharp
internal Result RemoveRole(Role role)
```

- Elimina un rol del usuario.
- Lanza el evento `UserRoleRemovedDomainEvent`.

```csharp
private static void GuardAgainstInvalidFirstName(string? firstName)
```

- Valida que el nombre del usuario no sea nulo y no exceda los 256 caracteres.
- Lanza la excepción `UserDomainException` si el nombre del usuario es nulo o excede los 256 caracteres.

```csharp
private static void GuardAgainstInvalidLastName(string? lastName)
```

- Valida que el apellido del usuario no sea nulo y no exceda los 256 caracteres.
- Lanza la excepción `UserDomainException` si el apellido del usuario es nulo o excede los 256 caracteres.

## Invariantes

- `Id` no puede ser `null` en ningún momento.
- `Email` no puede ser `null` en ningún momento y debe ser un formato de email válido.
- `PasswordHash` no puede ser `null` o vacío en ningún momento.
- `IsEmailConfirmed` debe ser un valor booleano (`true` o `false`) consistente.
- `Active` debe ser un valor booleano (`true` o `false`) consistente.
- `FirstName` puede ser `null`, pero si está presente, no debe exceder los 256 caracteres.
- `LastName` puede ser `null`, pero si está presente, no debe exceder los 256 caracteres.
- `Roles` debe ser una colección de roles que puede ser vacía pero nunca `null`.

## Reglas de negocio

- **Unicidad de Identidad**:
  - `Id` debe ser único en toda la aplicación.
  - `Email` debe ser único en toda la aplicación.

- **Confirmación de Email**:
  - Un usuario puede confirmar su email solo si no está ya confirmado (`IsEmailConfirmed` pasa de `false` a `true`).

- **Estado de Actividad**:
  - Un usuario puede cambiar su estado de actividad (`Active`) y este cambio debe ser registrado.

- **Roles**:
  - Un usuario puede tener múltiples roles, pero no roles duplicados.
  - Los roles asociados a un usuario deben permitir la correcta autorización dentro del sistema.

- **Tokens de Actualización**:
  - `Token` debe ser único y gestionado de manera segura, asegurando que solo un token activo esté asociado con el usuario en cualquier momento.

- **Actualización de Información Personal**:
  - `FirstName` y `LastName` pueden actualizarse, pero deben respetar las limitaciones de longitud y no ser nulos si están presentes.
  - El `Email` puede ser actualizado y debe seguir siendo único en la aplicación después de la actualización.

## Estado y Transiciones

La clase `User` tiene diferentes estados que dependen de ciertas condiciones y acciones realizadas dentro del sistema.

- **Estados Iniciales**:
  - **Estado Inicial**: Al crear un nuevo `User`, los estados iniciales son los siguientes:
    - `IsEmailConfirmed`: `false` (el email del usuario no está confirmado).
    - `Active`: `true` (el usuario está activo por defecto).

- **Estados Posibles**:
  - **IsEmailConfirmed**:
    - **`true`**: El email del usuario ha sido confirmado.
    - **`false`**: El email del usuario no ha sido confirmado.

- **Active**:
  - **`true`**: El usuario está activo y puede interactuar con el sistema.
  - **`false`**: El usuario está inactivo y no puede interactuar con el sistema.

- **Transiciones Permitidas**:
  - **Transiciones de `IsEmailConfirmed`**:
    - De **`false` a `true`**: Ocurre cuando el usuario confirma su email a través del enlace de confirmación enviado a su correo electrónico.
      - **Condición**: La confirmación debe ser iniciada por el usuario.

  - **Transiciones de `Active`**:
    - De **`true` a `false`**: Puede ocurrir en cualquier momento cuando el personal autorizado decide desactivar al usuario.
      - **Condición**: Solo personal autorizado puede cambiar el estado a inactivo.
    - De **`false` a `true`**: Puede ocurrir en cualquier momento cuando el personal autorizado decide reactivar al usuario.
    - **Condición**: Solo personal autorizado puede cambiar el estado a activo.

- **Condiciones para Transiciones**:
  - **Confirmación de Email**:
    - Solo puede ocurrir una vez después de que el usuario se ha registrado y ha completado el proceso de confirmación de email.
  - **Cambio de Estado de Actividad**:
    - Puede ocurrir en cualquier momento, pero solo puede ser realizado por personal autorizado. Este cambio debe ser registrado y auditado adecuadamente dentro del sistema para mantener la integridad y seguridad.

  Esta documentación ayudará a entender mejor el ciclo de vida de un `User` y las condiciones bajo las cuales pueden ocurrir ciertas transiciones de estado.

## Dependencias

- **Entidades**:
  - [Role](./role.md): Lista de roles asociados a este usuario.
- **Services**:
  - [UserManager](./services/user-manager.md): Gestiona la creación, actualización y eliminación de usuarios.
  - [IUserRepository](./interfaces/i-user-repository.md): Interfaz para acceder a los datos de los usuarios.
  - [IPasswordHasher](./interfaces/i-password-hasher.md): Interfaz para hashear contraseñas.
  - [AuthenticationService](./services/authentication-service.md): Gestiona la autenticación de usuarios.
  - [IEmailUniquenessChecker](./interfaces/i-email-uniqueness-checker.md): Valida que el email del usuario sea único.
- **Managers**:
  - [UserManager](./services/user-manager.md): Gestiona la creación, actualización y eliminación de usuarios.
  - [AuthorizationManager](./services/authorization-manager.md): Gestiona la autorización de usuarios.
- **Policies**:
  - [IPasswordPolicy](./interfaces/i-password-policy.md): Define las políticas de contraseña.
- **Value Objects**:
  - [UserId](./value-objects/user-id.md) `Id`: Identificador único del usuario.
  - [PasswordHash](./value-objects/password-hash.md) `PasswordHash`: Hash de la contraseña del usuario.
  - [RefreshToken](../common/value-objects/token/token.md) `Token`: Token de actualización para el usuario.
  - [EmailAddress](../common/value-objects/email-address/email-address.md) `Email`: Dirección de correo electrónico del usuario.

## Interceptores EF Core

- `UserAuditInterceptor`: Intercepta las operaciones de creación, actualización y eliminación de usuarios en la base de datos para registrar los cambios de `Active` en la tabla `AuditRecord`.

## Ejemplos de Uso

La entidad `User` no puede ser instanciada fuera de un contexto de dominio. Para crear un nuevo usuario, se debe utilizar el método `CreateUserAsync` del service model `UserManager`.

```csharp
// Crear un nuevo usuario a partir de un identificador único.
var user = new User(
    userId: UserId.From(Guid.Parse("123e4567-e89b-12d3-a456-426614174000")),
    emailAddress: EmailAddress.From("john.doe@example.com"),
    passwordHash: PasswordHash.FromHashed("PasswordHash"),
    firstName: "John",
    LastName: "Doe",
    active: true,
    emailConfirmed: true)

// Crear un nuevo usuario con un identificador único aleatorio.
var user = new User(
    userId: UserId.create(),
    emailAddress: EmailAddress.From("john.doe@example.com"),
    passwordHash: PasswordHash.FromHashed("PasswordHash"),
    firstName: "John",
    LastName: "Doe",
    active: true,
    emailConfirmed: true)
```
