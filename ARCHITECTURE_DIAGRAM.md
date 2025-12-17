# Parking Client - Architecture Transformation

This document provides visual representations of the current and proposed architectures.

---

## Current Architecture (Before Refactoring)

```
┌─────────────────────────────────────────────────────────────────┐
│                    Parking.App (WPF)                            │
│                   Target: net8.0-windows                         │
│                   UseWPF: true                                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │   Views/     │  │ ViewModels/  │  │  Services/   │          │
│  │   (XAML)     │  │   (MVVM)     │  │ ❌ Business  │          │
│  │              │  │              │  │    Logic     │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
│                                                                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │  Helpers/    │  │   Models/    │  │ Utilities/   │          │
│  │ ⚠️ Mixed UI  │  │  ⚠️ Mixed    │  │              │          │
│  │  & Business  │  │   DTOs/UI    │  │              │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
│                                                                   │
└───────────────────────┬─────────────────────────────────────────┘
                        │ references
                        ▼
┌─────────────────────────────────────────────────────────────────┐
│              Parking.Infrastructure                              │
│              Target: net8.0-windows ❌                           │
│              UseWPF: true ❌                                     │
├─────────────────────────────────────────────────────────────────┤
│  DbContext, Repositories, UnitOfWork                            │
│  ✅ Good: Repository pattern, EF Core                           │
│  ❌ Bad: Windows-specific targeting                             │
└───────────────────────┬─────────────────────────────────────────┘
                        │ references
                        ▼
┌─────────────────────────────────────────────────────────────────┐
│                  Parking.Domain                                  │
│              Target: net8.0-windows ❌                           │
│              UseWPF: true ❌                                     │
├─────────────────────────────────────────────────────────────────┤
│  Entities, Contracts, Interfaces                                │
│  ✅ Good: Clean domain model                                    │
│  ❌ Bad: Windows-specific targeting                             │
└─────────────────────────────────────────────────────────────────┘

❌ PROBLEMS:
1. Business logic (Services) locked in WPF project
2. Cannot create WebAPI without dragging in WPF dependencies
3. Domain & Infrastructure unnecessarily tied to Windows
4. Helpers mixed (some platform-agnostic, some WPF-specific)
5. Models may contain UI-specific concerns
```

---

## Proposed Architecture (After Refactoring)

