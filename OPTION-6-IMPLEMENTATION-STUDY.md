# SYNFLOX Option 6 Implementation - Ultimate Deep Dive Study

## 📊 Current Architecture Analysis

### Project Dependency Graph (Clean Architecture)

```
                    ┌─────────────┐
                    │   WebAPI    │
                    │ (50 files)  │
                    └──────┬──────┘
                           │ references
                           ▼
                  ┌────────────────────┐
                  │   Infrastructure   │
                  │   (180 files)      │
                  └────────┬───────────┘
                           │ references
               ┌───────────┴───────────┐
               │                       │
               ▼                       ▼
      ┌─────────────────┐     ┌─────────────┐
      │   Application   │     │   Domain    │
      │   (208 files)   │◄────│ (103 files) │
      └─────────────────┘     └─────────────┘
              │ references        (no deps)
              └────────►
```

### Current File Distribution

| Layer | Files | Purpose |
|-------|-------|---------|
| **Domain** | 103 | Entities, Enums, Interfaces, Exceptions |
| **Application** | 208 | DTOs (155), Mapping (9), Service Interfaces (28) |
| **Infrastructure** | 180 | Services (42), Repositories (27), Migrations (53), Background Jobs (7) |
| **WebAPI** | 50 | Controllers (24), Middlewares (11), Configurations (4) |
| **TOTAL** | **541** | |

### Controllers Split (Admin vs Client)

#### ADMIN Controllers (19) - Used by Admin Dashboard
```
AdminAuthenticationController    → api/admin/auth
AdminManagementController        → api/admins
AdminProfileController           → api/admin/profile
AdminTypesController             → api/admin-types
CompanyAdminController           → api/admin/company-admins
CompanyController                → api/companies
CustomEmailController            → api/custom-email
DashboardController              → api/dashboard
DownloadsController              → api/downloads
MenuItemController               → api/menuitem
ModulesController                → api/modules
OfflineLicenseAdminController    → api/admin/offline-license-tokens
OnlineTokenAdminController       → api/admin/online-tokens
PlanEntitlementsController       → api/plan-entitlements
PlansController                  → api/plans
ProjectsController               → api/projects
SearchController                 → api/search
SubscriptionsController          → api/subscriptions
UploadsController                → api/uploads
```

#### CLIENT Controllers (5) - Used by Client Applications
```
ClientDeviceController           → api/client/devices
ClientOnlineController           → api/client/online
CompanyAdminAuthController       → api/client/admin-auth
OfflineLicenseController         → api/offline-license
OnlineClientController           → api/online
```

### Shared Services Used by Both

| Service | Used by Admin | Used by Client |
|---------|---------------|----------------|
| IOfflineLicenseAdminService | ✅ | ✅ |
| IOnlineClientService | ✅ | ✅ |
| ILocalizationService | ✅ | ✅ |
| ICompanyAdminService | ✅ | ✅ |
| IMapper | ✅ | ✅ |

### Services Only Used by Admin
```
IAuthenticationService, IPasswordResetService, IBackupCodeService
IAdminService, IAdminProfileService, IAdminTypeService
ICompanyService, IDashboardService, IEmailService
IMenuItemsService, IModuleService, IProjectService
ISearchService, ISubscriptionService, ISubscriptionPlanService
IUploadService, IDownloadService, IPlanEntitlementService
ISecurityAnalyticsService, ISecurityReportService
```

### Services Only Used by Client
```
IOnlineJwtService, IOfflineLicenseService
```

---

## 🏗️ Architecture: Smart Sharing (Loose Coupling) ✅

