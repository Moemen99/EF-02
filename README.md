# Entity Framework Core - Mapping by Convention

## Project Structure
```
ProjectName/
├── Domain/
│   └── Entities/  # or Models
│       └── Employee.cs
```

## POCO Classes (Plain Old CLR Objects)

POCO classes are simple classes without any special functionality or behavior, used for mapping to database tables.

```csharp
namespace Day_02_G02.Entities
{
    internal class Employee
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public double Salary { get; set; }
        public int? Age { get; set; }
    }
}
```

## Mapping by Convention

Entity Framework Core follows specific conventions for mapping classes to database objects.

### Primary Key Conventions
```mermaid
graph TD
    A[Property Naming] --> B{Is Property Name}
    B -->|Yes| C[Id]
    B -->|Yes| D[EntityNameId]
    C --> E[Makes Primary Key]
    D --> E
    E --> F[Adds Identity<br/>1,1]
```

| Property Pattern | Example | Result |
|-----------------|---------|---------|
| `Id` | `public int Id` | Primary Key |
| `EntityNameId` | `public int EmployeeId` | Primary Key |

### Data Type Mapping

| C# Type | Nullability | SQL Server Type | Constraint |
|---------|------------|-----------------|------------|
| `int` | Not Nullable | `int` | Required |
| `string` (.NET 5) | Reference Type | `nvarchar(max)` | Optional |
| `string` (.NET 6+) | Reference Type | `nvarchar(max)` | Required |
| `string?` (.NET 6+) | Nullable Reference | `nvarchar(max)` | Optional |
| `double` | Value Type | `float` | Required |
| `int?` | Nullable Value | `int` | Optional |

## Default Conventions

1. **Primary Key**
   - Auto-detection of `Id` or `{EntityName}Id`
   - Automatic identity specification (1,1)

2. **Nullability**
   - Value types: Not nullable by default
   - Reference types (.NET 5): Nullable by default
   - Reference types (.NET 6+): Not nullable by default
   - Explicit nullable types (`string?`, `int?`): Always nullable

3. **String Properties**
   - Maps to `nvarchar(max)`
   - Nullability based on type declaration

4. **Numeric Properties**
   - Integer types map to `int`
   - Floating-point types map to `float`
   - Precision based on C# type

## DbContext Setup Requirements

To complete the mapping:
1. Create a DbContext class
2. Include DbSet properties
3. Configure connection string
4. Register with dependency injection (if using ASP.NET Core)

```csharp
// Basic DbContext structure (to be expanded)
public class ApplicationDbContext : DbContext
{
    public DbSet<Employee> Employees { get; set; }
    // Connection string configuration to be added
}
```

## Best Practices

1. **Project Organization**
   - Keep entity classes in dedicated folder
   - Use clear naming conventions
   - Group related entities together

2. **POCO Design**
   - Keep entities simple
   - Avoid business logic in entities
   - Use appropriate data types

3. **Conventions**
   - Follow EF Core naming conventions
   - Be consistent with nullability
   - Document any deviations

4. **Migration Considerations**
   - Plan for future changes
   - Consider column size constraints
   - Think about indexing requirements



# Entity Framework Core - DbContext Setup and Configuration

## Project Structure
```
ProjectName/
├── Context/
│   └── CompanyDbContext.cs
├── Domain/
│   └── Entities/
```

## Package Installation

### Installing Required Packages
```powershell
# Latest version (not recommended for specific projects)
Install-Package Microsoft.EntityFrameworkCore.SqlServer

# Specific version (recommended)
Install-Package Microsoft.EntityFrameworkCore.SqlServer -v 6.0.15
```

### Database Provider Packages
| Database | Package Name |
|----------|--------------|
| SQL Server | Microsoft.EntityFrameworkCore.SqlServer |
| MySQL | Microsoft.EntityFrameworkCore.MySQL |
| Oracle | Microsoft.EntityFrameworkCore.Oracle |

## DbContext Implementation

```csharp
using Microsoft.EntityFrameworkCore;

namespace YourNamespace.Context
{
    public class CompanyDbContext : DbContext
    {
        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            optionsBuilder.UseSqlServer(
                "Server=.;Database=CompanyDB;Trusted_Connection=True;"
            );
        }
    }
}
```

## Connection String Components

### Modern Keys (Entity Framework Core)
```mermaid
graph TD
    A[Connection String] --> B[Server]
    A --> C[Database]
    A --> D[Authentication]
    D --> E[Trusted_Connection]
    D --> F[User/Password]
```

| Component | Key | Description | Example |
|-----------|-----|-------------|---------|
| Server | `Server` | SQL Server instance | `Server=.` or `Server=SERVERNAME` |
| Database | `Database` | Database name | `Database=CompanyDB` |
| Windows Auth | `Trusted_Connection` | Windows authentication | `Trusted_Connection=True` |
| SQL Auth | `User Id`, `Password` | SQL Server authentication | `User Id=sa;Password=pass` |

