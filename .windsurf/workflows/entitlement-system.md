---
description: Enterprise Entitlement System v2.0 - Plan-Level Access Control
auto_execution_mode: 3
---

# 🏢 SYNFLOX Enterprise Entitlement System v2.0

## 📋 Executive Summary

**Core Principle**: Access control is defined at the **PLAN level**, not subscription level.
All subscriptions of the same plan have IDENTICAL access rights.

### Key Changes from v1:
- ❌ REMOVE `SubscriptionEntitlement` entity (wrong design)
- ✅ ADD `PlanEntitlement` entity (correct design)
- ✅ All access defined on Plan, inherited by all subscribers
- ✅ Plan changes can be scheduled for next billing cycle
- ✅ Email notifications for plan changes

---

## 📊 Progress Tracker

**Current Phase:** Phase 2 - Create PlanEntitlement Entity
**Status:** Phase 1 Complete, Ready for Phase 2
**Last Updated:** December 1, 2025 - 9:07 PM

### Implementation Phases
- [x] **Phase 1:** Remove Old Entitlement System ✅ COMPLETE
- [ ] **Phase 2:** Create PlanEntitlement Entity ⬅️ NEXT
- [ ] **Phase 3:** Plan Change Scheduling System
- [ ] **Phase 4:** Email Notification System
- [ ] **Phase 5:** Client API Updates
- [ ] **Phase 6:** Frontend - Plan Entitlements Management
- [ ] **Phase 7:** Frontend - Subscription Access View (Read-Only)
- [ ] **Phase 8:** Testing & Migration

### Phase 1 Completed Items
- ✅ Removed `SubscriptionEntitlement` entity
- ✅ Removed `ISubscriptionEntitlementRepository` interface
- ✅ Removed `SubscriptionEntitlementRepository` implementation
- ✅ Removed `SubscriptionEntitlementConfiguration` EF config
- ✅ Removed `IEntitlementService` interface
- ✅ Removed `EntitlementService` implementation
- ✅ Removed `EntitlementsController`
- ✅ Removed all Entitlement DTOs (17 files)
- ✅ Removed `EntitlementMappingProfile`
- ✅ Updated `Subscription` entity (removed Entitlements nav property)
- ✅ Updated `SubscriptionPlan` entity (added EntitlementVersion)
- ✅ Updated `LicenseService` to use plan-based entitlements
- ✅ Updated `ClientApiService` to use plan-based entitlements
- ✅ Updated `AccessModeTransitionJob` to use plan-based versioning
- ✅ Updated `EntitlementsVersionHeaderMiddleware`
- ✅ Updated `ClientApiController`
- ✅ Removed frontend entitlement files (model, mapper, service, viewmodel, view)
- ✅ Removed frontend entitlements page
- ✅ Cleaned up service provider and exports
- ✅ Backend builds successfully (0 errors)
- ✅ Frontend builds successfully

---

## 🏗️ ARCHITECTURE v2.0

### The Correct Flow
```
┌─────────────────────────────────────────────────────────────────┐
│                      PLAN DEFINITION                             │
│  "Pro Plan" includes:                                            │
│    ├── ERP Project → Full Access (CRUD + Export)                │
│    ├── HR Module → Read Only (Read + Export)                    │
│    └── Reports → Full Access                                     │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      SUBSCRIPTIONS                               │
│  Company A ──► Pro Plan ──┐                                     │
│  Company B ──► Pro Plan ──┼──► ALL get SAME access!            │
│  Company C ──► Pro Plan ──┘                                     │
│  Company D ──► Basic Plan ──► Gets Basic Plan access            │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      CLIENT API                                  │
│  GET /api/client/entitlements                                   │
│  Returns: Plan's entitlements (not subscription-specific!)      │
└─────────────────────────────────────────────────────────────────┘
```

### Why This is Correct
| Old Design (WRONG) | New Design (CORRECT) |
|-------------------|----------------------|
| Each subscription has own entitlements | All same-plan subscriptions = same access |
| Admin grants per subscription | Admin defines per plan |
| Security risk (can grant anything) | Controlled by plan definition |
| Complex upgrade/fallback | Simple: just change planId |
| Manual work for each subscriber | Define once, apply to all |

---

