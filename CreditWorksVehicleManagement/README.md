# CreditWorks Vehicle Management

A web application for managing vehicles and vehicle weight categories, developed as part of the CreditWorks Software Engineer technical assignment.

The application allows users to:

- Add and view vehicles
- Select a manufacturer from the configured manufacturer list
- Automatically determine a vehicle's category from its weight
- Sort vehicles by owner, manufacturer, year, and weight
- Create, edit, and delete vehicle weight categories
- Assign an icon to each vehicle category
- Maintain continuous category ranges with no gaps or overlaps
- Automatically reflect category definition changes for existing vehicles

## Technology Stack

- C#
- ASP.NET Core MVC
- .NET 9
- Entity Framework Core 9
- Microsoft SQL Server
- SQL Server LocalDB for local development
- Razor Views
- Bootstrap
- xUnit
- Entity Framework Core InMemory provider for automated tests

## Setup

### Required Software

The following software is required to build and run the application locally:

- Visual Studio 2022 or later
- ASP.NET and web development workload for Visual Studio
- .NET 9 SDK
- Microsoft SQL Server 2019 or later
- SQL Server LocalDB for local development
- Git

### Database Requirements

The application uses Microsoft SQL Server through Entity Framework Core.

For local development, the application uses:

    (localdb)\MSSQLLocalDB

The database name is:

    CreditWorksVehicleManagementDb

The application connection string is configured in:

    CreditWorksVehicleManagement/appsettings.json

The configured connection string is intended for local development.

No production credentials, passwords, API keys, or other secrets are required by the application.

### Configuration

1. Clone the repository.
2. Open `CreditWorksVehicleManagement.sln` in Visual Studio.
3. Ensure SQL Server LocalDB is installed and available.
4. Check the `DefaultConnection` setting in `CreditWorksVehicleManagement/appsettings.json`.
5. If required, update the connection string to point to another local SQL Server or SQL Server Express instance.
6. Restore NuGet packages if Visual Studio does not restore them automatically.

## Database Creation and Migrations

Entity Framework Core migrations are included in the repository.

The migrations create the database schema and seed the initial manufacturers and vehicle categories.

### Visual Studio Package Manager Console

Open:

    Tools
    -> NuGet Package Manager
    -> Package Manager Console

Set the default project to:

    CreditWorksVehicleManagement

Run:

    Update-Database

This applies all pending migrations and creates the configured database.

### .NET CLI

From the solution directory, run:

    dotnet ef database update --project CreditWorksVehicleManagement

If the Entity Framework Core CLI tool is not installed, install it with:

    dotnet tool install --global dotnet-ef

## Initial Seed Data

The application seeds the following manufacturers:

- Mazda
- Mercedes
- Honda
- Ferrari
- Toyota

The initial vehicle categories are:

| Category | Minimum Weight | Maximum Weight | Icon |
|---|---:|---:|---|
| Light | 0 kg | 500 kg | 🚗 |
| Medium | 500 kg | 2500 kg | 🚚 |
| Heavy | 2500 kg | No maximum | 🚛 |

Category boundary behaviour is described in the Category Boundary Rules section.

## Build the Application

From Visual Studio:

    Build
    -> Build Solution

Alternatively, from the solution directory:

    dotnet build

The solution should build without compilation errors.

## Run the Application

From Visual Studio, select the HTTPS launch profile and run the application using:

    F5

or:

    Ctrl + F5

Alternatively, use the .NET CLI:

    dotnet run --project CreditWorksVehicleManagement

## Run Automated Tests

The solution contains a separate test project:

    CreditWorksVehicleManagement.Tests

From Visual Studio:

    Test
    -> Test Explorer
    -> Run All Tests

Alternatively:

    dotnet test

The automated tests cover important application behaviour including:

- Category determination
- Category boundary values
- Category configuration validation
- Gap prevention
- Overlap prevention
- Category changes
- Existing vehicle reclassification
- Vehicle validation
- Vehicle sorting