### The Right Approach: Share ONLY What MUST Be Shared

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          SYNFLOX-Parent (Launcher)                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                 SYNFLOX.Core (Submodule) - MINIMAL SHARED            │  │
│   │                                                                       │  │
│   │   ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────┐  │  │
│   │   │  Core.Domain    │  │ Core.Application │  │ Core.Infrastructure │  │  │
│   │   │                 │  │                  │  │                     │  │  │
│   │   │ • ALL Entities  │  │ • Shared DTOs    │  │ • DbContext         │  │  │
│   │   │ • ALL Enums     │  │ • Shared Mapping │  │ • Base Repository   │  │  │
│   │   │ • Repo Interfaces│  │ • Common Interfaces│ • Common Services   │  │  │
│   │   │ • Exceptions    │  │                  │  │ • Migrations        │  │  │
│   │   └─────────────────┘  └─────────────────┘  └─────────────────────┘  │  │
│   │                                                                       │  │
│   │   ┌─────────────────────────────────────────────────────────────┐    │  │
│   │   │                    Core.WebAPI                               │    │  │
│   │   │  • Common Middlewares (Exception, Logging, Unicode, Cache)   │    │  │
│   │   │  • Common Extensions (AddSharedServices, UseSharedMiddleware)│    │  │
│   │   └─────────────────────────────────────────────────────────────┘    │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                    │                                        │
│                    ┌───────────────┴───────────────┐                       │
│                    │ imports                       │ imports               │
│                    ▼                               ▼                       │
│   ┌─────────────────────────────────┐  ┌─────────────────────────────────┐│
│   │     SYNFLOX.Admin.API           │  │     SYNFLOX.Client.API          ││
│   │        (Submodule)              │  │        (Submodule)              ││
│   │                                 │  │                                 ││
│   │  ┌───────────────────────────┐ │  │  ┌───────────────────────────┐  ││
│   │  │    Admin.Application      │ │  │  │    Client.Application     │  ││
│   │  │  • Admin-specific DTOs    │ │  │  │  • Client-specific DTOs   │  ││
│   │  │  • Admin Service Interfaces│ │  │  │  • Client Service Interfaces││
│   │  │  • Admin Mapping Profiles │ │  │  │  • Client Mapping Profiles│  ││
│   │  └───────────────────────────┘ │  │  └───────────────────────────┘  ││
│   │                                 │  │                                 ││
│   │  ┌───────────────────────────┐ │  │  ┌───────────────────────────┐  ││
│   │  │   Admin.Infrastructure    │ │  │  │   Client.Infrastructure   │  ││
│   │  │  • Admin-specific Services│ │  │  │  • Client-specific Services│ ││
│   │  │  • Admin ServiceRegistration│ │ │  │  • Client ServiceRegistration││
│   │  │  • Admin Background Jobs  │ │  │  │  • Client Background Jobs │  ││
│   │  └───────────────────────────┘ │  │  └───────────────────────────┘  ││
│   │                                 │  │                                 ││
│   │  ┌───────────────────────────┐ │  │  ┌───────────────────────────┐  ││
│   │  │      Admin.WebAPI         │ │  │  │      Client.WebAPI        │  ││
│   │  │  • 19 Admin Controllers   │ │  │  │  • 5 Client Controllers   │  ││
│   │  │  • Admin Middlewares      │ │  │  │  • Client Middlewares     │  ││
│   │  │  • Admin Configurations   │ │  │  │  • Client Configurations  │  ││
│   │  │  • Program.cs             │ │  │  │  • Program.cs             │  ││
│   │  │  • appsettings.json       │ │  │  │  • appsettings.json       │  ││
│   │  └───────────────────────────┘ │  │  └───────────────────────────┘  ││
│   └─────────────────────────────────┘  └─────────────────────────────────┘│
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Why This Is LOOSE Coupling:

| Aspect | Explanation |
|--------|-------------|
| **Core is minimal** | Only entities (same DB schema), DbContext, base repos |
| **Each API is independent** | Has its own DTOs, services, configs, middlewares |
| **Each API imports Core** | Just like importing a NuGet package |
| **Each API has its own DI** | AdminServiceRegistration, ClientServiceRegistration |
| **No cross-dependency** | Admin doesn't know Client exists, and vice versa |

### What Goes Where:

#### SYNFLOX.Core (MUST be shared - same DB schema)
```
✅ ALL Entities (Company, License, Subscription, Admin, etc.)
✅ ALL Enums (LicenseType, SubscriptionStatus, etc.)
✅ Repository Interfaces (IBaseRepository, ICompanyRepository, etc.)
✅ DbContext (ApplicationDBContext - same schema for both DBs)
✅ Migrations (run on Admin DB, replicated to Client DB)
✅ Common Services (ILocalizationService, IIdEncryptionService)
✅ Common Middlewares (ExceptionHandling, RequestLogging, Unicode)
```

