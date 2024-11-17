Hola, una pregunta.

Tengo este código:

```csharp
public void UpdateConfiguration(CalendarConfigurationId configurationId, string category, string selectedKey)
{
    var calendarConfiguration = _configurations.FirstOrDefault(x => x.Id == configurationId);

    if (calendarConfiguration is null)
    {
        throw new CalendarConfigurationDomainException("Invalid calendar configuration option.");
    }

    if (calendarConfiguration.Update(category, selectedKey))
    {
        calendarConfiguration.AddDomainEvent(
            new CalendarConfigurationUpdatedDomainEvent(Id, calendarConfiguration.Id));
    }
}
```

Mi pregunta es, cuando lanzo el evento `CalendarConfigurationUpdatedDomainEvent`, ¿lo debo lanzar tal y como esta o debería ser `AddDomainEvent(new CalendarConfigurationUpdatedDomainEvent(Id, calendarConfiguration.Id));`? sin `CalendarConfiguration.`