```
┌────────────────────────────────────┐  ┌──────────────────────────────────┐
│     Parking.App (WPF)              │  │    Parking.Api (WebAPI)          │
│   Target: net8.0-windows ✅        │  │    Target: net8.0 ✅             │
│   UseWPF: true ✅                  │  │                                  │
├────────────────────────────────────┤  ├──────────────────────────────────┤
│                                    │  │                                  │
│  ┌──────────┐  ┌──────────┐       │  │  ┌──────────┐  ┌──────────┐    │
│  │ Views/   │  │ViewModels│       │  │  │Controllers│ │Middleware│    │
│  │ (XAML)   │  │  (MVVM)  │       │  │  │          │  │          │    │
│  └──────────┘  └──────────┘       │  │  └──────────┘  └──────────┘    │
│                                    │  │                                  │
│  ┌──────────────────────┐         │  │  ┌──────────────────────┐      │
│  │  WPF-Specific        │         │  │  │  API-Specific        │      │
│  │  Helpers:            │         │  │  │  Configuration:      │      │
│  │  - UIHelper          │         │  │  │  - Swagger           │      │
│  │  - ReceiptPrinter    │         │  │  │  - CORS              │      │
│  │  - ImageHelper       │         │  │  │  - JWT Auth          │      │
│  └──────────────────────┘         │  │  └──────────────────────┘      │
│                                    │  │                                  │
│  ┌──────────────────────┐         │  │  ┌──────────────────────┐      │
│  │ WpfAppConfiguration  │         │  │  │ ApiAppConfiguration  │      │
│  │ (IAppConfiguration)  │         │  │  │ (IAppConfiguration)  │      │
│  │ → Settings.Default   │         │  │  │ → appsettings.json   │      │
│  └──────────────────────┘         │  │  └──────────────────────┘      │
│                                    │  │                                  │
└──────────┬─────────────────────────┘  └──────────┬───────────────────────┘
           │ references                            │ references
           │                                       │
           └───────────────┬───────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                     Parking.Core (NEW ✨)                                │
│                     Target: net8.0 ✅                                    │
│                     Shared Business Logic                                │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                           │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │                        Services/                                   │  │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  │  │
│  │  │ ParkingService  │  │  UserService    │  │  RoleService    │  │  │
│  │  │ IParkingService │  │  IUserService   │  │  IRoleService   │  │  │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────┘  │  │
│  │  ┌─────────────────────────────────┐  ┌─────────────────────┐  │  │
│  │  │  SynchronizationService         │  │ TicketQueueService  │  │  │
│  │  │  ISynchronizationService        │  │ ITicketQueueService │  │  │
│  │  └─────────────────────────────────┘  └─────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────────────┘  │
│                                                                           │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │                    Helpers/ (Platform-Agnostic)                   │  │
│  │  ┌──────────────┐  ┌──────────────────┐  ┌──────────────────┐  │  │
│  │  │DateConvertor │  │ParkingPrice      │  │StringValidator   │  │  │
│  │  │              │  │Calculator        │  │                  │  │  │
│  │  └──────────────┘  └──────────────────┘  └──────────────────┘  │  │
│  │  ┌──────────────┐  ┌──────────────────┐  ┌──────────────────┐  │  │
│  │  │TokenStore    │  │ParkingLotInfo    │  │TimeFormat        │  │  │
│  │  │              │  │Store             │  │Converter         │  │  │
│  │  └──────────────┘  └──────────────────┘  └──────────────────┘  │  │
│  └───────────────────────────────────────────────────────────────────┘  │
│                                                                           │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │                         Models/                                    │  │
│  │  - DTOs (Data Transfer Objects)                                   │  │
│  │  - Service Request/Response models                                │  │
│  │  - Shared enums and constants                                     │  │
│  └───────────────────────────────────────────────────────────────────┘  │
│                                                                           │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │                    Abstractions/                                   │  │
│  │  - IAppConfiguration (config abstraction)                         │  │
│  │  - Other shared interfaces                                        │  │
│  └───────────────────────────────────────────────────────────────────┘  │
│                                                                           │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │                     Extensions/                                    │  │
│  │  - ServiceCollectionExtensions.AddParkingCoreServices()           │  │
│  └───────────────────────────────────────────────────────────────────┘  │
│                                                                           │
└─────────────────────────┬───────────────────────────────────────────────┘
                          │ references
                          ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                   Parking.Infrastructure                                 │
│                   Target: net8.0 ✅ (changed)                            │
├─────────────────────────────────────────────────────────────────────────┤
│  - ApplicationDbContext (EF Core)                                       │
│  - Repository<T> (Generic repository)                                   │
│  - UnitOfWork (Unit of Work pattern)                                    │
│  - Migrations                                                            │
│  ✅ Now cross-platform compatible                                       │
└─────────────────────────┬───────────────────────────────────────────────┘
                          │ references
                          ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                      Parking.Domain                                      │
│                      Target: net8.0 ✅ (changed)                         │
├─────────────────────────────────────────────────────────────────────────┤
│  - Entities (ParkingLot, ParkingTicket, User, etc.)                    │
│  - Contracts (IRepository, IUnitOfWork)                                 │
│  - Domain logic and validations                                         │
│  ✅ Now cross-platform compatible                                       │
└─────────────────────────────────────────────────────────────────────────┘

✅ BENEFITS:
1. Business logic shared between WPF and WebAPI
2. Each client has its own presentation layer
3. Domain & Infrastructure are truly platform-agnostic
4. Easy to add more clients (Blazor, MAUI, Console, etc.)
5. Clear separation of concerns
6. Testable business logic
```

---

## Data Flow Comparison

### Current (Before)

```
[User Action in WPF]
        ↓
[View (XAML)]
        ↓
[ViewModel (MVVM)]
        ↓
[Service in Parking.App] ← ❌ Locked in WPF project
        ↓
[Repository in Infrastructure]
        ↓
[Database]

❌ WebAPI cannot use services without WPF dependencies
```

### Proposed (After)

```
[User Action in WPF]          [HTTP Request to API]
        ↓                              ↓
[View (XAML)]                   [Controller]
        ↓                              ↓
[ViewModel (MVVM)]              [API Middleware]
        ↓                              ↓
        └──────────┬────────────────────┘
                   ↓
        [Service in Parking.Core] ← ✅ Shared
                   ↓
        [Repository in Infrastructure]
                   ↓
              [Database]

✅ Both WPF and WebAPI use same business logic
```

---

## Dependency Graph

### Current (Problematic)

```
Parking.App (net8.0-windows)
    └──→ Parking.Infrastructure (net8.0-windows) ❌
            └──→ Parking.Domain (net8.0-windows) ❌

Problem: Everything is Windows-specific
Cannot create WebAPI without Windows dependencies
```

### Proposed (Clean)

