# User

- **Aggregate Root**: `CalendarConfiguration`
- **Namespace**: `AgendaManager.Domain.Calendars.Entities`
- **Tipo**: Entidad de Dominio Sellada (sealed)
- **Herencia**: `Entity`

## Descripción

Esta entidad se utiliza para almacenar las configuraciones específicas de un calendario. Las configuraciones son específicas para cada calendario y pueden ser modificadas por el usuario. Las opciones de cada configuración están definidas en `CalendarConfigurationKeys`, que actúa como un registro central de todas las configuraciones posibles y sus valores permitidos.

### Vista General

Las configuraciones de calendario permiten personalizar el comportamiento de cada calendario en la aplicación. Cada calendario tiene un conjunto de configuraciones que determinan cómo manejar:

- Creación y gestión de citas
- Comprobar si requiere validar los horarios de los recursos antes de crear una cita
- Manejo de solapamientos
- Gestión de días festivos
- Configuración de zona horaria

## Tipos de Configuración

### Configuraciones con Opciones Predefinidas

Configuraciones que solo permiten seleccionar entre un conjunto predefinido de valores:

- `AppointmentConfirmationStrategy`

  - **Descripción**: Define el estado inicial de las citas al ser creadas: automáticamente aceptadas (Accepted) o pendientes de confirmación por email (Pending)
  - **Opciones** AutoAccept, RequireConfirmation
  - **Default**: AutoAccept

- `AppointmentOverlappingStrategy`:

  - **Descripción**: Define cómo gestionar los conflictos temporales al crear o modificar citas: permitir la creación aunque existan solapamientos con otras citas, o rechazarla
  - **Opciones** AllowOverlapping, RejectIfOverlapping
  - **Default**: RejectIfOverlapping

- `HolidayConflictStrategy`

  - **Descripción**: Define cómo gestionar los conflictos cuando un día festivo se solapa con citas existentes: permitir que coexistan, rechazar la operación, o cancelar automáticamente las citas afectadas
  - **Opciones** AllowOverlapping, RejectIfOverlapping, CancelOverlapping
  - **Default**: RejectIfOverlapping

- `ResourceAvailabilityStrategy`
  - **Descripción**: Define si se validan los horarios de disponibilidad de los recursos requeridos al crear una cita. Los requisitos del servicio siempre se comprueban, pero la validación de los horarios puede ser opcional
  - **Opciones** ValidateSchedules, IgnoreSchedules
  - **Default**: ValidateSchedules

### Configuraciones de Valor Único (UnitValue)

Configuraciones que aceptan un valor personalizado con validación específica:

- IanaTimeZone (ejemplo: "Europe/Madrid")

### Responsabilidades

- **Creación de una nueva configuración**:

  - Añadir los valores por defecto de las opciones de configuración.
  - Los valores con una `Key` -> `UnitValue`, se deberá proporcionar un valor obtenido por un usuario.

- **Actualización de una configuración**:

  - Comprobar si la categoría de configuración existe en `CalendarConfigurationKeys.Metadata.Options`.
  - Se deberán proporcionar la `Category` y `SelectedKey` de la configuración a editar.
  - Se deberá comprobar si la `Category` y `SelectedKey` existen en `CalendarConfigurationKeys.Metadata.Options`.
    - Si la `Category` tiene una `Key` `UnitValue`, dependerá de la configuración que se haya asignado en `CalendarConfigurationKeys`.

- **Validación de valores UnitValue**:

  - Para configuraciones con `Key = 'UnitValue'`, validar que el valor proporcionado cumple con el formato esperado (ej: formato de zona horaria IANA).
  - La creación de una nueva configuración deberá permitir generar un lambda para la validación de los valores.

  ## Propiedades

| Nombre        | Tipo                      | Descripción                                                     |
| ------------- | ------------------------- | --------------------------------------------------------------- |
| `Id`          | `CalendarConfigurationId` | Identificador único de la configuración.                        |
| `CalendarId`  | `CalendarId`              | Identificador del calendario al que pertenece la configuración. |
| `Calendar`    | `Calendar`                | Referencia al calendario al que pertenece la configuración.     |
| `Category`    | `string`                  | Categoría de la configuración.                                  |
| `SelectedKey` | `string`                  | Clave seleccionada de la configuración.                         |

## Invariantes

- `Id` no puede ser `null` en ningún momento y debe ser único en toda la aplicación.
- `CalendarId` no puede ser `null` en ningún momento.
- `CalendarId` y `Category` deben ser únicos en toda la aplicación.
- `Category` no puede ser nulo y debe tener entre 1 y 100 caracteres.
- `SelectedKey` no puede ser nulo y debe tener entre 1 y 100 caracteres.

## Reglas de negocio

- **Unicidad de Identificador**:

  - `Id` debe ser único en toda la aplicación.
  - `CalendarId` y `Category` deben ser únicos en toda la aplicación.

- **Validación de Valores**:
  - Los valores para configuraciones con `Key = 'UnitValue'` deben cumplir con el formato específico de cada categoría.
  - Para otras configuraciones, `SelectedKey` debe existir en `CalendarConfigurationKeys` para la categoría correspondiente.

