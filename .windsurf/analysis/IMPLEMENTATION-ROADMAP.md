# 🚀 SYNFLOX Enterprise Access Control - IMPLEMENTATION ROADMAP

**Created:** November 30, 2025  
**Status:** APPROVED - READY TO IMPLEMENT  
**Total Estimated Time:** 15-17 days

---

## ✅ CONFIRMED DECISIONS

| Decision | Confirmed |
|----------|-----------|
| Free Tier Access | ReadOnly (view + export only) |
| Export Grace Days | 30 days after blocked |
| Upgrade Strategy | Additive (keep custom grants) |
| Fallback Plan | Optional per plan |

---

## 📋 PHASE BREAKDOWN

### PHASE 1: Domain Layer (Day 1-2)
**Goal:** Create all new entities and enums

#### Day 1: Enums & Core Entity
- [ ] Create `Domain/Enums/EntitlementGrantType.cs`
- [ ] Create `Domain/Enums/EntitlementSource.cs`
- [ ] Create `Domain/Enums/EntitlementAccessLevel.cs`
- [ ] Create `Domain/Enums/SubscriptionAccessMode.cs`
- [ ] Create `Domain/Entities/Subscriptions/SubscriptionEntitlement.cs`

#### Day 2: Update Existing Entities
- [ ] Update `Domain/Entities/Subscriptions/Subscription.cs`
  - Add `AccessMode` property
  - Add `FallbackPlanId` property
  - Add `ExportDeadlineUtc` property
  - Add `AccessRestrictionMessage` property
  - Add computed properties (`EffectiveAccessMode`, `IsInGracePeriod`, etc.)
  - Add `Entitlements` navigation collection
- [ ] Update `Domain/Entities/Subscriptions/SubscriptionPlan.cs`
  - Add `IsFreeTier` property
  - Add `FallbackAccessMode` property
  - Add `ExportGraceDays` property
  - Add `DefaultFallbackPlanId` property
- [ ] Create `Domain/Interfaces/ISubscriptionEntitlementRepository.cs`

---

### PHASE 2: Infrastructure - Data Layer (Day 3-4)
**Goal:** EF Configuration, Repository, Migration

#### Day 3: EF Configuration
- [ ] Create `Infrastructure/Configurations/SubscriptionEntitlementConfiguration.cs`
- [ ] Update `Infrastructure/Configurations/SubscriptionConfiguration.cs`
- [ ] Update `Infrastructure/Configurations/SubscriptionPlanConfiguration.cs`
- [ ] Update `Infrastructure/Context/ApplicationDBContext.cs` - Add DbSet

#### Day 4: Repository & Migration
- [ ] Create `Infrastructure/Repositories/SubscriptionEntitlementRepository.cs`
- [ ] Create database migration
- [ ] Create migration script for existing subscriptions (copy plan entitlements)
- [ ] Add proper indexes for performance

---

### PHASE 3: Application Layer - DTOs (Day 5)
**Goal:** Create all DTOs for the new system

- [ ] Create `Application/DTOs/Entitlements/SubscriptionEntitlementDto.cs`
- [ ] Create `Application/DTOs/Entitlements/GrantEntitlementRequest.cs`
- [ ] Create `Application/DTOs/Entitlements/UpdateEntitlementRequest.cs`
- [ ] Create `Application/DTOs/Entitlements/RevokeEntitlementRequest.cs`
- [ ] Create `Application/DTOs/Entitlements/AccessCheckRequest.cs`
- [ ] Create `Application/DTOs/Entitlements/AccessCheckResult.cs`
- [ ] Create `Application/DTOs/Entitlements/EntitlementMatrix.cs`
- [ ] Create `Application/DTOs/Entitlements/ProjectEntitlement.cs`
- [ ] Create `Application/DTOs/Entitlements/ModuleEntitlement.cs`
- [ ] Update `Application/DTOs/Subscriptions/SubscriptionDto.cs` - Add access mode fields
- [ ] Update `Application/DTOs/PlanDto/` - Add free tier and fallback fields
- [ ] Create `Application/Services/IEntitlementService.cs`

---

### PHASE 4: Infrastructure - Services (Day 6-8)
**Goal:** Implement the core entitlement service

