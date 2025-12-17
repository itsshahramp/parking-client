# Parking Client - Refactoring Implementation Guide

This document provides step-by-step instructions for implementing the refactoring plan described in REFACTORING_ANALYSIS.md.

---

## Prerequisites

- .NET 8 SDK installed
- Visual Studio 2022 or JetBrains Rider
- SQL Server accessible
- Git for version control
- Backup of current working application

---

## Phase 1: Prepare Domain and Infrastructure

### Step 1.1: Update Parking.Domain Project File

**File:** `Parking.Domain/Parking.Domain.csproj`

**Change:**
```xml
<!-- BEFORE -->
<PropertyGroup>
  <TargetFramework>net8.0-windows</TargetFramework>
  <Nullable>enable</Nullable>
  <UseWPF>true</UseWPF>
  <ImplicitUsings>enable</ImplicitUsings>
  <Configurations>Debug;Release;DebugWithANPR</Configurations>
</PropertyGroup>

<!-- AFTER -->
<PropertyGroup>
  <TargetFramework>net8.0</TargetFramework>
  <Nullable>enable</Nullable>
  <ImplicitUsings>enable</ImplicitUsings>
  <Configurations>Debug;Release;DebugWithANPR</Configurations>
</PropertyGroup>
```

### Step 1.2: Update Parking.Infrastructure Project File

**File:** `Parking.Infrastructure/Parking.Infrastructure.csproj`

**Change:**
```xml
<!-- BEFORE -->
<PropertyGroup>
  <TargetFramework>net8.0-windows</TargetFramework>
  <Nullable>enable</Nullable>
  <UseWPF>true</UseWPF>
  <ImplicitUsings>enable</ImplicitUsings>
  <Configurations>Debug;Release;DebugWithANPR</Configurations>
</PropertyGroup>

<!-- AFTER -->
<PropertyGroup>
  <TargetFramework>net8.0</TargetFramework>
  <Nullable>enable</Nullable>
  <ImplicitUsings>enable</ImplicitUsings>
  <Configurations>Debug;Release;DebugWithANPR</Configurations>
</PropertyGroup>
```

### Step 1.3: Verify No Windows-Specific Code

Run these commands to check:

```bash
# Check for System.Windows references in Domain
grep -r "using System.Windows" Parking.Domain/

# Check for System.Windows references in Infrastructure
grep -r "using System.Windows" Parking.Infrastructure/

# Both should return no results
```

### Step 1.4: Test Compilation

```bash
# Build Domain project
dotnet build Parking.Domain/Parking.Domain.csproj

# Build Infrastructure project
dotnet build Parking.Infrastructure/Parking.Infrastructure.csproj

# Both should succeed
```

### Step 1.5: Commit Changes

```bash
git add Parking.Domain/Parking.Domain.csproj
git add Parking.Infrastructure/Parking.Infrastructure.csproj
git commit -m "refactor: Remove Windows-specific targeting from Domain and Infrastructure"
```

---

## Phase 2: Create Parking.Core Library

### Step 2.1: Create New Project

```bash
# Create new class library
dotnet new classlib -n Parking.Core -f net8.0

# Add to solution
dotnet sln add Parking.Core/Parking.Core.csproj
```

### Step 2.2: Configure Project File

**File:** `Parking.Core/Parking.Core.csproj`

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>

  <ItemGroup>
    <ProjectReference Include="..\Parking.Domain\Parking.Domain.csproj" />
    <ProjectReference Include="..\Parking.Infrastructure\Parking.Infrastructure.csproj" />
  </ItemGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.Extensions.DependencyInjection.Abstractions" Version="9.0.3" />
    <PackageReference Include="Microsoft.Extensions.Logging.Abstractions" Version="9.0.3" />
    <PackageReference Include="Microsoft.Extensions.Http" Version="9.0.3" />
    <PackageReference Include="Microsoft.AspNetCore.Identity" Version="2.3.1" />
    <PackageReference Include="Microsoft.EntityFrameworkCore" Version="8.0.14" />
    <PackageReference Include="Newtonsoft.Json" Version="13.0.3" />
  </ItemGroup>