## Métodos

### Create

```csharp
internal static CalendarConfiguration Create(
    CalendarConfigurationId id,
    CalendarId calendarId,
    string category,
    string selectedKey)
```

- **Descripción**: Crea una nueva configuración de calendario.
- **Parámetros**:
  - `id`: Identificador único de la configuración.
  - `calendarId`: Identificador del calendario al que pertenece la configuración.
  - `category`: Categoría de la configuración.
  - `selectedKey`: Clave seleccionada de la configuración.
- **Retorna**: Nueva instancia de `CalendarConfiguration`.

### Update

```csharp
internal bool Update(string category, string selectedKey)
```

- **Descripción**: Actualiza la configuración de calendario.
- **Parámetros**:
  - `category`: Categoría de la configuración.
  - `selectedKey`: Nueva clave seleccionada para la configuración.
- **Retorna**: `true` si la configuración fue actualizada, `false` si no hubo cambios.

### IsUnitValueConfiguration

```csharp
internal bool IsUnitValueConfiguration()
```

- **Descripción**: Verifica si la configuración es de tipo `UnitValue`.
- **Retorna**: `true` si la configuración es de tipo `UnitValue`, `false` en caso contrario.

### GuardAgainstInvalidConfiguration

```csharp
private void GuardAgainstInvalidConfiguration(string category, string selectedKey)
```

- **Descripción**: Verifica que la configuración sea válida según las reglas definidas en `CalendarConfigurationKeys`.
- **Parámetros**:
  - `category`: Categoría de la configuración.
  - `selectedKey`: Clave seleccionada de la configuración.
- **Excepciones**:
  - `CalendarConfigurationDomainException`: Si la configuración no es válida.

### GuardAgainstInvalidCategory

```csharp
private void GuardAgainstInvalidCategory(string category)
```

- **Descripción**: Verifica que la categoría de la configuración sea válida.
- **Parámetros**:
  - `category`: Categoría de la configuración.
- **Excepciones**:
  - `CalendarConfigurationDomainException`: Si la categoría no es válida.

### GuardAgainstInvalidSelectedKey

```csharp
private void GuardAgainstInvalidSelectedKey(string selectedKey)
```

- **Descripción**: Verifica que la clave seleccionada de la configuración sea válida.
- **Parámetros**:
  - `selectedKey`: Clave seleccionada de la configuración.
- **Excepciones**:
  - `CalendarConfigurationDomainException`: Si la clave seleccionada no es válida.

### HasChanges

```csharp
private bool HasChanges(string category, string selectedKey)
```

- **Descripción**: Verifica si la configuración ha cambiado.
- **Parámetros**:
  - `category`: Nueva categoría de la configuración.
  - `selectedKey`: Nueva clave seleccionada de la configuración.
- **Retorna**: `true` si la configuración ha cambiado, `false` en caso contrario.

## Estado y Transiciones

- **No Aplica**

## Dependencias

### Directas

- **Entidades Base**:

  - `AuditableEntity`: Base class que proporciona capacidades de auditoría
    - Hereda gestión de eventos de dominio (`Entity`)

- `Calendar`: Entidad que representa un calendario.

### Interfaces

- `ICalendarConfigurationRepository`: Repositorio de configuraciones de calendario.

### Managers

- **No Aplica**

### Policies

- **No Aplica**

### Value Objects

- `CalendarConfigurationId`: Identificador único de la configuración.
- `CalendarId`: Identificador único del calendario.

### Errores

### Unexpected

- **Identifier**: `KeyNotFound` Sel lanza cuando no se encuentra una configuración con la clave especificada.
  - **Code**: `CalendarConfigurationErrors.KeyNotFound`
  - **Description**: Key not found.

## Ejemplos de Uso

### Consultar configuración

```csharp
var strategy = await calendarConfigurationRepository
    .GetBySelectedKeyAsync(
        calendarId,
        CalendarConfigurationKeys.Holidays.CreateStrategy);

if (strategy is CalendarConfigurationKeys.Holidays.CreationOptions.RejectIfOverlapping)
{
    // Lógica específica para rechazar solapamientos
}
```

### Validar configuración

```csharp
if (CalendarConfigurationKeys.Metadata.IsValidConfiguration(category, selectedKey))
{
    // Configuración válida, proceder con la operación
}
```

### Obtener todas las configuraciones disponibles

```csharp
var configurations = CalendarConfigurationKeys.Metadata.Options.Values
    .Select(option => new
    {
        Category = option.Category,
        Description = option.Description,
        IsUnitValue = option.IsUnitValue,
        AvailableOptions = option.AvailableKeys
    });
```

## Glosario

- **Category**: Identifica un grupo de configuraciones relacionadas (ej: IAppointmentConfirmationStrategyPolicy)
- **SelectedKey**: El valor específico elegido para una configuración (ej: AutoAccept)
- **UnitValue**: Configuración que acepta un valor personalizado con validación (ej: IanaTimeZone)
- **DefaultKey**: Valor predeterminado para una configuración cuando se crea un nuevo calendario

