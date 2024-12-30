# ResourceType

- **Aggregate Root**: `ResourceType`
- **Namespace**: `AgendaManager.Domain.ResourceTypes`
- **Tipo**: Entidad de Dominio Sellada (sealed)
- **Herencia**: `AggregateRoot`

## Descripción General

El agregado root `ResourceType` representa un tipo de recurso en el sistema de gestión de agenda. Define la naturaleza y características base de los recursos que pueden ser gestionados en el sistema. Cada tipo de recurso puede representar tanto recursos humanos como recursos no humano.

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

- **Eventos de Dominio**:

  - Disparar eventos de dominio cuando se realicen cambios significativos en el tipo de recurso.

- **Validación**:
  - Asegurarse de que `Name` y `Description` no exceden la longitud permitida.

## Propiedades

| Propiedad     | Tipo                      | Descripción                                              |
| ------------- | ------------------------- | -------------------------------------------------------- |
| `Id`          | `ResourceTypeId`          | Identificador único del tipo de recurso.                 |
| `Category`    | `ResourceCategory`        | Categoría del tipo de recurso (humano o físico).         |
| `Name`        | `string`                  | Nombre del tipo de recurso.                              |
| `Description` | `string`                  | Descripción del tipo de recurso.                         |
| `Resources`   | `IReadOnlyList<Resource>` | Colección de recursos asociados al tipo de recurso.      |
| `Services`    | `IReadOnlyList<Service>`  | Colección de servicios que requieren el tipo de recurso. |

## Invariantes

- `Id` no puede ser `null` en ningún momento
- `Category` no puede ser `null` en ningún momento y debe ser uno de los valores de `ResourceCategory`
- `Name` no puede ser `null` o vacío en ningún momento, no debe exceder los 50 caracteres
- `Description` no puede ser `null` o vacío en ningún momento, no debe exceder los 500 caracteres

## Reglas de negocio

- **Unicidad de Identificador**:

  - `Id` debe ser único en toda la aplicación.
  - `Name` debe ser único en toda la aplicación.

- **Nombre y Descripción**:

  - El nombre del tipo de recurso debe ser único en toda la aplicación y debe tener entre 1 y 50 caracteres.
  - La descripción del calendario debe tener entre 1 y 500 caracteres.

- **Categoría**:

  - Un `ResourceType` puede ser uno de estos tipos:
    - `Staff`: Recurso humano que requiere de un rol `Employee`
    - `Place`: Espacio físico requerido para realizar
    - `Equipment`: Equipo o recurso físico necesario para realizar

- **Eliminación**:

  - No se puede eliminar un `ResourceType` que tenga recursos asociados
  - No se puede eliminar un `ResourceType` que esté siendo usado en servicios

- **Modificación**:

  - Un `ResourceType` solo permite modificar su nombre y descripción
  - La `Category` se establece en la creación y no puede ser modificado posteriormente
  - La modificación del nombre debe mantener la restricción de unicidad
  - La modificación de la descripción debe mantener las restricciones de longitud

- **Gestión y Validaciones**:

  - Toda creación y modificación debe realizarse a través de `ResourceTypeManager`
  - Los métodos de creación y modificación del agregado son `internal` para forzar el uso del manager
  - El manager es responsable de:
    - Verificar la unicidad del nombre en la base de datos
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

## Enums

- `ResourceCategory`: Enumeración que define las categorías de un tipo de recurso.

### Errores

#### Conflict

- **Identifier**: `CannotDeleteResourceType`

  - **Code**: `ResourceTypeErrors.CannotDeleteResourceType`
  - **Descripción**: Cannot delete the resource type.

#### NotFound

- **Identifier**: `ResourceTypeNotFound`

  - **Code**: `ResourceTypeErrors.ResourceTypeNotFound`
  - **Descripción**: Indica que un tipo de recurso no fue encontrado.

#### Validation

- **Identifier**: `NameAlreadyExists` Se lanza cuando el nombre del tipo de recurso ya existe.

  - **Code**: `Name`
  - **Description**: The name of the resource type already exists.

## Comentarios adicionales

- **No Aplica**

### Ejemplos de Tipos de Recursos

#### Clínica Dental

- **Con Role**:

  - **Nombre**: "Odontólogo"
  - **Descripción**: "Profesional especializado en diagnóstico y tratamiento dental"
  - **Category**: `ResourceCategory.Staff`

- **Sin Role**:

  - **Nombre**: "Gabinete Dental"
  - **Descripción**: "Sala equipada con unidad dental completa"
  - **Category**: `ResourceCategory.Place`

#### Empresa de Calesas

- **Con Role**:

  - **Nombre**: "Cochero"
  - **Descripción**: "Conductor profesional de carruajes"
  - **Category**: `ResourceCategory.Staff`

- **Sin Role**:

  - **Nombre**: "Carruaje"
  - **Descripción**: "Vehículo de tracción animal para pasajeros"
  - **Category**: `ResourceCategory.Equipment`

## Ejemplos de Uso

```csharp
// ❌ No permitido (los métodos son internal)
var resourceType = ResourceType.Create(...);

// ✅ Uso correcto a través del manager
var resourceType = await resourceTypeManager.CreateResourceTypeAsync(...);
```
