# Parking Client

A comprehensive parking management system built with WPF and .NET 8.

---

## 🚨 Refactoring Analysis Available

A detailed analysis has been completed to assess the feasibility of refactoring this WPF application to support both desktop and WebAPI interfaces with shared business logic.

### 📚 Analysis Documentation

**👉 Start Here:** [README_ANALYSIS.md](README_ANALYSIS.md)

This comprehensive analysis includes:
- **Feasibility Assessment:** MEDIUM-HIGH ✅ (Recommended to proceed)
- **Detailed Analysis:** Current issues and proposed solutions
- **Implementation Guide:** Step-by-step refactoring instructions
- **Architecture Diagrams:** Visual representations of transformation
- **Effort Estimates:** 26-52 hours across 5 phases
- **Implementation Checklist:** Track your progress

### Quick Links

| Document | Purpose | Audience | Time |
|----------|---------|----------|------|
| [REFACTORING_SUMMARY.md](REFACTORING_SUMMARY.md) | Executive summary | Managers, Stakeholders | 15 min |
| [REFACTORING_ANALYSIS.md](REFACTORING_ANALYSIS.md) | Detailed analysis | Architects, Tech Leads | 45 min |
| [IMPLEMENTATION_GUIDE.md](IMPLEMENTATION_GUIDE.md) | Step-by-step guide | Developers | 60 min |
| [ARCHITECTURE_DIAGRAM.md](ARCHITECTURE_DIAGRAM.md) | Visual diagrams | All stakeholders | 20 min |
| [REFACTORING_CHECKLIST.md](REFACTORING_CHECKLIST.md) | Implementation tracker | Developers | During work |

### Key Findings

**Problem:** Business logic is tightly coupled to WPF presentation layer, preventing code reuse for WebAPI.

**Solution:** Create `Parking.Core` shared library to enable code sharing between WPF and WebAPI.

**Benefits:**
- ✅ Code reusability
- ✅ WebAPI support for mobile/web clients
- ✅ Better testability
- ✅ Future-proof architecture

**Effort:** 26-52 hours (2 weeks full-time or 6 weeks part-time)

**Recommendation:** ✅ Proceed with incremental refactoring

---

## Current Architecture

### Projects

- **Parking.Domain** - Domain entities and contracts
- **Parking.Infrastructure** - Data access layer with Entity Framework Core
- **Parking.App** - WPF desktop application

### Technologies

- .NET 8.0
- WPF (Windows Presentation Foundation)
- Entity Framework Core 8.0
- SQL Server
- ASP.NET Identity
- Serilog (Logging)
- CommunityToolkit.Mvvm (MVVM pattern)

### Features

- Parking ticket management (entry/exit)
- User and role management
- Real-time ticket queue
- Price calculation
- Reports and statistics
- License plate recognition (ANPR) support
- Card-based access
- Database synchronization

---

## Getting Started

### Prerequisites

- .NET 8 SDK
- Visual Studio 2022 or JetBrains Rider
- SQL Server (LocalDB or full instance)
- Windows OS (for WPF application)

### Building the Project

```bash
# Clone the repository
git clone https://github.com/itsshahramp/parking-client.git
cd parking-client

# Restore dependencies
dotnet restore

# Build the solution
dotnet build

# Run the WPF application
cd Parking.App
dotnet run
```

### Database Setup

The application uses Entity Framework Core migrations. Connection string is configured in `Settings.settings`.

```bash
# Apply migrations
cd Parking.Infrastructure
dotnet ef database update --startup-project ../Parking.App
```

---

## Refactoring Plan Overview

### Proposed Architecture

After refactoring, the solution will consist of:

- **Parking.Domain** - Domain entities (cross-platform)
- **Parking.Infrastructure** - Data access (cross-platform)
- **Parking.Core** - Shared business logic (NEW)
- **Parking.App** - WPF application (references Core)
- **Parking.Api** - ASP.NET Core WebAPI (NEW, references Core)

### Implementation Phases

1. **Phase 1** (2-4h): Update Domain/Infrastructure to target `net8.0`
2. **Phase 2** (8-16h): Create Parking.Core shared library
3. **Phase 3** (4-8h): Refactor WPF to use Parking.Core
4. **Phase 4** (4-8h): Create WebAPI using Parking.Core
5. **Phase 5** (8-16h): Testing and validation

**Total Effort:** 26-52 hours

See [IMPLEMENTATION_GUIDE.md](IMPLEMENTATION_GUIDE.md) for detailed instructions.

---

## Contributing

Before making changes, please review the refactoring analysis to understand the proposed architecture and ongoing improvements.

---

## License

[Specify your license here]

---

## Contact

[Add contact information]

---

## Additional Resources

- [Refactoring Analysis Documentation](README_ANALYSIS.md) - Complete guide to understanding and implementing the refactoring
- [Visual Architecture Diagrams](ARCHITECTURE_DIAGRAM.md) - See the before/after transformation
- [Implementation Checklist](REFACTORING_CHECKLIST.md) - Track your refactoring progress

---

**Status:** Active development - Refactoring analysis complete ✅  
**Version:** 1.0.26  
**Last Updated:** 2025-12-17
