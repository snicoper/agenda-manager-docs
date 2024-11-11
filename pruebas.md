Ahora en el constructor de los **ValueObject**

```csharp
public class UserId : ValueObject
{
    private UserId(Guid value)
    {
        ArgumentNullException.ThrowIfNull(value);

        Value = value;
    }

    //  ...
}
```

De esta manera me ahorro hacer la validación en el constructor de las entidades.
