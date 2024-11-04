# ResourceSchedules

* `ResourceScheduleId` como identificador único.
* `ResourceId` como asociación de un `Resource`.
* `Period` ValueObject con `Start` y `EndDate`
* `ResourceScheduleType` como enumeración con `Available` o `Unavailable`.
* `DayOfWeek[]` como enumeración con `Monday`, `Tuesday`, etc. Días de la semana que se aplica el `ResourceSchedule` dentro del `Period`.

Si yo creo una regla de disponibilidad

```txt
ResourceScheduleId: 1
ResourceId: 123456789
Period: 2020-09-01T09:00:00Z - 2040-09-01T14:59:59Z
ResourceScheduleType: Available
DayOfWeek: Monday, Tuesday, Wednesday, Thursday, Friday
```

Esto significa que desde el 1 de septiembre de 2020 hasta el 1 de septiembre de 2040, el recurso con ID 123456789 estará disponible de lunes a viernes, de 9:00 a 14:59.

De hay las `Unavailable`, imaginemos que el recurso con ID 123456789 se va de vacaciones del 1 de agosto de 2024 hasta el 1 de septiembre de 2024, entonces el recurso estará `Unavailable` durante ese periodo.

```txt
ResourceScheduleId: 1
ResourceId: 123456789
Period: 2024-08-01T09:00:00Z - 2024-09-01T14:59:59Z
ResourceScheduleType: Unavailable
DayOfWeek: Monday, Tuesday, Wednesday, Thursday, Friday, Saturday, Sunday
```