</Project>
```

### Step 2.3: Delete Default Class1.cs

```bash
rm Parking.Core/Class1.cs
```

### Step 2.4: Create Folder Structure

```bash
mkdir -p Parking.Core/Services/Interfaces
mkdir -p Parking.Core/Helpers
mkdir -p Parking.Core/Models
mkdir -p Parking.Core/Extensions
mkdir -p Parking.Core/Abstractions
```

### Step 2.5: Create Configuration Abstraction

**File:** `Parking.Core/Abstractions/IAppConfiguration.cs`

```csharp
namespace Parking.Core.Abstractions;

/// <summary>
/// Abstraction for application configuration to avoid Settings.Default coupling
/// </summary>
public interface IAppConfiguration
{
    string DbHostAddress { get; }
    string DbName { get; }
    string DbUsername { get; }
    string DbPassword { get; }
    string ApiServerAddress { get; }
    bool DbActiveStatus { get; }
}
```

### Step 2.6: Copy Service Interfaces

Copy from `Parking.App/Services/Interfaces/` to `Parking.Core/Services/Interfaces/`:

```bash
cp Parking.App/Services/Interfaces/IParkingService.cs Parking.Core/Services/Interfaces/
cp Parking.App/Services/Interfaces/IUserService.cs Parking.Core/Services/Interfaces/
cp Parking.App/Services/Interfaces/IRoleService.cs Parking.Core/Services/Interfaces/
cp Parking.App/Services/Interfaces/ISynchronizationService.cs Parking.Core/Services/Interfaces/
cp Parking.App/Services/Interfaces/ITicketQueueService.cs Parking.Core/Services/Interfaces/
```

**Update namespace in each file:**
```csharp
// Change from:
namespace Parking.App.Services;

// To:
namespace Parking.Core.Services;
```

### Step 2.7: Copy Service Implementations

```bash
cp Parking.App/Services/ParkingService.cs Parking.Core/Services/
cp Parking.App/Services/UserService.cs Parking.Core/Services/
cp Parking.App/Services/RoleService.cs Parking.Core/Services/
cp Parking.App/Services/SynchronizationService.cs Parking.Core/Services/
cp Parking.App/Services/TicketQueueService.cs Parking.Core/Services/
```

**Update namespace in each file:**
```csharp
// Change from:
namespace Parking.App.Services;

// To:
namespace Parking.Core.Services;
```

**Update using statements** (add where needed):
```csharp
using Parking.Core.Models;
using Parking.Core.Helpers;
```

### Step 2.8: Handle Settings.Default References

In services that use `Settings.Default`, inject `IAppConfiguration`:

**Example for SynchronizationService.cs:**

```csharp
// BEFORE
TokenStore.BaseUrl = Settings.Default.Application_ApiServerAddress;

// AFTER
private readonly IAppConfiguration _configuration;

public SynchronizationService(
    IUnitOfWork _unitOfWork,
    ILogger<SynchronizationService> logger,
    IHttpClientFactory _httpClientFactory,
    IAppConfiguration configuration,
    UserManager<ApplicationUser> _userManager,
    RoleManager<ApplicationRole> _roleManager)
{
    _configuration = configuration;
    // ... rest of constructor
}

// In methods:
TokenStore.BaseUrl = _configuration.ApiServerAddress;
```

### Step 2.9: Copy Platform-Agnostic Helpers

```bash
# Copy helpers (check each one doesn't have WPF dependencies)
cp Parking.App/Helpers/DateConvertor.cs Parking.Core/Helpers/
cp Parking.App/Helpers/StringValidator.cs Parking.Core/Helpers/
cp Parking.App/Helpers/Constants.cs Parking.Core/Helpers/
cp Parking.App/Helpers/TokenStore.cs Parking.Core/Helpers/
cp Parking.App/Helpers/ParkingLotInfoStore.cs Parking.Core/Helpers/
cp Parking.App/Helpers/TimeFormatConverter.cs Parking.Core/Helpers/
cp Parking.App/Helpers/PlateTypeHelper.cs Parking.Core/Helpers/
cp Parking.App/Helpers/DigestHeader.cs Parking.Core/Helpers/
cp Parking.App/Helpers/DatabaseConnectionTester.cs Parking.Core/Helpers/
cp Parking.App/Helpers/PriceRoundingHelper.cs Parking.Core/Helpers/

# Copy PriceCalculation folder
cp -r Parking.App/Helpers/PriceCalculation Parking.Core/Helpers/
```

**Update namespaces:**
```csharp
// Change from:
namespace Parking.App.Helpers;