## 📦 PHASE 1: Remove Old Entitlement System

### 1.1 Entities to REMOVE/DEPRECATE
```
Domain/Entities/Entitlement/
├── SubscriptionEntitlement.cs  ──► DELETE
└── EntitlementMatrix.cs        ──► DELETE (will be calculated from plan)
```

### 1.2 Services to REMOVE/REFACTOR
```
Application/Services/
├── IEntitlementService.cs      ──► MAJOR REFACTOR (plan-based)

Infrastructure/Services/
├── EntitlementService.cs       ──► MAJOR REFACTOR
```

### 1.3 DTOs to REMOVE
```
Application/DTOs/Entitlements/
├── SubscriptionEntitlementDto.cs   ──► DELETE
├── CreateEntitlementRequest.cs     ──► DELETE
├── UpdateEntitlementRequest.cs     ──► DELETE
├── GrantEntitlementRequest.cs      ──► DELETE
├── RevokeEntitlementRequest.cs     ──► DELETE
└── ... (most subscription-specific DTOs)
```

### 1.4 Frontend to REMOVE
```
synflox-frontend/
├── views/entitlement-detail-view.tsx      ──► REPLACE with read-only view
├── viewmodels/entitlement-viewmodel.ts    ──► DELETE
├── domain/models/entitlement.model.ts     ──► REFACTOR for plan entitlements
└── services/entitlement.service.ts        ──► DELETE
```

---

## 📦 PHASE 2: Create PlanEntitlement Entity

### 2.1 New Entity: PlanEntitlement
```csharp
// Domain/Entities/Plans/PlanEntitlement.cs
public class PlanEntitlement : AuditEntity<Guid>
{
    // ═══════════════════════════════════════════
    // RELATIONSHIPS
    // ═══════════════════════════════════════════
    public Guid PlanId { get; set; }
    public virtual SubscriptionPlan Plan { get; set; } = null!;
    
    // Target: Project OR Module (one must be set)
    public Guid? ProjectId { get; set; }
    public virtual Project? Project { get; set; }
    
    public Guid? ModuleId { get; set; }
    public virtual Module? Module { get; set; }
    
    // ═══════════════════════════════════════════
    // ACCESS CONFIGURATION
    // ═══════════════════════════════════════════
    public EntitlementAccessLevel AccessLevel { get; set; } = EntitlementAccessLevel.Full;
    
    // CRUD Permissions
    public bool CanCreate { get; set; } = true;
    public bool CanRead { get; set; } = true;
    public bool CanUpdate { get; set; } = true;
    public bool CanDelete { get; set; } = true;
    public bool CanExport { get; set; } = true;
    
    // UI
    public bool DisplayInMenu { get; set; } = true;
    
    // ═══════════════════════════════════════════
    // COMPUTED
    // ═══════════════════════════════════════════
    public string TargetType => ProjectId.HasValue ? "Project" : "Module";
    public string TargetName => Project?.Name ?? Module?.Name ?? "Unknown";
}
```

### 2.2 Update SubscriptionPlan Entity
```csharp
// Add to SubscriptionPlan.cs
public virtual ICollection<PlanEntitlement> Entitlements { get; set; } = new List<PlanEntitlement>();

// Entitlement Version (increments when entitlements change)
public int EntitlementVersion { get; set; } = 1;
```

### 2.3 AccessLevel Enum (Keep existing)
```csharp
public enum EntitlementAccessLevel
{
    Full = 1,        // Full CRUD access
    ReadOnly = 2,    // Can only read/view
    ExportOnly = 3,  // Can only export data
    Blocked = 4      // No access (hidden)
}
```