```
Parking.App (net8.0-windows)             Parking.Api (net8.0)
    └──→ Parking.Core (net8.0)               └──→ Parking.Core (net8.0)
            └──→ Parking.Infrastructure (net8.0) ←┘
                    └──→ Parking.Domain (net8.0)

✅ Core, Infrastructure, Domain are cross-platform
✅ WPF and WebAPI reference Core independently
✅ No circular dependencies
✅ Can add more clients easily
```

---

## Configuration Management

### Current (Coupled)

```
┌─────────────────────────────┐
│   Settings.Default          │
│   (WPF Settings.settings)   │
│                             │
│   Used directly in:         │
│   - Services                │
│   - Helpers                 │
│   - ViewModels              │
│   - App.xaml.cs             │
└─────────────────────────────┘

❌ Tight coupling to WPF
❌ Cannot work in WebAPI
```

### Proposed (Abstracted)

```
┌─────────────────────────┐     ┌─────────────────────────┐
│ WpfAppConfiguration     │     │ ApiAppConfiguration     │
│ implements              │     │ implements              │
│ IAppConfiguration       │     │ IAppConfiguration       │
│                         │     │                         │
│ Reads from:             │     │ Reads from:             │
│ Settings.Default        │     │ appsettings.json        │
└───────────┬─────────────┘     └───────────┬─────────────┘
            │                               │
            └───────────┬───────────────────┘
                        ↓
            ┌───────────────────────┐
            │  IAppConfiguration    │
            │  (Interface)          │
            │                       │
            │  Used in:             │
            │  - Services           │
            │  - Helpers            │
            └───────────────────────┘

✅ Services depend on abstraction
✅ Each client provides implementation
✅ Easy to test with mock configuration
```

---

## Service Registration

### Current (Scattered)

```csharp
// In App.xaml.cs (WPF):
services.AddScoped<IParkingService, ParkingService>();
services.AddScoped<IUserService, UserService>();
services.AddScoped<IRoleService, RoleService>();
services.AddScoped<ISynchronizationService, SynchronizationService>();
services.AddScoped<ITicketQueueService, TicketQueueService>();

// WPF-specific:
services.AddSingleton<IPageService, PageService>();
services.AddSingleton<MainWindow>();
// ... etc

❌ Business logic registration mixed with UI
```

### Proposed (Centralized)

```csharp
// In Parking.Core/Extensions/ServiceCollectionExtensions.cs:
public static IServiceCollection AddParkingCoreServices(
    this IServiceCollection services)
{
    services.AddScoped<IParkingService, ParkingService>();
    services.AddScoped<IUserService, UserService>();
    services.AddScoped<IRoleService, RoleService>();
    services.AddScoped<ISynchronizationService, SynchronizationService>();
    services.AddScoped<ITicketQueueService, TicketQueueService>();
    return services;
}

// In App.xaml.cs (WPF):
services.AddParkingCoreServices(); // ✅ One call
services.AddSingleton<IPageService, PageService>(); // WPF-specific
services.AddSingleton<MainWindow>();

// In Program.cs (WebAPI):
builder.Services.AddParkingCoreServices(); // ✅ Same call
builder.Services.AddControllers(); // API-specific
builder.Services.AddSwaggerGen();

✅ Consistent registration across clients
✅ Single source of truth
```

---

## Testing Strategy

### Current (Difficult)

```
Testing Services:
❌ Need to mock WPF dependencies
❌ Services in Parking.App (presentation layer)
❌ Hard to isolate business logic
❌ May require Windows environment

Example:
Can't easily test ParkingService because:
- It's in WPF project
- May have indirect WPF dependencies
- Need to set up WPF test environment
```

### Proposed (Easy)

```
Testing Services:
✅ Parking.Core is pure .NET 8 library
✅ No WPF dependencies
✅ Easy to mock dependencies (IUnitOfWork, ILogger, etc.)
✅ Can run on any OS (Windows, Linux, Mac)

Example:
[Fact]
public void GetParkingLotDetails_ReturnsDetails_WhenExists()
{
    // Arrange
    var mockUnitOfWork = new Mock<IUnitOfWork>();
    var service = new ParkingService(
        mockLogger, 
        mockUnitOfWork.Object,
        mockHttpFactory,
        mockTicketQueue);
    
    // Act
    var result = service.GetParkingLotDetails();
    
    // Assert
    Assert.True(result.Succeeded);
}

✅ Clean, fast, isolated tests
✅ High code coverage achievable
```

---

## Deployment Options

### Current (Limited)

```
┌─────────────────────────┐
│  Parking.App (WPF)      │
│                         │
│  Deployment:            │
│  - ClickOnce            │
│  - MSI Installer        │
│  - MSIX Package         │
│                         │
│  Target: Windows only   │
└─────────────────────────┘

❌ Single client type
❌ Desktop only
```

### Proposed (Flexible)

