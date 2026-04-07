# Next Steps

## Issues resolved
- Transformed Bookstore.Domain.csproj to net8.0
- Transformed Bookstore.Data.csproj to net8.0
- Transformed Bookstore.Web.csproj to net8.0
- Transformed Bookstore.Cdk.csproj to net8.0
- Transformed Bookstore.Domain.Tests.csproj to net8.0

## Overview

The solution build produced no errors across all five projects:

- `Bookstore.Data`
- `Bookstore.Domain.Tests`
- `Bookstore.Cdk`
- `Bookstore.Web`
- `Bookstore.Domain`

This indicates the transformation to cross-platform .NET was completed without introducing any build-breaking changes. The steps below focus on validating correctness and preparing for deployment.

---

## 1. Restore and Build the Solution Locally

Confirm the solution builds cleanly on your local machine using the .NET CLI:

```bash
dotnet restore
dotnet build --configuration Release
```

Resolve any warnings that surface during the build, particularly those related to nullable reference types, deprecated APIs, or platform compatibility.

---

## 2. Run the Existing Test Suite

Execute the tests in `Bookstore.Domain.Tests` to verify that domain logic behaves correctly after the transformation:

```bash
dotnet test --configuration Release --verbosity normal
```

Review the test output for:
- Any failing tests
- Any skipped tests that were previously passing
- Unexpected exceptions related to platform-specific behavior

If test coverage is low, consider adding tests for critical paths in `Bookstore.Domain` and `Bookstore.Data` before proceeding.

---

## 3. Validate Data Layer Behavior

Since `Bookstore.Data` handles persistence, verify the following:

- **Database provider compatibility**: Confirm the EF Core provider (e.g., SQL Server, SQLite, PostgreSQL) is compatible with the target .NET version.
- **Migrations**: Run any pending Entity Framework Core migrations against a test database:

```bash
dotnet ef database update --project Bookstore.Data --startup-project Bookstore.Web
```

- **Connection strings**: Ensure connection strings in `appsettings.json` or environment variables are correctly configured for the target environment.

---

## 4. Run and Smoke Test the Web Application

Start the `Bookstore.Web` project locally and manually verify core functionality:

```bash
dotnet run --project Bookstore.Web --configuration Release
```

Check the following:
- Application starts without runtime exceptions
- Key pages and routes load correctly
- Data reads and writes function as expected against the database
- Any authentication or session handling works correctly
- Static assets are served properly

Review the application logs for any runtime warnings or errors that did not surface at build time.

---

## 5. Verify CDK Project Configuration

Review `Bookstore.Cdk` to ensure infrastructure definitions reflect the correct runtime target (e.g., `dotnet8` or the appropriate Lambda/compute runtime identifier). Confirm that any runtime strings, environment variables, or resource configurations align with the migrated application.

---

## 6. Cross-Platform Validation

If the application will run on Linux (common in cloud deployments), test it explicitly on a Linux environment or within a Linux-based runtime:

- Check for any file path issues (`\` vs `/`)
- Confirm case-sensitive file references are correct
- Verify any native dependencies or interop calls are compatible

---

## 7. Review Target Framework and Package Versions

Open each `.csproj` file and confirm:

- `<TargetFramework>` is set to the intended version (e.g., `net8.0`)
- NuGet package versions are current and compatible with the target framework
- No packages reference `netstandard2.0` or older framework monikers in a way that could introduce compatibility gaps

```bash
dotnet list package --outdated
```

Update packages as needed, then re-run the build and tests.

---

## 8. Deploy to the Target Environment

Once all validation steps pass:

1. Publish the web application:

```bash
dotnet publish Bookstore.Web --configuration Release --output ./publish
```

2. Deploy the published output to your target hosting environment.
3. Run a post-deployment smoke test against the live environment to confirm the application is functioning correctly.