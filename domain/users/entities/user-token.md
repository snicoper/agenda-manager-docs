# UserToken

- **Entity**: `UserToken`

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

| Propiedad          | Tipo              | Descripción                                                   |
|--------------------|-------------------|---------------------------------------------------------------|
| `Id`               | `UserTokenId`     | Identificador único del token.                                |
| `UserId`           | `UserId`          | Identificador único del usuario asociado al token.            |
| `Token`            | `Token`           | El token generado para la acción específica.                  |
| `UserTokenType`    | `UserTokenType`   | Tipo de token, como `PasswordReset` o `EmailConfirmation`.    |
| `IsExpired`        | `bool`            | Indica si el token ha expirado.                              |

## Relaciones

- Un `UserToken` tiene un **ValueObject** `UserTokenId` único que lo identifica.
- Un `UserToken` tiene un **ValueObject** `UserId` que identifica al usuario asociado.
- Un `UserToken` tiene un **ValueObject** `Token` que representa el token específico.
- Un `UserToken` tiene un **Enum** `UserTokenType` que indica el tipo de token.

## Métodos

```csharp
public static UserToken Create(UserTokenId id, UserId userId, Token token, UserTokenType type)
```

- Crea una nueva instancia de `UserToken`.
- `id`: Identificador único del token.
- `userId`: Identificador único del usuario asociado al token.
- `token`: El token generado para la acción específica.
- `type`: Tipo de token, como `PasswordReset` o `EmailConfirmation`.
- Lanza el evento `UserTokenCreated` con el nuevo token.
- Devuelve un nuevo `UserToken`.

```csharp
public static Result<UserToken> CreateEmailConfirmation(UserId userId, TimeSpan? validityPeriod = null)
```

- Crea un nuevo token de confirmación de email.
- `userId`: Identificador único del usuario asociado al token.
- `validityPeriod`: Duración del token. Por defecto, 7 días.
- Return un nuevo `Result<UserToken>` con el nuevo token de confirmación de email.

```csharp
public static Result<UserToken> CreatePasswordReset(UserId userId, TimeSpan? validityPeriod = null)
```

- Crea un nuevo token de restablecimiento de contraseña.
- `userId`: Identificador único del usuario asociado al token.
- `validityPeriod`: Duración del token. Por defecto, 1 hora.
- Return un nuevo `Result<UserToken>` con el nuevo token de restablecimiento de contraseña.

```csharp
public Result Consume(string tokenValue)
```

- Consume el token para la operación correspondiente.
- `tokenValue`: El valor del token a consumir.
- Devuelve un `Result` success en caso de éxito o un error si el token no es válido o ha expirado.

## Estado y Transiciones

El token pasa por los siguientes estados:

1. **Creación**: Se crea con un tipo específico y tiempo de vida
2. **Uso**: Se consume para la operación correspondiente
3. **Eliminación**: Se elimina después de:
   - Ser utilizado
   - Expira
   - El usuario solicita un nuevo token del mismo tipo

## Dependencias

- **Entities**:
- **Servicios**:
  - [IUserTokenRepository](./interfaces/i-user-token-repository.md): Repositorio para acceder a los tokens de usuario.
- **Managers**:
- **Políticas**:
- **Value Objects**:
  - [UserTokenId](./value-objects/user-token-id.md): Identificador único del token.
  - [UserId](../users/value-objects/user-id.md): Identificador único del usuario asociado al token.
  - [Token](./value-objects/token.md): Token específico para la acción.
- **Enums**:
  - [UserTokenType](./enums/user-token-type.md): Tipo de token para diferentes acciones (`EmailVerification`, `PasswordReset`).

## Interceptores EF Core

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
