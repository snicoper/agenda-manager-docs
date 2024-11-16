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
- Aplicar las reglas de negocio en creación y modificaciones
- Gestionar las transiciones de estado válidas
- Emitir eventos de dominio para cambios significativos:
  - Cuando se crea una nueva cita
  - Cuando se modifica una cita existente
  - Cuando cambia el estado de una cita
  - Cuando se cancela una cita

## Invariantes

- Una cita debe tener siempre un calendario, servicio y usuario asociados
- Una cita siempre debe estar programada para una fecha/hora futura
- El período de la cita debe ser válido (fecha inicio < fecha fin)
- La duración de la cita no puede ser cero o negativa
- El estado actual debe ser coherente con el historial de cambios
- Los recursos asignados deben estar disponibles durante el período de la cita
- Una cita cancelada no puede volver a estados activos
- Una cita completada no puede modificarse

## Reglas de negocio

### Creación de Citas

- Debe tener asociados:
  - Un `Calendar` válido y activo
  - Un `Service` disponible en el calendario
  - Un `User` con permisos para reservar el servicio
- Debe especificar:
  - Un `Period` dentro del horario del calendario y en fecha futura
    - La fecha/hora de inicio debe ser posterior al momento actual más un margen mínimo configurable
    - No se permiten citas en el pasado o demasiado inmediatas
  - Los recursos requeridos por el servicio deben estar disponibles
- La `Duration` inicial será la del servicio, modificable dentro de límites
- El estado inicial será `Accepted` si pasa todas las validaciones
- Debe validar:
  - Que la fecha/hora sea futura y respete el margen mínimo de antelación
  - Disponibilidad según `AppointmentStrategy`
  - Disponibilidad de recursos necesarios
  - Permisos del usuario
  - Horario del calendario

### Modificación de Citas

- Solo modificables en estados: `Accepted`
- Campos modificables:
  - Duración (dentro de límites establecidos)
  - Período (respetando horarios y siempre en fecha futura)
    - Al modificar el período, se aplican las mismas reglas de fecha futura que en la creación
    - No se permite modificar una cita para establecerla en el pasado
  - Recursos (verificando disponibilidad)
- Requiere revalidación completa:
  - Validación de fecha futura y margen mínimo
  - Disponibilidad temporal
  - Disponibilidad de recursos necesarios
  - Reglas de `AppointmentStrategy`
- Debe generar evento de modificación

### Eliminación de Citas

- Solo se podrá eliminar la cita si está en estado `Accepted` y es futura
- La eliminación de la cita deberá ser permanente en el sistema y no se podrá recuperar
- Debe generar evento de eliminación

### Gestión de Estados

- Estados posibles:
  - `Accepted`: Confirmada y programada (estado inicial al crear)
  - `Waiting`: Usuario presente en ubicación
  - `Cancelled`: Cancelada definitivamente
  - `Completed`: Finalizada correctamente
- Cada cambio de estado:
  - Debe registrarse en `StatusChanges` con:
    - Fecha y hora del cambio (campo auditable)
    - Nuevo estado
    - Usuario que realiza el cambio (campo auditable)
    - Descripción o motivo (cuando aplique)
  - Debe seguir transiciones permitidas
  - Debe generar evento de dominio
- Restricciones de transición:
  - Solo estado `Accepted` puede pasar a `Waiting`
  - `Cancelled` y `Completed` son estados finales
  - No se permite transición a estados anteriores una vez en `Waiting`
- Validaciones específicas por estado:
  - `Waiting`: Solo se puede establecer si la fecha/hora actual está dentro del período de la cita
  - `Completed`: Solo se puede establecer si la cita estaba en estado `Waiting`
  - `Cancelled`: Se puede establecer desde cualquier estado no final

### Solapamiento y Calendario

- Controlado por `AppointmentStrategy` del calendario
- Estrategia por defecto: `RejectIfOverlapping`
- Debe respetar:
  - Horario del calendario
  - Disponibilidad de recursos
  - Reglas específicas del calendario
  - Políticas de solapamiento configuradas

### Notificaciones y Eventos

- Debe generar eventos de dominio para:
  - Creación de cita
  - Modificación de cita
  - Cambios de estado
  - Cancelación de cita
  - Finalización de cita
- Los eventos deben incluir:
  - Información relevante del cambio
  - Fecha y hora del evento
  - Usuario que generó el evento
  - Datos adicionales según el tipo de evento

### Validaciones Generales

- Todas las operaciones deben verificar:
  - Permisos del usuario que realiza la acción
  - Estado actual de la cita
  - Reglas temporales aplicables
  - Disponibilidad de recursos
  - Restricciones del calendario
- Cualquier violación de estas reglas debe generar una excepción de dominio apropiada

## Propiedades

| Nombre          | Tipo                                     | Descripción                                      |
| --------------- | ---------------------------------------- |--------------------------------------------------|
| `Id`            | `AppointmentId`                          | Identificador único de la cita                   |
| `ServiceId`     | `ServiceId`                              | Identificador del servicio asociado a la cita    |
| `Service`       | `Service`                                | Servicio asociado a la cita                      |
| `CalendarId`    | `CalendarId`                             | Identificador del calendario asociado a la cita  |
| `Calendar`      | `Calendar`                               | Calendario asociado a la cita                    |
| `UserId`        | `UserId`                                 | Identificador del usuario asociado a la cita     |
| `User`          | `User`                                   | Usuario asociado a la cita                       |
| `Period`        | `Period`                                 | Período de la cita                               |
| `Duration`      | `Duration`                               | Duración de la cita                              |
| `Status`        | `AppointmentStatus`                      | Estado de la cita                                |
| `StatusChanges` | `IReadOnlyList<AppointmentStatusChange>` | Historial de cambios de estado de la cita        |

- **Acceso**: `Id` es **get**, el resto son **get/private set**.

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

## Eventos de Dominio

- **No Aplica**

## Comentarios adicionales

- **No Aplica**

## Ejemplos de Uso

- **No Aplica**