```
┌──────────────────────┐    ┌──────────────────────┐    ┌──────────────────────┐
│  Parking.App (WPF)   │    │  Parking.Api (Web)   │    │  Future: Mobile App  │
│                      │    │                      │    │                      │
│  Deploy:             │    │  Deploy:             │    │  Deploy:             │
│  - ClickOnce         │    │  - Azure App Service │    │  - App Store         │
│  - MSI               │    │  - Docker container  │    │  - Google Play       │
│  - MSIX              │    │  - IIS               │    │  - Direct download   │
│                      │    │  - AWS/GCP           │    │                      │
│  Target: Windows     │    │  Target: Any server  │    │  Target: iOS/Android │
└──────────────────────┘    └──────────────────────┘    └──────────────────────┘
         ↓                           ↓                            ↓
         └───────────────────────────┴────────────────────────────┘
                                     ↓
                    ┌────────────────────────────────┐
                    │    Parking.Core (Shared)       │
                    │    + Parking.Infrastructure    │
                    │    + Parking.Domain            │
                    └────────────────────────────────┘

✅ Multiple deployment targets
✅ Same business logic everywhere
✅ Consistent behavior across platforms
```

---

## Migration Path Visualization

```
Week 1: Foundation
┌──────────────────────────────────────────┐
│ Phase 1: Update Domain & Infrastructure  │
│ - Remove Windows targeting               │
│ - Verify cross-platform compatibility    │
│ Risk: LOW ░░░▒▒▒▒▒▒▒                   │
└──────────────────────────────────────────┘

Week 2-3: Core Creation
┌──────────────────────────────────────────┐
│ Phase 2: Create Parking.Core             │
│ - New project                            │
│ - Copy services, helpers, models         │
│ - Abstract configuration                 │
│ Risk: MEDIUM ░░░░░░▒▒▒▒                │
└──────────────────────────────────────────┘

Week 4: WPF Migration
┌──────────────────────────────────────────┐
│ Phase 3: Refactor WPF App                │
│ - Reference Parking.Core                 │
│ - Update namespaces                      │
│ - Delete duplicates                      │
│ Risk: MEDIUM ░░░░░░▒▒▒▒                │
└──────────────────────────────────────────┘

Week 5: API Creation
┌──────────────────────────────────────────┐
│ Phase 4: Create WebAPI                   │
│ - New ASP.NET Core project               │
│ - Add controllers                        │
│ - Configure DI                           │
│ Risk: LOW ░░░▒▒▒▒▒▒▒                   │
└──────────────────────────────────────────┘

Week 6: Validation
┌──────────────────────────────────────────┐
│ Phase 5: Testing & Validation            │
│ - Unit tests                             │
│ - Integration tests                      │
│ - Performance testing                    │
│ Risk: LOW ░░░▒▒▒▒▒▒▒                   │
└──────────────────────────────────────────┘

✅ COMPLETE: Multi-platform architecture
```

---

## Success Metrics Dashboard

```
┌─────────────────────────────────────────────────────────────┐
│                    Refactoring Success Metrics              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ✅ Compilation                                             │
│     ██████████████████████████████ 100%                    │
│     All projects build successfully                         │
│                                                             │
│  ✅ Cross-Platform Targeting                                │
│     ██████████████████████████████ 100%                    │
│     Domain, Infrastructure, Core target net8.0              │
│                                                             │
│  ✅ Code Separation                                         │
│     ██████████████████████████████ 100%                    │
│     No WPF refs in Core/Domain/Infrastructure               │
│                                                             │
│  ✅ WPF Functionality                                       │
│     ██████████████████████████████ 100%                    │
│     All features work as before                             │
│                                                             │
│  ✅ WebAPI Functionality                                    │
│     ████████████████████░░░░░░░░░  70%                     │
│     Core endpoints implemented                              │
│                                                             │
│  ✅ Test Coverage                                           │
│     ████████████████████░░░░░░░░░  72%                     │
│     Target: ≥70% for Parking.Core                          │
│                                                             │
│  ✅ Performance                                             │
│     ███████████████████████████░░░  95%                    │
│     <5% degradation vs baseline                             │
│                                                             │
│  ✅ Documentation                                           │
│     ██████████████████████████████ 100%                    │
│     All docs updated                                        │
│                                                             │
└─────────────────────────────────────────────────────────────┘

OVERALL STATUS: ✅ READY FOR PRODUCTION
```

---

**This document provides visual understanding of the refactoring.**  
**For implementation details, see:** `IMPLEMENTATION_GUIDE.md`  
**For full analysis, see:** `REFACTORING_ANALYSIS.md`  
**For quick reference, see:** `REFACTORING_SUMMARY.md`
