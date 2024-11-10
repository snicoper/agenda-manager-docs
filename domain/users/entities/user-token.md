¡No hay problema! Aquí tienes la documentación revisada en español:

---

# Clase: UserToken

## Descripción

La clase `UserToken` representa un token de seguridad temporal asociado a un usuario para operaciones que requieren verificación adicional, como la confirmación de email o el restablecimiento de contraseña. Esta entidad forma parte del agregado `User`.

### Responsabilidades

- **Gestión de tokens**:
  - Almacenar y gestionar tokens específicos para diferentes acciones dentro del sistema.
  - Eliminar tokens después de su uso.
  - Eliminar tokens expirados.

- **Validación y seguridad**:
  - Validar la autenticidad de los tokens.
  - Asegurar que los tokens solo se usen una vez.
  - Validar el tipo correcto de token para cada operación.

- **Eventos de Dominio**:
  - Disparar eventos de dominio cuando se crean los tokens.

## Propiedades

| Propiedad          | Tipo              | Descripción                                                   |
|--------------------|-------------------|---------------------------------------------------------------|
| `Id`               | `UserTokenId`     | Identificador único del token.                                |
| `UserId`           | `UserId`          | Identificador único del usuario asociado al token.            |
| `Token`            | `Token`           | El token generado para la acción específica.                  |
| `UserTokenType`    | `UserTokenType`   | Tipo de token, como `PasswordReset` o `EmailConfirmation`.    |

## Asociaciones

- Un `UserToken` tiene un **ValueObject** `UserTokenId` único que lo identifica.
- Un `UserToken` tiene un **ValueObject** `UserId` que identifica al usuario asociado.
- Un `UserToken` tiene un **ValueObject** `Token` que representa el token específico.
- Un `UserToken` tiene un **Enum** `UserTokenType` que indica el tipo de token.

## Métodos

## Invariantes

- `Id` no puede ser `null` en ningún momento.
- `UserId` debe ser un `UserId` válido.
- `Token` no puede ser `null` en ningún momento y debe ser un formato válido.
- `UserTokenType` debe ser un valor válido del enum `UserTokenType`.
- `ExpiryAt` a la hora de creación debe ser una fecha futura.

## Reglas de negocio

- **Unicidad de identidad**:
  - El `Id` debe ser único en toda la aplicación.
  - El `Token` debe ser único en toda la aplicación.

- **Token**:
  - El `Token` no puede ser `null` en ningún momento y debe ser criptográficamente seguro.
  - La longitud mínima del token debe ser de 32 caracteres.
  - La longitud máxima del token debe ser de 128 caracteres.
  - Cada tipo de token tiene un tiempo de expiración específico:
    - `EmailVerification`: 24 horas
    - `PasswordReset`: 1 hora
  - Un token expirado no puede ser utilizado.
  - Un token utilizado no puede ser utilizado nuevamente.
  - Al usar un token incorrecto más de 3 veces, se deben bloquear los intentos por 15 minutos.

- **Validación de UserTokenType**:
  - El `UserTokenType` debe ser un valor válido del enum `UserTokenType`.
  - Los tipos de `UserTokenType` dentro del enum deben ser únicos e incluir `EmailVerification` y `PasswordReset`.

- **Validación de ExpiryAt**:
  - La fecha de expiración del token debe ser una fecha futura.

## Estado y Transiciones

## Dependencias

- **Entidades**:
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
var user = await userRepository.GetByEmailAsync(email, cancellationToken);

if (user is null)
{
    return UserErrors.UserNotFound;
}

var userToken = UserToken.Create(
    id: UserTokenId.Create(),
    userId: user.Id,
    token: Token.Generate(TimeSpan.FromDays(1)),
    type: UserTokenType.EmailConfirmation);

user.AddUserToken(userToken);

userRepository.Update(user);
await unitOfWork.SaveChangesAsync(cancellationToken);
```
