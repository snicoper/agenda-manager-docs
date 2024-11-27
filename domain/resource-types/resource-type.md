# ResourceType

- **Aggregate Root**: `ResourceType`
- **Namespace**: `AgendaManager.Domain.ResourceTypes`
- **Tipo**: Entidad de Dominio Sellada (sealed)
- **Herencia**: `AggregateRoot`

## Descripción General

El agregado root `ResourceType` representa un tipo de recurso en el sistema de gestión de agenda. Define la naturaleza y características base de los recursos que pueden ser gestionados en el sistema. Cada tipo de recurso puede representar tanto recursos humanos (que requieren un rol) como recursos físicos (sin rol asociado).

Un `ResourceType` sirve como plantilla para la creación de recursos específicos y es fundamental para:

- Definir qué tipos de recursos pueden existir en el sistema
- Especificar qué recursos son necesarios para los servicios
- Categorizar y organizar los recursos disponibles

### Responsabilidades

- **Clasificación**:

  - Definir si un tipo de recurso requiere un rol (recursos humanos) o no (recursos físicos)
  - Servir como base para la creación de recursos específicos

- **Integridad**:

  - Mantener la consistencia en la definición de tipos de recursos
  - Asegurar que los tipos de recursos con rol tengan un rol válido

- **Eventos de Dominio**:

  - Disparar eventos de dominio cuando se realicen cambios significativos en el tipo de recurso.

- **Validación**:
  - Asegurarse de que `Name` y `Description` no exceden la longitud permitida.

## Propiedades

| Propiedad     | Tipo                      | Descripción                                         |
| ------------- | ------------------------- | --------------------------------------------------- |
| `Id`          | `ResourceTypeId`          | Identificador único del tipo de recurso.            |
| `Name`        | `string`                  | Nombre del tipo de recurso.                         |
| `Description` | `string`                  | Descripción del tipo de recurso.                    |
| `RoleId`      | `RoleId?`                 | Identificador del rol asociado al tipo de recurso.  |
| `Resources`   | `IReadOnlyList<Resource>` | Colección de recursos asociados al tipo de recurso. |

## Invariantes

- `Id` no puede ser `null` en ningún momento
- `Name` no puede ser `null` o vacío en ningún momento, no debe exceder los 50 caracteres
- `Description` no puede ser `null` o vacío en ningún momento, no debe exceder los 500 caracteres
- `RoleId` puede ser `null` o no, pero si no es `null` debe ser un valor válido

## Reglas de negocio

- **Unicidad de Identificador**:

  - `Id` debe ser único en toda la aplicación.
  - `RoleId` si no es `null` debe ser único en toda la aplicación.
  - `Name` debe ser único en toda la aplicación.

- **Nombre y Descripción**:

  - El nombre del tipo de recurso debe ser único en toda la aplicación y debe tener entre 1 y 50 caracteres.
  - La descripción del calendario debe tener entre 1 y 500 caracteres.

- **Rol Asociado**:

  - Un `ResourceType` puede tener o no un rol asociado
  - Si tiene rol, representa un recurso que requiere personal (ejemplo: Odontólogo)
  - Si no tiene rol, representa un recurso físico (ejemplo: Gabinete)

- **Eliminación**:

  - No se puede eliminar un `ResourceType` que tenga recursos asociados
  - No se puede eliminar un `ResourceType` que esté siendo usado en servicios

- **Modificación**:

  - Un `ResourceType` solo permite modificar su nombre y descripción
  - El `RoleId` se establece en la creación y no puede ser modificado posteriormente
  - La modificación del nombre debe mantener la restricción de unicidad
  - La modificación de la descripción debe mantener las restricciones de longitud

- **Gestión y Validaciones**:

  - Toda creación y modificación debe realizarse a través de `ResourceTypeManager`
  - Los métodos de creación y modificación del agregado son `internal` para forzar el uso del manager
  - El manager es responsable de:
    - Verificar la unicidad del nombre en la base de datos
    - Validar la existencia y validez del rol si se especifica
    - Aplicar reglas de negocio que requieran consulta a base de datos
  - Esta restricción garantiza:
    - La aplicación de todas las validaciones de negocio
    - La consistencia en la creación/modificación
    - El control transaccional adecuado