#### SYNFLOX.Admin.API (Independent)
```
✅ Admin-specific DTOs (AdminDto, AdminTypeDto, DashboardDto)
✅ Admin-specific Services (IAuthenticationService, IDashboardService)
✅ Admin Controllers (19 controllers)
✅ Admin Middlewares (CustomClaimsPrincipal, DeletedUser)
✅ Admin Configurations (Authorization policies, SignalR hub)
✅ Admin Background Jobs (SecurityAnalytics, ExportCleanup)
✅ AdminServiceRegistration (registers Core + Admin services)
✅ Admin appsettings.json (Admin DB connection, Admin JWT settings)
```

#### SYNFLOX.Client.API (Independent)
```
✅ Client-specific DTOs (OnlineTokenDto, DeviceBindingDto)
✅ Client-specific Services (IOnlineJwtService, IOfflineLicenseService)
✅ Client Controllers (5 controllers)
✅ Client Middlewares (ClientTokenValidation)
✅ Client Configurations (Client-specific policies)
✅ Client Background Jobs (TokenExpiry, SubscriptionChange)
✅ ClientServiceRegistration (registers Core + Client services)
✅ Client appsettings.json (Client DB connection, Client JWT settings)
```

---

## 📁 Proposed Repository Structure

### Git Repositories to Create (3 repos, not 4!)

```
1. synflox-parent       (Parent/Launcher + SYNFLOX.Core inside)
2. synflox-admin-api    (Full Admin Clean Architecture)
3. synflox-client-api   (Full Client Clean Architecture)
```

### Why Core is INSIDE Parent (Not a Submodule):
- Core changes rarely (entities/schema are stable)
- Avoids nested submodule complexity
- Both APIs reference it via relative path `../core/`
- Still version controlled in parent repo

### Directory Structure

```
synflox-parent/
├── .git/
├── .gitmodules                      # Only 2 submodules
├── docker-compose.yml               # Run everything locally
├── docker-compose.prod.yml          # Production config
├── SYNFLOX.sln                      # Opens everything (all 7 projects)
├── README.md
│
├── core/                            # INSIDE parent (not submodule)
│   ├── SYNFLOX.Core.Domain/
│   │   ├── Entities/                # ALL entities (shared DB schema)
│   │   ├── Enums/                   # ALL enums
│   │   ├── Interfaces/              # Repository interfaces
│   │   ├── Exceptions/              # Common exceptions
│   │   └── SYNFLOX.Core.Domain.csproj
│   │
│   ├── SYNFLOX.Core.Application/
│   │   ├── DTOs/                    # ONLY shared DTOs
│   │   ├── Mapping/                 # Common AutoMapper converters
│   │   ├── Interfaces/              # Common service interfaces
│   │   └── SYNFLOX.Core.Application.csproj
│   │
│   ├── SYNFLOX.Core.Infrastructure/
│   │   ├── Context/                 # ApplicationDBContext
│   │   ├── Migrations/              # ALL migrations
│   │   ├── Repositories/            # Base repositories
│   │   ├── Services/                # Common services only
│   │   └── SYNFLOX.Core.Infrastructure.csproj
│   │
│   └── SYNFLOX.Core.WebAPI/
│       ├── Middlewares/             # Common middlewares
│       ├── Extensions/              # AddCoreServices(), UseCoreMiddleware()
│       └── SYNFLOX.Core.WebAPI.csproj
│
├── admin-api/                       # Git submodule → synflox-admin-api
│   ├── src/
│   │   ├── SYNFLOX.Admin.Application/
│   │   │   ├── DTOs/                # Admin-specific DTOs
│   │   │   ├── Interfaces/          # Admin service interfaces
│   │   │   ├── Mapping/             # Admin mapping profiles
│   │   │   └── SYNFLOX.Admin.Application.csproj
│   │   │
│   │   ├── SYNFLOX.Admin.Infrastructure/
│   │   │   ├── Services/            # Admin-specific services
│   │   │   ├── BackgroundJobs/      # Admin background jobs
│   │   │   ├── ServiceRegistration.cs  # Admin DI
│   │   │   └── SYNFLOX.Admin.Infrastructure.csproj
│   │   │
│   │   └── SYNFLOX.Admin.WebAPI/
│   │       ├── Controllers/         # 19 Admin controllers
│   │       ├── Middlewares/         # Admin-specific middlewares
│   │       ├── Configurations/      # Admin configs
│   │       ├── Hubs/                # SignalR hubs
│   │       ├── Program.cs           # Admin startup
│   │       ├── appsettings.json     # Admin DB connection
│   │       └── SYNFLOX.Admin.WebAPI.csproj
│   │
│   └── SYNFLOX.Admin.API.sln        # Admin-only solution
│
└── client-api/                      # Git submodule → synflox-client-api
    ├── src/
    │   ├── SYNFLOX.Client.Application/
    │   │   ├── DTOs/                # Client-specific DTOs
    │   │   ├── Interfaces/          # Client service interfaces
    │   │   ├── Mapping/             # Client mapping profiles
    │   │   └── SYNFLOX.Client.Application.csproj
    │   │
    │   ├── SYNFLOX.Client.Infrastructure/
    │   │   ├── Services/            # Client-specific services
    │   │   ├── BackgroundJobs/      # Client background jobs
    │   │   ├── ServiceRegistration.cs  # Client DI
    │   │   └── SYNFLOX.Client.Infrastructure.csproj
    │   │
    │   └── SYNFLOX.Client.WebAPI/
    │       ├── Controllers/         # 5 Client controllers
    │       ├── Middlewares/         # Client-specific middlewares
    │       ├── Configurations/      # Client configs
    │       ├── Program.cs           # Client startup
    │       ├── appsettings.json     # Client DB connection
    │       └── SYNFLOX.Client.WebAPI.csproj
    │
    └── SYNFLOX.Client.API.sln       # Client-only solution
```