// To:
namespace Parking.Core.Helpers;
```

### Step 2.10: Copy Required Models

Copy DTO models from `Parking.App/Models/` to `Parking.Core/Models/`:

```bash
# Copy model directories (review each for WPF dependencies)
cp -r Parking.App/Models/Dto Parking.Core/Models/
cp -r Parking.App/Models/GeneralServiceResponse Parking.Core/Models/
cp -r Parking.App/Models/Login Parking.Core/Models/
cp -r Parking.App/Models/Tickets Parking.Core/Models/
cp -r Parking.App/Models/Config Parking.Core/Models/
```

**Update namespaces:**
```csharp
// Change from:
namespace Parking.App.Models.Dto.XXX;

// To:
namespace Parking.Core.Models.Dto.XXX;
```

**Note:** Review each model file for these WPF-specific items:
- `INotifyPropertyChanged` (remove if present)
- `ObservableCollection` (change to `List` or `ICollection`)
- WPF binding attributes

### Step 2.11: Create Service Registration Extension

**File:** `Parking.Core/Extensions/ServiceCollectionExtensions.cs`

```csharp
using Microsoft.Extensions.DependencyInjection;
using Parking.Core.Services;

namespace Parking.Core.Extensions;

public static class ServiceCollectionExtensions
{
    /// <summary>
    /// Register all Parking Core services
    /// </summary>
    public static IServiceCollection AddParkingCoreServices(
        this IServiceCollection services)
    {
        // Register services
        services.AddScoped<IParkingService, ParkingService>();
        services.AddScoped<IUserService, UserService>();
        services.AddScoped<IRoleService, RoleService>();
        services.AddScoped<ISynchronizationService, SynchronizationService>();
        services.AddScoped<ITicketQueueService, TicketQueueService>();

        return services;
    }
}
```

### Step 2.12: Test Parking.Core Compilation

```bash
dotnet build Parking.Core/Parking.Core.csproj
```

Fix any compilation errors:
- Missing using statements
- Incorrect namespaces
- WPF references that need removal

### Step 2.13: Commit Phase 2

```bash
git add Parking.Core/
git commit -m "feat: Create Parking.Core shared library with business logic"
```

---

## Phase 3: Refactor WPF Application

### Step 3.1: Add Parking.Core Reference

**File:** `Parking.App/Parking.App.csproj`

```xml
<ItemGroup>
  <ProjectReference Include="..\Parking.Domain\Parking.Domain.csproj" />
  <ProjectReference Include="..\Parking.Infrastructure\Parking.Infrastructure.csproj" />
  <ProjectReference Include="..\Parking.Core\Parking.Core.csproj" />  <!-- ADD THIS -->
</ItemGroup>
```

### Step 3.2: Create WPF Configuration Adapter

**File:** `Parking.App/Adapters/WpfAppConfiguration.cs`

```csharp
using Parking.Core.Abstractions;

namespace Parking.App.Adapters;

/// <summary>
/// Adapter to provide Settings.Default values through IAppConfiguration interface
/// </summary>
public class WpfAppConfiguration : IAppConfiguration
{
    public string DbHostAddress => Settings.Default.Application_DbHostAddress;
    public string DbName => Settings.Default.Application_DbName;
    public string DbUsername => Settings.Default.Application_DbUsername;
    public string DbPassword => Settings.Default.Application_DbPassword;
    public string ApiServerAddress => Settings.Default.Application_ApiServerAddress;
    public bool DbActiveStatus => Settings.Default.Application_DbActiveStatus;
}
```

### Step 3.3: Update App.xaml.cs Service Registration

**File:** `Parking.App/App.xaml.cs`

```csharp
using Parking.Core.Extensions;
using Parking.Core.Abstractions;
using Parking.App.Adapters;

// In CreateDefaultBuilder().ConfigureServices():

// Add configuration abstraction
services.AddSingleton<IAppConfiguration, WpfAppConfiguration>();

// Replace individual service registrations with:
services.AddParkingCoreServices();

// Keep WPF-specific registrations:
services.AddSingleton<IPageService, PageService>();
services.AddSingleton<IThemeService, ThemeService>();
services.AddSingleton<ITaskBarService, TaskBarService>();
// ... ViewModels, Pages, Windows, etc.
```

### Step 3.4: Update GlobalUsings.cs

**File:** `Parking.App/GlobalUsings.cs`

```csharp
// Keep WPF-related global usings (they're needed for Views and ViewModels)
global using System.Windows;
global using System.Windows.Controls;
global using System.Windows.Data;
// ... other System.Windows usings

