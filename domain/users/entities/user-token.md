# UserToken

- **Entity**: `UserToken`
- **Namespace**: `AgendaManager.Domain.Users.Entities`
- **Tipo**: Entidad de Dominio Sellada (sealed)
- **Herencia**: `AuditableEntity`

## Descripción

La clase `UserToken` representa un token de seguridad temporal asociado a un usuario para operaciones que requieren verificación adicional, como la confirmación de email o el restablecimiento de contraseña. Esta entidad forma parte del agregado `User`.

Los tokens son de un solo uso y tienen un tiempo de vida limitado definido en su creación. Una vez utilizados o expirados, deben ser eliminados del sistema.

### Responsabilidades

- **Gestión de tokens**:
  - Almacenar y gestionar tokens específicos para diferentes acciones dentro del sistema.
  - Eliminar tokens después de su uso.
  - Eliminar tokens expirados.

- **Eventos de Dominio**:
  - Disparar eventos de dominio cuando se realicen cambios significativos en el user token.

- **Validación y seguridad**:
  - Validar la autenticidad de los tokens
  - Asegurar que los tokens solo se usen una vez
  - Validar el tipo correcto de token para cada operación
  - Asegurar que el token no haya expirado antes de su uso
  - Garantizar que un usuario no pueda tener múltiples tokens activos del mismo tipo

## Invariantes

- `Id` no puede ser `null` en ningún momento
- `UserId` no puede ser `null` y debe referenciar a un usuario existente
- `Token` no puede ser `null` y debe ser válido según las reglas del ValueObject `Token`
- `Type` no puede ser `UserTokenType.None`

## Reglas de negocio

- Un usuario solo puede tener un token activo por tipo en un momento dado
- Los tokens de reseteo de contraseña tienen una validez máxima de 1 hora
- Los tokens de verificación de email tienen una validez máxima de 7 días
- Un token no puede ser utilizado después de su expiración
- Un token no puede ser utilizado más de una vez
- Al generar un nuevo token del mismo tipo, el token anterior (si existe) debe ser eliminado

## Propiedades

| Propiedad          | Tipo              | Acceso           | Descripción                                                 |
|--------------------|-------------------|--------------------------------------------------------------------------------|
| `Id`               | `UserTokenId`     | get              | Identificador único del token.                              |
| `UserId`           | `UserId`          | get/private set  | Identificador único del usuario asociado al token.          |
| `Token`            | `Token`           | get/private set  | El token generado para la acción específica.                |
| `UserTokenType`    | `UserTokenType`   | get/private set  | Tipo de token, como `PasswordReset` o `EmailConfirmation`.  |
| `IsExpired`        | `bool`            | get/private set  | Indica si el token ha expirado.                             |

## Métodos

### Create

```csharp
public static UserToken Create(UserTokenId id, UserId userId, Token token, UserTokenType type)
```

- **Descripción**: Crea una nueva instancia de `UserToken`.
- **Parámetros**:
  - `id`: Identificador único del token.
  - `userId`: Identificador único del usuario asociado al token.
  - `token`: El token generado para la acción específica.
  - `type`: Tipo de token, como `PasswordReset` o `EmailConfirmation`.
- **Eventos**: `UserTokenCreatedDomainEvent` disparado cuando se crea un nuevo token.
- **Retorno**: Un nuevo `UserToken`.

### CreateEmailConfirmation

```csharp
public static Result<UserToken> CreateEmailConfirmation(UserId userId, TimeSpan? validityPeriod = null)
```

- **Descripción**: Crea un nuevo token de confirmación de email.
- **Parámetros**:
  - `userId`: Identificador único del usuario asociado al token.
  - `validityPeriod`: Duración del token. Por defecto, 7 días.
- **Retorno**: Un nuevo `Result<UserToken>` con el nuevo token de confirmación de email.

### CreatePasswordReset

```csharp
public static Result<UserToken> CreatePasswordReset(UserId userId, TimeSpan? validityPeriod = null)
```

- **Descripción**: Crea un nuevo token de restablecimiento de contraseña.
- **Parámetros**:
  - `userId`: Identificador único del usuario asociado al token.
  - `validityPeriod`: Duración del token. Por defecto, 1 hora.
- **Retorno**: Un nuevo `Result<UserToken>` con el nuevo token de restablecimiento de contraseña.

### Consume

```csharp
public Result Consume(string tokenValue)
```

- **Descripción**: Consume el token para la operación correspondiente.
- **Parámetros**:
  - `tokenValue`: El valor del token a consumir.
- **Retorno**: Un `Result` que indica si la operación fue exitosa o no.

## Estado y Transiciones

El token pasa por los siguientes estados:

1. **Creación**: Se crea con un tipo específico y tiempo de vida
2. **Uso**: Se consume para la operación correspondiente
3. **Eliminación**: Se elimina después de:
   - Ser utilizado
   - Expira
   - El usuario solicita un nuevo token del mismo tipo

## Dependencias

### Directas

- **Entidades Base**:
  - `AuditableEntity`: Base class que proporciona capacidades de auditoría
    - Hereda gestión de eventos de dominio (`Entity`)

### Entidades

- **No Aplica**

### Servicios

- `IUserTokenRepository`: Repositorio para acceder a los tokens de usuario.

### Managers

- **No Aplica**

### Políticas

- **No Aplica**

### Value Objects

- `UserTokenId`: Identificador único del token.
- `UserId`: Identificador único del usuario asociado al token.
- `Token`: Token específico para la acción.
- `UserTokenType`: Tipo de token para diferentes acciones (`EmailVerification`, `PasswordReset`).

## Interceptores EF Core

- **No Aplica**

## Ejemplos de Uso

```csharp
// Usando el método Create original
var passwordResetToken = UserToken.Create(
    id: UserTokenId.Create(),
    userId: user.Id,
    token: Token.Generate(TimeSpan.FromHours(1)),
    type: UserTokenType.PasswordReset);

// Usando los nuevos métodos factory
var passwordResetResult = UserToken.CreatePasswordReset(
    userId: user.Id,
    validityPeriod: TimeSpan.FromHours(2));

if (passwordResetResult.IsSuccess)
{
    var token = passwordResetResult.Value;
    // Usar el token...
}

// Ejemplo de consumo de token
var consumeResult = token.Consume("token-value-here");
if (consumeResult.IsSuccess)
{
    // Token consumido correctamente
}
else
{
    // Manejar el error (token expirado o inválido)
}
```
