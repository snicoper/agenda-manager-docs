# Caso de Uso: Confirmar Recuperación de Contraseña

## Objetivo

Permitir a los usuarios confirmar su contraseña si la han olvidado.

## Actores

- Usuario no autenticado

## Flujo Principal

- El usuario accede a la página de recuperación de contraseña desde el correo electrónico
- El sistema muestra el formulario con:
  - Campo para insertar la nueva contraseña
  - Campo para repetir la nueva contraseña
  - Botón "Restablecer"
  - Botón con enlace a "Inicia Sesión"
- El usuario introduce la nueva contraseña y la repite
- El sistema valida la nueva contraseña
- El sistema notifica al usuario el resultado de la validación
- El sistema elimina el token de la base de datos si existe, sea cual sea el resultado

## Flujos Alternativos

### Token inválido o expirado

- El sistema muestra mensaje de error: "Token inválido o expirado"

## Validaciones

- El token debe ser un token válido y no expirado
- Verifica que la contraseña cumple con las reglas de seguridad

## Notas Técnicas

- Ruta: `/accounts/confirm-recovery-password`
- API Endpoint: `POST /api/v1/accounts/confirm-recovery-password`
- Muestra un toast de éxito y redirige a la página de login en caso de éxito

## Criterios de Aceptación

- [x] El formulario valida campos antes de enviar
- [x] El botón de restableces se deshabilita durante el proceso
- [x] Se muestran mensajes de error apropiados
- [x] Se redirige correctamente tras validación del token
