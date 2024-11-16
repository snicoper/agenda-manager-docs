# Appointment

- **Aggregate Root**: `Appointment`
- **Namespace**: `AgendaManager.Domain.Appointments`
- **Tipo**: Entidad de Dominio Sellada (sealed)
- **Herencia**: `AggregateRoot`

## Descripción General

Un `Appointment` representa una cita en el sistema que pertenece a un calendario. Cada cita tiene un servicio asociado y un usuario al que se le asigna la cita.

### Responsabilidades

- Requisitos necesarios para la creación de una cita
-

## Invariantes

- **No Aplica**

## Reglas de negocio

- **No Aplica**

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