// Add Parking.Core usings
global using Parking.Core.Services;
global using Parking.Core.Helpers;
global using Parking.Core.Models;
global using Parking.Core.Models.Dto;

// Remove (no longer needed - now in Parking.Core):
// global using Parking.App.Services;  (remove this line)
// global using Parking.App.Helpers;    (partially - keep for WPF helpers)
```

### Step 3.5: Update Using Statements in View Files

For files that use services, update:

```csharp
// BEFORE
using Parking.App.Services;

// AFTER
using Parking.Core.Services;
```

Use find-and-replace in your IDE:
- Find: `using Parking.App.Services;`
- Replace: `using Parking.Core.Services;`

### Step 3.6: Update Using Statements for Models

```csharp
// BEFORE
using Parking.App.Models.Dto.XXX;

// AFTER  
using Parking.Core.Models.Dto.XXX;
```

Use find-and-replace:
- Find: `using Parking.App.Models`
- Replace: `using Parking.Core.Models`

### Step 3.7: Delete Duplicate Service Files

**IMPORTANT:** Only after verifying compilation succeeds!

```bash
# Delete duplicate service files
rm Parking.App/Services/ParkingService.cs
rm Parking.App/Services/UserService.cs
rm Parking.App/Services/RoleService.cs
rm Parking.App/Services/SynchronizationService.cs
rm Parking.App/Services/TicketQueueService.cs

# Delete duplicate service interfaces
rm Parking.App/Services/Interfaces/IParkingService.cs
rm Parking.App/Services/Interfaces/IUserService.cs
rm Parking.App/Services/Interfaces/IRoleService.cs
rm Parking.App/Services/Interfaces/ISynchronizationService.cs
rm Parking.App/Services/Interfaces/ITicketQueueService.cs
```

### Step 3.8: Delete Duplicate Helper Files

**IMPORTANT:** Only delete platform-agnostic helpers that were copied!

```bash
rm Parking.App/Helpers/DateConvertor.cs
rm Parking.App/Helpers/StringValidator.cs
rm Parking.App/Helpers/Constants.cs
rm Parking.App/Helpers/TokenStore.cs
rm Parking.App/Helpers/ParkingLotInfoStore.cs
# ... etc (only the ones copied to Parking.Core)
```

**Keep WPF-specific helpers:**
- UIHelper.cs
- ReceiptPrinter.cs
- ImageHelper.cs
- SingleInstanceApp.cs
- EnumToBooleanConverter.cs

### Step 3.9: Delete Duplicate Model Files

```bash
# Delete models that were moved to Parking.Core
rm -rf Parking.App/Models/Dto
rm -rf Parking.App/Models/GeneralServiceResponse
rm -rf Parking.App/Models/Login
# etc.
```

### Step 3.10: Test WPF Application Build

```bash
dotnet build Parking.App/Parking.App.csproj
```

Fix any compilation errors.

### Step 3.11: Run WPF Application

Launch the application and test all major features:
- Login
- Create parking ticket
- View ticket history
- User management
- Reports
- Settings

### Step 3.12: Commit Phase 3

```bash
git add Parking.App/
git commit -m "refactor: Update WPF app to use Parking.Core shared library"
```

---

## Phase 4: Create WebAPI Project

### Step 4.1: Create WebAPI Project

```bash
dotnet new webapi -n Parking.Api -f net8.0
dotnet sln add Parking.Api/Parking.Api.csproj
```

### Step 4.2: Add Project References

**File:** `Parking.Api/Parking.Api.csproj`

```xml
<ItemGroup>
  <ProjectReference Include="..\Parking.Core\Parking.Core.csproj" />
  <ProjectReference Include="..\Parking.Infrastructure\Parking.Infrastructure.csproj" />
  <ProjectReference Include="..\Parking.Domain\Parking.Domain.csproj" />
</ItemGroup>

<ItemGroup>
  <PackageReference Include="Microsoft.AspNetCore.Identity.EntityFrameworkCore" Version="8.0.14" />
  <PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="8.0.14" />
  <PackageReference Include="Microsoft.EntityFrameworkCore.Design" Version="8.0.14">
    <PrivateAssets>all</PrivateAssets>
    <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
  </PackageReference>
  <PackageReference Include="Swashbuckle.AspNetCore" Version="6.5.0" />
