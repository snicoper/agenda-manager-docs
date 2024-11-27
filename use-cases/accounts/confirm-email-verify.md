# Caso de Uso: Verificar Correo de Confirmación

## ToDo List

## Objetivo

Verificar el correo electrónico de un usuario.

## Actores

- Usuario no autenticado

## Flujo Principal

- El usuario accede a la página para verificar el correo electrónico.
- En la uri `token?={token}` se obtiene el email del usuario.
- Al acceder a la página, el sistema verifica que el token sea válido y no esté expirado.
- El sistema elimina el token de la base de datos en caso de que sea válido.
- El sistema muestra un mensaje de éxito y un botón "Iniciar sesión".

## Respuesta al Usuario

### Token válido

- El sistema muestra un mensaje de éxito y un botón "Iniciar sesión".

### Token inválido

- El sistema muestra un mensaje de error: "La confirmación de correo electrónico ha fallado o expiro."
  - Cuanto el token no existe
  - Cuando el usuario esta inactivo
  - Cuando el usuario ya ha verificado su cuenta
  - Cuando el token ha expirado

## Validaciones

## Notas Técnicas

- Ruta: `/accounts/confirm-email-verify?token={token}`
- API Endpoint: `POST /api/v1/accounts/confirm-email-verify`
  - **Argumentos**:
  - `token`: Token de verificación
- Muestra un mensaje de de éxito o error

## Criterios de Aceptación

- [x] El sistema valida que el token que exista
- [x] El sistema valida que el usuario esté activo
- [x] El sistema valida que el usuario no ha verificado su cuenta
- [x] El sistema valida que el token no ha expirado
- [x] El sistema elimina el token de la base de datos en caso de que sea válido
- [x] El sistema muestra un mensaje de éxito y un botón "Iniciar sesión"
- [x] El sistema muestra un mensaje de error en caso de fallar la verificación