# Project Structure

The solution contains two projects:

    CreditWorksVehicleManagement
    CreditWorksVehicleManagement.Tests

## CreditWorksVehicleManagement

The main ASP.NET Core MVC application.

### Controllers

    Controllers

Handles HTTP requests and coordinates application operations.

Main controllers include:

- `HomeController`
- `VehiclesController`
- `CategoriesController`

### Models

    Models

Contains the application's domain entities.

Main entities include:

- `Vehicle`
- `Manufacturer`
- `VehicleCategory`

### ViewModels

    ViewModels

Contains models designed specifically for the user interface.

Examples include:

- `VehicleCreateViewModel`
- `VehicleListViewModel`

The vehicle creation form uses a dedicated ViewModel rather than directly binding the database entity.

### Services

    Services

Contains application business logic.

The main service is:

    CategoryService

The category service is responsible for:

- Determining the category for a vehicle weight
- Validating category configurations
- Creating categories
- Updating categories
- Deleting categories
- Maintaining continuous category ranges

### Data

    Data

Contains Entity Framework Core database configuration.

The main database context is:

    ApplicationDbContext

It provides access to:

- Vehicles
- Manufacturers
- VehicleCategories

### Migrations

    Migrations

Contains Entity Framework Core migrations used to create and update the SQL Server database schema.

### Views

    Views

Contains Razor Views used by the ASP.NET Core MVC frontend.

Views are provided for:

- Home
- Vehicles
- Categories
- Error handling

### wwwroot

    wwwroot

Contains static frontend assets such as:

- CSS
- JavaScript
- Other static resources

## CreditWorksVehicleManagement.Tests

The test project contains automated tests covering important application behaviour.

Test areas include:

- Category service behaviour
- Category boundary behaviour
- Category configuration validation
- Category changes
- Vehicle validation
- Vehicle sorting

The tests use the Entity Framework Core InMemory provider where database-backed service behaviour is required.

# Overall Architecture

The application uses a straightforward ASP.NET Core MVC architecture appropriate for the size and scope of the assignment.

The main application flow is:

    Browser
       |
       v
    ASP.NET Core MVC Controller
       |
       +----------------------+
       |                      |
       v                      v
    ViewModel            Category Service
       |                      |
       |                      v
       |                Business Rules
       |                      |
       +--------------> ApplicationDbContext
                              |
                              v
                          SQL Server

The main responsibilities are separated as follows.

### Presentation

ASP.NET Core MVC Controllers and Razor Views handle:

- HTTP requests
- Form submissions
- User input
- Validation messages
- Rendering application pages

### ViewModels

ViewModels represent data required by specific UI operations.

For example:

    VehicleCreateViewModel

is used for vehicle creation.

### Business Logic

`CategoryService` contains the vehicle category business rules.

This keeps category calculation and category configuration validation outside the controllers.

### Data Access

Entity Framework Core and `ApplicationDbContext` handle database access.

### Database

Microsoft SQL Server stores:

- Vehicles
- Manufacturers
- Vehicle categories

The architecture intentionally remains simple rather than introducing unnecessary enterprise patterns or additional layers.

# Database Design

The application uses three main database entities.

## Manufacturers

Stores the available vehicle manufacturers.

Main fields:

    Id
    Name

A manufacturer can have multiple vehicles.

## Vehicles

Stores vehicle information.

Main fields:

    Id
    OwnerName
    ManufacturerId
    YearOfManufacture
    WeightKg

`ManufacturerId` is a foreign key to the `Manufacturers` table.

The vehicle does not store a `CategoryId`.

## VehicleCategories

Stores configurable vehicle weight categories.

Main fields:

    Id
    Name
    MinWeightKg
    MaxWeightKg
    Icon

`MaxWeightKg` is nullable.

A null maximum represents the final open-ended category.

## Vehicle and Manufacturer Relationship

