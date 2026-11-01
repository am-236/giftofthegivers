# Gift of the Givers Prototype

A .NET 10 prototype for a Gift of the Givers donation platform. The repository currently contains an ASP.NET Core web application with ASP.NET Core Identity and supporting Azure Functions for donation logging, tax-certificate generation, and login-timestamp telemetry.

## Repository structure

| Project | Purpose |
| --- | --- |
| `GiftOfTheGiversPrototype` | ASP.NET Core web application using MVC controllers/views, Razor Pages for Identity, and SQL Server through Entity Framework Core. |
| `DonationLoggerFunctions` | HTTP-triggered Azure Function that accepts a donation payload and stores a JSON log in the `donation-logs` Azure Blob Storage container. |
| `GIftOfTheGivers.Functions` | HTTP-triggered Azure Function that returns a tax-certificate response for a donation. |
| `LogInTimeStamp` | Azure Functions project containing the login-timestamp and Azure Table Storage/OpenTelemetry scaffolding. |

## Prerequisites

- .NET 10 SDK
- SQL Server LocalDB or another SQL Server instance
- Azure Functions Core Tools v4 for running the Functions projects locally
- An Azure Storage account, Azurite, or another compatible local storage emulator when testing blob/table functionality
- Visual Studio 2026 or another editor that supports .NET 10

## Run the web application

From the repository root:

```powershell
dotnet restore .\GiftOfTheGiversPrototype.csproj
dotnet run --project .\GiftOfTheGiversPrototype.csproj
```

The development launch profile uses:

- HTTP: `http://localhost:5086`
- HTTPS: `https://localhost:7179`

The web application uses the `DefaultConnection` connection string from `appsettings.json` or `appsettings.Development.json`. Apply the existing Entity Framework Core migrations if the database has not been created:

```powershell
dotnet ef database update --project .\GiftOfTheGiversPrototype.csproj
```

When running in Development, the application seeds the development users configured under `DevelopmentUsers`. Do not reuse those credentials outside local development.

## Run an Azure Function locally

Each Functions project is independent. Run a project from its own directory after configuring its local settings:

```powershell
cd .\DonationLoggerFunctions
dotnet build
func start
```

Use the same pattern for `GIftOfTheGivers.Functions` or `LogInTimeStamp`.

### Available HTTP endpoints

With the default Azure Functions host configuration, the endpoints are:

- `POST http://localhost:7071/api/logdonation` — accepts a non-empty donation payload and returns a generated log ID.
- `GET` or `POST http://localhost:7071/api/taxcertificate/{donationId}` — returns a tax-certificate response for the supplied donation ID.

The donation logger uses the `AzureWebJobsStorage` setting. If it is unavailable, the function logs a warning and continues without uploading the blob.

## Configuration and secrets

- Keep connection strings, storage keys, Application Insights connection strings, and credentials in user secrets, environment variables, or local settings files.
- Do not commit production secrets or real donor information.
- Azure Functions local configuration belongs in `local.settings.json`; this file is intended for local development and should not be used as production configuration.
- For telemetry, set `APPLICATIONINSIGHTS_CONNECTION_STRING` when Application Insights/OpenTelemetry export is required.

## Build the solution

```powershell
dotnet build .\GiftOfTheGiversPrototype.slnx
```

## Current status

This is a prototype. The web application currently provides the base application shell, Identity integration, database context, and starter Home/Privacy pages. The supporting Functions include working donation-log and tax-certificate HTTP entry points, while the login-timestamp/Table Storage functionality remains scaffolding for further implementation.