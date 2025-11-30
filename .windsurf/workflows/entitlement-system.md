---
description: Enterprise Entitlement System - Complete Implementation Workflow
auto_execution_mode: 3
---

# 🏢 SYNFLOX Enterprise Entitlement System

## 📊 Progress Tracker

**Current Phase:** Phase 4 - Core Services  
**Current Step:** Step 4.1 - EntitlementService CRUD  
**Last Updated:** November 30, 2025 - 5:25 PM

### Phase Status
- [x] **Phase 1:** Domain Layer (Enums & Entities) ✅ COMPLETE
- [x] **Phase 2:** Infrastructure Data Layer (EF Config, Repository, Migration) ✅ COMPLETE
- [x] **Phase 3:** Application Layer (DTOs & Interfaces) ✅ COMPLETE
- [ ] **Phase 4:** Core Services (EntitlementService) ⬅️ NEXT
- [ ] **Phase 5:** Token Architecture (Thin Token + Versioned Entitlements)
- [ ] **Phase 6:** Offline License System
- [ ] **Phase 7:** Background Jobs (Access Mode Transitions)
- [ ] **Phase 8:** API Layer (Controllers)
- [ ] **Phase 9:** Frontend Domain (Models, Mappers, Services)
- [ ] **Phase 10:** Frontend UI (Components)
- [ ] **Phase 11:** Testing & Documentation

---

## 🏗️ ARCHITECTURE DECISIONS

### Online Systems: Thin Token + Dynamic Authorization
```
Token (JWT):
├── company_id
├── subscription_id  
├── token_id
├── token_version
├── exp, iat
└── NO entitlements (fetched separately)

Entitlements:
├── Fetched via GET /api/client/entitlements
├── Cached by client (24 hours or until version change)
├── Version tracked via X-Entitlements-Version header
└── Changes instantly without token regeneration
```

### Offline Systems: Self-Contained License
```
License Key (Encrypted):
├── company_id, subscription_id
├── FULL entitlement matrix
├── access_mode
├── expiry dates (expiry, grace_end, export_deadline)
├── version number
└── cryptographic signature
```

### Background Job Purpose
```
AccessModeTransitionJob (Runs every hour):
├── Check subscriptions where ExpiryDateUtc < now
├── Transition: Active → GracePeriod
├── Transition: GracePeriod → ExportOnly/ReadOnly  
├── Transition: ExportOnly → Blocked
├── Increment entitlements_version on change
└── NOT for checking client access (that's DB-driven)
```

### Caching Strategy
```
Server-side:
├── Cache entitlement matrix per subscription
├── Invalidate on: grant, revoke, upgrade, access mode change
├── Version incremented on any change

Client-side:
├── Cache entitlements for 24 hours (server time)
├── Check X-Entitlements-Version on every response
├── Refresh if version mismatch
├── Clear cache on upgrade/token refresh
```

---

## 📋 IMPLEMENTATION PHASES

---

### PHASE 1: Domain Layer
**Status:** ⬜ Not Started  
**Estimated:** Day 1-2

#### Step 1.1: Create Enums
- [ ] `Domain/Enums/EntitlementGrantType.cs`
- [ ] `Domain/Enums/EntitlementSource.cs`
- [ ] `Domain/Enums/EntitlementAccessLevel.cs`
- [ ] `Domain/Enums/SubscriptionAccessMode.cs`

#### Step 1.2: Create SubscriptionEntitlement Entity
- [ ] `Domain/Entities/Subscriptions/SubscriptionEntitlement.cs`

#### Step 1.3: Update Subscription Entity
- [ ] Add `AccessMode` property
- [ ] Add `FallbackPlanId` property
- [ ] Add `ExportDeadlineUtc` property
- [ ] Add `EntitlementsVersion` property (NEW - for versioning)
- [ ] Add `Entitlements` navigation

#### Step 1.4: Update SubscriptionPlan Entity
- [ ] Add `IsFreeTier` property
- [ ] Add `FallbackAccessMode` property
- [ ] Add `ExportGraceDays` property
- [ ] Add `DefaultFallbackPlanId` property

#### Step 1.5: Create Repository Interface
- [ ] `Domain/Interfaces/ISubscriptionEntitlementRepository.cs`