The relationship between manufacturers and vehicles is:

    Manufacturer 1 ---- * Vehicle

One manufacturer can be associated with many vehicles.

Each vehicle references one manufacturer through:

    ManufacturerId

Manufacturers are stored as database records rather than being hard-coded throughout the application.

The initial manufacturer list is provided through Entity Framework Core seed data.

# Why Category Is Not Stored on Vehicle

The vehicle category is deliberately calculated from:

    Vehicle.WeightKg

and the current category definitions.

A `CategoryId` is not stored on the `Vehicle` entity.

This prevents category information from becoming stale when category definitions are changed.

For example, a vehicle weighing:

    2200 kg

initially belongs to:

    Medium

when the configuration is:

    Medium: 500 to less than 2500 kg
    Heavy: 2500 kg and above

If the configuration is changed to:

    Medium: 500 to less than 2000 kg
    Heavy: 2000 kg and above

the same vehicle is immediately classified as:

    Heavy

without modifying the vehicle's stored weight.

# Category Calculation

Vehicle categories are determined from the vehicle's current weight and the configured category ranges.

The category lookup follows this rule:

    Weight >= MinWeightKg
    AND
    Weight < MaxWeightKg

For the final category, where `MaxWeightKg` is null, the rule becomes:

    Weight >= MinWeightKg

Therefore the category calculation uses:

    Minimum = inclusive
    Maximum = exclusive

This ensures that a valid vehicle weight resolves to exactly one category when the category configuration is valid.

# Category Boundary Rules

The application uses inclusive minimum boundaries and exclusive maximum boundaries.

The initial categories are:

    Light:
    0 kg <= weight < 500 kg

    Medium:
    500 kg <= weight < 2500 kg

    Heavy:
    weight >= 2500 kg

Examples:

| Weight | Category |
|---:|---|
| 0.01 kg | Light |
| 499.99 kg | Light |
| 500.00 kg | Medium |
| 500.01 kg | Medium |
| 2499.99 kg | Medium |
| 2500.00 kg | Heavy |
| 2500.01 kg | Heavy |

Therefore:

    500.00 kg -> Medium
    2500.00 kg -> Heavy

This boundary convention is documented so that there is no ambiguity about which category owns an exact boundary value.

# Category Configuration Rules

The application validates the complete category configuration.

A valid category configuration must:

1. Contain at least one category.
2. Start at 0 kg.
3. Have non-negative minimum weights.
4. Have a maximum greater than the minimum when a maximum exists.
5. Have no gaps between adjacent categories.
6. Have no overlapping ranges.
7. Have exactly one final open-ended category.
8. Have a category name.
9. Have a category icon.

## Valid Configuration

    Light: 0 to less than 500
    Medium: 500 to less than 2500
    Heavy: 2500 and above

## Invalid Configuration - Gap

    Light: 0 to less than 500
    Medium: 600 to less than 2500
    Heavy: 2500 and above

The range from 500 kg to less than 600 kg is not covered.

## Invalid Configuration - Overlap

    Light: 0 to less than 600
    Medium: 500 to less than 2500
    Heavy: 2500 and above

Weights between 500 kg and less than 600 kg belong to two ranges.

The service rejects invalid configurations before saving them.

# Category Changes

Category changes are handled through `CategoryService`.

When a category boundary is changed, the matching adjacent category boundary is updated so that the complete configuration remains continuous.

For example, the initial configuration may be:

    Medium:
    500 to less than 2500

    Heavy:
    2500 and above

If Medium is changed to end at 2000 kg, the resulting configuration becomes:

    Medium:
    500 to less than 2000

    Heavy:
    2000 and above

The complete candidate configuration is validated before the changes are saved.

Existing vehicle weights remain unchanged.

Their categories are calculated from the new configuration.

# Category Creation and Deletion

Category creation and deletion are handled through the category service.

The resulting category configuration is validated before the operation is saved.

