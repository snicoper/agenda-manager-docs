# AppointmentStatusHistory

- **Aggregate Root**: `AppointmentStatusHistory`
- **Namespace**: `AgendaManager.Domain.Appointments.Entities`
- **Tipo**: Entidad de Dominio Sellada (sealed)
- **Herencia**: `AuditableEntity`

## ToDo list

- [ ] Revisar **Consideraciones de Rendimiento**

## Descripción General

Entidad que representa el historial de estados de una cita en el sistema. Cada instancia captura un cambio de estado específico en un momento determinado, manteniendo así un registro completo y auditable de la evolución de la cita a lo largo de su ciclo de vida. Esta entidad es fundamental para:

- Trazabilidad de cambios de estado
- Auditoría del ciclo de vida de las citas
- Análisis histórico de patrones de cambio de estado
- Cumplimiento de requisitos regulatorios de registro

### Responsabilidades

- Mantener la integridad de los datos históricos de estados de cita
- Validar las reglas de negocio relacionadas con el registro de cambios de estado
- Capturar el contexto temporal de cada cambio de estado (Period)
- Mantener la trazabilidad de la relación con la cita padre
- Gestionar el flag de estado actual
- Proporcionar una descripción opcional del cambio de estado
- Soportar la auditoría de cambios de estado

## Propiedades

| Nombre           | Tipo                         | Descripción                                               |
| ---------------- | ---------------------------- | --------------------------------------------------------- |
| `Id`             | `AppointmentStatusHistoryId` | Identificador único de la historia de estado de la cita   |
| `AppointmentId`  | `AppointmentId`              | Identificador único de la cita                            |
| `Appointment`    | `Appointment`                | Cita asociada a la historia de estado                     |
| `Period`         | `Period`                     | Período de tiempo asociado a la historia de estado        |
| `CurrentState`   | `AppointmentCurrentState`    | Estado actual de la cita                                  |
| `IsCurrentState` | `bool`                       | Indica si el estado actual es el estado actual de la cita |
| `Description`    | `string?`                    | Descripción del estado actual                             |

## Invariantes

- `Id` no puede ser `null` en ningún momento
- `AppointmentId` no puede ser `null` en ningún momento
- `Period` no puede ser `null` en ningún momento
- `CurrentState` no puede ser `null` en ningún momento
- `IsCurrentState` no puede ser `null` en ningún momento, en la creación debe ser `true`
- `Description` si no es `null` no debe tener un máximo de 200 caracteres

## Reglas de negocio

- **Unicidad de Identidad**:
  - `Id` debe ser único en toda la aplicación.
  - `Id` y `AppointmentId` deben ser únicos en toda la aplicación.

## Casos de Uso

1. **Registro de Nuevo Estado**

   - Cuando una cita cambia de estado
   - Captura del momento exacto del cambio
   - Registro opcional de motivo/descripción

2. **Consulta de Historial**

   - Visualización de la progresión de estados
   - Análisis de tiempos entre cambios de estado
   - Auditoría de cambios

3. **Desactivación de Estado Actual**
   - Cuando la cita cambia a un nuevo estado
   - Actualización del flag IsCurrentState

## Métodos

### Create

```csharp
internal static AppointmentStatusHistory Create(
    AppointmentStatusChangeId appointmentStatusChangeId,
    AppointmentId appointmentId,
    Period period,
    AppointmentCurrentState state,
    string? description)
```

- **Descripción**: Crea una nueva historia de estado de cita.
- **Parámetros**:
  - `appointmentStatusChangeId`: Identificador único de la historia de estado de la cita.
  - `appointmentId`: Identificador único de la cita.
  - `period`: Período de tiempo asociado a la historia de estado.
  - `state`: Estado actual de la cita.
  - `description`: Descripción del estado actual.
    **Retorno**: Nueva instancia de `AppointmentStatusHistory`.

### DeactivateCurrentState

```csharp
internal void DeactivateCurrentState()
```