---

### PHASE 2: Infrastructure Data Layer
**Status:** ⬜ Not Started  
**Estimated:** Day 3-4

#### Step 2.1: EF Configurations
- [ ] `Infrastructure/Configurations/SubscriptionEntitlementConfiguration.cs`
- [ ] Update `SubscriptionConfiguration.cs`
- [ ] Update `SubscriptionPlanConfiguration.cs`
- [ ] Update `ApplicationDBContext.cs`

#### Step 2.2: Repository Implementation
- [ ] `Infrastructure/Repositories/SubscriptionEntitlementRepository.cs`

#### Step 2.3: Database Migration
- [ ] Create migration file
- [ ] Add SubscriptionEntitlements table
- [ ] Alter Subscriptions table
- [ ] Alter SubscriptionPlans table
- [ ] Create migration script for existing data

---

### PHASE 3: Application Layer
**Status:** ⬜ Not Started  
**Estimated:** Day 5

#### Step 3.1: Entitlement DTOs
- [ ] `Application/DTOs/Entitlements/SubscriptionEntitlementDto.cs`
- [ ] `Application/DTOs/Entitlements/GrantEntitlementRequest.cs`
- [ ] `Application/DTOs/Entitlements/UpdateEntitlementRequest.cs`
- [ ] `Application/DTOs/Entitlements/RevokeEntitlementRequest.cs`

#### Step 3.2: Access Control DTOs
- [ ] `Application/DTOs/Entitlements/AccessCheckRequest.cs`
- [ ] `Application/DTOs/Entitlements/AccessCheckResult.cs`
- [ ] `Application/DTOs/Entitlements/EntitlementMatrixDto.cs`
- [ ] `Application/DTOs/Entitlements/ProjectEntitlementDto.cs`
- [ ] `Application/DTOs/Entitlements/ModuleEntitlementDto.cs`

#### Step 3.3: Update Existing DTOs
- [ ] Update `SubscriptionDto.cs` - Add access mode fields
- [ ] Update `SubscriptionDetailsDto.cs` - Add entitlements
- [ ] Update Plan DTOs - Add free tier fields

#### Step 3.4: Service Interface
- [ ] `Application/Services/IEntitlementService.cs`

---

### PHASE 4: Core Services
**Status:** ⬜ Not Started  
**Estimated:** Day 6-8

#### Step 4.1: EntitlementService - CRUD
- [ ] Create `Infrastructure/Services/EntitlementService.cs`
- [ ] `GetSubscriptionEntitlementsAsync`
- [ ] `GetEntitlementByIdAsync`
- [ ] `GrantEntitlementAsync` (increments version)
- [ ] `UpdateEntitlementAsync` (increments version)
- [ ] `RevokeEntitlementAsync` (increments version)

#### Step 4.2: EntitlementService - Bulk Operations
- [ ] `CopyPlanEntitlementsToSubscriptionAsync`
- [ ] `AddUpgradeEntitlementsAsync`
- [ ] `ReplaceEntitlementsAsync`
- [ ] `DowngradeToFallbackAsync`

#### Step 4.3: EntitlementService - Access Checking
- [ ] `CheckProjectAccessAsync`
- [ ] `CheckModuleAccessAsync`
- [ ] `CheckFeatureAccessAsync`
- [ ] `CheckOperationAsync`
- [ ] `BuildEntitlementMatrixAsync`

#### Step 4.4: EntitlementService - Versioning
- [ ] `GetEntitlementsVersionAsync`
- [ ] `IncrementVersionAsync` (called on any change)

#### Step 4.5: Register Service
- [ ] Register in DI container

---

### PHASE 5: Token Architecture (Online)
**Status:** ⬜ Not Started  
**Estimated:** Day 9-10

#### Step 5.1: Redesign ClientJwtService
- [ ] Remove entitlements from token claims
- [ ] Keep only: company_id, subscription_id, token_id, version
- [ ] Add entitlements_version claim (for client to compare)

#### Step 5.2: Create EntitlementsEndpoint
- [ ] `GET /api/client/entitlements` - Returns full matrix
- [ ] Returns: version, access_mode, projects, modules, usage_limits
- [ ] Cached server-side per subscription

