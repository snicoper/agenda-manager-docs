# User

- **Aggregate Root**: `CalendarSettings`
- **Namespace**: `AgendaManager.Domain.Calendars.Entities`
- **Tipo**: Entidad de Dominio Sellada (sealed)
- **Herencia**: `AuditableEntity`

## ToDo List

- **No Aplica**

## Descripción General

La entidad `CalendarSettings` representa la configuración de un calendario. Esta entidad contiene la información necesaria para determinar el comportamiento del calendario, como la zona horaria, y diferentes estrategias de validación y manejo de citas.

## Vista General

Las configuraciones de calendario permiten personalizar el comportamiento de cada calendario en la aplicación. Cada calendario tiene un conjunto de configuraciones que determinan cómo manejar:

- Creación y gestión de citas
- Comprobar si requiere validar los horarios de los recursos antes de crear una cita
- Manejo de solapamientos
- Gestión de días festivos
- Configuración de zona horaria

## Tipos de Configuración

### Configuraciones con Opciones Predefinidas

Configuraciones que solo permiten seleccionar entre un conjunto predefinido de valores:

- `AppointmentConfirmationRequirementStrategy`

  - **Descripción**: Define el estado inicial de las citas al ser creadas: automáticamente aceptadas (Accepted) o pendientes de confirmación por email (Pending)
  - **Opciones** AutoAccept, Require
  - **Default**: AutoAccept

- `AppointmentOverlappingStrategy`:

  - **Descripción**: Define cómo gestionar los conflictos temporales al crear o modificar citas: permitir la creación aunque existan solapamientos con otras citas, o rechazarla
  - **Opciones** Allow, Reject
  - **Default**: Reject

- `HolidayConflictStrategy`

  - **Descripción**: Define cómo gestionar los conflictos cuando un día festivo se solapa con citas existentes: permitir que coexistan, rechazar la operación, o cancelar automáticamente las citas afectadas
  - **Opciones** Allow, Reject, Cancel
  - **Default**: Reject

- `ResourceScheduleValidationStrategy`
  - **Descripción**: Define si se validan los horarios de disponibilidad de los recursos requeridos al crear una cita. Los requisitos del servicio siempre se comprueban, pero la validación de los horarios puede ser opcional
  - **Opciones** Validate, Ignore
  - **Default**: Validate

### Responsabilidades

1. **Gestión de configuración temporal**
   - Mantener la zona horaria correcta del calendario
   - Garantizar que todas las operaciones temporales se realicen en la zona horaria configurada

2. **Control de creación de citas**
   - Definir el flujo de confirmación de citas (automática o con confirmación requerida)
   - Gestionar las reglas de solapamiento entre citas
   - Controlar la validación de disponibilidad de recursos

3. **Gestión de conflictos con días festivos**
   - Determinar cómo se manejan las citas existentes cuando se crean días festivos
   - Garantizar la consistencia entre citas y días festivos según la estrategia configurada

4. **Validación de configuración**
   - Asegurar que todas las estrategias configuradas son válidas y coherentes entre sí
   - Mantener la integridad de la configuración del calendario

## Propiedades

| Nombre | Tipo             | Descripción                     |
| ------ | ---------------- |-------------------------------- |
| `Id`     | `CalendarSettingsId`           | Identificador único de la configuración del calendario. |
| `CalendarId` | `CalendarId` | Identificador único del calendario al que pertenece la configuración. |
| `Calendar` | `Calendar` | Calendario al que pertenece la configuración. |
| `Timezone` | `IanaTimeZone` | Zona horaria del calendario. |
| `ConfirmationRequirement`| `AppointmentConfirmationRequirementStrategy` | Determina la estrategia de confirmación de citas. |
| `OverlapBehavior` | `AppointmentOverlappingStrategy` | Determina la estrategia de superposición de citas. |
| `HolidayAppointmentHandling` | `HolidayConflictType` | Determina la estrategia de manejo de citas al crear días festivos. |
| `ResourceScheduleValidationStrategy` | `ScheduleValidation` | Determina la estrategia de validación de horarios de recursos. |

## Invariantes

- `Id` no puede ser `null` en ningún momento y debe ser único en toda la aplicación.
- `CalendarId` no puede ser `null` en ningún momento.
- `Timezone` no puede ser `null` en ningún momento.
- `ConfirmationRequirement` no puede ser `null` en ningún momento.
- `OverlapBehavior` no puede ser `null` en ningún momento.
- `HolidayAppointmentHandling` no puede ser `null` en ningún momento.
- `ResourceScheduleValidationStrategy` no puede ser `null` en ningún momento.

