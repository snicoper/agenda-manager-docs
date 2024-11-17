# README.md

Actualmente estas son las tablas de la base de datos:

## Tabla `CalendarConfigurationOptions`

Esta es la que estamos mirando de eliminar

```textplain
OptionId                            |Category                      |Key                |DefaultValue|Description          |CreatedBy|CreatedAt                    |LastModifiedBy|LastModifiedAt               |Version|
------------------------------------+------------------------------+-------------------+------------+---------------------+---------+-----------------------------+--------------+-----------------------------+-------+
2a0c43ff-64ae-4797-b35b-4a50e3c429a1|AppointmentCreationStrategy   |RequireConfirmation|false       |Require confirmation |System   |2024-11-16 20:17:13.380 +0100|System        |2024-11-16 20:17:13.380 +0100|      1|
2d32245d-3b77-492b-9de1-5229ba77046c|HolidayCreateStrategy         |RejectIfOverlapping|true        |Reject if overlapping|System   |2024-11-16 20:17:13.380 +0100|System        |2024-11-16 20:17:13.380 +0100|      1|
6d93cf82-21f0-45bd-b427-2080f475c2c8|AppointmentOverlappingStrategy|AllowOverlapping   |false       |Allow overlapping    |System   |2024-11-16 20:17:13.380 +0100|System        |2024-11-16 20:17:13.380 +0100|      1|
7dccd558-3d0b-41ca-a92c-1b9327d6dd16|HolidayCreateStrategy         |CancelOverlapping  |false       |Cancel overlapping   |System   |2024-11-16 20:17:13.380 +0100|System        |2024-11-16 20:17:13.380 +0100|      1|
ab89b7a9-1304-4c3a-ac3e-956b06fbef40|AppointmentCreationStrategy   |Direct             |true        |Direct               |System   |2024-11-16 20:17:13.380 +0100|System        |2024-11-16 20:17:13.380 +0100|      1|
e024653b-7b4c-49a9-8385-4bb35710e895|IanaTimeZone                  |UnitValue          |true        |Time zone            |System   |2024-11-16 20:17:13.380 +0100|System        |2024-11-16 20:17:13.380 +0100|      1|
eaea7f13-4f41-462e-a2a6-bffb3ee09757|HolidayCreateStrategy         |AllowOverlapping   |false       |Allow overlapping    |System   |2024-11-16 20:17:13.380 +0100|System        |2024-11-16 20:17:13.380 +0100|      1|
febb98ae-280b-436e-8aa4-56b538bf8e76|AppointmentOverlappingStrategy|RejectIfOverlapping|true        |Reject if overlapping|System   |2024-11-16 20:17:13.380 +0100|System        |2024-11-16 20:17:13.380 +0100|      1|
```

## Tabla `CalendarConfiguration`

Esta si que debe quedar

```textplain
Id                                  |CalendarId                          |Category                      |SelectedKey        |CreatedBy|CreatedAt                    |LastModifiedBy|LastModifiedAt               |Version|
------------------------------------+------------------------------------+------------------------------+-------------------+---------+-----------------------------+--------------+-----------------------------+-------+
2728eefe-7fab-4363-b9f9-6553aaf9cf35|6c8926c3-aed5-4ff5-960b-39def35143fc|AppointmentOverlappingStrategy|RejectIfOverlapping|System   |2024-11-16 20:17:15.334 +0100|System        |2024-11-16 20:17:15.334 +0100|      1|
3ab3370e-bfc7-4edf-8049-d99be824ced7|6c8926c3-aed5-4ff5-960b-39def35143fc|IanaTimeZone                  |Europe/Madrid      |System   |2024-11-16 20:17:15.334 +0100|System        |2024-11-16 20:17:15.334 +0100|      1|
ab7e15a4-2ef1-430f-b283-3daf3e4361ec|6c8926c3-aed5-4ff5-960b-39def35143fc|AppointmentCreationStrategy   |Direct             |System   |2024-11-16 20:17:15.334 +0100|System        |2024-11-16 20:17:15.334 +0100|      1|
b23980da-5021-4e2f-8c0e-159e6477f47c|6c8926c3-aed5-4ff5-960b-39def35143fc|HolidayCreateStrategy         |RejectIfOverlapping|System   |2024-11-16 20:17:15.334 +0100|System        |2024-11-16 20:17:15.334 +0100|      1|
```

