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

- **Gestión de Información Personal**:
  - Almacenar y validar la dirección de correo electrónico.
  - Gestionar los datos opcionales como `FirstName` y `LastName`.

- **Gestión de Estado**:
  - Manejar el estado de confirmación del email (`IsEmailConfirmed`).
  - Controlar el estado de actividad del usuario (`IsActive`).

- **Roles**:
  - Añadir y remover roles asociados al usuario.
  - Proveer una colección de roles asignados al usuario.

- **Eventos de Dominio**:
  - Disparar eventos de dominio cuando se realicen cambios significativos en el usuario.

- **Validación**:
  - Asegurarse de que `FirstName` y `LastName` no exceden la longitud permitida.

## Invariantes

- `Id` no puede ser `null` en ningún momento
- `Email` no puede ser `null` en ningún momento y debe ser un formato de email válido
- `PasswordHash` no puede ser `null` o vacío en ningún momento
- `IsEmailConfirmed` y `IsActive` debe ser un valor booleano (`true` o `false`) consistente
- `IsActive` debe ser un valor booleano (`true` o `false`) consistente
- `FirstName` puede ser `null`, pero si está presente, no debe exceder los 256 caracteres
- `LastName` puede ser `null`, pero si está presente, no debe exceder los 256 caracteres
- `Roles` debe ser una colección de roles que puede ser vacía pero nunca `null`
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

- **Tokens de Actualización**:
  - `Token` debe ser único y gestionado de manera segura, asegurando que solo un token activo esté asociado con el usuario en cualquier momento.

- **Actualización de Información Personal**:
  - `FirstName` y `LastName` pueden actualizarse, pero deben respetar las limitaciones de longitud y no ser nulos si están presentes.
  - El `Email` puede ser actualizado y debe seguir siendo único en la aplicación después de la actualización.

## Propiedades

| Propiedad           | Tipo                        |Acceso           | Descripción                                                |
|---------------------|-----------------------------|-----------------|------------------------------------------------------------|
| `UserId`            | `UserId`                    | get             | Identificador único del usuario.                           |
| `PasswordHash`      | `PasswordHash`              | get/private set | Hash de la contraseña del usuario.                         |
| `Email`             | `EmailAddress`              | get/private set | Dirección de correo electrónico única del usuario.         |
| `IsEmailConfirmed`  | `bool`                      | get/private set | Estado de confirmación del email del usuario.              |
| `FirstName`         | `string?`                   | get/private set | Nombre del usuario, puede ser `null`.                      |
| `LastName`          | `string?`                   | get/private set | Apellido del usuario, puede ser `null`.                    |
| `IsActive`          | `bool`                      | get/private set | Estado de actividad del usuario.                           |
| `RefreshToken`      | `Token?`                    | get/private set | Token de refresco de autorización del usuario.             |
| `Roles`             | `IReadOnlyCollection<Role>` | get/private set | Lista de roles asociados al usuario.                       |
| `Tokens`            | `IReadOnlyCollection<Token>`| get/private set | Lista de tokens asociados al usuario.                      |

## Métodos

### UpdatePassword

```csharp
public Result UpdatePassword(PasswordHash newPasswordHash)
```

- **Descripción**: Actualiza la contraseña del usuario.
- **Parámetros**:
  - `newPasswordHash`: Hash de la nueva contraseña del usuario.
- **Eventos**:
  - `UserPasswordUpdatedDomainEvent`: Se dispara cuando se actualiza la contraseña del usuario.
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
  - `UserEmailUpdatedDomainEvent`: Se dispara cuando se actualiza la dirección de correo electrónico del usuario.
**Retorno**: `Result` Un resultado que indica si la actualización fue exitosa o no.

### UpdateRefreshToken

```csharp
public void UpdateRefreshToken(Token token)
```

- **Descripción**: Actualiza el token de refresco del usuario.
- **Parámetros**:
  - `token`: Token de refresco del usuario.
- **Eventos**:
  - `UserRefreshTokenUpdatedDomainEvent`: Se dispara cuando se actualiza el token de refresco del usuario.

### SetEmailConfirmed

```csharp
public void SetEmailConfirmed()
```

- **Descripción**: Marca el correo electrónico del usuario como confirmado.
- **Eventos**:
  - `UserEmailConfirmedDomainEvent`: Se dispara cuando se marca el correo electrónico del usuario como confirmado.

### Activate

```csharp
public void Activate()
```

- **Descripción**: Activa el usuario.
- **Eventos**:
  - `UserActivatedDomainEvent`: Se dispara cuando se activa el usuario.

### Deactivate

```csharp
public void Deactivate()
```

- **Descripción**: Desactiva el usuario.
- **Eventos**:
  - `UserDeactivatedDomainEvent`: Se dispara cuando se desactiva el usuario.

### AddUserToken

```csharp
public void AddUserToken(Token token)
```

- **Descripción**: Agrega un token de usuario.
- **Parámetros**:
  - `token`: Token de usuario.
- **Eventos**:
  - `UserTokenAddedDomainEvent`: Se dispara cuando se agrega un token de usuario.

### Create

```csharp
internal static User Create(
    UserId userId,
    EmailAddress email,
    PasswordHash passwordHash,
    string? firstName,
    string? lastName,
    bool isActive = true,
    bool emailConfirmed = false)
```

- **Descripción**: Crea una nueva instancia de `User`.
- **Parámetros**:
  - `userId`: Identificador único del usuario.
  - `email`: Dirección de correo electrónico única del usuario.
  - `passwordHash`: Hash de la contraseña del usuario.
  - `firstName`: Nombre del usuario, puede ser `null`.
  - `lastName`: Apellido del usuario, puede ser `null`.
  - `isActive`: Estado de