The application prevents a category operation from leaving an invalid configuration containing:

- Gaps
- Overlaps
- Invalid ranges
- Missing zero-based coverage
- More than one open-ended category

If an operation would create an invalid configuration, the operation is rejected and an appropriate validation message is shown to the user.

# Vehicle Validation

Vehicle creation performs server-side validation.

## Owner Name

Owner name is required.

Whitespace-only input is rejected.

## Manufacturer

A manufacturer must be selected.

The selected manufacturer ID is checked against the database on the server.

## Year of Manufacture

The year must be within the configured validation range and cannot be in the future.

## Weight

Vehicle weight must:

- Be greater than zero
- Have no more than two decimal places
- Fit within the configured database decimal precision

The weight must also resolve to a valid vehicle category.

# Vehicle List

The vehicle list displays:

- Owner name
- Manufacturer
- Year of manufacture
- Weight
- Category icon
- Category name

The vehicle category is calculated from the current category configuration.

This means category changes are reflected when existing vehicles are displayed.

# Vehicle Sorting

The vehicle list supports sorting by:

- Owner
- Manufacturer
- Year
- Weight

Both ascending and descending sorting are supported.

The selected sort field and direction are retained in the page so that the current sort order is clear to the user.

# Error Handling

The application handles expected errors without exposing unnecessary technical details to the user.

Examples include:

- Invalid user input
- Invalid category configuration
- Invalid category operations
- Missing records
- Invalid manufacturer selection
- Database-related application errors

For unexpected application errors, the application uses ASP.NET Core exception handling in non-development environments.

The error page provides a safe user-facing message and request reference rather than exposing stack traces.

# Security

The application includes basic security practices appropriate to the assignment.

## Server-Side Validation

Important validation is performed on the server rather than relying only on browser-side validation.

## Anti-Forgery Protection

POST forms use ASP.NET Core anti-forgery validation.

## Database Queries

Entity Framework Core LINQ queries are used rather than constructing SQL statements through string concatenation.

## Output Encoding

Razor's normal output encoding is used when displaying user-provided values.

## Secrets

No passwords, API keys, or production credentials are included in source control.

The configured connection string is intended for local development.

Authentication and authorisation are not implemented because they are outside the requirements of this assignment.

# Automated Testing

The automated test suite focuses on important application behaviour rather than every implementation detail.

## Category Determination

Tests verify boundary behaviour including:

    499.99 -> Light
    500.00 -> Medium
    2499.99 -> Medium
    2500.00 -> Heavy

## Category Configuration Validation

Tests verify rejection of invalid configurations including:

- Gaps
- Overlaps
- Configuration not starting at zero
- Invalid final category configuration

## Category Changes

Tests verify that updating a category boundary:

- Moves the matching adjacent boundary
- Preserves a valid overall configuration
- Causes existing vehicle weights to resolve to the appropriate current category

## Vehicle Validation

Tests cover:

- Required owner name
- Year validation
- Positive vehicle weight
- Weight validation behaviour

## Vehicle Sorting

Tests cover sorting by:

- Owner
- Manufacturer
- Year
- Weight

# Important Assumptions

The following assumptions were made where the assignment requirements allowed implementation choices.

## Category Boundaries

Minimum values are inclusive and maximum values are exclusive.

Therefore:

    500.00 kg -> Medium
    2500.00 kg -> Heavy

## Final Category

The final category must be open-ended and therefore has no maximum weight.

## Vehicle Category

The category is calculated from the vehicle's current weight rather than permanently stored on the vehicle.

## Manufacturers

Manufacturers are represented as database records.

The initial manufacturer list is seeded through Entity Framework Core.

## Weight Precision

Vehicle weight is stored using SQL Server decimal precision with two decimal places.

## Authentication

Authentication and authorisation are not implemented because they are not required by the assignment.

# Significant Design Decisions

## ASP.NET Core MVC