Y tengo este script de SQL:

```sql
```sql
INSERT INTO "CalendarConfigurationOptions" ("Key", "DefaultValue", "Description", "CreatedAt", "CreatedBy")
VALUES ('NewFeatureStrategy', 'DefaultOption', 'Description of the new feature', CURRENT_TIMESTAMP, 'System');
```

```sql
INSERT INTO "CalendarConfigurations" ("CalendarId", "Category", "SelectedKey", "CreatedAt", "CreatedBy", "LastModifiedAt", "LastModifiedBy", "Version")
SELECT
    c."Id" AS "CalendarId",
    'NewFeatureStrategy' AS "Category",
    (SELECT "DefaultValue" FROM "CalendarConfigurationOptions" WHERE "Key" = 'NewFeatureStrategy') AS "SelectedKey",
    CURRENT_TIMESTAMP AS "CreatedAt",
    'System' AS "CreatedBy",
    CURRENT_TIMESTAMP AS "LastModifiedAt",
    'System' AS "LastModifiedBy",
    1 AS "Version"
FROM "Calendars" c
WHERE NOT EXISTS (
    SELECT 1
    FROM "CalendarConfigurations" cc
    WHERE cc."CalendarId" = c."Id"
      AND cc."Category" = 'NewFeatureStrategy'
);
```

Como se puede ver, el primer script inserta una nueva opción de configuración en la tabla `CalendarConfigurationOptions`. Luego, el segundo script selecciona los calendarios que aún no tienen una configuración para la nueva categoría y los inserta en la tabla `CalendarConfigurations` (con tantos `CalendarId` haya en la tabla).

Pero hay en `CalendarConfigurationOptions` un campo `Key` con posibles valores `UnitValue`, si nos fijamos, esos `Category` solo tienen un valor.

Por lo tanto el segundo script debería añadir en `CalendarConfigurations` el valor `Category` -> `Key` -> `DefaultValue` = true por cada `CalendarId` que haya en la tabla `CalendarConfigurations`.

Por otro lado, supongo, que me hará falta otro script para añadir por cada `CalendarId` en la tabla `CalendarConfigurations` un valor concreto para los que tengan `Key` = `UnitValue` en la tabla `CalendarConfigurations`.

Espero haberme explicado bien, por que es un poco lioso. 😥

---

Yo genero el calendario desde un manager, básicamente es lo mismo a tu ejemplo en el factory create de la clase `Calendar`.

En la parte que le añado las configuraciones, el time zone no requiere de validación ya que es un ValueObject y ya esta validado.

```csharp

// ...
private static Result<Calendar> AddDefaultConfigurationsAsync(
        Calendar calendar,
        CalendarId calendarId,
        IanaTimeZone ianaTimeZone,
        CancellationToken cancellationToken)
    {
        foreach (var config in CalendarConfigurationKeys2.Metadata.Options.Values.Where(o => !o.IsUnitValue))
        {
            calendar.AddConfiguration(
                CalendarConfiguration.Create(
                    id: CalendarConfigurationId.Create(),
                    calendarId: calendarId,
                    category: config.Category,
                    selectedKey: config.DefaultKey!));
        }

        calendar.AddConfiguration(
            CalendarConfiguration.Create(
                CalendarConfigurationId.Create(),
                calendarId,
                CalendarConfigurationKeys2.TimeZone.Category,
                ianaTimeZone.Value));

// ...
```

- **NOTA**: El `CalendarConfigurationKeys2` es por que yo tengo una clase `CalendarConfigurationKeys` y como estoy probando sin eliminar lo anterior...

---

Jajaja no no, has acertado en casí todo, menos en el `ErrorOr`, supongo que dices la de **Amichai Mantinband** pero, me hice uno propio para aprender y para tener el control de como manejar los errores.

Lo que te quería comentar, cuando ejecuto los test que ya tengo, el único que falla es por el IanaTimeZone.

```textplain
AgendaManager.Domain.Calendars.Exceptions.CalendarConfigurationDomainException: Invalid configuration combination: IanaTimeZone - Europe/Madrid
```