#### Step 5.3: Add Version Header to All Responses
- [ ] Create middleware/filter
- [ ] Add `X-Entitlements-Version` header to all client API responses
- [ ] Client compares to cached version

#### Step 5.4: Update Token Generation
- [ ] Don't regenerate token on entitlement change
- [ ] Only regenerate on: expiry, revocation, security concern

---

### PHASE 6: Offline License System
**Status:** ⬜ Not Started  
**Estimated:** Day 11-12

#### Step 6.1: Update OfflineLicenseData Structure
- [ ] Add `EntitlementMatrix`
- [ ] Add `AccessMode`
- [ ] Add `GraceEndDateUtc`
- [ ] Add `ExportDeadlineUtc`
- [ ] Add `Version` (bump to 3)

#### Step 6.2: Update LicenseService
- [ ] `GenerateOfflineLicenseKeyAsync` - Include full entitlements
- [ ] `ValidateOfflineLicenseKey` - Validate entitlements

#### Step 6.3: License Sync Endpoint
- [ ] `POST /api/client/license/sync`
- [ ] Request: current_version
- [ ] Response: new_license_key if changed, or up_to_date flag

#### Step 6.4: Backward Compatibility
- [ ] Handle v1 and v2 licenses
- [ ] Graceful upgrade path

---

### PHASE 7: Background Jobs
**Status:** ⬜ Not Started  
**Estimated:** Day 12-13

#### Step 7.1: AccessModeTransitionJob
- [ ] Create `Infrastructure/BackgroundJobs/AccessModeTransitionJob.cs`
- [ ] Run every hour
- [ ] Check for expiring subscriptions
- [ ] Transition access modes
- [ ] Increment entitlements_version on transition
- [ ] Log all transitions

#### Step 7.2: EntitlementsCacheCleanupJob
- [ ] Create cleanup job for stale cache entries
- [ ] Run daily

#### Step 7.3: Register Jobs
- [ ] Register with Hangfire/Quartz
- [ ] Configure schedules

---

### PHASE 8: API Layer
**Status:** ⬜ Not Started  
**Estimated:** Day 13-14

#### Step 8.1: EntitlementsController (Admin)
- [ ] Create `WebAPI/Controllers/EntitlementsController.cs`
- [ ] `GET /api/subscriptions/{id}/entitlements`
- [ ] `POST /api/subscriptions/{id}/entitlements`
- [ ] `PUT /api/entitlements/{id}`
- [ ] `DELETE /api/entitlements/{id}`

#### Step 8.2: Update ClientApiController
- [ ] Add `GET /api/client/entitlements`
- [ ] Add `POST /api/client/access/check`
- [ ] Add version header middleware

#### Step 8.3: Update Existing Controllers
- [ ] SubscriptionsController - Include entitlements in details
- [ ] PlansController - Add free tier configuration

#### Step 8.4: AutoMapper Profiles
- [ ] Create `EntitlementMappingProfile.cs`
- [ ] Update existing profiles

---

### PHASE 9: Frontend Domain
**Status:** ⬜ Not Started  
**Estimated:** Day 15

#### Step 9.1: Models
- [ ] `domain/models/entitlement.model.ts`
- [ ] `domain/models/access-check.model.ts`
- [ ] Update `subscription.model.ts`
- [ ] Update `subscription-plan.model.ts`

#### Step 9.2: Mappers
- [ ] `domain/mappers/entitlement.mapper.ts`

#### Step 9.3: Service
- [ ] `services/entitlement.service.ts`

#### Step 9.4: Config Updates
- [ ] `config/api-endpoints.ts`
- [ ] `providers/service-provider.tsx`

---

### PHASE 10: Frontend UI
**Status:** ⬜ Not Started  
**Estimated:** Day 16-17

#### Step 10.1: Entitlement Management Components
- [ ] `components/entitlements/entitlement-list.tsx`
- [ ] `components/entitlements/grant-entitlement-dialog.tsx`
- [ ] `components/entitlements/edit-entitlement-dialog.tsx`
- [ ] `components/entitlements/revoke-entitlement-dialog.tsx`

#### Step 10.2: Access Mode Components
- [ ] `components/access/access-mode-badge.tsx`
- [ ] `components/access/access-mode-banner.tsx`
- [ ] `components/access/upgrade-prompt.tsx`

