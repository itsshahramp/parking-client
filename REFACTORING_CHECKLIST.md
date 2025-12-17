# Parking Client - Refactoring Implementation Checklist

Use this checklist to track your progress through the refactoring process.

---

## Pre-Implementation Checklist

- [ ] Read **REFACTORING_SUMMARY.md** (understand the why)
- [ ] Read **IMPLEMENTATION_GUIDE.md** (understand the how)
- [ ] Review **ARCHITECTURE_DIAGRAM.md** (visualize the transformation)
- [ ] Get stakeholder approval
- [ ] Allocate developer time (26-52 hours estimated)
- [ ] Set up development environment
  - [ ] .NET 8 SDK installed
  - [ ] Visual Studio 2022 or Rider ready
  - [ ] SQL Server accessible
  - [ ] Git configured
- [ ] Create backup
  - [ ] `git tag pre-refactoring-backup`
- [ ] Create feature branch
  - [ ] `git checkout -b feature/refactor-to-shared-core`
- [ ] Document current functionality (baseline)
  - [ ] List all WPF features
  - [ ] Document expected behaviors
  - [ ] Create test scenarios

---

## Phase 1: Prepare Domain and Infrastructure

**Estimated Time:** 2-4 hours  
**Risk Level:** LOW  

### 1.1 Update Parking.Domain Project

- [ ] Open `Parking.Domain/Parking.Domain.csproj`
- [ ] Change `<TargetFramework>net8.0-windows</TargetFramework>` to `<TargetFramework>net8.0</TargetFramework>`
- [ ] Remove `<UseWPF>true</UseWPF>` line
- [ ] Save file

### 1.2 Update Parking.Infrastructure Project

- [ ] Open `Parking.Infrastructure/Parking.Infrastructure.csproj`
- [ ] Change `<TargetFramework>net8.0-windows</TargetFramework>` to `<TargetFramework>net8.0</TargetFramework>`
- [ ] Remove `<UseWPF>true</UseWPF>` line
- [ ] Save file

### 1.3 Verification

- [ ] Search for Windows-specific references
  - [ ] `grep -rn --include='*.cs' "using System.Windows" Parking.Domain/` (should return nothing)
  - [ ] `grep -rn --include='*.cs' "using System.Windows" Parking.Infrastructure/` (should return nothing)
- [ ] Test compilation
  - [ ] `dotnet build Parking.Domain/Parking.Domain.csproj` (should succeed)
  - [ ] `dotnet build Parking.Infrastructure/Parking.Infrastructure.csproj` (should succeed)

### 1.4 Commit Changes

- [ ] Stage changes: `git add Parking.Domain/ Parking.Infrastructure/`
- [ ] Commit: `git commit -m "refactor: Remove Windows targeting from Domain/Infrastructure"`
- [ ] Push: `git push origin feature/refactor-to-shared-core`

### 1.5 Phase 1 Sign-off

- [ ] All tests pass
- [ ] Projects compile successfully
- [ ] Changes reviewed
- [ ] **Phase 1 Complete ✅**

---

## Phase 2: Create Parking.Core Library

**Estimated Time:** 8-16 hours  
**Risk Level:** MEDIUM  

### 2.1 Create Project

- [ ] Create new project: `dotnet new classlib -n Parking.Core -f net8.0`
- [ ] Add to solution: `dotnet sln add Parking.Core/Parking.Core.csproj`
- [ ] Delete default `Class1.cs`

### 2.2 Configure Project File

- [ ] Edit `Parking.Core/Parking.Core.csproj`
- [ ] Add project references (Domain, Infrastructure)
- [ ] Add NuGet packages:
  - [ ] Microsoft.Extensions.DependencyInjection.Abstractions
  - [ ] Microsoft.Extensions.Logging.Abstractions
  - [ ] Microsoft.Extensions.Http
  - [ ] Microsoft.AspNetCore.Identity
  - [ ] Microsoft.EntityFrameworkCore
  - [ ] Newtonsoft.Json
- [ ] Save file
- [ ] Restore packages: `dotnet restore Parking.Core/`

### 2.3 Create Folder Structure

- [ ] `mkdir -p Parking.Core/Services/Interfaces`
- [ ] `mkdir -p Parking.Core/Helpers`
- [ ] `mkdir -p Parking.Core/Models`
- [ ] `mkdir -p Parking.Core/Extensions`
- [ ] `mkdir -p Parking.Core/Abstractions`