## Reglas de negocio

- **Unicidad de Identificador**:

  - `Id` debe ser único en toda la aplicación.
  - `CalendarId` debe ser único en toda la aplicación.

- **Validación de Valores**:
  - Los valores para configuraciones con `Key = 'UnitValue'` deben cumplir con el formato específico de cada categoría.
  - Para otras configuraciones, `SelectedKey` debe existir en `CalendarConfigurationKeys` para la categoría correspondiente.

## Métodos

### Constructor

```csharp
private CalendarSettings(
    CalendarSettingsId calendarSettingsId,
    CalendarId calendarId,
    IanaTimeZone timeZone,
    AppointmentConfirmationRequirementStrategy confirmationRequirement,
    AppointmentOverlappingStrategy overlapBehavior,
    HolidayConflictStrategy holidayAppointmentHandling,
    ResourceScheduleValidationStrategy scheduleValidation)
```

- **Descripción**: Crea una nueva instancia de `CalendarSettings` con los valores proporcionados.
- **Parámetros**:
  - `calendarSettingsId`: Identificador único de la configuración del calendario.
  - `calendarId`: Identificador único del calendario al que pertenece la configuración.
  - `timeZone`: Zona horaria del calendario.
  - `confirmationRequirement`: Estrategia de confirmación de citas.
  - `overlapBehavior`: Estrategia de superposición de citas.
  - `holidayAppointmentHandling`: Estrategia de manejo de citas al crear días festivos.
  - `scheduleValidation`: Estrategia de validación de horarios de recursos.

### Create

```csharp
internal static CalendarSettings Create(
    CalendarSettingsId calendarSettingsId,
    CalendarId calendarId,
    IanaTimeZone timeZone,
    AppointmentConfirmationRequirementStrategy confirmationRequirement,
    AppointmentOverlappingStrategy overlapBehavior,
    HolidayConflictStrategy holidayAppointmentHandling,
    ResourceScheduleValidationStrategy scheduleValidation)
```

- **Descripción**: Crea una nueva instancia de `CalendarSettings` con los valores proporcionados.
- **Parámetros**:
  - `calendarSettingsId`: Identificador único de la configuración del calendario.
  - `calendarId`: Identificador único del calendario al que pertenece la configuración.
  - `timeZone`: Zona horaria del calendario.
  - `confirmationRequirement`: Estrategia de confirmación de citas.
  - `overlapBehavior`: Estrategia de superposición de citas.
  - `holidayAppointmentHandling`: Estrategia de manejo de citas al crear días festivos.
  - `scheduleValidation`: Estrategia de validación de horarios de recursos.
- **Eventos**:
  - `new CalendarSettingsCreatedDomainEvent(calendarSettingsId)`:
    - **Descripción** Se lanza cuando se crea una nueva instancia de `CalendarSettings`.
    - **Parámetros**:
      - `calendarSettingsId`: Identificador único de la configuración del calendario.
- **Retorno**: Nueva instancia de `CalendarSettings`.

### HasChanges

```csharp
internal bool HasChanges(CalendarSettings settings)
```

- **Descripción**: Comprueba si la configuración actual es diferente de otra configuración.
- **Parámetros**:
  - `settings`: Configuración con la que comparar.
- **Retorno**: `true` si la configuración actual es igual de `settings`, `false` en caso contrario.

## Estado y Transiciones

- **No Aplica**

## Dependencias

### Directas

- **Entidades Base**:

  - `AuditableEntity`: Base class que proporciona capacidades de auditoría
    - Hereda gestión de eventos de dominio (`Entity`)

- `Calendar`: Entidad que representa un calendario.

### Interfaces

- **No Aplica**

### Managers

- **No Aplica**

### Policies

- **No Aplica**

### Value Objects

- `CalendarSettingsId`: Identificador único de la configuración del calendario.
- `IanaTimeZone`: Zona horaria IANA.

### Enums

- `AppointmentConfirmationRequirementStrategy`: Estrategia de confirmación de citas.
- `AppointmentOverlappingStrategy`: Estrategia de superposición de citas.
- `HolidayConflictStrategy`: Estrategia de manejo de citas al crear días festivos.
- `ResourceScheduleValidationStrategy`: Estrategia de validación de horarios de recursos.

## Errores

## Comentarios adicionales

- **No Aplica**

## Ejemplos de Uso

- **No Aplica**
