# README.md

TODO: Mover a documentación apuntes-next

Enum con `[Flags]` `WeekDays`

```csharp
[Flags]
public enum WeekDays
{
    None = 0,
    Monday = 1 << 0,
    Tuesday = 1 << 1,
    Wednesday = 1 << 2,
    Thursday = 1 << 3,
    Friday = 1 << 4,
    Saturday = 1 << 5,
    Sunday = 1 << 6,
    Weekdays = Monday | Tuesday | Wednesday | Thursday | Friday,
    Weekend = Saturday | Sunday,
    All = Weekdays | Weekend
}
```

En front end

```ts
// Angular - Interface
interface Schedule {
  workDays: number;
}

// Angular - Enum
export enum WeekDays {
  None = 0,
  Monday = 1,
  Tuesday = 2,
  Wednesday = 4,
  Thursday = 8,
  Friday = 16,
  Saturday = 32,
  Sunday = 64,
  Weekdays = Monday | Tuesday | Wednesday | Thursday | Friday,
  Weekend = Saturday | Sunday,
  All = Weekdays | Weekend
}

// Angular - Uso
const schedule: Schedule = {
  workDays: WeekDays.Monday | WeekDays.Wednesday
};
```

FluentValidation

```csharp
public class CreateResourceRequestValidator
    : AbstractValidator<CreateResourceRequest>
{
    public CreateResourceRequestValidator()
    {
        RuleFor(x => x.AvailableDays)
            .NotNull()
            .WithMessage("Los días disponibles son requeridos");

        RuleForEach(x => x.AvailableDays)
            .InclusiveBetween(1, 7)
            .WithMessage("Los días deben estar entre 1 y 7");

        RuleFor(x => x.AvailableDays)
            .Must(days => days.Distinct().Count() == days.Length)
            .WithMessage("No puede haber días duplicados");
    }
}
```

Method Extensions

```csharp
public static class WeekDaysExtensions
{
    /// <summary>
    /// Converts the given <see cref="WeekDays" /> to an array of their corresponding day numbers.
    /// </summary>
    /// <param name="weekDays">The input <see cref="WeekDays" />.</param>
    /// <returns>An array of day numbers.</returns>
    /// <remarks>
    /// The array will be ordered by day number.
    /// </remarks>
    public static int[] ToNumberArray(this WeekDays weekDays)
    {
        var days = Enum.GetValues<WeekDays>()
            .Where(
                d => d != WeekDays.None &&
                     d != WeekDays.All &&
                     d != WeekDays.Weekend &&
                     d != WeekDays.Weekdays &&
                     weekDays.HasFlag(d))
            .Select(d => (int)Math.Log2((int)d) + 1)
            .OrderBy(d => d)
            .ToArray();

        return days;
    }

    /// <summary>
    /// Converts an array of day numbers to the corresponding <see cref="WeekDays" /> value.
    /// </summary>
    /// <param name="days">An array of integers representing the days of the week (1 for Monday, 7 for Sunday).</param>
    /// <returns>
    /// A <see cref="WeekDays" /> value representing the combination of days specified in the array.
    /// Returns <see cref="WeekDays.None" /> if the array is null or empty.
    /// </returns>
    /// <exception cref="WeekDaysException">
    /// Thrown when any day number is out of the 1-7 range or if there are duplicate day numbers.
    /// </exception>
    public static WeekDays FromNumberArray(this int[]? days)
    {
        if (days is null || days.Length == 0)
        {
            return WeekDays.None;
        }

        // Validación de rango.
        if (days.Any(d => d is < 1 or > 7))
        {
            throw new WeekDaysException("The day range must be between 1 and 7.");
        }

        // Validación de duplicados.
        if (days.Distinct().Count() != days.Length)
        {
            throw new WeekDaysException("There can be no duplicate days.");
        }

        return days.Aggregate(WeekDays.None, (current, day) => current | (WeekDays)(1 << (day - 1)));
    }
}
```

## NO BORRAR

```text
# Los Mandamientos del EPA (El Puto Amo / Aggregate Root)

1. El EPA siempre tiene la razón
   - El Aggregate Root es la fuente de verdad y consistencia

2. Si el EPA no tiene la razón, consulte el primer mandamiento
   - No hay excepciones a las reglas del agregado

3. El EPA solo habla a través de sus eventos, el resto es ruido
   - La comunicación con otros agregados debe ser a través de eventos de dominio

4. No tocarás las entidades del EPA sin su permiso
   - Toda modificación debe pasar por el Aggregate Root

5. No modificarás directamente lo que el EPA protege
   - No bypass de la encapsulación

6. Honrarás al EPA y sus invariantes
   - Respetar siempre las reglas de negocio del agregado

7. No codiciarás el estado interno de otro EPA
   - Mantener los agregados independientes

8. El EPA es el único que puede cambiar su estado
   - Encapsulación y control de modificaciones

9. No tomarás el nombre del EPA en vano
   - No hacer bypass de las validaciones del agregado

10. El EPA descansará el séptimo día
    - Pero siempre vigilando la consistencia 😉
```