### 2.4 EF Configuration
```csharp
// Infrastructure/Configuration/PlanEntitlementConfiguration.cs
public class PlanEntitlementConfiguration : IEntityTypeConfiguration<PlanEntitlement>
{
    public void Configure(EntityTypeBuilder<PlanEntitlement> builder)
    {
        builder.ToTable("PlanEntitlements");
        
        builder.HasKey(e => e.Id);
        
        // Plan relationship
        builder.HasOne(e => e.Plan)
            .WithMany(p => p.Entitlements)
            .HasForeignKey(e => e.PlanId)
            .OnDelete(DeleteBehavior.Cascade);
        
        // Project relationship (optional)
        builder.HasOne(e => e.Project)
            .WithMany()
            .HasForeignKey(e => e.ProjectId)
            .OnDelete(DeleteBehavior.SetNull);
        
        // Module relationship (optional)
        builder.HasOne(e => e.Module)
            .WithMany()
            .HasForeignKey(e => e.ModuleId)
            .OnDelete(DeleteBehavior.SetNull);
        
        // Indexes
        builder.HasIndex(e => e.PlanId);
        builder.HasIndex(e => new { e.PlanId, e.ProjectId, e.ModuleId }).IsUnique();
        
        // Check constraint: must have Project OR Module (not both, not neither)
        builder.ToTable(t => t.HasCheckConstraint(
            "CK_PlanEntitlement_Target",
            "(ProjectId IS NOT NULL AND ModuleId IS NULL) OR (ProjectId IS NULL AND ModuleId IS NOT NULL)"
        ));
    }
}
```

---

## 📦 PHASE 3: Plan Change Scheduling System

### 3.1 New Entity: PlanChangeSchedule
```csharp
// Domain/Entities/Plans/PlanChangeSchedule.cs
public class PlanChangeSchedule : AuditEntity<Guid>
{
    public Guid PlanId { get; set; }
    public virtual SubscriptionPlan Plan { get; set; } = null!;
    
    // What changed
    public PlanChangeType ChangeType { get; set; }
    public string ChangeDescription { get; set; } = string.Empty;
    public string ChangedFieldsJson { get; set; } = "{}"; // JSON of changed fields
    
    // When to apply
    public bool ApplyImmediately { get; set; } = false;
    public DateTime? ScheduledForUtc { get; set; } // If not immediate
    
    // Status
    public PlanChangeStatus Status { get; set; } = PlanChangeStatus.Pending;
    public DateTime? AppliedAtUtc { get; set; }
    
    // Notification
    public bool NotifySubscribers { get; set; } = true;
    public bool NotificationSent { get; set; } = false;
    public DateTime? NotificationSentAtUtc { get; set; }
}

public enum PlanChangeType
{
    PriceChange = 1,
    EntitlementAdded = 2,
    EntitlementRemoved = 3,
    EntitlementModified = 4,
    FeaturesChanged = 5,
    DurationChanged = 6
}

public enum PlanChangeStatus
{
    Pending = 1,
    Applied = 2,
    Cancelled = 3
}
```

### 3.2 Plan Update Service
```csharp
public interface IPlanUpdateService
{
    // Update plan with scheduling option
    Task<PlanChangeSchedule> UpdatePlanAsync(
        Guid planId, 
        UpdatePlanRequest request,
        bool applyImmediately = false,
        bool notifySubscribers = true);
    
    // Get pending changes for a plan
    Task<List<PlanChangeSchedule>> GetPendingChangesAsync(Guid planId);
    
    // Cancel a pending change
    Task CancelPendingChangeAsync(Guid changeId);
    
    // Apply pending changes (called by background job)
    Task ApplyPendingChangesAsync();
}
```

### 3.3 Background Job: ApplyPlanChangesJob
```csharp
// Runs hourly
public class ApplyPlanChangesJob : IJob
{
    public async Task Execute()
    {
        // Find all pending changes where ScheduledForUtc <= now
        var pendingChanges = await GetPendingChanges();
        
        foreach (var change in pendingChanges)
        {
            // Apply the change
            await ApplyChange(change);
            
            // Send notification emails if requested
            if (change.NotifySubscribers && !change.NotificationSent)
            {
                await SendNotifications(change);
            }
        }
    }
}
```

---

## 📦 PHASE 4: Email Notification System

### 4.1 Notification Events
```csharp
public interface IPlanNotificationService
{
    // Notify all subscribers of a plan about changes
    Task NotifyPlanChangeAsync(Guid planId, PlanChangeNotification notification);
    
    // Notify specific subscription about access change
    Task NotifyAccessChangeAsync(Guid subscriptionId, AccessChangeNotification notification);
}

public class PlanChangeNotification
{
    public string ChangeType { get; set; } // "Price", "Features", "Access"
    public string ChangeDescription { get; set; }
    public DateTime EffectiveDate { get; set; }
    public Dictionary<string, object> OldValues { get; set; }
    public Dictionary<string, object> NewValues { get; set; }
}
```