---

## 🔧 Solution File Structure

### Parent Solution (SYNFLOX.Parent.sln)

```xml
Microsoft Visual Studio Solution File, Format Version 12.00

Project("{2150E333-8FDC-42A3-9474-1A3956D46DE8}") = "Shared", "Shared", "{GUID}"
EndProject

Project("{FAE04EC0-301F-11D3-BF4B-00C04F79EFBC}") = "Domain", "shared\Domain\Domain.csproj", "{GUID}"
EndProject

Project("{FAE04EC0-301F-11D3-BF4B-00C04F79EFBC}") = "Application", "shared\Application\Application.csproj", "{GUID}"
EndProject

Project("{FAE04EC0-301F-11D3-BF4B-00C04F79EFBC}") = "Infrastructure", "shared\Infrastructure\Infrastructure.csproj", "{GUID}"
EndProject

Project("{FAE04EC0-301F-11D3-BF4B-00C04F79EFBC}") = "Shared.WebAPI", "shared\Shared.WebAPI\Shared.WebAPI.csproj", "{GUID}"
EndProject

Project("{2150E333-8FDC-42A3-9474-1A3956D46DE8}") = "APIs", "APIs", "{GUID}"
EndProject

Project("{FAE04EC0-301F-11D3-BF4B-00C04F79EFBC}") = "SYNFLOX.Admin.API", "admin-api\SYNFLOX.Admin.API.csproj", "{GUID}"
EndProject

Project("{FAE04EC0-301F-11D3-BF4B-00C04F79EFBC}") = "SYNFLOX.Client.API", "client-api\SYNFLOX.Client.API.csproj", "{GUID}"
EndProject

Global
    GlobalSection(SolutionConfigurationPlatforms) = preSolution
        Debug|Any CPU = Debug|Any CPU
        Release|Any CPU = Release|Any CPU
    EndGlobalSection
    
    GlobalSection(ProjectConfigurationPlatforms) = postSolution
        # All projects build in all configurations
    EndGlobalSection
EndGlobal
```

### Running Both Projects

**Option 1: Visual Studio Multiple Startup Projects**
```
Right-click Solution → Properties → Startup Project → Multiple startup projects
- SYNFLOX.Admin.API: Start
- SYNFLOX.Client.API: Start
```

**Option 2: docker-compose (Recommended)**
```yaml
# docker-compose.yml
version: '3.8'
services:
  admin-api:
    build:
      context: .
      dockerfile: admin-api/Dockerfile
    ports:
      - "5035:80"
    depends_on:
      - db-admin
      - redis
      
  client-api:
    build:
      context: .
      dockerfile: client-api/Dockerfile
    ports:
      - "5036:80"
    depends_on:
      - db-client
      - redis
      
  db-admin:
    image: mcr.microsoft.com/mssql/server:2022-latest
    ports:
      - "1433:1433"
      
  db-client:
    image: mcr.microsoft.com/mssql/server:2022-latest
    ports:
      - "1434:1433"
      
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
```

