# Caso de Uso: Recuperación de Contraseña

## ToDo List

- [ ] Mejorar diseño de la página

## Objetivo

Permitir a los usuarios recuperar su contraseña si la han olvidado.

## Actores

- Usuario no autenticado

## Flujo Principal

- El usuario accede a la página de recuperación de contraseña
- El sistema muestra el formulario con:
  - Campo para email
  - Botón "Recuperar Contraseña"
  - Botón con enlace a "Inicia Sesión"
- El usuario introduce su email
- El sistema valida el email
- El sistema notifica en base al resultado la validación
- El sistema envía un correo electrónico con un enlace para recuperar la contraseña

## Respuesta al Usuario

### Email encontrado

- El sistema muestra mensaje: "Se ha enviado un correo electrónico con un enlace para recuperar la contraseña"

### Email no encontrado

- El sistema muestra mensaje de error: "El correo electrónico no está registrado"

### Usuario no activo

- El sistema muestra mensaje: "Su cuenta no está activa. Contacte con el administrador"
- Se proporciona enlace al formulario de contacto

### Usuario pendiente de confirmación

- El sistema muestra mensaje: "Su cuenta esta pendiente de confirmación. Revise su correo"

## Validaciones

- El email debe ser un email válido
- Verifica que el email existe en la base de datos
- Verifica que la cuenta del usuario está activa
- Verifica que la cuanta del usuario está pendiente de confirmación

## Notas Técnicas

- Ruta: `/accounts/recovery-password`
- API Endpoint: `POST /api/v1/accounts/recovery-password`
- Muestra un mensaje de de éxito o error

## Criterios de Aceptación

- [x] El formulario valida campos antes de enviar
- [x] Se muestran mensajes de error o éxito apropiados
- [x] El botón de recuperación se deshabilita durante el proceso
- [x] El usuario puede volver a la página de inicio de sesión