#### Day 6: EntitlementService Core
- [ ] Create `Infrastructure/Services/EntitlementService.cs`
  - `GetSubscriptionEntitlementsAsync`
  - `GrantEntitlementAsync`
  - `RevokeEntitlementAsync`
  - `UpdateEntitlementAsync`

#### Day 7: EntitlementService Bulk Operations
- [ ] Add to `EntitlementService.cs`
  - `CopyPlanEntitlementsToSubscriptionAsync`
  - `AddUpgradeEntitlementsAsync`
  - `ReplaceEntitlementsAsync`
  - `DowngradeToFallbackAsync`

#### Day 8: EntitlementService Access Checking
- [ ] Add to `EntitlementService.cs`
  - `CheckProjectAccessAsync`
  - `CheckModuleAccessAsync`
  - `CheckFeatureAccessAsync`
  - `CheckOperationAsync`
  - `BuildEntitlementMatrixAsync`
- [ ] Register service in DI container

---

### PHASE 5: Update Existing Services (Day 9-10)
**Goal:** Integrate entitlements into existing flows

#### Day 9: SubscriptionService Updates
- [ ] Update `SubscriptionService.CreateSubscriptionAsync`
  - Call `CopyPlanEntitlementsToSubscriptionAsync` after creation
  - Set initial `AccessMode = Full`
- [ ] Update `SubscriptionService.UpgradeSubscriptionAsync`
  - Call `AddUpgradeEntitlementsAsync` for additive upgrades
- [ ] Update `SubscriptionService.CancelSubscriptionAsync`
  - Trigger fallback/export-only logic
- [ ] Update other lifecycle methods to handle access modes

#### Day 10: Background Job & Access Mode Transitions
- [ ] Create `Infrastructure/BackgroundJobs/AccessModeTransitionJob.cs`
  - Run hourly
  - Check expiry → transition to GracePeriod
  - Check grace end → transition to ExportOnly or ReadOnly
  - Check export deadline → transition to Blocked
- [ ] Update `SubscriptionService` to set `ExportDeadlineUtc` when needed
- [ ] Register background job

---

### PHASE 6: Token & License Key Redesign (Day 11-12)
**Goal:** Include entitlement matrix in tokens and license keys

#### Day 11: Token Redesign
- [ ] Update `Infrastructure/Services/ClientJwtService.cs`
  - Add `access_mode` claim
  - Add `allowed_operations` claim
  - Add `entitlements` claim with full matrix
  - Add `fallback_plan` claim
  - Add `days_remaining` claim
  - Add `blocked_features` claim
  - Add `upgrade_message` claim
- [ ] Update `Infrastructure/Services/ClientTokenService.cs`
  - Regenerate token when entitlements change
  - Include access mode in response

#### Day 12: License Key Redesign
- [ ] Update `Infrastructure/Services/LicenseService.cs`
  - Update `OfflineLicenseData` structure
  - Include `AccessMode`
  - Include `EntitlementMatrix`
  - Include `FallbackPlan` info
  - Include `ExportDeadlineUtc`
  - Bump version to 2
- [ ] Update license validation to check entitlements
- [ ] Add backward compatibility for v1 licenses

---

### PHASE 7: API Layer (Day 13)
**Goal:** Create new endpoints and update existing ones

- [ ] Create `WebAPI/Controllers/EntitlementsController.cs` (Admin management)
  - `GET /api/subscriptions/{id}/entitlements`
  - `POST /api/subscriptions/{id}/entitlements` (Grant)
  - `PUT /api/entitlements/{id}` (Update)
  - `DELETE /api/entitlements/{id}` (Revoke)
- [ ] Update `WebAPI/Controllers/ClientApiController.cs`
  - Add `POST /api/client/access/check`
  - Add `GET /api/client/entitlements`
  - Update `GET /api/client/subscription/status` to include access mode
  - Update `GET /api/client/plan/features` to include entitlements
- [ ] Update `WebAPI/Controllers/SubscriptionsController.cs`
  - Include entitlements in responses
- [ ] Update `WebAPI/Controllers/PlansController.cs`
  - Include free tier and fallback options

---

### PHASE 8: AutoMapper Profiles (Day 13 continued)
**Goal:** Add mapping configurations