**Option 3: VS Code Tasks**
```json
// .vscode/tasks.json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "Run Both APIs",
            "dependsOn": ["Run Admin API", "Run Client API"],
            "dependsOrder": "parallel"
        },
        {
            "label": "Run Admin API",
            "type": "process",
            "command": "dotnet",
            "args": ["run", "--project", "admin-api/SYNFLOX.Admin.API.csproj"]
        },
        {
            "label": "Run Client API",
            "type": "process",
            "command": "dotnet",
            "args": ["run", "--project", "client-api/SYNFLOX.Client.API.csproj"]
        }
    ]
}
```

---

## 🗄️ Database & Migrations Deep Dive

### How Migrations Work with 2 DBs

```
┌─────────────────────────────────────────────────────────────────┐
│                     MIGRATION FLOW                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Developer runs: dotnet ef migrations add NewFeature           │
│                          │                                      │
│                          ▼                                      │
│   ┌─────────────────────────────────────────┐                  │
│   │    shared/Infrastructure/Migrations/     │                  │
│   │    └── 20241210_NewFeature.cs           │                  │
│   └─────────────────────────────────────────┘                  │
│                          │                                      │
│                          ▼                                      │
│   Developer runs: dotnet ef database update                     │
│   (on Admin API with Admin DB connection)                       │
│                          │                                      │
│                          ▼                                      │
│   ┌─────────────────┐                ┌─────────────────┐       │
│   │   ADMIN DB      │ ═══════════════► CLIENT DB       │       │
│   │   (Primary)     │  SQL Replication │   (Replica)    │       │
│   │   Updated ✓     │  auto-syncs     │   Updated ✓    │       │
│   └─────────────────┘                └─────────────────┘       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Migration Commands

```bash
# Create migration (run from parent folder)
dotnet ef migrations add NewFeature \
    --project shared/Infrastructure \
    --startup-project admin-api

# Apply to Admin DB (Primary)
dotnet ef database update \
    --project shared/Infrastructure \
    --startup-project admin-api

# Client DB (Replica) syncs automatically via SQL Server replication
# NO MANUAL UPDATE NEEDED for replica!
```

### Development Mode (Without Replication)

For local development without SQL Server replication:

```csharp
// In Admin API Program.cs
if (app.Environment.IsDevelopment())
{
    // Apply migrations to Admin DB
    using var scope = app.Services.CreateScope();
    var adminDb = scope.ServiceProvider.GetRequiredService<ApplicationDBContext>();
    await adminDb.Database.MigrateAsync();
}

// In Client API Program.cs
if (app.Environment.IsDevelopment())
{
    // Apply same migrations to Client DB (simulating replication)
    using var scope = app.Services.CreateScope();
    var clientDb = scope.ServiceProvider.GetRequiredService<ApplicationDBContext>();
    await clientDb.Database.MigrateAsync();
}
```

### Production Mode (With Replication)

```csharp
// Only Admin API applies migrations
// admin-api/Program.cs
if (app.Environment.IsProduction())
{
    using var scope = app.Services.CreateScope();
    var adminDb = scope.ServiceProvider.GetRequiredService<ApplicationDBContext>();
    await adminDb.Database.MigrateAsync();
    // Client DB syncs via SQL Server Always-On replication
}
```

---

## 📦 Shared.WebAPI - What Goes There?

### Files to Move to Shared.WebAPI

```
Shared.WebAPI/
├── Middlewares/                     # Shared by both APIs
│   ├── ExceptionHandlingMiddleware.cs
│   ├── CacheHeadersMiddleware.cs
│   ├── CacheHeadersOptions.cs
│   ├── ETagMiddleware.cs
│   ├── ETagOptions.cs
│   ├── RequestLoggingMiddleware.cs
│   ├── UnicodeHeaderMiddleware.cs
│   ├── EarlyUnicodeHeaderMiddleware.cs
│   └── CustomRequestCultureProvider.cs
│
├── Configurations/                  # Shared configurations
│   ├── CorsConfiguration.cs
│   ├── SwaggerConfiguration.cs
│   └── RateLimiterConfiguration.cs
│
├── Extensions/                      # Shared startup extensions
│   ├── MiddlewareExtensions.cs      # UseSharedMiddlewares()
│   ├── ServiceExtensions.cs         # AddSharedServices()
│   └── LocalizationExtensions.cs    # UseSharedLocalization()
│
└── Shared.WebAPI.csproj
```

### Files That Stay in Each API

```
Admin API Only:
├── Configurations/
│   └── AuthorizationConfiguration.cs  # Admin-specific policies
├── Hubs/
│   └── SecurityNotificationHub.cs     # Admin real-time notifications
└── Middlewares/
    ├── CustomClaimsPrincipalMiddleware.cs  # Admin JWT claims
    └── DeletedUserMiddleware.cs            # Admin user checks

