# CalendarSettings

- **Entity**: `CalendarSettings`
- **Namespace**: `AgendaManager.Domain.Calendars.Entities`
- **Tipo**: Entidad de Dominio Sellada (sealed)
- **Herencia**: `AuditableEntity`

## Descripción General

> Representa la configuración específica de un calendario en el sistema, gestionando aspectos críticos como la zona horaria y las estrategias de manejo de holidays (días festivos o no laborables). Esta entidad es fundamental para el correcto funcionamiento y la coherencia temporal de los calendarios en la aplicación.

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

## Invariantes

- **Integridad de Identificadores**:
  - `Id` debe ser un `CalendarSettingsId` válido y no nulo.
  - `CalendarId` debe ser un `CalendarId` válido y no nulo.
  - La combinación de `Id` y `CalendarId` debe ser única en el sistema.

- **Validación de Zona Horaria**:
  - `IanaTimeZone` debe contener un identificador IANA válido.
  - No se permiten zonas horarias nulas o inválidas.

- **Estrategia de Holidays**:
  - `HolidayCreationStrategy` debe estar siempre definida y ser válida.
  - No se permite un valor nulo para la estrategia.

## Reglas de Negocio

- **Creación y Modificación**:
  - La creación de nuevas configuraciones solo se permite mediante el método factory `Create`.
  - Las modificaciones solo se realizan a través del método `Update`.
  - Se verifica la existencia de cambios reales antes de aplicar actualizaciones.

- **Gestión de Eventos de Dominio**:
  - Se notifica la creación mediante `CalendarSettingsCreatedDomainEvent`.
  - Las modificaciones significativas generan eventos de dominio apropiados.

- **Estrategias de Manejo de Holidays**:

  ```csharp
  public enum HolidayCreationStrategy
  {
      RejectIfOverlapping,    // Rechaza nuevos holidays si hay conflictos
      CancelOverlapping,      // Cancela eventos existentes para dar prioridad al holiday
      AllowOverlapping        // Permite la coexistencia de holidays y otros eventos
  }
  ```

## Propiedades

| Propiedad                 | Tipo                     | Acceso       | Descripción                                                          |
|--------------------------|---------------------------|--------------|----------------------------------------------------------------------|
| `Id`                     | `CalendarSettingsId`      | get         | Identificador único de la configuración. Inmutable tras creación.     |
| `CalendarId`             | `CalendarId`              | get/private set | Identificador del calendario asociado. Inmutable tras creación.   |
| `Calendar`               | `Calendar`                | get/private set | Navegación a la entidad Calendar asociada.                         |
| `IanaTimeZone`           | `IanaTimeZone`            | get/private set | Zona horaria del calendario. Modificable vía Update.              |
| `HolidayCreationStrategy`| `HolidayCreationStrategy` | get/private set | Estrategia de gestión de holidays. Modificable vía Update.        |

## Métodos

### Create

```csharp
internal static CalendarSettings Create(
    CalendarSettingsId id,
    CalendarId calendarId,
    IanaTimeZone ianaTimeZone,
    HolidayCreationStrategy holidayCreationStrategy)
```

- Factory method para crear nuevas instancias de configuración.
- `id`: Identificador único de la configuración.
- `calendarId`: Identificador del calendario asociado.
- `ianaTimeZone`: Zona horaria del calendario.
- `holidayCreationStrategy`: Estrategia de gestión de holidays.
- **Eventos**: Emite `CalendarSettingsCreatedDomainEvent`
- **Retorno**: Nueva instancia válida de `CalendarSettings`

### Update

```csharp
internal void Update(
    IanaTimeZone ianaTimeZone,
    HolidayCreationStrategy holidayCreationStrategy)
```

- Método para actualizar la configuración existente
- `ianaTimeZone`: Nueva zona horaria
- `holidayCreationStrategy`: Nueva estrategia de gestión de holidays
- **Eventos**: Emite `CalendarSettingsUpdatedDomainEvent`
- **Retorno**: `void`

### HasChanges

```csharp
internal bool HasChanges(
    IanaTimeZone ianaTimeZone,
    HolidayCreationStrategy holidayCreationStrategy)
```

- Método para determinar si hay cambios reales
- `ianaTimeZone`: Nueva zona horaria
- `holidayCreationStrategy`: Nueva estrategia de gestión de holidays
- **Retorno**: `true` si hay cambios reales, `false` en caso contrario

## Estado y Transiciones

## Dependencias

### Directas

- **Entidades Base**:
  - `AuditableEntity`: Base class que proporciona capacidades de auditoría
    - Hereda gestión de eventos de dominio (`Entity`)

### Value Objects

- `CalendarSettingsId`: Identificador de la configuración
- `CalendarId`: Identificador del calendario
- `IanaTimeZone`: Representa una zona horaria válida
- `HolidayCreationStrategy`: Enum de estrategias disponibles

### Eventos de Dominio

- `CalendarSettingsCreatedDomainEvent`: Notifica la creación
- `CalendarSettingsUpdatedDomainEvent`: Notifica cambios significativos

## Interceptores EF Core

## Comentarios adicionales

## Ejemplos de Uso

```csharp
// Creación de nueva configuración
var settings = CalendarSettings.Create(
    new CalendarSettingsId(Guid.NewGuid()),
    existingCalendarId,
    new IanaTimeZone("Europe/Madrid"),
    HolidayCreationStrategy.RejectIfOverlapping);

// Actualización de configuración
settings.Update(
    new IanaTimeZone("Europe/London"),
    HolidayCreationStrategy.CancelOverlapping);

// Verificación de cambios
bool hasChanges = settings.HasChanges(
    currentTimeZone,
    newStrategy);
```