- [ ] Create `Application/Mapping/EntitlementMappingProfile.cs`
- [ ] Update `Application/Mapping/SubscriptionMappingProfile.cs`
- [ ] Update `Application/Mapping/PlanMappingProfile.cs`

---

### PHASE 9: Frontend - Domain Layer (Day 14)
**Goal:** Create frontend models and mappers

- [ ] Create `domain/models/entitlement.model.ts`
  - `SubscriptionEntitlement` class
  - `GrantEntitlementRequest` class
  - `AccessCheckResult` class
  - `EntitlementMatrix` class
  - Enums: `EntitlementGrantType`, `EntitlementSource`, `EntitlementAccessLevel`, `SubscriptionAccessMode`
- [ ] Create `domain/mappers/entitlement.mapper.ts`
- [ ] Update `domain/models/subscription.model.ts` - Add access mode fields
- [ ] Update `domain/models/subscription-plan.model.ts` - Add free tier fields
- [ ] Update `domain/index.ts` - Export new models

---

### PHASE 10: Frontend - Service Layer (Day 14 continued)
**Goal:** Create entitlement service

- [ ] Create `services/entitlement.service.ts`
  - `getSubscriptionEntitlements`
  - `grantEntitlement`
  - `updateEntitlement`
  - `revokeEntitlement`
  - `checkAccess`
- [ ] Update `config/api-endpoints.ts` - Add entitlement endpoints
- [ ] Register service in `providers/service-provider.tsx`

---

### PHASE 11: Frontend - UI Components (Day 15-16)
**Goal:** Create UI for managing entitlements

#### Day 15: Subscription Entitlements Tab
- [ ] Create `components/entitlements/entitlement-list.tsx`
- [ ] Create `components/entitlements/grant-entitlement-dialog.tsx`
- [ ] Create `components/entitlements/revoke-entitlement-dialog.tsx`
- [ ] Create `components/entitlements/access-mode-badge.tsx`
- [ ] Update subscription details page - Add Entitlements tab

#### Day 16: Plan Free Tier Configuration
- [ ] Update plan create/edit form - Add free tier options
- [ ] Add fallback plan selector
- [ ] Add export grace days input
- [ ] Create `components/plans/fallback-config.tsx`

---

### PHASE 12: Frontend - Access Mode UI (Day 16 continued)
**Goal:** Show access restrictions to users

- [ ] Create `components/access/access-mode-banner.tsx` (for client apps)
- [ ] Create `components/access/upgrade-prompt.tsx`
- [ ] Create `components/access/export-countdown.tsx`
- [ ] Update subscription status displays

---

### PHASE 13: Translations (Day 17)
**Goal:** Add all i18n keys

- [ ] Update `locales/en.ts` - Add entitlement keys (50+ keys)
- [ ] Update `locales/ar.ts` - Add Arabic translations

---

### PHASE 14: Testing & Documentation (Day 17)
**Goal:** Ensure everything works

- [ ] Test all scenarios:
  - [ ] Create subscription → entitlements copied from plan
  - [ ] Grant custom entitlement
  - [ ] Revoke entitlement
  - [ ] Upgrade subscription → additive entitlements
  - [ ] Subscription expires → grace period
  - [ ] Grace ends → export only or free tier
  - [ ] Export deadline → blocked
  - [ ] Token contains correct entitlements
  - [ ] License key contains correct entitlements
  - [ ] Access check endpoint works
- [ ] Update API documentation
- [ ] Create client SDK documentation

---

## 📁 FILES TO CREATE (Summary)

### Domain Layer
```
Domain/
├── Enums/
│   ├── EntitlementGrantType.cs (NEW)
│   ├── EntitlementSource.cs (NEW)
│   ├── EntitlementAccessLevel.cs (NEW)
│   └── SubscriptionAccessMode.cs (NEW)
├── Entities/Subscriptions/
│   ├── SubscriptionEntitlement.cs (NEW)
│   ├── Subscription.cs (UPDATE)
│   └── SubscriptionPlan.cs (UPDATE)
└── Interfaces/
    └── ISubscriptionEntitlementRepository.cs (NEW)
```