### 4.2 Email Templates
```
Templates/Emails/
├── PlanPriceChange.html        ──► "Your subscription price will change"
├── PlanFeaturesAdded.html      ──► "New features added to your plan!"
├── PlanFeaturesRemoved.html    ──► "Some features will be removed"
├── PlanAccessChange.html       ──► "Your access level has changed"
└── PlanUpgradeAvailable.html   ──► "Upgrade to get more features"
```

---

## 📦 PHASE 5: Client API Updates

### 5.1 Updated Entitlements Endpoint
```csharp
// GET /api/client/entitlements
// Returns the PLAN's entitlements for this subscription
public async Task<ActionResult<ClientEntitlementMatrixDto>> GetEntitlements()
{
    var subscription = await GetCurrentSubscription();
    var plan = await _planService.GetByIdAsync(subscription.PlanId);
    
    return new ClientEntitlementMatrixDto
    {
        Version = plan.EntitlementVersion,
        AccessMode = subscription.AccessMode,
        DaysRemaining = subscription.DaysRemaining,
        IsInGracePeriod = subscription.IsInGracePeriod,
        
        // Entitlements come from PLAN, not subscription
        Projects = plan.Entitlements
            .Where(e => e.ProjectId.HasValue)
            .Select(MapToProjectAccess),
            
        Modules = plan.Entitlements
            .Where(e => e.ModuleId.HasValue)
            .Select(MapToModuleAccess)
    };
}
```

### 5.2 Version Checking
```csharp
// Client sends: X-Entitlements-Version: 5
// Server checks: Plan.EntitlementVersion = 7
// Response: Full entitlement matrix (version changed)

// Client sends: X-Entitlements-Version: 7
// Server checks: Plan.EntitlementVersion = 7
// Response: 304 Not Modified (use cached version)
```

### 5.3 Status Endpoint (Already exists - minor update)
```csharp
// GET /api/client/status
// Returns subscription status WITH plan access info
public async Task<ActionResult<ClientStatusDto>> GetStatus()
{
    var subscription = await GetCurrentSubscription();
    var plan = await _planService.GetByIdAsync(subscription.PlanId);
    
    return new ClientStatusDto
    {
        IsValid = subscription.IsActive && !subscription.IsExpired,
        SubscriptionStatus = subscription.Status,
        PlanName = plan.Name,
        ExpiresAt = subscription.ExpiryDateUtc,
        DaysRemaining = subscription.DaysRemaining,
        AccessMode = subscription.AccessMode,
        
        // Summary of access
        TotalProjects = plan.Entitlements.Count(e => e.ProjectId.HasValue),
        TotalModules = plan.Entitlements.Count(e => e.ModuleId.HasValue),
        
        // Version for cache checking
        EntitlementVersion = plan.EntitlementVersion
    };
}
```

---

## 📦 PHASE 6: Frontend - Plan Entitlements Management

### 6.1 Plan Detail Page - New "Entitlements" Tab
```
/plans/[id]
├── Overview Tab (existing)
├── Pricing Tab (existing)  
├── Entitlements Tab (NEW) ◄── Define what this plan can access
└── Subscribers Tab (existing)
```

### 6.2 Entitlements Tab UI
```tsx
// Plan Entitlements Management
<Card>
  <CardHeader>
    <CardTitle>Plan Entitlements</CardTitle>
    <CardDescription>
      Define which projects and modules subscribers of this plan can access
    </CardDescription>
    <Button onClick={addEntitlement}>+ Add Entitlement</Button>
  </CardHeader>
  
  <CardContent>
    {/* List of entitlements */}
    {plan.entitlements.map(ent => (
      <EntitlementRow 
        key={ent.id}
        target={ent.projectName || ent.moduleName}
        type={ent.projectId ? "Project" : "Module"}
        accessLevel={ent.accessLevel}
        permissions={ent.permissions}
        onEdit={() => editEntitlement(ent)}
        onDelete={() => removeEntitlement(ent.id)}
      />
    ))}
  </CardContent>
</Card>

{/* Add/Edit Dialog */}
<Dialog>
  <Select label="Type" options={["Project", "Module"]} />
  <Select label="Target" options={projectsOrModules} />
  <Select label="Access Level" options={["Full", "ReadOnly", "ExportOnly"]} />
  
  <Checkboxes>
    <Checkbox label="Can Create" />
    <Checkbox label="Can Read" />
    <Checkbox label="Can Update" />
    <Checkbox label="Can Delete" />
    <Checkbox label="Can Export" />
    <Checkbox label="Display in Menu" />
  </Checkboxes>
  
  {/* Apply Options */}
  <RadioGroup label="When to apply?">
    <Radio value="immediate" label="Apply immediately to all subscribers" />
    <Radio value="nextCycle" label="Apply on next billing cycle" />
  </RadioGroup>
  
  <Checkbox label="Send email notification to all subscribers" defaultChecked />
</Dialog>
```

