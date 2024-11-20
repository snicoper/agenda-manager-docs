Como verías este enfoque, solo he echo un pequeño ejemplo y me comentas si es un enfoque correcto (o cercano a lo correcto)

```csharp
public sealed class User : AggregateRoot
{
    private readonly List<Role> _roles = [];

    public IReadOnlyCollection<Role> Roles => _roles.AsReadOnly();

    public bool HasRole(RoleId roleId)
    {
        return _roles.Any(x => x.Id == roleId);
    }

    public bool HasPermission(PermissionId permissionId)
    {
        return _roles.Any(role => role.HasPermission(permissionId));
    }

    public IReadOnlyList<Permission> GetAllPermissions()
    {
        return _roles.SelectMany(x => x.Permissions)
            .ToList()
            .AsReadOnly();
    }

    internal Result AddRole(Role role)
    {
        _roles.Add(role);

        AddDomainEvent(new UserRoleAddedDomainEvent(Id, role.Id));

        return Result.Success();
    }

    internal Result RemoveRole(Role role)
    {
        _roles.Remove(role);

        AddDomainEvent(new UserRoleRemovedDomainEvent(Id, role.Id));

        return Result.Success();
    }

    internal Result AddPermissionToRole(Role role, Permission permission)
    {
        if (role.HasPermission(permission.Id))
        {
            return Result.Success();
        }

        role.AddPermission(permission);

        return Result.Success();
    }

    internal Result RemovePermissionFromRole(Role role, Permission permission)
    {
        if (!role.HasPermission(permission.Id))
        {
            return Result.Success();
        }

        role.RemovePermission(permission);

        return Result.Success();
    }
}
```

Pueden faltar eventos, etc, hablamos del concepto.

```csharp
public class UserManager(IUserRepository userRepository)
{
    // ...

    // Ejemplo de uso
    public async Task<Result> AddRoleToUserAsync(UserId userId, Role role, CancellationToken cancellationToken)
    {
        var user = await userRepository.GetByIdWithPermissionsAsync(userId, cancellationToken);

        if (user is null)
        {
            return UserErrors.UserNotFound;
        }

        var addRoleResult = user.AddRole(role);

        if (addRoleResult.IsFailure)
        {
            return addRoleResult;
        }

        userRepository.Update(user);

        return Result.Success();
    }

    public async Task<Result> RemoveRoleFromUserAsync(UserId userId, Role role, CancellationToken cancellationToken) {}

    public async Task<Result> AddPermissionToUserAsync(UserId userId, Permission permission, CancellationToken cancellationToken) {}

    public async Task<Result> RemovePermissionFromUserAsync(UserId userId, Permission permission, CancellationToken cancellationToken) {}
}
```