### 2.4 Create Configuration Abstraction

- [ ] Create `Parking.Core/Abstractions/IAppConfiguration.cs`
- [ ] Define interface with properties:
  - [ ] DbHostAddress
  - [ ] DbName
  - [ ] DbUsername
  - [ ] DbPassword
  - [ ] ApiServerAddress
  - [ ] DbActiveStatus

### 2.5 Copy Service Interfaces

- [ ] Copy `IParkingService.cs` from App/Services/Interfaces to Core/Services/Interfaces
- [ ] Copy `IUserService.cs`
- [ ] Copy `IRoleService.cs`
- [ ] Copy `ISynchronizationService.cs`
- [ ] Copy `ITicketQueueService.cs`
- [ ] Update namespaces: `Parking.App.Services` → `Parking.Core.Services`

### 2.6 Copy Service Implementations

- [ ] Copy `ParkingService.cs` from App/Services to Core/Services
- [ ] Copy `UserService.cs`
- [ ] Copy `RoleService.cs`
- [ ] Copy `SynchronizationService.cs`
- [ ] Copy `TicketQueueService.cs`
- [ ] Update namespaces: `Parking.App.Services` → `Parking.Core.Services`

### 2.7 Refactor Services for Configuration

- [ ] In each service using `Settings.Default`:
  - [ ] Add `IAppConfiguration` parameter to constructor
  - [ ] Replace `Settings.Default.XXX` with `_configuration.XXX`
- [ ] Services updated:
  - [ ] ParkingService
  - [ ] UserService
  - [ ] RoleService
  - [ ] SynchronizationService
  - [ ] TicketQueueService

### 2.8 Copy Platform-Agnostic Helpers

- [ ] Copy `DateConvertor.cs` from App/Helpers to Core/Helpers
- [ ] Copy `StringValidator.cs`
- [ ] Copy `Constants.cs`
- [ ] Copy `TokenStore.cs`
- [ ] Copy `ParkingLotInfoStore.cs`
- [ ] Copy `TimeFormatConverter.cs`
- [ ] Copy `PlateTypeHelper.cs`
- [ ] Copy `DigestHeader.cs`
- [ ] Copy `DatabaseConnectionTester.cs`
- [ ] Copy `PriceRoundingHelper.cs`
- [ ] Copy `PriceCalculation/` folder (if exists)
- [ ] Update namespaces: `Parking.App.Helpers` → `Parking.Core.Helpers`

### 2.9 Copy Required Models

- [ ] Copy `Models/Dto/` folder to Core/Models/
- [ ] Copy `Models/GeneralServiceResponse/` folder
- [ ] Copy `Models/Login/` folder
- [ ] Copy `Models/Tickets/` folder
- [ ] Copy `Models/Config/` folder
- [ ] Update namespaces: `Parking.App.Models` → `Parking.Core.Models`
- [ ] Review models for WPF-specific code:
  - [ ] Remove `INotifyPropertyChanged` if present
  - [ ] Change `ObservableCollection` to `List` if needed
  - [ ] Remove WPF binding attributes

### 2.10 Create Service Registration Extension

- [ ] Create `Parking.Core/Extensions/ServiceCollectionExtensions.cs`
- [ ] Implement `AddParkingCoreServices()` extension method
- [ ] Register all services:
  - [ ] IParkingService → ParkingService
  - [ ] IUserService → UserService
  - [ ] IRoleService → RoleService
  - [ ] ISynchronizationService → SynchronizationService
  - [ ] ITicketQueueService → TicketQueueService

### 2.11 Test Compilation

- [ ] Build Parking.Core: `dotnet build Parking.Core/Parking.Core.csproj`
- [ ] Fix any compilation errors
- [ ] Verify no WPF references: `grep -rn --include='*.cs' "System.Windows" Parking.Core/`

### 2.12 Commit Phase 2

- [ ] Stage changes: `git add Parking.Core/`
- [ ] Commit: `git commit -m "feat: Create Parking.Core shared library"`
- [ ] Push changes

### 2.13 Phase 2 Sign-off

- [ ] Parking.Core compiles successfully
- [ ] No WPF references in Core
- [ ] All services and helpers copied
- [ ] Configuration abstraction created
- [ ] **Phase 2 Complete ✅**

---

## Phase 3: Refactor WPF Application

**Estimated Time:** 4-8 hours  
**Risk Level:** MEDIUM  

### 3.1 Add Reference