Client API Only:
├── Middlewares/
│   └── ClientTokenMiddleware.cs       # Client token validation
└── (minimal - uses shared middlewares)
```

---

## 🔐 Cache Invalidation Architecture

### Redis Pub/Sub for Cache Sync

```
┌──────────────────────────────────────────────────────────────────────┐
│                        CACHE INVALIDATION FLOW                        │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   1. Admin updates license                                           │
│      │                                                               │
│      ▼                                                               │
│   ┌─────────────────────────────────────────────────────────┐       │
│   │  AdminLicenseService.UpdateAsync()                       │       │
│   │                                                          │       │
│   │  // 1. Write to Admin DB                                 │       │
│   │  await _adminDb.SaveChangesAsync();                      │       │
│   │                                                          │       │
│   │  // 2. Invalidate local cache                            │       │
│   │  await _cache.RemoveAsync($"license:{id}");              │       │
│   │                                                          │       │
│   │  // 3. Publish event WITH new data                       │       │
│   │  await _redis.PublishAsync("license:updated",            │       │
│   │      JsonSerializer.Serialize(new {                      │       │
│   │          LicenseId = id,                                 │       │
│   │          Data = updatedLicense,                          │       │
│   │          Timestamp = DateTime.UtcNow                     │       │
│   │      }));                                                │       │
│   └─────────────────────────────────────────────────────────┘       │
│      │                                                               │
│      │ Pub/Sub                                                       │
│      ▼                                                               │
│   ┌─────────────────────────────────────────────────────────┐       │
│   │                    REDIS                                  │       │
│   │                                                          │       │
│   │  Channel: "license:updated"                              │       │
│   │  Subscribers: [ClientAPI-1, ClientAPI-2, ClientAPI-3]    │       │
│   └─────────────────────────────────────────────────────────┘       │
│      │                                                               │
│      │ Broadcast                                                     │
│      ▼                                                               │
│   ┌─────────────────────────────────────────────────────────┐       │
│   │  ClientCacheSubscriber (BackgroundService)               │       │
│   │                                                          │       │
│   │  // Option A: Just invalidate                            │       │
│   │  await _cache.RemoveAsync($"license:{event.LicenseId}"); │       │
│   │                                                          │       │
│   │  // Option B: Update with new data (FASTER!)             │       │
│   │  if (event.Data != null)                                 │       │
│   │  {                                                       │       │
│   │      await _cache.SetAsync(                              │       │
│   │          $"license:{event.LicenseId}",                   │       │
│   │          event.Data);                                    │       │
│   │  }                                                       │       │
│   └─────────────────────────────────────────────────────────┘       │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

### Cache Service Interface

