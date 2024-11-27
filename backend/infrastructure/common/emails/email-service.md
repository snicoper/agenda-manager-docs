# EmailService

- **Aggregate Root**: `EmailService`
- **Namespace**: `AgendaManager.Infrastructure.Common.Emails`
- **Tipo**: Entidad de Dominio Sellada (sealed)
- **Herencia**: `IEmailService`

## Descripción General

La clase `EmailService` proporciona una implementación robusta para el envío de correos electrónicos en la aplicación. Soporta dos modalidades principales:

- Envío de correos electrónicos simples con contenido directo
- Envío de correos electrónicos basados en plantillas Razor con contenido dinámico

Esta implementación se integra con SMTP para el envío de correos y utiliza el motor de vistas Razor para la generación de contenido HTML dinámico.

## Características Principales

- Envío asíncrono de correos electrónicos
- Soporte para múltiples destinatarios
- Plantillas HTML personalizables con Razor
- Validación de configuración en el inicio
- Manejo de prioridades de correo
- Logging de actividad (en entornos no productivos)

## Configuración

### appsettings.json

La configuración del servicio de correo se realiza a través del archivo `appsettings.json`:

```json
"Email": {
  "Host": "smtp.test.com",
  "DefaultFrom": "noreply@test.com",
  "Username": "smtp_user@test.com",
  "Password": "smtp_password",
  "Port": 587
}
```

### Modelo de Configuración

La configuración se valida mediante un modelo fuertemente tipado con anotaciones de datos:

```csharp
public sealed class EmailSettings
{
    public const string SectionName = "Email";

    [Required]
    public string Host { get; set; } = default!;

    [Required]
    public int Port { get; set; }

    [Required]
    public string Username { get; set; } = default!;

    [Required]
    public string Password { get; set; } = default!;

    [Required]
    public string DefaultFrom { get; set; } = default!;
}
```

### Registro del Servicio

```csharp
services.AddOptions<EmailSettings>()
    .Bind(configuration.GetSection(EmailSettings.SectionName))
    .ValidateDataAnnotations()
    .ValidateOnStart();

services.AddScoped<IEmailService, EmailService>();
```

## Modelos de Datos

### EmailMessage

Representa un correo electrónico básico:

```csharp
public sealed record EmailMessage(
    IEnumerable<string> To,
    string Subject,
    string Body,
    bool IsHtml = false,
    MailPriority Priority = MailPriority.Normal);
```

### EmailTemplate

Define una plantilla de correo con modelo dinámico:

```csharp
public sealed record EmailTemplate<TModel>(
    IEnumerable<string> To,
    string Subject,
    string TemplateName,
    TModel Model)
    where TModel : class;
```

## Ejemplos de Uso

### Envío de Email Simple

```csharp
await emailService.SendEmailAsync(new EmailMessage(
    To: new[] { "usuario@ejemplo.com" },
    Subject: "Notificación Importante",
    Body: "Este es un mensaje de prueba",
    IsHtml: false
));
```

### Envío con Plantilla Razor

```csharp
var welcomeModel = new WelcomeEmailModel
{
    Username = "Juan",
    ActivationLink = "https://ejemplo.com/activate"
};

await emailService.SendTemplatedEmailAsync(new EmailTemplate<WelcomeEmailModel>(
    To: new[] { "usuario@ejemplo.com" },
    Subject: "Bienvenido a nuestra aplicación",
    TemplateName: EmailViewNames.Welcome,
    Model: welcomeModel
));
```

## Estructura de Plantillas

Las plantillas Razor deben ubicarse en:

```
Views/
└── Emails/
    ├── _Layout.cshtml          # Plantilla base para todos los emails
    └── Welcome.cshtml          # Plantillas específicas
```

## Manejo de Errores

El servicio puede lanzar las siguientes excepciones:

- `EmailSenderException`: Para errores de validación de parámetros
- `InvalidOperationException`: Para errores en la búsqueda de plantillas
- `SmtpException`: Para errores en el envío de correos

## Dependencias

- **Microsoft.AspNetCore.Mvc.Razor**: Motor de vistas Razor
- **Microsoft.Extensions.Options**: Configuración tipada
- **System.Net.Mail**: Funcionalidad SMTP

## Consideraciones de Seguridad

- Las credenciales SMTP se manejan de forma segura mediante configuración
- No se guardan logs de contenido de emails en producción
- Se utiliza SSL/TLS para la conexión SMTP
- Se validan los emails destinatarios antes del envío

## Logging

En entornos no productivos, el servicio registra:

- Destinatarios
- Asunto
- Éxito/fallo del envío
- Errores detallados en caso de fallo