### Application Layer
```
Application/
├── DTOs/Entitlements/
│   ├── SubscriptionEntitlementDto.cs (NEW)
│   ├── GrantEntitlementRequest.cs (NEW)
│   ├── UpdateEntitlementRequest.cs (NEW)
│   ├── RevokeEntitlementRequest.cs (NEW)
│   ├── AccessCheckRequest.cs (NEW)
│   ├── AccessCheckResult.cs (NEW)
│   ├── EntitlementMatrix.cs (NEW)
│   ├── ProjectEntitlement.cs (NEW)
│   └── ModuleEntitlement.cs (NEW)
├── Services/
│   └── IEntitlementService.cs (NEW)
└── Mapping/
    └── EntitlementMappingProfile.cs (NEW)
```

### Infrastructure Layer
```
Infrastructure/
├── Configurations/
│   └── SubscriptionEntitlementConfiguration.cs (NEW)
├── Repositories/
│   └── SubscriptionEntitlementRepository.cs (NEW)
├── Services/
│   ├── EntitlementService.cs (NEW)
│   ├── ClientJwtService.cs (UPDATE)
│   ├── ClientTokenService.cs (UPDATE)
│   ├── LicenseService.cs (UPDATE)
│   └── SubscriptionService.cs (UPDATE)
└── BackgroundJobs/
    └── AccessModeTransitionJob.cs (NEW)
```

### WebAPI Layer
```
WebAPI/
└── Controllers/
    ├── EntitlementsController.cs (NEW)
    ├── ClientApiController.cs (UPDATE)
    ├── SubscriptionsController.cs (UPDATE)
    └── PlansController.cs (UPDATE)
```

### Frontend
```
synflox-frontend/
├── domain/
│   ├── models/
│   │   ├── entitlement.model.ts (NEW)
│   │   ├── subscription.model.ts (UPDATE)
│   │   └── subscription-plan.model.ts (UPDATE)
│   └── mappers/
│       └── entitlement.mapper.ts (NEW)
├── services/
│   └── entitlement.service.ts (NEW)
├── components/
│   ├── entitlements/
│   │   ├── entitlement-list.tsx (NEW)
│   │   ├── grant-entitlement-dialog.tsx (NEW)
│   │   ├── revoke-entitlement-dialog.tsx (NEW)
│   │   └── access-mode-badge.tsx (NEW)
│   ├── access/
│   │   ├── access-mode-banner.tsx (NEW)
│   │   ├── upgrade-prompt.tsx (NEW)
│   │   └── export-countdown.tsx (NEW)
│   └── plans/
│       └── fallback-config.tsx (NEW)
└── locales/
    ├── en.ts (UPDATE)
    └── ar.ts (UPDATE)
```

---

## 🔄 DATABASE CHANGES

### New Table: SubscriptionEntitlements
```sql
CREATE TABLE SubscriptionEntitlements (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWSEQUENTIALID(),
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
    -- Audit
    CreatedTimestamp DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    UpdatedTimestamp DATETIME2 NULL,
    CreatedBy NVARCHAR(100) NULL,
    UpdatedBy NVARCHAR(100) NULL,
    IsDeleted BIT NOT NULL DEFAULT 0,
    -- Foreign Keys
    CONSTRAINT FK_SubscriptionEntitlements_Subscription 
        FOREIGN KEY (SubscriptionId) REFERENCES Subscriptions(Id),
    CONSTRAINT FK_SubscriptionEntitlements_Project 
        FOREIGN KEY (ProjectId) REFERENCES Projects(Id),
    CONSTRAINT FK_SubscriptionEntitlements_Module 
        FOREIGN KEY (ModuleId) REFERENCES Modules(Id),
    CONSTRAINT FK_SubscriptionEntitlements_Admin 
        FOREIGN KEY (GrantedByAdminId) REFERENCES Admins(Id)
);

-- Indexes
CREATE INDEX IX_SubEntitlements_SubscriptionId 
    ON SubscriptionEntitlements(SubscriptionId) WHERE IsDeleted = 0;
CREATE INDEX IX_SubEntitlements_ProjectId 
    ON SubscriptionEntitlements(ProjectId) WHERE IsDeleted = 0;
CREATE INDEX IX_SubEntitlements_ModuleId 
    ON SubscriptionEntitlements(ModuleId) WHERE IsDeleted = 0;
CREATE INDEX IX_SubEntitlements_Active 
    ON SubscriptionEntitlements(SubscriptionId, IsActive) WHERE IsDeleted = 0;
```