```csharp
// shared/Infrastructure/Services/ICacheService.cs
public interface ICacheService
{
    // Get from cache
    Task<T?> GetAsync<T>(string key);
    
    // Set with TTL
    Task SetAsync<T>(string key, T value, TimeSpan? ttl = null);
    
    // Remove from cache
    Task RemoveAsync(string key);
    
    // Pub/Sub
    Task PublishAsync<T>(string channel, T message);
    Task SubscribeAsync<T>(string channel, Func<T, Task> handler);
}

// shared/Infrastructure/Services/RedisCacheService.cs
public class RedisCacheService : ICacheService
{
    private readonly IConnectionMultiplexer _redis;
    private readonly IDatabase _db;
    private readonly JsonSerializerOptions _jsonOptions;
    
    public RedisCacheService(IConnectionMultiplexer redis)
    {
        _redis = redis;
        _db = redis.GetDatabase();
        _jsonOptions = new JsonSerializerOptions
        {
            PropertyNamingPolicy = JsonNamingPolicy.CamelCase
        };
    }
    
    public async Task<T?> GetAsync<T>(string key)
    {
        var value = await _db.StringGetAsync(key);
        return value.HasValue 
            ? JsonSerializer.Deserialize<T>(value!, _jsonOptions) 
            : default;
    }
    
    public async Task SetAsync<T>(string key, T value, TimeSpan? ttl = null)
    {
        var json = JsonSerializer.Serialize(value, _jsonOptions);
        await _db.StringSetAsync(key, json, ttl ?? TimeSpan.FromMinutes(5));
    }
    
    public async Task RemoveAsync(string key)
    {
        await _db.KeyDeleteAsync(key);
    }
    
    public async Task PublishAsync<T>(string channel, T message)
    {
        var json = JsonSerializer.Serialize(message, _jsonOptions);
        var subscriber = _redis.GetSubscriber();
        await subscriber.PublishAsync(channel, json);
    }
    
    public async Task SubscribeAsync<T>(string channel, Func<T, Task> handler)
    {
        var subscriber = _redis.GetSubscriber();
        await subscriber.SubscribeAsync(channel, async (_, message) =>
        {
            if (message.HasValue)
            {
                var data = JsonSerializer.Deserialize<T>(message!, _jsonOptions);
                if (data != null)
                {
                    await handler(data);
                }
            }
        });
    }
}
```

---

## 📋 Implementation Steps

### Phase 1: Create Git Repositories (Day 1)

**You need to create these repos:**
1. `synflox-parent` - Main launcher
2. `synflox-shared` - Domain + Application + Infrastructure
3. `synflox-admin-api` - Admin WebAPI
4. `synflox-client-api` - Client WebAPI

```bash
# After you create the repos, give me the URLs and I'll set up:
# - Git submodules configuration
# - Project references
# - Solution file
# - Docker compose
```

### Phase 2: Move Current Code (Day 1-2)

```
1. Move Domain/ → synflox-shared/Domain/
2. Move Application/ → synflox-shared/Application/
3. Move Infrastructure/ → synflox-shared/Infrastructure/
4. Create Shared.WebAPI with middlewares
5. Split WebAPI controllers:
   - 19 Admin → synflox-admin-api
   - 5 Client → synflox-client-api
```

### Phase 3: Add Redis Caching (Day 3)

```
1. Add ICacheService interface
2. Add RedisCacheService implementation
3. Add CacheInvalidationService
4. Update services to use caching
```

### Phase 4: Configure Databases (Day 4)

```
1. Set up Admin DB connection
2. Set up Client DB connection
3. Configure SQL Server replication (or simulate in dev)
4. Test migrations flow
```

### Phase 5: Docker & Testing (Day 5)

```
1. Create Dockerfiles
2. Create docker-compose.yml
3. Test running both APIs
4. Test cache invalidation
5. Test DB replication
```

---

## ❓ Questions for You

Before I proceed, please:

### 1. Create Git Repositories

Create these 4 repos and give me the URLs:
- `synflox-parent`
- `synflox-shared`
- `synflox-admin-api`
- `synflox-client-api`

### 2. Confirm Submodule Structure

Do you want:
- **Option A**: shared, admin-api, client-api as submodules under parent ✅ (Recommended)
- **Option B**: Only admin-api and client-api as submodules, shared inside parent

### 3. Confirm Sharing Approach

Do you want:
- **Option A**: Share Domain + Application + Infrastructure + Shared.WebAPI ✅ (Recommended)
- **Option B**: Copy Domain/Application to each API (complete isolation)

---

## 📊 Final Summary

| Aspect | Decision |
|--------|----------|
| **Repositories** | 4 (parent + 3 submodules) |
| **Shared Code** | Domain, Application, Infrastructure, Shared.WebAPI |
| **Admin API** | 19 controllers + Admin-specific middlewares |
| **Client API** | 5 controllers + minimal middlewares |
| **Databases** | 2 (Primary + Replica) |
| **Cache** | 1 Redis cluster (shared) |
| **Migrations** | Run on Admin DB, replica auto-syncs |
| **Dev Cost** | $0 (Docker) |
| **Prod Cost** | ~$175/month |
| **Implementation** | ~5 days |

---

**Ready to start! Just create the repos and give me the URLs!** 🚀