#### Step 10.3: Plan Configuration
- [ ] Update plan forms with free tier options
- [ ] Fallback plan selector

#### Step 10.4: Subscription Details
- [ ] Add Entitlements tab
- [ ] Show access mode status
- [ ] Show version info

---

### PHASE 11: Testing & Documentation
**Status:** ⬜ Not Started  
**Estimated:** Day 17-18

#### Step 11.1: Backend Testing
- [ ] Entitlement CRUD operations
- [ ] Access mode transitions
- [ ] Token versioning
- [ ] License generation/validation
- [ ] Access check endpoint

#### Step 11.2: Frontend Testing
- [ ] Entitlement management UI
- [ ] Access mode display
- [ ] Plan configuration

#### Step 11.3: Integration Testing
- [ ] Full upgrade flow
- [ ] Full expiry → fallback flow
- [ ] Offline license sync

#### Step 11.4: Documentation
- [ ] API documentation
- [ ] Client SDK guide
- [ ] Architecture documentation

#### Step 11.5: Translations
- [ ] `locales/en.ts` additions
- [ ] `locales/ar.ts` additions

---

## 🔧 TECHNICAL SPECIFICATIONS

### New Database Schema

```sql
-- New Table
CREATE TABLE SubscriptionEntitlements (
    Id UNIQUEIDENTIFIER PRIMARY KEY,
    SubscriptionId UNIQUEIDENTIFIER NOT NULL,
    ProjectId UNIQUEIDENTIFIER NULL,
    ModuleId UNIQUEIDENTIFIER NULL,
    GrantType INT NOT NULL,
    AccessLevel INT NOT NULL DEFAULT 1,
    Source INT NOT NULL,
    IsCustom BIT NOT NULL DEFAULT 0,
    CanCreate BIT NOT NULL DEFAULT 1,
    CanRead BIT NOT NULL DEFAULT 1,
    CanUpdate BIT NOT NULL DEFAULT 1,
    CanDelete BIT NOT NULL DEFAULT 1,
    CanExport BIT NOT NULL DEFAULT 1,
    Features NVARCHAR(2000) NULL,
    UsageLimits NVARCHAR(2000) NULL,
    GrantedByAdminId UNIQUEIDENTIFIER NULL,
    ExpiresAt DATETIME2 NULL,
    Notes NVARCHAR(500) NULL,
    IsActive BIT NOT NULL DEFAULT 1,
    -- Audit fields
    CreatedTimestamp DATETIME2 NOT NULL,
    UpdatedTimestamp DATETIME2 NULL,
    CreatedBy NVARCHAR(100) NULL,
    UpdatedBy NVARCHAR(100) NULL,
    IsDeleted BIT NOT NULL DEFAULT 0
);

-- Subscriptions additions
ALTER TABLE Subscriptions ADD
    AccessMode INT NOT NULL DEFAULT 1,
    FallbackPlanId UNIQUEIDENTIFIER NULL,
    ExportDeadlineUtc DATETIME2 NULL,
    EntitlementsVersion INT NOT NULL DEFAULT 1,
    AccessRestrictionMessage NVARCHAR(500) NULL;

-- SubscriptionPlans additions
ALTER TABLE SubscriptionPlans ADD
    IsFreeTier BIT NOT NULL DEFAULT 0,
    FallbackAccessMode INT NOT NULL DEFAULT 2,
    ExportGraceDays INT NOT NULL DEFAULT 30,
    DefaultFallbackPlanId UNIQUEIDENTIFIER NULL;
```

### Token Structure (Thin - Online)

```json
{
  "jti": "token-guid",
  "iat": 1701302400,
  "exp": 1732838400,
  "company_id": "guid",
  "subscription_id": "guid",
  "token_id": "guid",
  "token_version": "1.0",
  "entitlements_version": 5
}
```

### Entitlements Response Structure