</ItemGroup>
```

### Step 4.3: Create API Configuration Adapter

**File:** `Parking.Api/Configuration/ApiAppConfiguration.cs`

```csharp
using Parking.Core.Abstractions;

namespace Parking.Api.Configuration;

public class ApiAppConfiguration : IAppConfiguration
{
    private readonly IConfiguration _configuration;

    public ApiAppConfiguration(IConfiguration configuration)
    {
        _configuration = configuration;
    }

    public string DbHostAddress => _configuration["Database:Host"] ?? "";
    public string DbName => _configuration["Database:Name"] ?? "";
    public string DbUsername => _configuration["Database:Username"] ?? "";
    public string DbPassword => _configuration["Database:Password"] ?? "";
    public string ApiServerAddress => _configuration["ApiSettings:ServerAddress"] ?? "";
    public bool DbActiveStatus => _configuration.GetValue<bool>("Database:Active");
}
```

### Step 4.4: Configure appsettings.json

**File:** `Parking.Api/appsettings.json`

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "Database": {
    "Host": "your-sql-server",
    "Name": "ParkingDB",
    "Username": "sa",
    "Password": "YourPassword123",
    "Active": true
  },
  "ConnectionStrings": {
    "DefaultConnection": "Server=your-sql-server;Database=ParkingDB;User Id=sa;Password=YourPassword123;TrustServerCertificate=true;MultipleActiveResultSets=True;"
  },
  "ApiSettings": {
    "ServerAddress": "https://api.example.com"
  }
}
```

### Step 4.5: Configure Program.cs

**File:** `Parking.Api/Program.cs`

```csharp
using Microsoft.AspNetCore.Identity;
using Microsoft.EntityFrameworkCore;
using Parking.Core.Abstractions;
using Parking.Core.Extensions;
using Parking.Domain.Contracts;
using Parking.Domain.Contracts.Base;
using Parking.Domain.Entities.User;
using Parking.Infrastructure.Context;
using Parking.Infrastructure.Repositories.Base;
using Parking.Infrastructure.Uow;
using Parking.Api.Configuration;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Add Database Context
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

// Add Identity
builder.Services.AddIdentity<ApplicationUser, ApplicationRole>()
    .AddEntityFrameworkStores<ApplicationDbContext>()
    .AddDefaultTokenProviders();

// Add Infrastructure Services
builder.Services.AddScoped(typeof(IRepository<>), typeof(Repository<>));
builder.Services.AddScoped<IUnitOfWork, UnitOfWork>();

// Add Configuration Abstraction
builder.Services.AddSingleton<IAppConfiguration, ApiAppConfiguration>();

// Add Parking Core Services
builder.Services.AddParkingCoreServices();

// Add HttpClient
builder.Services.AddHttpClient();

var app = builder.Build();

// Configure the HTTP request pipeline
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

### Step 4.6: Create Controllers

**File:** `Parking.Api/Controllers/ParkingController.cs`

```csharp
using Microsoft.AspNetCore.Mvc;
using Parking.Core.Services;
using Parking.Core.Models.Dto.Parking.ParkingLot;

namespace Parking.Api.Controllers;

[ApiController]
[Route("api/[controller]")]
public class ParkingController : ControllerBase
{
    private readonly IParkingService _parkingService;
    private readonly ILogger<ParkingController> _logger;

    public ParkingController(
        IParkingService parkingService,
        ILogger<ParkingController> logger)
    {
        _parkingService = parkingService;
        _logger = logger;
    }

    [HttpGet("details")]
    public ActionResult<ParkingLotModel> GetParkingLotDetails()
    {
        try
        {
            var result = _parkingService.GetParkingLotDetails();
            
            if (result.Succeeded)
                return Ok(result);
            
            return NotFound(result.Message);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error getting parking lot details");
            return StatusCode(500, "Internal server error");
        }
    }

    // Add more endpoints as needed
}
```

**File:** `Parking.Api/Controllers/UserController.cs`

```csharp
using Microsoft.AspNetCore.Mvc;
using Parking.Core.Services;

namespace Parking.Api.Controllers;

[ApiController]
[Route("api/[controller]")]
public class UserController : ControllerBase
{
    private readonly IUserService _userService;
    private readonly ILogger<UserController> _logger;

