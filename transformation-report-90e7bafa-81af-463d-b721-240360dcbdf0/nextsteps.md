# Next Steps

## Issues resolved
- Transformed Bookstore.Domain.csproj to net8.0
- Transformed Bookstore.Data.csproj to net8.0
- Transformed Bookstore.Web.csproj to net8.0
- Transformed Bookstore.Cdk.csproj to net8.0
- Transformed Bookstore.Domain.Tests.csproj to net8.0

## Overview

The solution build output contains no errors across all five projects:

- `Bookstore.Data`
- `Bookstore.Domain.Tests`
- `Bookstore.Cdk`
- `Bookstore.Web`
- `Bookstore.Domain`

This indicates the transformation to cross-platform .NET has completed without introducing any build-time errors. The following steps outline how to validate, test, and deploy the solution.

---

## 1. Restore and Build the Solution

Run a clean restore and build from the solution root to confirm the error-free state is reproducible in your local environment:

```bash
dotnet restore
dotnet build --configuration Release
```

Verify that no warnings are promoted to errors and that all projects compile successfully.

---

## 2. Run the Unit Tests

Execute the test project to confirm that existing test coverage passes under the new runtime:

```bash
dotnet test app/Bookstore.Domain.Tests/Bookstore.Domain.Tests.csproj --configuration Release --verbosity normal
```

Review the output for:
- Any failing tests that previously passed in the legacy project.
- Any skipped tests that may indicate platform-specific code that was not fully migrated.
- Any runtime exceptions that do not surface at build time.

---

## 3. Validate the Data Layer

The `Bookstore.Data` project likely contains database access logic such as Entity Framework Core migrations or a data context. Perform the following checks:

- Confirm the correct database provider package is referenced (e.g., `Microsoft.EntityFrameworkCore.SqlServer`, `Npgsql.EntityFrameworkCore.PostgreSQL`, etc.).
- If using Entity Framework Core, verify that migrations are up to date:

```bash
dotnet ef migrations list --project app/Bookstore.Data/Bookstore.Data.csproj
```

- Apply migrations to a local or development database:

```bash
dotnet ef database update --project app/Bookstore.Data/Bookstore.Data.csproj
```

---

## 4. Run the Web Application Locally

Start the `Bookstore.Web` project and verify it runs correctly on the target platform:

```bash
dotnet run --project app/Bookstore.Web/Bookstore.Web.csproj --configuration Release
```

Check the following:
- The application starts without runtime exceptions.
- All routes and pages load as expected.
- Any authentication, session, or middleware configuration behaves correctly under ASP.NET Core.
- Static files, bundling, and any front-end assets are served correctly.

---

## 5. Review Configuration Files

Legacy projects often rely on `Web.config` or `App.config`. These are replaced by `appsettings.json` in cross-platform .NET. Confirm:

- All connection strings have been moved to `appsettings.json` or environment variables.
- Any environment-specific settings use `appsettings.Development.json` and `appsettings.Production.json` appropriately.
- Secrets are not stored in source-controlled configuration files. Use the .NET Secret Manager for local development:

```bash
dotnet user-secrets init --project app/Bookstore.Web/Bookstore.Web.csproj
dotnet user-secrets set "ConnectionStrings:Default" "your_connection_string"
```

---

## 6. Validate the CDK Project

The `Bookstore.Cdk` project defines infrastructure. Confirm it synthesizes correctly:

```bash
cd app/Bookstore.Cdk
dotnet build --configuration Release
cdk synth
```

Review the synthesized output for any resource definitions that may reference platform-specific assumptions from the legacy project (e.g., Windows-specific AMIs, IIS-based configurations).

---

## 7. Check for Remaining Platform-Specific Code

Even without build errors, runtime issues can arise from code that was valid in .NET Framework but behaves differently in cross-platform .NET. Search the solution for common problem areas:

- `System.Web` namespace usage (should have been removed or replaced).
- `Registry` access via `Microsoft.Win32`.
- Windows-specific file path separators (use `Path.Combine` and `Path.DirectorySeparatorChar`).
- `AppDomain` usage that is not supported in the same way.
- Any P/Invoke calls targeting Windows-only native libraries.

You can use the .NET Upgrade Assistant compatibility analyzer or the `Microsoft.DotNet.PlatformAbstractions` tooling to assist with this review.

---

## 8. Perform Integration and Smoke Testing

After unit tests pass and the application runs locally, perform a broader validation:

- Test all major user-facing workflows end to end (browsing, searching, purchasing, etc.).
- Verify database read and write operations function correctly.
- Check application logs for any warnings or handled exceptions that indicate degraded functionality.