---

## 📦 PHASE 7: Frontend - Subscription Access View

### 7.1 Subscription Detail - Simplified View
```
/subscriptions/[id]
├── Overview Tab
├── Access Tab (NEW - READ ONLY) ◄── Shows what plan allows
├── History Tab
└── Analytics Tab
```

### 7.2 Access Tab UI (Read-Only)
```tsx
// Shows plan's entitlements - CANNOT be edited here
<Card>
  <CardHeader>
    <CardTitle>Subscription Access</CardTitle>
    <CardDescription>
      Access is defined by the plan: {subscription.planName}
      <Link href={`/plans/${subscription.planId}`}>Edit Plan Entitlements</Link>
    </CardDescription>
  </CardHeader>
  
  <CardContent>
    {/* Current Access Mode */}
    <AccessModeCard mode={subscription.accessMode} />
    
    {/* Read-only list of what they can access */}
    <h3>Projects</h3>
    {plan.entitlements.filter(e => e.projectId).map(ent => (
      <AccessRow 
        key={ent.id}
        name={ent.projectName}
        accessLevel={ent.accessLevel}
        permissions={ent.permissions}
        readOnly={true}  // Cannot edit!
      />
    ))}
    
    <h3>Modules</h3>
    {plan.entitlements.filter(e => e.moduleId).map(ent => (
      <AccessRow 
        key={ent.id}
        name={ent.moduleName}
        accessLevel={ent.accessLevel}
        permissions={ent.permissions}
        readOnly={true}  // Cannot edit!
      />
    ))}
  </CardContent>
</Card>
```

---

## 📦 PHASE 8: Testing & Migration

### 8.1 Database Migration
```sql
-- 1. Create PlanEntitlements table
CREATE TABLE PlanEntitlements (
    Id UNIQUEIDENTIFIER PRIMARY KEY,
    PlanId UNIQUEIDENTIFIER NOT NULL REFERENCES SubscriptionPlans(Id),
    ProjectId UNIQUEIDENTIFIER NULL REFERENCES Projects(Id),
    ModuleId UNIQUEIDENTIFIER NULL REFERENCES Modules(Id),
    AccessLevel INT NOT NULL DEFAULT 1,
    CanCreate BIT NOT NULL DEFAULT 1,
    CanRead BIT NOT NULL DEFAULT 1,
    CanUpdate BIT NOT NULL DEFAULT 1,
    CanDelete BIT NOT NULL DEFAULT 1,
    CanExport BIT NOT NULL DEFAULT 1,
    DisplayInMenu BIT NOT NULL DEFAULT 1,
    -- Audit fields
    CreatedAt DATETIME2 NOT NULL,
    CreatedBy NVARCHAR(100),
    ModifiedAt DATETIME2,
    ModifiedBy NVARCHAR(100),
    IsDeleted BIT NOT NULL DEFAULT 0,
    CONSTRAINT CK_PlanEntitlement_Target CHECK (
        (ProjectId IS NOT NULL AND ModuleId IS NULL) OR 
        (ProjectId IS NULL AND ModuleId IS NOT NULL)
    )
);

-- 2. Add EntitlementVersion to SubscriptionPlans
ALTER TABLE SubscriptionPlans ADD EntitlementVersion INT NOT NULL DEFAULT 1;

-- 3. Create PlanChangeSchedules table
CREATE TABLE PlanChangeSchedules (
    Id UNIQUEIDENTIFIER PRIMARY KEY,
    PlanId UNIQUEIDENTIFIER NOT NULL REFERENCES SubscriptionPlans(Id),
    ChangeType INT NOT NULL,
    ChangeDescription NVARCHAR(500),
    ChangedFieldsJson NVARCHAR(MAX),
    ApplyImmediately BIT NOT NULL DEFAULT 0,
    ScheduledForUtc DATETIME2,
    Status INT NOT NULL DEFAULT 1,
    AppliedAtUtc DATETIME2,
    NotifySubscribers BIT NOT NULL DEFAULT 1,
    NotificationSent BIT NOT NULL DEFAULT 0,
    NotificationSentAtUtc DATETIME2,
    -- Audit fields
    CreatedAt DATETIME2 NOT NULL,
    CreatedBy NVARCHAR(100),
    ModifiedAt DATETIME2,
    ModifiedBy NVARCHAR(100),
    IsDeleted BIT NOT NULL DEFAULT 0
);

-- 4. DROP old SubscriptionEntitlements table (after backup)
-- BACKUP: SELECT * INTO SubscriptionEntitlements_Backup FROM SubscriptionEntitlements;
-- DROP TABLE SubscriptionEntitlements;
```

