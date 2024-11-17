# Appointment

- **Aggregate Root**: `Appointment`
- **Namespace**: `AgendaManager.Domain.Appointments`
- **Tipo**: Entidad de Dominio Sellada (sealed)
- **Herencia**: `AggregateRoot`

## Descripción General

Un `Appointment` representa una cita programada en el sistema que pertenece a un calendario específico. Es la unidad central de reserva que conecta un servicio concreto con un usuario en un período de tiempo determinado. Gestiona todo el ciclo de vida de la cita, incluyendo su programación, estados y finalización, asegurando que se cumplan todas las reglas de negocio y restricciones del calendario asociado.

### Responsabilidades

- Gestionar el ciclo de vida completo de una cita
- Validar la disponibilidad temporal y de recursos
- Mantener la integridad y consistencia del estado de la cita
- Registrar el historial de cambios de estado
- Gestionar el proceso de confirmación cuando sea necesario
- Aplicar las reglas de negocio en creación y modificaciones
- Gestionar las transiciones de estado válidas
- Emitir eventos de dominio para cambios significativos

## Propiedades y Estructura

### Propiedades Principales

| Nombre              | Tipo                                     | Descripción                           |
| ------------------- | ---------------------------------------- | ------------------------------------- |
| `Id`                | `AppointmentId`                          | Identificador único de la cita        |
| `ServiceId`         | `ServiceId`                              | Identificador del servicio asociado   |
| `Service`           | `Service`                                | Servicio asociado a la cita           |
| `CalendarId`        | `CalendarId`                             | Identificador del calendario asociado |
| `Calendar`          | `Calendar`                               | Calendario asociado a la cita         |
| `UserId`            | `UserId`                                 | Identificador del usuario asociado    |
| `User`              | `User`                                   | Usuario asociado a la cita            |
| `Period`            | `Period`                                 | Período de la cita                    |
| `Duration`          | `Duration`                               | Duración de la cita                   |
| `Status`            | `AppointmentStatus`                      | Estado actual de la cita              |
| `StatusChanges`     | `IReadOnlyList<AppointmentStatusChange>` | Historial de cambios de estado        |
| `ConfirmationToken` | `string`                                 | Token de confirmación (si aplica)     |

### Invariantes

- `Id` no puede ser `null` en ningún momento
- Una cita debe tener siempre un calendario, servicio y usuario asociados
- Una cita siempre debe estar programada para una fecha/hora futura
- El período de la cita debe ser válido (fecha inicio < fecha fin)
- La duración de la cita no puede ser cero o negativa
- El estado actual debe ser coherente con el historial de cambios
- Los recursos asignados deben estar disponibles durante el período
- Una cita cancelada no puede volver a estados activos ni modificarse
- Una cita completada no puede modificarse
- Si `RequireConfirmation` está activo, la cita debe tener un token válido asociado

## Configuración y Comportamiento

### Estrategias de Configuración

#### AppointmentCreationStrategy

- **Direct**
  - Las citas se crean directamente en estado `Accepted`
  - No requiere confirmación adicional

- **RequireConfirmation**
  - Las citas se crean en estado `Pending`
  - Requiere confirmación del cliente
  - Se genera token de confirmación
  - Tiene tiempo límite de confirmación

#### AppointmentOverlappingStrategy

- **AllowOverlapping**
  - Permite solapamiento de citas
  - No realiza validaciones adicionales de disponibilidad

- **RejectIfOverlapping**
  - Rechaza citas que se solapan
  - Valida disponibilidad de recursos y horarios

## Ciclo de Vida

### Estados y Transiciones

#### Estados Posibles

- `Pending`: Esperando confirmación del cliente
- `Accepted`: Confirmada y programada
- `Waiting`: Usuario presente en ubicación
- `Cancelled`: Cancelada definitivamente
- `RequiresRescheduling`: Requiere reprogramación
- `InProgress`: En proceso, la cita está en curso
- `Completed`: Finalizada correctamente

#### Transiciones Permitidas

- `Pending` → `Accepted`, `RequiresRescheduling`, `Cancelled`
- `Accepted` → `Waiting`, `Cancelled`, `RequiresRescheduling`
- `Waiting` → `InProgress`, `Cancelled`
- `InProgress` → `Completed`, `Cancelled`
- `RequiresRescheduling` → `Accepted`, `Cancelled`

#### Validaciones por Estado

- `Pending`: Debe tener token de confirmación válido si aplica
- `Waiting`: Solo dentro del período de la cita (con tolerancia)
- `InProgress`: Solo si estaba en estado `Waiting`
- `Completed`: Solo si estaba en estado `InProgress`
- `Cancelled`: Permitido desde cualquier estado no final

## Operaciones y Reglas de Negocio

### Creación de Citas

#### Requisitos Básicos

- Calendario, servicio y usuario válidos y activos
- Período dentro del horario del calendario y en fecha futura
- Recursos requeridos disponibles
- Duración inicial según el servicio

#### Proceso de Creación

1. Validación de requisitos básicos
2. Verificación de disponibilidad según estrategia de solapamiento
3. Asignación de estado inicial según estrategia de creación
4. Generación de token si requiere confirmación
5. Emisión de eventos correspondientes

### Confirmación de Citas

#### Proceso de Confirmación

1. Verificación de token válido y no expirado
2. Validación de estado `Pending`
3. Revalidación de disponibilidad de recursos
4. Cambio a estado `Accepted`
5. Emisión de evento de confirmación

### Modificación de Citas

#### Campos Modificables

- Duración
- Período
- Recursos asignados

#### Reglas de Modificación

- Solo en estados `Pending` y `Accepted`
- Validación de fecha futura
- Revalidación de disponibilidad
- Verificación de reglas de solapamiento

### Eliminación de Citas

- Solo en estados `Pending` o `Accepted`
- Solo para citas futuras
- Eliminación permanente
- Generación de evento de eliminación

## Métodos

- **No Aplica**

## Estado y Transiciones

- **No Aplica**

## Dependencias

- **No Aplica**

### Directas

- **No Aplica**

### Interfaces

- **No Aplica**

### Managers

- **No Aplica**

### Policies

- **No Aplica**

### Value Objects

- **No Aplica**

## Comentarios adicionales

- **No Aplica**