```json
{
  "version": 5,
  "generated_at": "2025-11-30T12:00:00Z",
  "cache_until": "2025-12-01T12:00:00Z",
  "access_mode": "Full",
  "days_remaining": 45,
  "allowed_operations": ["GET", "POST", "PUT", "DELETE", "EXPORT"],
  
  "entitled": {
    "projects": [
      {
        "project_id": "guid",
        "project_name": "ERP",
        "grant_type": "FullProject",
        "access_level": "Full",
        "operations": ["GET", "POST", "PUT", "DELETE", "EXPORT"],
        "modules": [
          { "id": "hr-guid", "name": "HR", "access": "Full" },
          { "id": "finance-guid", "name": "Finance", "access": "Full" }
        ]
      }
    ],
    "standalone_modules": [],
    "usage_limits": {
      "api_calls_per_month": 10000,
      "storage_mb": 5120
    }
  },
  
  "available_upgrades": {
    "show_locked_menus": true,
    "locked_items": [
      {
        "type": "module",
        "project": "ERP",
        "id": "accounting-guid",
        "name": "Accounting",
        "description": "Manage accounts, ledgers, and financial reports",
        "icon": "calculator",
        "display_in_menu": true,
        "upgrade_cta": "Upgrade to Enterprise",
        "upgrade_url": "/upgrade?module=accounting"
      },
      {
        "type": "module",
        "project": "ERP",
        "id": "inventory-guid",
        "name": "Inventory",
        "display_in_menu": true,
        "upgrade_cta": "Upgrade to Enterprise"
      }
    ]
  },
  
  "menu_config": {
    "show_locked_items": true,
    "locked_item_style": "greyed_with_lock",
    "show_upgrade_badge": true,
    "group_locked_separately": false
  }
}
```

### License Key Structure (Offline)

```json
{
  "v": 3,
  "cid": "company-guid",
  "sid": "subscription-guid",
  "cn": "Acme Corp",
  "pn": "Enterprise",
  "iat": 1701302400,
  "exp": 1732838400,
  "gexp": 1733443200,
  "xdl": 1736035200,
  "am": "Full",
  "ent": {
    "projects": [...],
    "modules": [...],
    "limits": {...}
  },
  "sig": "hmac-signature"
}
```

---

## 🔐 SECURITY & CACHING MODEL

### Three Levels of Caching

```
LAYER 1: SYNFLOX Server Cache (Redis)
├── Cache entitlement matrix per subscription
├── TTL: Until version changes
├── Invalidated on: grant, revoke, upgrade, mode change
└── Security: ✅ Fully controlled

LAYER 2: Client's Backend Cache (Redis/Memory)
├── Cache response from SYNFLOX
├── TTL: 24 hours or until X-Entitlements-Version changes
├── Used for: Validating every user request
└── Security: ✅ User cannot access

LAYER 3: Client's Frontend Cache (localStorage/Memory)
├── Cache for UI display only
├── TTL: Session or until refresh
├── Used for: Showing menus, reducing API calls
├── Security: ⚠️ User CAN tamper (but doesn't matter - display only)
└── Encrypted: NO NEED (public info to user anyway)
```

### Why Frontend Tampering Doesn't Matter

```
User tampers with localStorage → Sees extra menus
        ↓
User clicks locked feature
        ↓
Request goes to Client's Backend
        ↓
Backend checks its SECURE cache
        ↓
Cache says: "Feature = LOCKED"
        ↓
Backend returns: 403 Forbidden
        ↓
Security Impact: ZERO ✅
```

### Key Security Principles

1. **Frontend is NEVER trusted** - It's just for display
2. **Security is ALWAYS server-side** - Both SYNFLOX and Client's Backend
3. **Entitlements are for authorization** - Token is for authentication
4. **Version header signals changes** - Client knows when to refresh
5. **Offline systems use signed licenses** - Can't tamper with encrypted data

---

## 📝 NOTES & REMINDERS

- **Server Time**: Background job uses server local time for daily operations
- **Version Increment**: Any entitlement change increments subscription's EntitlementsVersion
- **Cache Duration**: Client should cache entitlements for 24 hours
- **Force Refresh**: X-Entitlements-Version header signals version change
- **Offline Sync**: Offline clients should sync when online to get latest license
- **Marketing Display**: Use `available_upgrades` and `menu_config` to control locked item visibility

---

## 🔄 CHANGE LOG

| Date | Phase | Step | Change |
|------|-------|------|--------|
| - | - | - | - |

---

**Ready to start Phase 1!**