ASP.NET Core MVC was selected because it provides a straightforward structure suitable for the size and scope of this application.

It separates:

- Controllers
- Views
- Models
- ViewModels
- Business logic
- Data access

without introducing unnecessary architectural complexity.

## Category Service

Category business rules are implemented in `CategoryService` instead of being placed directly inside controllers.

This makes the rules easier to test and keeps controllers focused on request handling.

## Category Calculated Rather Than Stored

The vehicle category is calculated from the current vehicle weight and current category definitions.

This prevents stale category information when ranges change.

## Entity Framework Core

Entity Framework Core was selected for database access because it provides:

- Strongly typed database access
- LINQ queries
- Entity relationships
- Database migrations
- Seed data
- Asynchronous database operations

## SQL Server

Microsoft SQL Server was selected because SQL Server is required by the assignment.

SQL Server LocalDB provides a convenient local development environment.

## Database Seed Data

Initial manufacturers and vehicle categories are seeded through Entity Framework Core so that a newly created database contains the required initial configuration.

# Known Limitations

This application is designed for the scope of the CreditWorks technical assignment rather than as a full production enterprise system.

Known limitations include:

- Vehicle sorting is performed in application memory, which is appropriate for a small assignment dataset but would need reconsideration for a very large dataset.
- Automated tests use the EF Core InMemory provider for many database-backed tests, so they do not reproduce every SQL Server-specific behaviour.
- Authentication and authorisation are not implemented because they are outside the assignment requirements.
- SQL Server LocalDB is used for local development rather than production infrastructure.
- The application does not include advanced operational monitoring or distributed infrastructure.
- The application does not implement enterprise-scale caching or distributed processing.
- The application is intentionally focused on the requirements of the technical assignment rather than production-scale infrastructure.

# Potential Future Improvements

If the application were developed further for production use, possible improvements would include:

- Add structured application logging and monitoring.
- Add SQL Server-compatible integration tests.
- Move large vehicle sorting and filtering operations into database queries.
- Add pagination for large vehicle lists.
- Add authentication and role-based authorisation if required.
- Use environment-specific configuration for production databases.
- Store production secrets using a secure secret-management solution.
- Add additional integration and end-to-end tests.
- Add audit history for category configuration changes.
- Add automated CI/CD validation for builds and tests.
- Add database indexes based on production query patterns.
- Improve observability and operational diagnostics.

These improvements are intentionally outside the scope of the current assignment.

# Running the Application from a Fresh Clone

A reviewer can use the following process:

1. Clone the repository.
2. Open `CreditWorksVehicleManagement.sln`.
3. Ensure .NET 9 and SQL Server LocalDB are installed.
4. Restore NuGet packages.
5. Check the `DefaultConnection` configuration.
6. Apply the Entity Framework Core migrations:

       Update-Database

   or:

       dotnet ef database update --project CreditWorksVehicleManagement

7. Build the solution:

       dotnet build

8. Run the application:

       dotnet run --project CreditWorksVehicleManagement

9. Run the automated tests:

       dotnet test

The repository contains the required source code, migrations, seed configuration, and automated tests needed to establish and run the application.

# Source Control

The repository contains:

- Complete application source code
- Solution and project files
- Automated tests
- Entity Framework Core migrations
- README documentation
- Required application assets

The repository should not contain:

- Compiled binaries
- Build output
- Database backup files
- Passwords
- API keys
- Production connection strings
- Other secrets

# Assignment Scope

The application intentionally focuses on the core requirements of the CreditWorks Software Engineer technical assignment.

It does not introduce unnecessary:

- Enterprise architecture
- Microservices
- Production cloud infrastructure
- Authentication systems
- Unrelated application features

The primary focus is:

- Functional correctness
- Clear software structure
- Vehicle management
- Category management
- Category business rules
- Boundary handling
- Data integrity
- Server-side validation
- Automated testing
- Maintainability
- Documentation