- [ ] Edit `Parking.App/Parking.App.csproj`
- [ ] Add `<ProjectReference Include="..\Parking.Core\Parking.Core.csproj" />`
- [ ] Save file

### 3.2 Create Configuration Adapter

- [ ] Create `Parking.App/Adapters/` folder
- [ ] Create `WpfAppConfiguration.cs` implementing `IAppConfiguration`
- [ ] Map Settings.Default properties to interface properties

### 3.3 Update Service Registration

- [ ] Open `App.xaml.cs`
- [ ] Add using: `using Parking.Core.Extensions;`
- [ ] Add using: `using Parking.Core.Abstractions;`
- [ ] Register configuration: `services.AddSingleton<IAppConfiguration, WpfAppConfiguration>();`
- [ ] Replace individual service registrations with: `services.AddParkingCoreServices();`
- [ ] Keep WPF-specific registrations (Pages, Windows, ViewModels)

### 3.4 Update GlobalUsings.cs

- [ ] Open `Parking.App/GlobalUsings.cs`
- [ ] Add: `global using Parking.Core.Services;`
- [ ] Add: `global using Parking.Core.Helpers;`
- [ ] Add: `global using Parking.Core.Models;`
- [ ] Keep WPF usings (needed for Views/ViewModels)

### 3.5 Update Using Statements

- [ ] Find and replace in solution:
  - [ ] `using Parking.App.Services;` → `using Parking.Core.Services;`
  - [ ] `using Parking.App.Models.` → `using Parking.Core.Models.`
  - [ ] (Keep `using Parking.App.Helpers;` for WPF-specific helpers)

### 3.6 Test Compilation

- [ ] Build WPF app: `dotnet build Parking.App/Parking.App.csproj`
- [ ] Fix any compilation errors
- [ ] Fix any namespace mismatches

### 3.7 Delete Duplicate Files

**⚠️ IMPORTANT: Only after successful compilation!**

- [ ] Delete service files from `Parking.App/Services/`:
  - [ ] ParkingService.cs
  - [ ] UserService.cs
  - [ ] RoleService.cs
  - [ ] SynchronizationService.cs
  - [ ] TicketQueueService.cs
- [ ] Delete service interfaces from `Parking.App/Services/Interfaces/`:
  - [ ] IParkingService.cs
  - [ ] IUserService.cs
  - [ ] IRoleService.cs
  - [ ] ISynchronizationService.cs
  - [ ] ITicketQueueService.cs
- [ ] Delete duplicate helpers (only platform-agnostic ones):
  - [ ] DateConvertor.cs
  - [ ] StringValidator.cs
  - [ ] Constants.cs
  - [ ] TokenStore.cs
  - [ ] (etc. - ones copied to Core)
- [ ] Delete duplicate models:
  - [ ] Models/Dto/ (if copied to Core)
  - [ ] (other model folders moved to Core)

### 3.8 Test WPF Application

- [ ] Build: `dotnet build Parking.App/`
- [ ] Run application
- [ ] Test major features:
  - [ ] Login functionality
  - [ ] Create parking ticket
  - [ ] View ticket history
  - [ ] User management
  - [ ] Reports
  - [ ] Settings page
- [ ] Verify no regressions

### 3.9 Commit Phase 3

- [ ] Stage changes: `git add Parking.App/`
- [ ] Commit: `git commit -m "refactor: Update WPF to use Parking.Core"`
- [ ] Push changes

### 3.10 Phase 3 Sign-off

- [ ] WPF app compiles
- [ ] WPF app runs without errors
- [ ] All features work as before
- [ ] No performance degradation
- [ ] **Phase 3 Complete ✅**

---

## Phase 4: Create WebAPI Project

**Estimated Time:** 4-8 hours  
**Risk Level:** LOW  

### 4.1 Create WebAPI Project

- [ ] Create project: `dotnet new webapi -n Parking.Api -f net8.0`
- [ ] Add to solution: `dotnet sln add Parking.Api/Parking.Api.csproj`

### 4.2 Add References

- [ ] Edit `Parking.Api/Parking.Api.csproj`
- [ ] Add project references:
  - [ ] Parking.Core
  - [ ] Parking.Infrastructure
  - [ ] Parking.Domain
- [ ] Add NuGet packages:
  - [ ] Microsoft.AspNetCore.Identity.EntityFrameworkCore
  - [ ] Microsoft.EntityFrameworkCore.SqlServer
  - [ ] Microsoft.EntityFrameworkCore.Design
  - [ ] Swashbuckle.AspNetCore
