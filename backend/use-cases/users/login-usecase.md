# Use case Login

Autenticación de usuarios.

El usuario ha de poder autenticarse con su `Email` y `Password`. En caso de éxito deberá recibir un `TokenResult` con los datos del token.

En caso de error deberá devolver un `Result` con el código de error correspondiente.

En caso de que las credenciales no sean válidas deberá devolver un error `Error.Conflict` con un simple mensaje de error.

Crear endpoint `POST` que acepte un **request** con los parámetros `Email` y `Password` para la validación del usuario.

```cs
public record LoginRequest(string Email, string Password);
```

## LoginCommand

El LoginCommand ha de aceptar 2 parámetros, el `Email` y `Password`. y devolver un `TokenResult`

```cs
public record LoginCommand(string Email, string Password);
```

```cs
public record TokenResult(string AccessToken, string RefreshToken, DateTime Expires);
```

## Reglas de validación

- El campo `Email` debe ser un email valido y no deberá estar vacío.
- El campo `Password` debe ser un string no vacío.

Se deberá obtener el usuario por el email dado en el request y comprobar que exista.

Si no existe, deberá lanzar un error `Error.Conflict` con el mensaje `Invalid credentials`.

Si existe, se deberá comprobar que la contraseña sea correcta `User.VerifyPassword(string password, IPasswordHasher passwordHasher)`, en caso de que no sea correcta deberá lanzar un error `Error.Conflict` con el mensaje `Invalid credentials`.

El `IPasswordHasher` se requiere inyectar desde el `LoginCommandHandler` para la verificación de la contraseña.

Se deberá comprobar que el usuario este `Active` o deberá lanzar un error `Error.Conflict` con el mensaje `User is not active`.

## Generar tokens

Se deberá generar un `AccessToken`, `RefreshToken` y `Expires` con `IJwtTokenGenerator.GenerateAccessTokenAsync(User user)` y en caso de éxito, deberá actualizar al `User.RefreshToken` y devolver un `TokenResult`.

## Tests

- `Login_ShouldReturnSuccess_WithValidCredentials`
- `Login_ShouldReturnValidationError_WithInvalidEmail`
- `Login_ShouldReturnValidationError_WithEmptyEmail`
- `Login_ShouldReturnValidationError_WithEmptyPassword`
- `Login_ShouldReturnConflict_WithInvalidEmailCredentials`
- `Login_ShouldReturnConflict_WithInvalidPasswordCredentials`