### Alter Table: Subscriptions
```sql
ALTER TABLE Subscriptions ADD
    AccessMode INT NOT NULL DEFAULT 1,
    FallbackPlanId UNIQUEIDENTIFIER NULL,
    ExportDeadlineUtc DATETIME2 NULL,
    AccessRestrictionMessage NVARCHAR(500) NULL;

ALTER TABLE Subscriptions ADD CONSTRAINT 
    FK_Subscriptions_FallbackPlan FOREIGN KEY (FallbackPlanId) 
    REFERENCES SubscriptionPlans(Id);
```

### Alter Table: SubscriptionPlans
```sql
ALTER TABLE SubscriptionPlans ADD
    IsFreeTier BIT NOT NULL DEFAULT 0,
    FallbackAccessMode INT NOT NULL DEFAULT 2,
    ExportGraceDays INT NOT NULL DEFAULT 30,
    DefaultFallbackPlanId UNIQUEIDENTIFIER NULL;

ALTER TABLE SubscriptionPlans ADD CONSTRAINT 
    FK_Plans_DefaultFallback FOREIGN KEY (DefaultFallbackPlanId) 
    REFERENCES SubscriptionPlans(Id);
```

### Migration: Existing Subscriptions
```sql
-- Create entitlements from existing plan associations
INSERT INTO SubscriptionEntitlements (
    Id, SubscriptionId, ProjectId, ModuleId, GrantType, 
    AccessLevel, Source, IsCustom, IsActive, CreatedTimestamp
)
SELECT 
    NEWID(),
    s.Id,
    pp.ProjectId,
    NULL,
    1, -- FullProject
    1, -- Full
    1, -- PlanDefault
    0, -- Not custom
    1, -- Active
    GETUTCDATE()
FROM Subscriptions s
INNER JOIN SubscriptionPlans sp ON s.PlanId = sp.Id
INNER JOIN PlanProjects pp ON sp.Id = pp.PlanId
WHERE s.IsDeleted = 0;

-- For modules not tied to projects through plans
INSERT INTO SubscriptionEntitlements (
    Id, SubscriptionId, ProjectId, ModuleId, GrantType, 
    AccessLevel, Source, IsCustom, IsActive, CreatedTimestamp
)
SELECT 
    NEWID(),
    s.Id,
    NULL,
    pm.ModuleId,
    3, -- StandaloneModule
    1, -- Full
    1, -- PlanDefault
    0, -- Not custom
    1, -- Active
    GETUTCDATE()
FROM Subscriptions s
INNER JOIN SubscriptionPlans sp ON s.PlanId = sp.Id
INNER JOIN PlanModules pm ON sp.Id = pm.PlanId
WHERE s.IsDeleted = 0
AND NOT EXISTS (
    SELECT 1 FROM PlanProjects pp2 
    INNER JOIN ProjectModules pmod ON pp2.ProjectId = pmod.ProjectId
    WHERE pp2.PlanId = sp.Id AND pmod.ModuleId = pm.ModuleId
);
```

---

## 🎯 SUCCESS CRITERIA

Each phase is complete when:
- [ ] All files created/updated
- [ ] No build errors
- [ ] Unit tests pass (if applicable)
- [ ] Manual testing confirms functionality
- [ ] Translations added

Final success:
- [ ] All scenarios in Phase 14 pass
- [ ] Token contains correct entitlement matrix
- [ ] License key contains correct entitlement matrix
- [ ] Access check endpoint returns correct results
- [ ] Access mode transitions work correctly
- [ ] Fallback plan works after expiry
- [ ] Export deadline enforcement works
- [ ] Admin UI can manage entitlements
- [ ] Frontend shows access restrictions

---

## 🚀 READY TO START!

**Starting with Phase 1: Domain Layer**

Will create:
1. `EntitlementGrantType.cs`
2. `EntitlementSource.cs`
3. `EntitlementAccessLevel.cs`
4. `SubscriptionAccessMode.cs`
5. `SubscriptionEntitlement.cs`

**Proceed?**