- [ ] Save and restore: `dotnet restore Parking.Api/`

### 4.3 Create Configuration Adapter

- [ ] Create `Parking.Api/Configuration/` folder
- [ ] Create `ApiAppConfiguration.cs` implementing `IAppConfiguration`
- [ ] Read from `IConfiguration` (appsettings.json)

### 4.4 Configure appsettings.json

- [ ] Edit `Parking.Api/appsettings.json`
- [ ] Add database configuration section
- [ ] Add connection string
- [ ] Add API settings section

### 4.5 Configure Program.cs

- [ ] Open `Parking.Api/Program.cs`
- [ ] Add DbContext registration
- [ ] Add Identity registration
- [ ] Add Infrastructure services (Repository, UnitOfWork)
- [ ] Add configuration: `services.AddSingleton<IAppConfiguration, ApiAppConfiguration>()`
- [ ] Add Core services: `builder.Services.AddParkingCoreServices();`
- [ ] Add HttpClient: `builder.Services.AddHttpClient();`
- [ ] Configure middleware (Swagger, HTTPS, Auth)

### 4.6 Create Controllers

- [ ] Create `Parking.Api/Controllers/ParkingController.cs`
  - [ ] Inject `IParkingService`
  - [ ] Add endpoint: `GET /api/parking/details`
- [ ] Create `Parking.Api/Controllers/UserController.cs`
  - [ ] Inject `IUserService`
  - [ ] Add endpoint: `GET /api/user`
- [ ] Create additional controllers as needed

### 4.7 Test Compilation

- [ ] Build API: `dotnet build Parking.Api/Parking.Api.csproj`
- [ ] Fix any compilation errors

### 4.8 Run and Test WebAPI

