# Caso de Uso: Reenviar Correo de Confirmación

## ToDo List

## Objetivo

Reenvía el correo de confirmación de cuenta.

Solo se permite reenviar un código si el usuario esta activo y no ha sido verificado.

## Actores

- Usuario no autenticado

## Flujo Principal

1. El usuario accede a la página para reenviar el correo de confirmación.
2. El email se obtiene de la uri `email?={email}` el email del usuario.
3. El sistema muestra un mensaje La cuenta con el email {email} requiere ser verificada.
4. El sistema muestra un mensaje de confirmación y un botón "Reenviar código".
5. El usuario hace clic en el botón "Reenviar código".
6. El botón de reenviar se deshabilita durante el proceso.
7. El sistema valida que el email existe en la base de datos.
8. El sistema envía un correo electrónico con un enlace para confirmar la cuenta.
9. El sistema muestra un toast de éxito **Se ha enviado un nuevo código de verificación a su correo.**
10. El botón de "Reenviar código" cambia el texto a "Código enviado con éxito" y otro botón "Iniciar sesión".
11. El botón de **Código enviado con éxito** queda deshabilitado.

## Respuesta al Usuario

### Token enviado

- El sistema muestra un toast: "Se ha enviado un nuevo código de verificación a su correo."

### Email no encontrado

- El sistema muestra un toast error: "No se ha podido enviar el código de verificación."

### Usuario ya verificado

- El sistema muestra un toast error: "No se ha podido enviar el código de verificación."

### Usuario inactivo

- El sistema muestra un toast error: "No se ha podido enviar el código de verificación."

## Validaciones

- Verifica que el email exista en la base de datos
- Verifica que el usuario está activo
- Verifica que el usuario no está verificado

## Notas Técnicas

- Ruta: `/accounts/confirm-email-resent`
- API Endpoint: `POST /api/v1/accounts/confirm-email-resent`
  - **Argumentos**:
  - `email`
- Muestra un mensaje de de éxito o error

## Criterios de Aceptación

- [x] Se muestran mensajes de error o éxito apropiados
- [x] El botón de reenviar se deshabilita durante el proceso
- [x] En caso de éxito, el botón de "Reenviar código" cambia el texto a "Código enviado con éxito" y otro botón "Iniciar sesión".
