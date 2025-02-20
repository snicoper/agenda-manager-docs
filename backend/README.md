# Agenda Manager Api

## Certificado

```bash
dotnet dev-certs https -ep ./.containers/https/aspnetapp.pfx -p Test1234!
```

## Migrations

Install `dotnet tool install --global dotnet-ef`

```bash
cd src/WebApi

dotnet ef migrations add Initial -p ../Infrastructure/Infrastructure.csproj  -c AppDbContext  -o ../Infrastructure/Common/Persistence/Migrations

#dotnet ef database update -c AppDbContext
```

## UserSecrets

Mover `compose/usersecrets/employee-manager-55f1c750-41fd-463e-92d8-f15f12886245`