- **Descripción**: Desactiva el estado actual de la cita.

### AgainstInvalidDescription

```csharp
private static void AgainstInvalidDescription(string? description)
```

- **Descripción**: Valida la descripción del estado actual.
- **Parámetros**:
  - `description`: Descripción del estado actual.
- **Excepciones**:
  - `AppointmentStatusChangeDomainException`: Si la descripción no es válida.

## Estado y Transiciones

- **No Aplica**

## Dependencias

### Directas

- **Entidades Base**:
  - `AuditableEntity`: Base class que proporciona capacidades de auditoría
    - Hereda gestión de eventos de dominio (`Entity`)

### Interfaces

- `IAppointmentStatusHistoryRepository`: Repositorio de historias de estado de cita.

### Managers

### Policies

### Value Objects

- `AppointmentStatusChangeId`: Identificador único de la historia de estado de la cita.
- `AppointmentId`: Identificador único de la cita.
- `Period`: Período de tiempo asociado a la historia de estado.
- `AppointmentCurrentState`: Estado actual de la cita.

### Errores

### Conflict

- **Identifier**: `OnlyPendingAndAcceptedAllowed` Se lanza cuando se intenta cambiar el estado de una cita que no está en estado pendiente o aceptado.
  - **Code**: `AppointmentStatusChangeErrors.OnlyPendingAndAcceptedAllowed`
  - **Description**: Only pending and accepted appointments can be changed.

- **Identifier**: `OnlyPendingAllowed` Se lanza cuando se intenta cambiar el estado de una cita que no está en estado pendiente.
  - **Code**: `AppointmentStatusChangeErrors.OnlyPendingAllowed`
  - **Description**: Only pending appointments can be changed.

- **Identifier**: `OnlyPendingAndReschedulingAllowed` Se lanza cuando se intenta cambiar el estado de una cita que no está en estado pendiente o reprogramado.
  - **Code**: `AppointmentStatusChangeErrors.OnlyPendingAndReschedulingAllowed`
  - **Description**: Only pending and rescheduling appointments can be changed.

- **Identifier**: `AlreadyCancelledOrCompleted` Se lanza cuando se intenta cambiar el estado de una cita que ya está cancelada o completada.
  - **Code**: `AppointmentStatusChangeErrors.AlreadyCancelledOrCompleted`
  - **Description**: The appointment is already cancelled or completed.

- **Identifier**: `OnlyWaitingAllowed` Se lanza cuando se intenta cambiar el estado de una cita que no está en estado de espera.
  - **Code**: `AppointmentStatusChangeErrors.OnlyWaitingAllowed`
  - **Description**: Only waiting appointments can be changed.

- **Identifier**: `OnlyInProgressAllowed` Se lanza cuando se intenta cambiar el estado de una cita que no está en estado en progreso.
  - **Code**: `AppointmentStatusChangeErrors.OnlyInProgressAllowed`
  - **Description**: Only in progress appointments can be changed.

## Comentarios adicionales

La entidad no es accesible desde fuera de la capa de dominio y el responsable de añadir nuevas instancias es `Appointment`.

## Consideraciones de Rendimiento

- Los registros históricos son inmutables una vez creados
- Se debe considerar una estrategia de archivado para registros antiguos
- Índices recomendados:
  - (AppointmentId, IsCurrentState)
  - (AppointmentId, Period)

## Ejemplos de Uso

### Creación de nuevo registro histórico

```csharp
var statusHistory = AppointmentStatusHistory.Create(
    AppointmentStatusChangeId.Create(),
    appointmentId,
    Period.Create(DateTimeOffset.UtcNow, DateTimeOffset.UtcNow.AddHours(1)),
    AppointmentCurrentState.FromEnum(AppointmentStatus.Accepted),
    "Cliente confirmó la cita por email"
);

// Desactivación de estado actual
currentStatusHistory.DeactivateCurrentState();
```