### Legacy Keys (Traditional ADO.NET)
| Modern Key | Legacy Key |
|------------|------------|
| `Server` | `Data Source` |
| `Database` | `Initial Catalog` |
| `Trusted_Connection` | `Integrated Security` |

## Authentication Methods

### 1. Windows Authentication
```csharp
// Using Windows Authentication
optionsBuilder.UseSqlServer(
    "Server=.;Database=CompanyDB;Trusted_Connection=True;"
);
```

### 2. SQL Server Authentication
```csharp
// Using SQL Server Authentication
optionsBuilder.UseSqlServer(
    "Server=.;Database=CompanyDB;User Id=sa;Password=yourpassword;"
);
```

## Best Practices

### Connection String Management
1. **Don't hardcode** connection strings in DbContext
2. Use configuration files or environment variables
3. Consider using secrets management in development

### Server Specification
- Use `.` or `(localdb)\MSSQLLocalDB` for local development
- Use full server name for specific instances
- Consider using aliases for production environments

### Security
- Prefer Windows Authentication in corporate environments
- Use strong passwords for SQL Authentication
- Never commit connection strings to source control
- Use secure connection strings in production

### Code Organization
- Keep DbContext in dedicated folder
- Follow naming convention: `[Name]DbContext`
- Group related configurations together
- Consider using separate configuration classes for complex models

## Common Setup Steps
1. Create Context folder
2. Create DbContext class
3. Install required provider package
4. Inherit from DbContext
5. Override OnConfiguring
6. Configure connection string

## Troubleshooting
- Verify SQL Server instance name
- Check authentication credentials
- Ensure database exists
- Verify network connectivity
- Check firewall settings



# Entity Framework Core - DbContext Initialization Flow

## Object Creation Process

```mermaid
sequenceDiagram
    participant P as Program
    participant CDC as CompanyDbContext Constructor
    participant DBC as DbContext Constructor
    participant OC as OnConfiguring
    
    P->>CDC: new CompanyDbContext()
    CDC->>DBC: base()
    DBC->>OC: Call OnConfiguring
    OC-->>DBC: Return Configuration
    DBC-->>CDC: Complete Initialization
    CDC-->>P: Return DbContext Instance
```

## Constructor Chain

### 1. Initial Object Creation
```csharp
// In Program.cs
CompanyDbContext dbContext = new CompanyDbContext();
```

### 2. Implicit Constructor Definition
```csharp
public class CompanyDbContext : DbContext
{
    // Automatically generated - no need to write explicitly
    public CompanyDbContext() : base() { }
}
```

## Initialization Flow

1. **Program Creates Object**
   - Calls parameterless constructor
   - Triggers constructor chain

2. **CompanyDbContext Constructor**
   - Implicit or explicit constructor
   - Chains to base class (DbContext)

3. **DbContext Constructor**
   - Takes DbContextOptions
   - Triggers OnConfiguring
   - Sets up database connection

4. **OnConfiguring Method**
   - Configures database provider
   - Sets connection string
   - Establishes connection parameters


```mermaid
graph TD
    A["Program: new CompanyDbContext()"] --> B["CompanyDbContext Constructor"]
    B -->|"base()"| C["DbContext Constructor"]
    C -->|"Dynamic Binding"| D["OnConfiguring"]
    D --> E["Database Connection Established"]
    style D fill:#f9f,stroke:#333,stroke-width:4px
```

## Key Points

### Constructor Behavior
- Parameterless constructor is automatically generated
- No need for explicit constructor definition
- Base constructor chaining happens automatically

### Dynamic Binding
- OnConfiguring is called through dynamic binding
- Allows for runtime configuration
- Supports different environments

### Connection Lifecycle
1. Object instantiation begins
2. Constructor chain executes
3. Configuration is applied
4. Connection is established
5. DbContext is ready for use

## Best Practices

1. **Constructor Usage**
   - Let EF Core handle constructor generation
   - Only create custom constructors when needed
   - Maintain the base constructor chain

2. **Configuration Timing**
   - OnConfiguring is the right place for connection setup
   - Avoid configuration in constructors
   - Consider using dependency injection in ASP.NET Core

3. **Error Handling**
   - Connection errors are thrown during initialization
   - Handle exceptions appropriately
   - Log connection issues for debugging

## Common Scenarios

### Basic Usage
```csharp
// Simple instantiation
using var context = new CompanyDbContext();
```

### With Dependency Injection (ASP.NET Core)
```csharp
services.AddDbContext<CompanyDbContext>();
```

### Custom Constructor (When Needed)
```csharp
public class CompanyDbContext : DbContext
{
    // Custom configuration
    public CompanyDbContext(DbContextOptions<CompanyDbContext> options)
        : base(options)
    {
    }
}
```