- [ ] Run: `cd Parking.Api && dotnet run`
- [ ] Navigate to Swagger UI (e.g., https://localhost:5001/swagger)
- [ ] Test endpoints:
  - [ ] GET /api/parking/details
  - [ ] GET /api/user
- [ ] Verify responses

### 4.9 Commit Phase 4

- [ ] Stage changes: `git add Parking.Api/`
- [ ] Commit: `git commit -m "feat: Create WebAPI using Parking.Core"`
- [ ] Push changes

### 4.10 Phase 4 Sign-off

- [ ] WebAPI compiles
- [ ] WebAPI runs without errors
- [ ] Endpoints return correct data
- [ ] Swagger documentation works
- [ ] **Phase 4 Complete ✅**

---

## Phase 5: Testing and Validation

**Estimated Time:** 8-16 hours  
**Risk Level:** LOW  

### 5.1 Create Test Project

- [ ] Create test project: `dotnet new xunit -n Parking.Core.Tests -f net8.0`
- [ ] Add to solution: `dotnet sln add Parking.Core.Tests/`
- [ ] Add test packages:
  - [ ] xunit
  - [ ] Moq
  - [ ] FluentAssertions
- [ ] Add reference to Parking.Core

### 5.2 Write Unit Tests

- [ ] Create test class for ParkingService
- [ ] Create test class for UserService
- [ ] Create test class for helpers (DateConvertor, etc.)
- [ ] Write tests for critical paths
- [ ] Aim for ≥70% code coverage

### 5.3 Run Unit Tests

- [ ] Run tests: `dotnet test Parking.Core.Tests/`
- [ ] All tests pass
- [ ] Check coverage: `dotnet test /p:CollectCoverage=true`

### 5.4 Manual Testing - WPF

- [ ] Launch WPF application
- [ ] Test feature: Login
- [ ] Test feature: Create parking ticket
- [ ] Test feature: Exit parking ticket
- [ ] Test feature: View ticket history
- [ ] Test feature: Search tickets
- [ ] Test feature: User management (add/edit/delete user)
- [ ] Test feature: Role management
- [ ] Test feature: Reports
- [ ] Test feature: Settings page
- [ ] Test feature: Database connection monitoring
- [ ] Verify no crashes
- [ ] Verify no performance issues

### 5.5 Manual Testing - WebAPI

- [ ] Launch WebAPI
- [ ] Test endpoint: GET /api/parking/details
- [ ] Test endpoint: GET /api/user
- [ ] Test endpoint: POST requests (if implemented)
- [ ] Verify error handling
- [ ] Verify response formats
- [ ] Test with invalid data

### 5.6 Integration Testing

- [ ] Run WPF and WebAPI simultaneously
- [ ] Verify both can access database
- [ ] Verify data consistency
- [ ] Create ticket in WPF, verify in API
- [ ] Create user in API, verify in WPF

### 5.7 Performance Testing

- [ ] Measure WPF startup time (compare to baseline)
- [ ] Measure ticket creation time
- [ ] Measure report generation time
- [ ] Verify no degradation > 5%
- [ ] For API: run load test (optional)
  - [ ] `ab -n 1000 -c 10 https://localhost:5001/api/parking/details`

### 5.8 Security Validation

- [ ] No secrets in code
- [ ] Connection strings in configuration
- [ ] API requires authentication (if implemented)
- [ ] No WPF dependencies in Core/Domain/Infrastructure

### 5.9 Code Quality Checks

- [ ] Run code analysis: `dotnet build /p:RunAnalyzers=true`
- [ ] Check for warnings
- [ ] Verify no Windows references in Core/Domain/Infrastructure:
  - [ ] `grep -rn --include='*.cs' "System.Windows" Parking.Core/ Parking.Domain/ Parking.Infrastructure/`
- [ ] Clean up unused using statements
- [ ] Format code consistently

### 5.10 Documentation

- [ ] Update main README.md with new architecture
- [ ] Document new projects (Parking.Core, Parking.Api)
- [ ] Update setup instructions
- [ ] Document API endpoints
- [ ] Add code comments where needed

### 5.11 Commit Phase 5

- [ ] Stage all test files and docs: `git add .`
- [ ] Commit: `git commit -m "test: Add unit tests and update documentation"`
- [ ] Push changes

### 5.12 Phase 5 Sign-off

- [ ] All unit tests pass
- [ ] WPF app fully functional
- [ ] WebAPI fully functional
- [ ] Performance acceptable
- [ ] Documentation updated
- [ ] **Phase 5 Complete ✅**

---

## Post-Implementation Checklist

### Final Verification

- [ ] All projects build: `dotnet build`
- [ ] All tests pass: `dotnet test`
- [ ] No compilation warnings
- [ ] WPF app runs correctly
- [ ] WebAPI runs correctly
- [ ] Code committed and pushed

### Success Criteria Verification

- [ ] ✅ Domain targets `net8.0`
- [ ] ✅ Infrastructure targets `net8.0`
- [ ] ✅ Parking.Core created and compiles
- [ ] ✅ WPF builds and runs (no functional changes)
- [ ] ✅ WebAPI provides endpoints
- [ ] ✅ All existing features work
- [ ] ✅ Unit tests pass (≥70% coverage)
- [ ] ✅ No WPF refs in Core/Domain/Infrastructure
- [ ] ✅ Documentation updated

### Final Sign-off

- [ ] Code review completed
- [ ] Stakeholder approval
- [ ] Merge to main branch
- [ ] Tag release: `git tag v2.0.0-refactored`
- [ ] Deploy (if applicable)
- [ ] **🎉 Refactoring Complete! 🎉**

---

## Rollback Procedure (If Needed)

If critical issues arise:

- [ ] Stop implementation
- [ ] Document the issue
- [ ] Check if quick fix is possible
- [ ] If not, rollback:
  - [ ] `git reset --hard pre-refactoring-backup`
  - [ ] Or: `git revert <commit-range>`
- [ ] Analyze what went wrong
- [ ] Plan corrective action
- [ ] Try again with fixes

---

## Notes & Issues

Use this section to track issues encountered during implementation:

**Date:** ______  
**Issue:** ______________________________________________________  
**Resolution:** _________________________________________________  

**Date:** ______  
**Issue:** ______________________________________________________  
**Resolution:** _________________________________________________  

**Date:** ______  
**Issue:** ______________________________________________________  
**Resolution:** _________________________________________________  

---

## Time Tracking

Track actual time spent vs. estimates:

| Phase | Estimated | Actual | Difference | Notes |
|-------|-----------|--------|------------|-------|
| Phase 1 | 2-4h | ___h | ___h | _____ |
| Phase 2 | 8-16h | ___h | ___h | _____ |
| Phase 3 | 4-8h | ___h | ___h | _____ |
| Phase 4 | 4-8h | ___h | ___h | _____ |
| Phase 5 | 8-16h | ___h | ___h | _____ |
| **Total** | **26-52h** | **___h** | **___h** | **_____** |

---

**Checklist Version:** 1.0  
**Last Updated:** 2025-12-17  
**Status:** Ready for Implementation