    public UserController(
        IUserService userService,
        ILogger<UserController> logger)
    {
        _userService = userService;
        _logger = logger;
    }

    [HttpGet]
    public async Task<IActionResult> GetAllUsers()
    {
        try
        {
            var users = await _userService.GetAllUsersAsync();
            return Ok(users);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error getting all users");
            return StatusCode(500, "Internal server error");
        }
    }

    // Add more endpoints
}
```

### Step 4.7: Test WebAPI Build

```bash
dotnet build Parking.Api/Parking.Api.csproj
```

### Step 4.8: Run WebAPI

```bash
cd Parking.Api
dotnet run
```

Navigate to: https://localhost:5001/swagger (or the port shown in console)

### Step 4.9: Test Endpoints

Use Swagger UI or curl:

```bash
# Test parking details endpoint
curl https://localhost:5001/api/parking/details

# Test users endpoint
curl https://localhost:5001/api/user
```

### Step 4.10: Commit Phase 4

```bash
git add Parking.Api/
git commit -m "feat: Create WebAPI using shared Parking.Core library"
```

---

## Phase 5: Testing and Validation

### Step 5.1: Create Test Project

```bash
dotnet new xunit -n Parking.Core.Tests -f net8.0
dotnet sln add Parking.Core.Tests/Parking.Core.Tests.csproj
```

### Step 5.2: Add Test Dependencies

**File:** `Parking.Core.Tests/Parking.Core.Tests.csproj`

```xml
<ItemGroup>
  <PackageReference Include="Microsoft.NET.Test.Sdk" Version="17.6.0" />
  <PackageReference Include="xunit" Version="2.4.2" />
  <PackageReference Include="xunit.runner.visualstudio" Version="2.4.5" />
  <PackageReference Include="Moq" Version="4.20.69" />
  <PackageReference Include="FluentAssertions" Version="6.12.0" />
</ItemGroup>

<ItemGroup>
  <ProjectReference Include="..\Parking.Core\Parking.Core.csproj" />
</ItemGroup>
```

### Step 5.3: Create Sample Unit Tests

**File:** `Parking.Core.Tests/Services/UserServiceTests.cs`

```csharp
using Moq;
using Xunit;
using FluentAssertions;
using Microsoft.Extensions.Logging;
using Parking.Core.Services;
using Parking.Domain.Contracts;
using Parking.Domain.Entities.User;
using Microsoft.AspNetCore.Identity;

namespace Parking.Core.Tests.Services;

public class UserServiceTests
{
    private readonly Mock<IUnitOfWork> _unitOfWorkMock;
    private readonly Mock<ILogger<UserService>> _loggerMock;
    private readonly Mock<UserManager<ApplicationUser>> _userManagerMock;
    private readonly Mock<RoleManager<ApplicationRole>> _roleManagerMock;
    private readonly UserService _sut;

    public UserServiceTests()
    {
        _unitOfWorkMock = new Mock<IUnitOfWork>();
        _loggerMock = new Mock<ILogger<UserService>>();
        
        // Mock UserManager (requires complex setup)
        var userStoreMock = new Mock<IUserStore<ApplicationUser>>();
        _userManagerMock = new Mock<UserManager<ApplicationUser>>(
            userStoreMock.Object, null, null, null, null, null, null, null, null);
        
        // Mock RoleManager
        var roleStoreMock = new Mock<IRoleStore<ApplicationRole>>();
        _roleManagerMock = new Mock<RoleManager<ApplicationRole>>(
            roleStoreMock.Object, null, null, null, null);

        _sut = new UserService(
            _unitOfWorkMock.Object,
            _loggerMock.Object,
            _userManagerMock.Object,
            _roleManagerMock.Object);
    }

    [Fact]
    public void GetAllUsers_ShouldReturnEmptyList_WhenNoUsersExist()
    {
        // Arrange
        _unitOfWorkMock.Setup(x => x.Users.GetAll())
            .Returns(new List<ApplicationUser>().AsQueryable());

        // Act
        var result = _sut.GetAllUsers();

        // Assert
        result.Should().NotBeNull();
        result.Should().BeEmpty();
    }

