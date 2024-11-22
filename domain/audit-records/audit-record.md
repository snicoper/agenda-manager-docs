# AuditRecord

- **Aggregate Root**: `AuditRecord`
- **Namespace**: `AgendaManager.Domain.AuditRecords`
- **Tipo**: Entidad de Dominio Sellada (sealed)
- **Herencia**: `AggregateRoot`

## Descripción General

La clase `AuditRecord` representa un registro de auditoría en el sistema, que captura los cambios realizados de un agregado.

### Responsabilidades

- Almacenar la información de auditoría, incluyendo el identificador único del agregado, el espacio de nombres del agregado, el nombre de la propiedad auditada, el valor anterior, el valor actual y el tipo de cambio.

## Propiedades

| Nombre          | Tipo                    | Descripción                                       |
| --------------- | ----------------------- | ------------------------------------------------- |
| `Id`            | `AuditRecordId`         | Identificador único del registro de auditoría     |
| `AggregateId`   | `string`                | Identificador único del agregado auditado         |
| `NamespaceName` | `string`                | Espacio de nombres del agregado auditado          |
| `PropertyName`  | `string`                | Nombre de la propiedad auditada                   |
| `OldValue`      | `string`                | Valor anterior de la propiedad auditada           |
| `NewValue`      | `string`                | Valor nuevo de la propiedad auditada              |
| `ActionType`    | `AuditRecordActionType` | Tipo de acción realizada en la propiedad auditada |

## Invariantes

- `Id` no puede ser nulo o vacío y debe ser único en la aplicación
- `AggregateId` no puede ser nulo o vacío.
- `NamespaceName` no puede ser nulo o vacío.
- `PropertyName` no puede ser nulo o vacío.
- `OldValue` no puede ser nulo o vacío.
- `NewValue` no puede ser nulo o vacío.
- `ActionType` no puede ser nulo y debe ser un valor válido de `AuditRecordActionType`.

## Reglas de negocio

- **Unicidad de Identidad**:
  - `Id` debe ser único en toda la aplicación.

## Métodos

### Create

```csharp
public static AuditRecord Create(
    AuditRecordId id,
    string aggregateId,
    string namespaceName,
    string aggregateName,
    string propertyName,
    string oldValue,
    string newValue,
    AuditRecordActionType actionType)
```

- **Descripción**: Crea una nueva instancia de `AuditRecord`.
- **Parámetros**:
  - `id`: Identificador único del registro de auditoría.
  - `aggregateId`: Identificador único del agregado auditado.
  - `namespaceName`: Espacio de nombres del agregado auditado.
  - `aggregateName`: Nombre del agregado auditado.
  - `propertyName`: Nombre de la propiedad auditada.
  - `oldValue`: Valor anterior de la propiedad auditada.
  - `newValue`: Valor nuevo de la propiedad auditada
  - `actionType`: Tipo de acción realizada en la propiedad auditada.
- **Eventos**: `AuditRecordCreatedDomainEvent` se dispara cuando se crea un nuevo registro de auditoría.
- **Retorno**: Una nueva instancia de `AuditRecord`.

## Estado y Transiciones

- **No Aplica**

## Dependencias

### Directas

- **Entidades Base**:
  - `AggregateRoot`: Base class que designa esta entidad como raíz de agregado, proporcionando control transaccional y consistencia del agregado
    - Hereda capacidades de auditoría (`AuditableEntity`)
    - Hereda gestión de eventos de dominio (`Entity`)

### Interfaces

- `IAuditRecordRepository`: Interfaz para el repositorio de registros de auditoría.

### Managers

- **No Aplica**

### Policies

- **No Aplica**

### Value Objects

- `AuditRecordId`: Identificador único del registro de auditoría.

## Comentarios adicionales

- **No Aplica**
