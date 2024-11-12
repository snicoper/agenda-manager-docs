Hola, estoy haciendo la configuracion de EF Core:

Pongo las partes del código importantes:

```csharp
public sealed class Calendar : AggregateRoot
{
    public CalendarId Id { get; } = null!;

    public CalendarSettingsId SettingsId { get; private set; } = null!;

    public CalendarSettings Settings { get; private set; } = null!;
}

public sealed class CalendarSettings : AuditableEntity
{
    public CalendarSettingsId Id { get; } = null!;

    public CalendarId CalendarId { get; private set; } = null!;

    public Calendar Calendar { get; private set; } = null!;
}
```

Es una relación de uno a uno.

```csharp
public class CalendarConfiguration : IEntityTypeConfiguration<Calendar>
{
    public void Configure(EntityTypeBuilder<Calendar> builder)
    {
        // ...
        builder.Property(c => c.SettingsId)
            .HasConversion(
                id => id.Value,
                value => CalendarSettingsId.From(value))
            .IsRequired();

        builder.HasOne(c => c.Settings)
            .WithOne(cs => cs.Calendar)
            .HasForeignKey<Calendar>(c => c.SettingsId)
            .OnDelete(DeleteBehavior.Cascade);
    }
}

public class CalendarSettingsConfiguration : IEntityTypeConfiguration<CalendarSettings>
{
    public void Configure(EntityTypeBuilder<CalendarSettings> builder)
    {
        // ...
        builder.Property(cs => cs.CalendarId)
            .HasConversion(
                id => id.Value,
                value => CalendarId.From(value))
            .IsRequired();

        builder.HasOne(cs => cs.Calendar)
            .WithOne(c => c.Settings)
            .HasForeignKey<CalendarSettings>(cs => cs.CalendarId)
            .OnDelete(DeleteBehavior.Cascade);
    }
}
```

El que "manda" es `Calendar` y lo deseable sería que no se pueda borrar un `CalendarSettings` si tiene un `Calendar` asociado.

Y si  se borra un `Calendar` se borra el `CalendarSettings` asociado.

Si me modificas la parte de los `OnDelete` y revisas si el resto es correcto, te lo agradezco.