    // Add more tests
}
```

### Step 5.4: Run Unit Tests

```bash
dotnet test Parking.Core.Tests/
```

### Step 5.5: Manual Testing Checklist

**WPF Application:**
- [ ] Application launches without errors
- [ ] Can log in
- [ ] Can create new parking ticket
- [ ] Can view ticket history
- [ ] Can search tickets
- [ ] Can manage users
- [ ] Can view reports
- [ ] Settings page works
- [ ] Database connection monitoring works
- [ ] No performance degradation

**WebAPI:**
- [ ] Swagger UI loads
- [ ] Can call GET endpoints
- [ ] Can call POST endpoints
- [ ] Proper error handling
- [ ] Returns correct data format
- [ ] Database queries work

### Step 5.6: Performance Testing

```bash
# Run WPF app and measure startup time
# Compare with pre-refactoring baseline

# For WebAPI, use a load testing tool
# Example with Apache Bench:
ab -n 1000 -c 10 https://localhost:5001/api/parking/details
```

### Step 5.7: Code Coverage

```bash
dotnet test /p:CollectCoverage=true /p:CoverletOutputFormat=opencover
```

Target: ≥ 70% coverage for Parking.Core

### Step 5.8: Final Verification

Run these checks:

```bash
# 1. Ensure no WPF references in Core, Domain, Infrastructure
grep -r "System.Windows" Parking.Core/ Parking.Domain/ Parking.Infrastructure/
# Should return nothing

# 2. Ensure all projects build
dotnet build

# 3. Ensure all tests pass
dotnet test

# 4. Check for unused using statements
# Use IDE cleanup feature
```

### Step 5.9: Documentation Update

Update README.md with new architecture:

```markdown
## Architecture

- **Parking.Domain**: Domain entities and contracts
- **Parking.Infrastructure**: Data access with EF Core
- **Parking.Core**: Shared business logic (NEW)
- **Parking.App**: WPF desktop application
- **Parking.Api**: ASP.NET Core WebAPI (NEW)
```

### Step 5.10: Final Commit

```bash
git add .
git commit -m "test: Add unit tests and complete refactoring validation"
git tag v2.0.0-refactored
```

---

## Rollback Plan

If issues arise during refactoring:

### Rollback to Previous Phase

```bash
# View commits
git log --oneline

# Rollback to specific commit
git reset --hard <commit-hash>
```

### Restore from Backup Tag

```bash
git checkout pre-refactoring-backup
git checkout -b feature/refactor-attempt-2
```

---

## Common Issues and Solutions

### Issue 1: "Type exists in both assemblies"

**Problem:** Duplicate types in Parking.App and Parking.Core

**Solution:**
```bash
# Ensure old files are deleted
# Check .csproj for duplicate Compile includes
# Clean and rebuild
dotnet clean
dotnet build
```

### Issue 2: Settings.Default not working in services

**Problem:** IAppConfiguration not registered

**Solution:**
```csharp
// In Program.cs or App.xaml.cs, ensure:
services.AddSingleton<IAppConfiguration, WpfAppConfiguration>();
// or
services.AddSingleton<IAppConfiguration, ApiAppConfiguration>();
```

### Issue 3: DbContext not found in WebAPI

**Problem:** Missing DbContext registration

**Solution:**
```csharp
// In Program.cs:
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
```

### Issue 4: WPF app performance issues

**Problem:** Additional layer causing slowdown

**Solution:**
- Profile application to identify bottlenecks
- Ensure Dependency Injection is using correct lifetimes (Scoped vs Singleton)
- Consider lazy loading for large data sets

---

## Success Metrics

✅ All phases completed  
✅ Zero compilation errors  
✅ All unit tests passing  
✅ WPF app fully functional  
✅ WebAPI endpoints working  
✅ Code coverage ≥ 70%  
✅ No WPF references in Core/Domain/Infrastructure  
✅ Documentation updated  

---

## Next Steps After Refactoring

1. **Add Authentication to WebAPI**
   - JWT tokens
   - Role-based authorization

2. **Add More WebAPI Endpoints**
   - Complete CRUD for all entities
   - Search and filtering

3. **Create Postman Collection**
   - Document all API endpoints
   - Include sample requests

4. **Deploy WebAPI**
   - Azure App Service
   - Docker container
   - IIS hosting

5. **Create Mobile App** (Optional)
   - Xamarin/MAUI app using Parking.Api
   - Consume WebAPI endpoints

---

**Implementation Guide Version:** 1.0  
**Last Updated:** 2025-12-17