- **Eventos**:
  - `UserCreatedDomainEvent`: Se dispara cuando se crea un nuevo usuario.
- **Retorno**: Una nueva instancia de `User`.

### Update

```csharp
internal void Update(string? firstName, string? lastName)
```

- **Descripción**: Actualiza el nombre y apellido del usuario.
- **Parámetros**:
  - `firstName`: Nuevo nombre del usuario.
  - `lastName`: Nuevo apellido del usuario.
**Eventos**:
  - `UserUpdatedDomainEvent`: Se dispara cuando se actual

### AddRole

```csharp
public void AddRole(Role role)
```

- **Descripción**: Agrega un rol al usuario.
- **Parámetros**:
  - `role`: Rol a agregar al usuario.
- **Eventos**:
  - `UserRoleAddedDomainEvent`: Se dispara cuando se agrega un rol al usuario.

### RemoveRole

```csharp
public void RemoveRole(Role role)
```

- **Descripción**: Elimina un rol del usuario.
- **Parámetros**:
  - `role`: Rol a eliminar del usuario.
- **Eventos**:
  - `UserRoleRemovedDomainEvent`: Se dispara cuando se elimina un rol del usuario.

### GuardAgainstInvalidFirstName

```csharp
private static void GuardAgainstInvalidFirstName(string? firstName)
```

- **Descripción**: Verifica que el nombre del usuario sea válido.
- **Parámetros**:
  - `firstName`: Nombre del usuario.
- **Excepciones**:
  - `UserDomainException`: Se lanza si el nombre del usuario es inválido.

### GuardAgainstInvalidLastName

```csharp
private static void GuardAgainstInvalidLastName(string? lastName)
```

- **Descripción**: Verifica que el apellido del usuario sea válido.
- **Parámetros**:
  - `lastName`: Apellido del usuario.
- **Excepciones**:
  - `UserDomainException`: Se lanza si el apellido del usuario es inválido.

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

### Interfaces

- `UserManager`: Gestiona la creación, actualización y eliminación de usuarios.
- `IUserRepository`: Interfaz para el repositorio de usuarios.
- `IPasswordHasher`: Interfaz para el servicio de hash de contraseñas.
- `AuthenticationService`: Servicio de autenticación.
- `IEmailUniquenessChecker`: Interfaz para verificar la unicidad del email.

### Managers

- `UserManager`: Gestiona la creación, actualización y eliminación de usuarios.
- `AuthorizationManager`: Gestiona la autorización de usuarios.

### Policies

- `PasswordPolicy`: Define las políticas de contraseña.

### Value Objects

- `UserId`: Identificador único del usuario.
- `EmailAddress`: Dirección de correo electrónico del usuario.
- `PasswordHash`: Hash de la contraseña del usuario.
- `RefreshToken`: Token de actualización del usuario.

## Eventos de Dominio

- `UserCreatedDomainEvent`: Se lanza cuando se crea un nuevo usuario.
- `UserUpdatedDomainEvent`: Se lanza cuando se actualiza un usuario.
- `UserPasswordUpdatedDomainEvent`: Se lanza cuando se actualiza la contraseña de un usuario.
- `UserEmailUpdatedDomainEvent`: Se lanza cuando se actualiza la dirección de correo electrónico de un usuario.
- `UserRefreshTokenUpdatedDomainEvent`: Se lanza cuando se actualiza el token de actualización de un usuario.
- `UserEmailConfirmedDomainEvent`: Se lanza cuando se confirma la dirección de correo electrónico de un usuario.
- `UserActivatedDomainEvent`: Se lanza cuando se activa un usuario.
- `UserDeactivatedDomainEvent`: Se lanza cuando se desactiva un usuario.
- `UserTokenAddedDomainEvent`: Se lanza cuando se agrega un token a un usuario.
- `UserTokenRemovedDomainEvent`: Se lanza cuando se elimina un token de un usuario.
- `UserRoleAddedDomainEvent`: Se lanza cuando se agrega un rol a un usuario.
- `UserRoleRemovedDomainEvent`: Se lanza cuando se elimina un rol de un usuario.

## Receptores de Eventos

### Users Application Event Handlers

#### Audit Handlers

- `AuditUserActivatedDomainEventHandler` ->  `UserActivatedDomainEvent`
  - Registra el cambio de `IsActive` en la tabla `AuditRecord`
  - Registra: `userId`, `oldValue: false`, `newValue: true`

- `AuditUserDeactivatedDomainEventHandler` ->  `UserDeactivatedDomainEvent`
  - Registra el cambio de `IsActive` en la tabla `AuditRecord`
  - Registra: `userId`, `oldValue: true`, `newValue: false`

## Comentarios adicionales

La entidad `User` no puede ser instanciada fuera de un contexto de dominio. Para crear un nuevo usuario, se debe utilizar el método `CreateUserAsync` del service model `UserManager`.

## Ejemplos de Uso

```csharp
// Crear un nuevo usuario a partir de un identificador único.
var user = new User(
    userId: UserId.From(Guid.Parse("123e4567-e89b-12d3-a456-426614174000")),
    emailAddress: EmailAddress.From("john.doe@example.com"),
    passwordHash: PasswordHash.FromHashed("PasswordHash"),
    firstName: "John",
    LastName: "Doe",
    isActive: true,
    emailConfirmed: true)

// Crear un nuevo usuario con un identificador único aleatorio.
var user = new User(
    userId: UserId.create(),
    emailAddress: EmailAddress.From("john.doe@example.com"),
    passwordHash: PasswordHash.FromHashed("PasswordHash"),
    firstName: "John",
    LastName: "Doe",
    isActive: true,
    emailConfirmed: true)
```