- **Uso en Servicios**:
  - Un `ResourceType` puede ser requerido por múltiples servicios
  - Define qué tipos de recursos son necesarios para realizar un servicio

## Métodos

### Create

```csharp
internal static ResourceType Create(ResourceTypeId id, string name, string description, RoleId? roleId = null)
```

- Crea una nueva instancia de `ResourceType`.
- `id`: Identificador único del tipo de recurso.
- `name`: Nombre del tipo de recurso.
- `description`: Descripción del tipo de recurso.
- `roleId`: Identificador del rol asociado al tipo de recurso.
- **Eventos** Lanza un evento `ResourceTypeCreatedDomainEvent` con la instancia de `ResourceType` creada.
- **Retorna** Una nueva instancia de `ResourceType`.

### Update

```csharp
internal void Update(string name, string description)
```

- Actualiza el nombre y la descripción de un tipo de recurso existente.
- `name`: Nuevo nombre del tipo de recurso.
- `description`: Nueva descripción del tipo de recurso.
- **Eventos** Lanza un evento `ResourceTypeUpdatedDomainEvent` con la instancia de `ResourceType` actualizada.
- **Retorna** `void`.

### GuardAgainstInvalidName

```csharp
private static void GuardAgainstInvalidName(string name)
```

- Valida que el nombre del tipo de recurso sea válido.
- `name`: Nombre del tipo de recurso.
- **Excepciones** Lanza `ResourceTypeDomainException` si el nombre no es válido.
- **Retorna** `void`.

### GuardAgainstInvalidDescription

```csharp
private static void GuardAgainstInvalidDescription(string description)
```

- Valida que la descripción del tipo de recurso sea válida.
- `description`: Descripción del tipo de recurso.
- **Excepciones** Lanza `ResourceTypeDomainException` si la descripción no es válida.
- **Retorna** `void`.

## Estado y Transiciones

- **No Aplica**

## Dependencias

### Directas

- **Entidades Base**:
  - `AggregateRoot`: Base class que designa esta entidad como raíz de agregado, proporcionando control transaccional y consistencia del agregado
    - Hereda capacidades de auditoría (`AuditableEntity`)
    - Hereda gestión de eventos de dominio (`Entity`)

### Interfaces

- `IResourceTypeRepository`: Define el contrato para la persistencia de `ResourceType`.

### Managers

- `ResourceTypeManager`: Gestiona la lógica de negocio relacionada con `ResourceType`.

### Policies

### Value Objects

- `ResourceTypeId`: Representa el identificador único de un tipo de recurso.
- `RoleId`: Representa el identificador único de un rol.

### Errores

### NotFound

- **Identifier**: `ResourceTypeNotFound`
  - **Code**: `ResourceTypeErrors.ResourceTypeNotFound`
  - **Descripción**: Indica que un tipo de recurso no fue encontrado.

### Validation

- **Identifier**: `NameAlreadyExists` Se lanza cuando el nombre del tipo de recurso ya existe.
  - **Code**: `Name`
  - **Description**: The name of the resource type already exists.

- **Identifier**: `DescriptionExists` Se lanza cuando la descripción del tipo de recurso ya existe.
  - **Code**: `Description`
  - **Description**: The description of the resource type already exists.

## Comentarios adicionales

- **No Aplica**

### Ejemplos de Tipos de Recursos

#### Clínica Dental

- **Con Role**:

  - **Nombre**: "Odontólogo"
  - **Descripción**: "Profesional especializado en diagnóstico y tratamiento dental"
  - **Role**: Employee

- **Sin Role**:
  - **Nombre**: "Gabinete Dental"
  - **Descripción**: "Sala equipada con unidad dental completa"
  - **Role**: null

#### Empresa de Calesas

- **Con Role**:

  - **Nombre**: "Cochero"
  - **Descripción**: "Conductor profesional de carruajes"
  - **Role**: Employee

- **Sin Role**:
  - **Nombre**: "Carruaje"
  - **Descripción**: "Vehículo de tracción animal para pasajeros"
  - **Role**: null

## Ejemplos de Uso

```csharp
// ❌ No permitido (los métodos son internal)
var resourceType = ResourceType.Create(...);

// ✅ Uso correcto a través del manager
var resourceType = await resourceTypeManager.CreateResourceTypeAsync(...);
```
