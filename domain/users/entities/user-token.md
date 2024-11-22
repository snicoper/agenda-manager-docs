# UserToken

- **Entity**: `UserToken`
- **Namespace**: `AgendaManager.Domain.Users.Entities`
- **Tipo**: Entidad de Dominio Sellada (sealed)
- **Herencia**: `AuditableEntity`

## Descripción General

La clase `UserToken` representa un token de seguridad temporal asociado a un usuario para operaciones que requieren verificación adicional, como la confirmación de email o el restablecimiento de contraseña. Esta entidad forma parte del agregado `User`.

Los tokens son de un solo uso y tienen un tiempo de vida limitado definido en su creación. Una vez utilizados o expirados, deben ser eliminados del sistema.

### Responsabilidades

- **Gestión de tokens**:
  - Almacenar y gestionar tokens específicos para diferentes acciones dentro del sistema.
  - Eliminar tokens después de su uso.
  - Eliminar tokens expirados.

- **Validación y seguridad**:
  - Validar la autenticidad de los tokens
  - Asegurar que los tokens solo se usen una vez
  - Validar el tipo correcto de token para cada operación
  - Asegurar que el token no haya expirado antes de su uso
  - Garantizar que un usuario no pueda tener múltiples tokens activos del mismo tipo

## Propiedades

| Propiedad          | Tipo              | Acceso           | Descripción                                                 |
|--------------------|-------------------|--------------------------------------------------------------------------------|
| `Id`               | `UserTokenId`     | get              | Identificador único del token.                              |
| `UserId`           | `UserId`          | get/private set  | Identificador único del usuario asociado al token.          |
| `Token`            | `Token`           | get/private set  | El token generado para la acción específica.                |
| `UserTokenType`    | `UserTokenType`   | get/private set  | Tipo de token, como `PasswordReset` o `EmailConfirmation`.  |
| `IsExpired`        | `bool`            | get/private set  | Indica si el token ha expirado.                             |

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

## Métodos

### Create

```csharp
internal static UserToken Create(UserTokenId id, UserId userId, Token token, UserTokenType type)
```

- **Descripción**: Crea una nueva instancia de `UserToken`.
- **Parámetros**:
  - `id`: Identificador único del token.
  - `userId`: Identificador único del usuario asociado al token.
  - `token`: El token generado para la acción específica.
  - `type`: Tipo de token, como `PasswordReset` o `EmailConfirmation`.
- **Retorno**: Un nuevo `UserToken`.

### Consume

```csharp
internal Result Consume(string tokenValue)
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

## Ejemplos de Uso