### 8.2 Data Migration
```csharp
// If there was any data in SubscriptionEntitlements, migrate to PlanEntitlements
// Group by plan and create unique entries
```

---

## 📊 SUMMARY: Old vs New

| Aspect | OLD (v1) | NEW (v2) |
|--------|----------|----------|
| **Entitlements defined on** | Subscription (each one unique) | Plan (same for all subscribers) |
| **Entity** | SubscriptionEntitlement | PlanEntitlement |
| **Manual granting** | Admin grants per subscription | Admin defines per plan |
| **Upgrade handling** | Copy entitlements manually | Automatic (new plan = new access) |
| **Fallback handling** | Complex recalculation | Automatic (fallback plan access) |
| **UI location** | /subscriptions/[id]/entitlements | /plans/[id] (Entitlements tab) |
| **Security** | Risk: can grant anything | Safe: plan defines limits |
| **Plan changes** | Not supported | Schedule for next cycle + notify |

---

## 🚀 IMPLEMENTATION ORDER

### Step 1: Backend - Remove Old System
1. Backup SubscriptionEntitlements data
2. Remove SubscriptionEntitlement entity
3. Remove related DTOs
4. Remove EntitlementService (old version)

### Step 2: Backend - Create New System
1. Create PlanEntitlement entity
2. Create PlanChangeSchedule entity
3. Update SubscriptionPlan entity
4. Create EF configurations
5. Run migration

### Step 3: Backend - New Services
1. Create IPlanEntitlementService
2. Create IPlanChangeService
3. Create IPlanNotificationService
4. Update ClientApiController

### Step 4: Frontend - Remove Old
1. Delete entitlement-detail-view.tsx
2. Delete entitlement-viewmodel.ts
3. Delete entitlement.service.ts
4. Delete entitlement.model.ts

### Step 5: Frontend - Plan Entitlements
1. Create plan-entitlements-view.tsx
2. Add Entitlements tab to plan detail page
3. Create add/edit entitlement dialog
4. Add change scheduling options

### Step 6: Frontend - Subscription Access
1. Create subscription-access-view.tsx (read-only)
2. Add Access tab to subscription detail page
3. Show plan's entitlements
4. Link to plan for editing

### Step 7: Testing
1. Create plan with entitlements
2. Subscribe company to plan
3. Verify client API returns plan entitlements
4. Test upgrade/downgrade
5. Test plan change scheduling
6. Test email notifications

---

## ✅ ACCEPTANCE CRITERIA

1. **All subscriptions of same plan have identical access**
2. **No manual per-subscription granting (except promotional)**
3. **Plan entitlements defined in plan management UI**
4. **Client API returns plan's entitlements**
5. **Plan changes can be scheduled for next billing cycle**
6. **Subscribers notified of plan changes via email**
7. **Upgrade automatically grants new plan's access**
8. **Fallback automatically uses fallback plan's access**
9. **Version number increments on entitlement changes**
10. **Clients can check version to know when to refresh cache**

---

*Last Updated: December 1, 2025*
*Version: 2.0 - Complete Redesign*