## Comentarios adicionales

### ConfigurationOption

```csharp
public class ConfigurationOption
{
    public string Category { get; }           // Categoría de la configuración
    public string? DefaultKey { get; }        // Valor por defecto (null para UnitValue)
    public string[] AvailableKeys { get; }    // Valores permitidos
    public bool IsUnitValue { get; }          // Indica si acepta valor personalizado
    public Func<string, bool>? Validator { get; }  // Validador para UnitValue
    public string Description { get; }        // Descripción de la configuración
}
```

### Estructura en Base de Datos

Esta es una representación de la tabla `CalendarConfigurations` en la base de datos:

| Id                                   | CalendarId                           | Category                               | SelectedKey         |
| ------------------------------------ | ------------------------------------ | -------------------------------------- | ------------------- |
| 2728eefe-7fab-4363-b9f9-6553aaf9cf35 | 6c8926c3-aed5-4ff5-960b-39def35143fc | AppointmentOverlappingStrategy         | RejectIfOverlapping |
| 3ab3370e-bfc7-4edf-8049-d99be824ced7 | 6c8926c3-aed5-4ff5-960b-39def35143fc | IanaTimeZone                           | Europe/Madrid       |
| ab7e15a4-2ef1-430f-b283-3daf3e4361ec | 6c8926c3-aed5-4ff5-960b-39def35143fc | IAppointmentConfirmationStrategyPolicy | AutoAccept              |
| b23980da-5021-4e2f-8c0e-159e6477f47c | 6c8926c3-aed5-4ff5-960b-39def35143fc | HolidayConflictStrategy                | RejectIfOverlapping |

### Guía: Añadir una Nueva Configuración

#### 1. Definir la Configuración en Código

```csharp
// En CalendarConfigurationKeys.cs
public static class NewFeature
{
    public const string Category = "NewFeatureStrategy";
    public static class Options
    {
        public const string OptionA = "OptionA";
        public const string OptionB = "OptionB";
    }
}

// En la sección Metadata.Options
[NewFeature.Category] = new ConfigurationOption(
    category: NewFeature.Category,
    defaultKey: NewFeature.Options.OptionA,
    availableKeys: new[]
    {
        NewFeature.Options.OptionA,
        NewFeature.Options.OptionB
    },
    description: "Description of new feature"
)

// Para UnitValue, usar este patrón:
public static class CustomFeature
{
    public const string Category = "CustomFeatureStrategy";
    public const string Key = "UnitValue";
}
```

#### 2. Crear Migración de Base de Datos

```sql
-- Para cada calendario existente, insertar la nueva configuración con su valor por defecto
INSERT INTO "CalendarConfiguration"
(Id, CalendarId, Category, SelectedKey, CreatedAt, CreatedBy, LastModifiedAt, LastModifiedBy, Version)
SELECT
    gen_random_uuid() AS Id,
    c.CalendarId AS CalendarId,
    'NewFeatureStrategy' AS Category,
    'OptionA' AS SelectedKey,  -- El DefaultKey de tu nueva regla
    CURRENT_TIMESTAMP AS CreatedAt,
    'System' AS CreatedBy,
    CURRENT_TIMESTAMP AS LastModifiedAt,
    'System' AS LastModifiedBy,
    1 AS Version
FROM "CalendarConfiguration" c
WHERE c.CalendarId NOT IN (
    SELECT DISTINCT CalendarId
    FROM "CalendarConfiguration"
    WHERE Category = 'NewFeatureStrategy'
)
GROUP BY c.CalendarId;
```

#### 3. Implementar la Migración en EF Core

```csharp
public class AddNewRuleConfiguration : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.Sql(@"
            INSERT INTO ""CalendarConfiguration"" (
                ""Id"", ""CalendarId"", ""Category"", ""SelectedKey"",
                ""CreatedAt"", ""CreatedBy"", ""LastModifiedAt"", ""LastModifiedBy"", ""Version"")
            SELECT
                gen_random_uuid(),
                c.""CalendarId"",
                'NewFeatureStrategy',
                'OptionA',
                CURRENT_TIMESTAMP,
                'System',
                CURRENT_TIMESTAMP,
                'System',
                1
            FROM ""CalendarConfiguration"" c
            WHERE c.""CalendarId"" NOT IN (
                SELECT DISTINCT ""CalendarId""
                FROM ""CalendarConfiguration""
                WHERE ""Category"" = 'NewFeatureStrategy'
            )
            GROUP BY c.""CalendarId"";
        ");
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.Sql(@"
            DELETE FROM ""CalendarConfiguration""
            WHERE ""Category"" = 'NewFeatureStrategy';
        ");
    }
}
```

#### 4. Checklist de Implementación

- [ ] Añadir constantes en `CalendarConfigurationKeys`
- [ ] Definir configuración en `Metadata.Options`
- [ ] Crear migración de base de datos
- [ ] Actualizar tests
- [ ] Actualizar documentación si es necesario
- [ ] Validar funcionamiento en ambiente de desarrollo
