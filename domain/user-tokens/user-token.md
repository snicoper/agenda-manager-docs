# class: UserToken

## Descripción

Una clase `UserToken` representa un token de seguridad temporal asociado a un usuario para operaciones que requieren verificación adicional, como la confirmación de email o el restablecimiento de contraseña. Esta entidad forma parte del agregado `User`.

### Responsabilidades

- **Gestión de Tokens**:
  - Almacenar y gestionar tokens específicos para diferentes acciones dentro del sistema.

- **Validación de Tokens**:
  - Validar la validez de los tokens antes de permitir ciertas acciones dentro del sistema.
  - Gestionar la expiración de los tokens.

- **Eventos de Dominio**:
  - Disparar eventos de dominio cuando se crean los tokens.

## Propiedades

| Propiedad           | Tipo                      | Descripción                                                |
|---------------------|---------------------------|------------------------------------------------------------|
| `Id`                | `UserTokenId`             | Identificador único del usuario.                           |
| `UserId`            | `UserId`                  | Identificador único del usuario asociado al token.         |
| `User`              | `User`                    | El usuario asociado al token.                              |
| `Token`             | `Token`                   | El token generado para la acción específica.               |
| `UserTokenType`     | `UserTokenType`           | Tipo de token, como `PasswordReset` o `EmailConfirmation`. |
| `ExpiryDate`        | `DateTimeOffset`          | Fecha de expiración del token.                             |

## Asociaciones

- Un `UserToken` tiene un **ValueObject** `UserTokenId` único que lo identifica.
- Un `UserToken` tiene un **ValueObject** `UserId` que identifica al usuario asociado.
- Un `UserToken` tiene un **ValueObject** `Token` que representa el token específico.
- Un `UserToken` tiene un **Enum** `UserTokenType` que indica el tipo de token.

## Métodos

## Invariantes

- `Id` debe ser único en toda la aplicación.
- `UserId` debe ser un `UserId` válido.
- `Token` no puede ser null en ningún momento y debe ser un formato válido.
- `UserTokenType` debe ser un valor válido del enum `UserTokenType`.
- `ExpiryDate` debe ser una fecha futura.

## Reglas de negocio

- **Unicidad de identidad**:
  - El `Id` debe ser único en toda la aplicación.
  - El `Token` debe ser único en toda la aplicación.

- **Validación de UserId**:
  - El `UserId` debe ser un `UserId` válido y debe existir en la base de datos.

- **Validación de Token**:
  - El `Token` no puede ser null en ningún momento y debe ser un formato válido.

- **Validación de UserTokenType**:
  - El `UserTokenType` debe ser un valor válido del enum `UserTokenType`.

- **Validación de ExpiryDate**:
  - La fecha de expiración del token debe ser una fecha futura.

- **Verificación de un token**:
  - Después de ser verificado, el token debe ser eliminado de la base de datos.

## Estado y Transiciones

## Dependencias

- **Entidades**:
  - [User](./user.md): El usuario asociado al token.
- **Services**:
  - [IUserTokenRepository](./interfaces/i-user-token-repository.md): Repositorio para acceder a los tokens de usuario.
- **Managers**:
- **Policies**:
- **Value Objects**:
  - [UserTokenId](./value-objects/user-token-id.md): Identificador único del usuario.
  - [UserId](../users/value-objects/user-id.md): Identificador único del usuario asociado al token.
  - [Token](./value-objects/token.md): Token específico para la acción.
- **Enums**:
  - [UserTokenType](./enums/user-token-type.md): Tipo de token para diferentes acciones (`EmailVerification`, `PasswordReset`).

## Interceptores EF Core

## Ejemplos de Uso
