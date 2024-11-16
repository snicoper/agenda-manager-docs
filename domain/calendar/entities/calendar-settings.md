# CalendarSettings

- **Entity**: `CalendarSettings`
- **Namespace**: `AgendaManager.Domain.Calendars.Entities`
- **Tipo**: Entidad de Dominio Sellada (sealed)
- **Herencia**: `AuditableEntity`

## Descripción General

Representa la configuración específica de un calendario en el sistema, gestionando aspectos críticos como la zona horaria y las estrategias de manejo de holidays (días festivos o no laborables) y manejo de citas en caso de solapamiento. Esta entidad es fundamental para el correcto funcionamiento y la coherencia temporal de los calendarios en la aplicación.

## Responsabilidades

- **Gestión de Configuración del Calendario**:
  - Mantiene la configuración específica para cada calendario del sistema.
  - Garantiza la integridad y validez de los ajustes del calendario.
  - Gestiona la relación entre calendarios y sus configuraciones.

- **Configuración de la Zona Horaria**:
  - Establece y mantiene la zona horaria específica para cada calendario.
  - Asegura que todas las operaciones temporales del calendario respeten la zona horaria configurada.
  - Valida que la zona horaria especificada sea un identificador IANA válido.

- **Gestión de Estrategias para Holidays**:
  - Implementa políticas de manejo de conflictos para holidays.
  - Controla el comportamiento del sistema cuando hay solapamientos entre holidays y otros eventos.
  - Permite la flexibilidad en la gestión de conflictos mediante diferentes estrategias.

- **Gestión de Estrategia para Citas**:
  - Define la estrategia para la creación de citas en el calendario en caso de solapamiento.

## Invariantes

- **Integridad de Identificadores**:
  - `Id` debe ser un `CalendarSettingsId` válido y no nulo.
  - `CalendarId` debe ser un `CalendarId` válido y no nulo.
  - La combinación de `Id` y `CalendarId` debe ser única en el sistema.

- **Validación de Zona Horaria**:
  - `IanaTimeZone` debe contener un identificador IANA válido.
  - No se permiten zonas horarias nulas o inválidas.

- **Estrategia de Holidays**:
  - `HolidayStrategy` debe estar siempre definida y ser válida.
  - No se permite un valor nulo para la estrategia.

- **Estrategia de Citas**:
  - `AppointmentStrategy` debe estar siempre definida y ser válida.
  - No se permite un valor nulo para la estrategia.

## Reglas de Negocio

- **Creación y Modificación**:
  - La creación de nuevas configuraciones solo se permite mediante el método factory `CalendarManager.CreateCalendarAsync` al crear un nuevo calendario.
  - Las modificaciones solo se realizan a través del método `UpdateSettings` de la entidad `Calendar`.
  - Se verifica la existencia de cambios reales antes de aplicar actualizaciones.
  - El valor por defecto de `HolidayStrategy` cuando se crea un nuevo calendario debe ser `RejectIfOverlapping`.
  - El valor por defecto de `AppointmentStrategy` cuando se crea un nuevo calendario debe ser `RejectIfOverlapping`.
  - Es requerido que la zona horaria especificada sea un identificador IANA válido.
  - La modificación se realiza desde el **record** `CalendarSettingsConfiguration`, estas son las propiedades editables:
    - `IanaTimeZone`
    - `HolidayStrategy`
    - `AppointmentStrategy`

- **Gestión de Eventos de Dominio**:
  - Emitir eventos de dominio cuando se realicen cambios significativos en la configuración del calendario.

- **Estrategias de Manejo de Holidays**:

  ```csharp
  public enum HolidayStrategy
  {
      [Display(Name = "Reject if overlapping")]
      RejectIfOverlapping = 1, // Rechazar si se solapan

      [Display(Name = "Cancel if overlapping")]
      CancelOverlapping = 2, // Cancelar si se solapan

      [Display(Name = "Allow overlapping")]
      AllowOverlapping = 3 // Permitir solapamiento
  }
  ```

- **Estrategia de Manejo de Citas**:

  ```csharp
    public enum AppointmentStrategy
    {
        [Display(Name = "Reject if overlapping")]
        RejectIfOverlapping = 1, // Rechazar si se solapan

        [Display(Name = "Allow overlapping")]
        AllowOverlapping = 2 // Permitir solapamiento
    }
    ```

## Propiedades

| Propiedad                    | Tipo                              | Descripción                                |
|------------------------------|-----------------------------------|--------------------------------------------|
| `Id`                         | `CalendarSettingsId`              | Identificador único de la configuración    |
| `CalendarId`                 | `CalendarId`                      | Identificador del calendario asociado.     |
| `Calendar`                   | `Calendar`                        | Navegación a la entidad Calendar asociada. |
| `IanaTimeZone`               | `IanaTimeZone`                    | Zona horaria del calendario.               |
| `HolidayStrategy`            | `HolidayStrategy`                 | Estrategia de gestión de holidays.         |
| `AppointmentStrategy`        | `AppointmentStrategy`             | Estrategia de gestión de citas.            |

- **Acceso**: `Id` es **get**, el resto son **get/private set**.

## Métodos

### Create

```csharp
internal static CalendarSettings Create(
    CalendarSettingsId id,
    CalendarId calendarId,
    IanaTimeZone ianaTimeZone,
    HolidayStrategy holidayStrategy,
    AppointmentStrategy appointmentStrategy)
```

- **Descripción**: Método para crear una nueva configuración de calendario
- **Parámetros**:
  - `id`: Identificador único de la configuración
  - `calendarId`: Identificador del calendario asociado
  - `ianaTimeZone`: Zona horaria del calendario
  - `holidayStrategy`: Estrategia de gestión de holidays
  - `appointmentStrategy`: Estrategia de gestión de citas
**Eventos**: Emite `CalendarSettingsCreatedDomainEvent`
**Retorno**: Nueva instancia de `CalendarSettings`

### Update

```csharp
internal void Update(CalendarSettingsConfiguration configuration)
```

- **Descripción**: Método para actualizar la configuración existente
- **Parámetros**:
  - `configuration`: Configuración actualizada
- **Eventos**: Emite `CalendarSettingsUpdatedDomainEvent`

### HasChanges

```csharp
internal bool HasChanges(CalendarSettingsConfiguration configuration)
```

- **Descripción**: Método para determinar si hay cambios reales
- **Parámetros**:
  - `configuration`: Configuración actualizada
- **Retorno**: `true` si hay cambios reales, `false` en caso contrario

## Estado y Transiciones

- **No Aplica**

## Dependencias

### Directas

- **Entidades Base**:
  - `AuditableEntity`: Base class que proporciona capacidades de auditoría
    - Hereda gestión de eventos de dominio (`Entity`)

### Value Objects

- `CalendarSettingsId`: Identificador de la configuración
- `CalendarId`: Identificador del calendario
- `IanaTimeZone`: Representa una zona horaria válida
- `HolidayStrategy`: Enum de estrategias disponibles

### Eventos de Dominio

- `CalendarSettingsCreatedDomainEvent`: Notifica la creación
- `CalendarSettingsUpdatedDomainEvent`: Notifica cambios significativos

### Records

- `CalendarSettingsDomainRequest`: Record para crear y actualizar configuraciones del calendario

## Comentarios adicionales

- **No Aplica**

## Ejemplos de Uso
