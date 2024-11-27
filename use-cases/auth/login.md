# Caso de Uso: Login de Usuario

## Objetivo

Permitir a los usuarios acceder a la aplicación mediante sus credenciales (email y contraseña).

## Actores

- Usuario no autenticado

## Flujo Principal

1. El usuario accede a la página de login
2. El sistema muestra el formulario con:
   - Campo para email
   - Campo para contraseña
   - Botón "Iniciar Sesión"
   - Enlace "¿Olvidaste tu contraseña?"
3. El usuario introduce sus credenciales
4. El sistema valida las credenciales
5. El sistema redirige al dashboard

## Flujos Alternativos

### Credenciales Inválidas

- El sistema muestra mensaje de error: "Email o contraseña incorrectos"
- El sistema mantiene el email introducido
- El sistema mantiene el campo de contraseña

### Usuario Bloqueado

- El sistema muestra mensaje: "Tu cuenta está bloqueada. Contacta con soporte"

### Usuario pendiente de confirmación

- Redirección a la página de confirmación de correo para enviar un nuevo código de confirmación

## Validaciones

Las validaciones siguen las reglas definidas en:

- [User](../../domain/users/user.md)

## Notas Técnicas

- Ruta: `/auth/login`
- API Endpoint: `POST /api/v1/auth/login`
- Redirección tras éxito: `returnUrl` si existe o `/dashboard` en caso contrario
- Tiempo máximo de sesión:
  - El token de acceso tiene una duración de 10 minutos
  - El token de actualización tiene una duración de 30 días

## Criterios de Aceptación

- [x] Permisos de acceso correctos
- [x] El formulario valida campos antes de enviar
- [x] Se muestran mensajes de error apropiados
- [x] Se almacena el access_token y refresh_token JWT en localStorage
- [x] Se redirige correctamente tras login exitoso
- [x] Tiempos de sesión correctos
- [x] El botón de inicio de sesión se deshabilita durante el proceso
