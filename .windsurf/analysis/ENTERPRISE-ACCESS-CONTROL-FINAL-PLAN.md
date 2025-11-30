# 🏢 SYNFLOX Enterprise Access Control - FINAL COMPREHENSIVE PLAN

**Date:** November 30, 2025  
**Version:** 2.0 (Complete Redesign)  
**Status:** READY FOR IMPLEMENTATION  
**Priority:** 🔴 CRITICAL - Core Business Architecture

---

## 📊 EXECUTIVE SUMMARY

This document contains the **complete enterprise-grade access control system** for SYNFLOX, addressing:

1. ✅ **Granular Entitlements** - Project/Module level access control
2. ✅ **Access Modes** - Full, ReadOnly, ExportOnly, Blocked
3. ✅ **Graceful Degradation** - Downgrade to free tier instead of blocking
4. ✅ **Online & Offline Support** - Both scenarios handled
5. ✅ **Upgrade/Downgrade Flows** - Proper entitlement management
6. ✅ **Token & License Key Redesign** - Self-contained access matrix

---

## 🎯 THE COMPLETE SOLUTION

### Before (Current - Broken)
```
Plan → Flat list of modules → Token has ["HR", "Finance"]
                                    ↓
                         Client must enforce (honor system)
                         No project context
                         No access modes
                         Expiry = Complete block
```

### After (Enterprise-Grade)
```
Subscription → SubscriptionEntitlements → Structured access matrix
                         ↓
              ┌─────────────────────────────────────────────┐
              │ Project: ERP                                 │
              │   ├── Module: HR (Full Access)              │
              │   └── Module: Accounting (ReadOnly)         │
              │ Project: Finance                             │
              │   └── Module: Payment (Full Access)         │
              │ Access Mode: Full / ReadOnly / ExportOnly   │
              │ Allowed Operations: [GET, POST, PUT, DELETE]│
              └─────────────────────────────────────────────┘
                         ↓
              Token/License contains ENFORCED access
              SYNFLOX validates, not just informs
```

---

## 🏗️ ARCHITECTURE OVERVIEW

### New Entity Relationship Diagram
```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         SYNFLOX ENTITY STRUCTURE                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌──────────────┐         ┌─────────────────────┐                           │
│  │   Company    │◄────────│   Subscription      │                           │
│  └──────────────┘    1:N  │  - PlanId (template)│                           │
│                           │  - AccessMode       │                           │
│                           │  - FallbackPlanId   │                           │
│                           └─────────┬───────────┘                           │
│                                     │ 1:N                                   │
│                                     ▼                                       │
│                     ┌───────────────────────────────────┐                   │
│                     │    SubscriptionEntitlement        │  ◄── NEW!         │
│                     │  - ProjectId (nullable)           │                   │
│                     │  - ModuleId (nullable)            │                   │
│                     │  - GrantType (Full/Specific/etc)  │                   │
│                     │  - AccessLevel (Full/Read/None)   │                   │
│                     │  - Operations (CRUD flags)        │                   │
│                     │  - Features (JSON)                │                   │
│                     │  - UsageLimits (JSON)             │                   │
│                     │  - Source (Plan/Admin/Upgrade)    │                   │
│                     │  - ExpiresAt (optional override)  │                   │
│                     └───────────────────────────────────┘                   │
│                              │              │                               │
│                              ▼              ▼                               │
│                     ┌────────────┐  ┌────────────┐                          │
│                     │  Project   │  │   Module   │                          │
│                     │  (ERP,CRM) │  │ (HR,Sales) │                          │
│                     └────────────┘  └────────────┘                          │
│                                                                              │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │                    SUBSCRIPTION PLAN (Template)                       │   │
│  │  - Default entitlements (copied to subscription on creation)         │   │
│  │  - IsFreeTier (for fallback)                                         │   │
│  │  - FallbackAccessMode (ReadOnly/ExportOnly)                          │   │
│  │  - GracePeriodDays                                                   │   │
│  │  - ExportGraceDays (after blocked)                                   │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 📐 DETAILED ENTITY DESIGNS

### 1. New Enums

```csharp
/// <summary>
/// How an entitlement was granted
/// </summary>
public enum EntitlementGrantType
{
    /// <summary>Access to entire project and ALL its modules (current + future)</summary>
    FullProject = 1,
    
    /// <summary>Access to specific modules within a project</summary>
    SpecificModules = 2,
    
    /// <summary>Access to a standalone module (not tied to project)</summary>
    StandaloneModule = 3,
    
    /// <summary>Access to specific features only</summary>
    FeatureOnly = 4
}

/// <summary>
/// Source of entitlement (for audit trail)
/// </summary>
public enum EntitlementSource
{
    /// <summary>Inherited from plan when subscription created</summary>
    PlanDefault = 1,
    
    /// <summary>Manually granted by admin</summary>
    AdminGrant = 2,
    
    /// <summary>Added during upgrade</summary>
    Upgrade = 3,
    
    /// <summary>Trial/promotional access (time-limited)</summary>
    Promotional = 4,
    
    /// <summary>Custom contract override</summary>
    ContractOverride = 5,
    
    /// <summary>Fallback/free tier (after expiry)</summary>
    Fallback = 6
}

/// <summary>
/// Access level for an entitlement
/// </summary>
public enum EntitlementAccessLevel
{
    /// <summary>Full access - all CRUD operations</summary>
    Full = 1,
    
    /// <summary>Read-only - view and export only</summary>
    ReadOnly = 2,
    
    /// <summary>Export-only - can only export, then blocked</summary>
    ExportOnly = 3,
    
    /// <summary>No access</summary>
    None = 4
}

/// <summary>
/// Current access mode for entire subscription
/// </summary>
public enum SubscriptionAccessMode
{
    /// <summary>Full access to all entitled features</summary>
    Full = 1,
    
    /// <summary>Grace period - full access but expiring soon</summary>
    GracePeriod = 2,
    
    /// <summary>Read-only - view and export only</summary>
    ReadOnly = 3,
    
    /// <summary>Export-only - data export before full block</summary>
    ExportOnly = 4,
    
    /// <summary>Completely blocked - upgrade required</summary>
    Blocked = 5
}
```

### 2. SubscriptionEntitlement Entity (NEW)

```csharp
/// <summary>
/// Granular entitlement record for a subscription
/// Defines exactly what a subscription can access at project/module/feature level
/// </summary>
public class SubscriptionEntitlement : AuditEntity<Guid>
{
    #region Core References
    
    /// <summary>The subscription this entitlement belongs to</summary>
    public Guid SubscriptionId { get; set; }
    
    /// <summary>Project being granted (null for standalone module or feature)</summary>
    public Guid? ProjectId { get; set; }
    
    /// <summary>Module being granted (null if granting full project)</summary>
    public Guid? ModuleId { get; set; }
    
    #endregion
    
    #region Grant Configuration
    
    /// <summary>How this entitlement was granted</summary>
    public EntitlementGrantType GrantType { get; set; }
    
    /// <summary>Access level for this entitlement</summary>
    public EntitlementAccessLevel AccessLevel { get; set; } = EntitlementAccessLevel.Full;
    
    /// <summary>Source of this entitlement (for audit)</summary>
    public EntitlementSource Source { get; set; }
    
    /// <summary>Whether this differs from plan default</summary>
    public bool IsCustom { get; set; }
    
    #endregion
    
    #region Operations & Features
    
    /// <summary>Can create new records</summary>
    public bool CanCreate { get; set; } = true;
    
    /// <summary>Can read/view records</summary>
    public bool CanRead { get; set; } = true;
    
    /// <summary>Can update/edit records</summary>
    public bool CanUpdate { get; set; } = true;
    
    /// <summary>Can delete records</summary>
    public bool CanDelete { get; set; } = true;
    
    /// <summary>Can export data</summary>
    public bool CanExport { get; set; } = true;
    
    /// <summary>
    /// Specific feature flags within the module (JSON array)
    /// null = all features, ["payroll", "leave"] = only these
    /// </summary>
    [StringLength(2000)]
    public string? Features { get; set; }
    
    /// <summary>
    /// Usage limits for metered features (JSON object)
    /// Example: {"apiCalls": 1000, "storage": "1GB", "users": 5}
    /// </summary>
    [StringLength(2000)]
    public string? UsageLimits { get; set; }
    
    #endregion
    
    #region Audit & Expiry
    
    /// <summary>Admin who granted this (if manual)</summary>
    public Guid? GrantedByAdminId { get; set; }
    
    /// <summary>Optional expiry (null = follows subscription)</summary>
    public DateTime? ExpiresAt { get; set; }
    
    /// <summary>Admin notes for audit</summary>
    [StringLength(500)]
    public string? Notes { get; set; }
    
    /// <summary>Is this entitlement currently active?</summary>
    public bool IsActive { get; set; } = true;
    
    #endregion
    
    #region Computed Properties
    
    /// <summary>Is this entitlement currently valid?</summary>
    public bool IsValid => IsActive && !IsDeleted && 
        (!ExpiresAt.HasValue || ExpiresAt.Value > DateTime.UtcNow);
    
    /// <summary>Allowed operations as list</summary>
    public List<string> AllowedOperations
    {
        get
        {
            var ops = new List<string>();
            if (CanRead) ops.Add("GET");
            if (CanCreate) ops.Add("POST");
            if (CanUpdate) ops.Add("PUT");
            if (CanDelete) ops.Add("DELETE");
            if (CanExport) ops.Add("EXPORT");
            return ops;
        }
    }
    
    #endregion
    
    #region Navigation
    
    public Subscription Subscription { get; set; } = null!;
    public Project? Project { get; set; }
    public Module? Module { get; set; }
    public Admin? GrantedByAdmin { get; set; }
    
    #endregion
}
```

### 3. Updated Subscription Entity

```csharp
public class Subscription : AuditEntity<Guid>
{
    // ... existing fields ...
    
    #region NEW: Access Control Fields
    
    /// <summary>
    /// Current access mode (Full, GracePeriod, ReadOnly, ExportOnly, Blocked)
    /// Updated by background job based on subscription state
    /// </summary>
    public SubscriptionAccessMode AccessMode { get; set; } = SubscriptionAccessMode.Full;
    
    /// <summary>
    /// Fallback plan for expired subscriptions (null = blocked after expiry)
    /// When set, subscription downgrades to this plan instead of blocking
    /// </summary>
    public Guid? FallbackPlanId { get; set; }
    
    /// <summary>
    /// When export-only mode ends (after which = Blocked)
    /// </summary>
    public DateTime? ExportDeadlineUtc { get; set; }
    
    /// <summary>
    /// Custom message to show when access is restricted
    /// </summary>
    [StringLength(500)]
    public string? AccessRestrictionMessage { get; set; }
    
    #endregion
    
    #region Computed Access Properties
    
    /// <summary>
    /// Effective access mode considering all factors
    /// </summary>
    public SubscriptionAccessMode EffectiveAccessMode
    {
        get
        {
            // Active and not expired = Full access
            if (IsActive && !IsExpired) 
                return SubscriptionAccessMode.Full;
            
            // Check grace period
            if (IsInGracePeriod) 
                return SubscriptionAccessMode.GracePeriod;
            
            // Check export deadline
            if (ExportDeadlineUtc.HasValue && DateTime.UtcNow <= ExportDeadlineUtc.Value)
                return SubscriptionAccessMode.ExportOnly;
            
            // Has fallback plan = use fallback access
            if (FallbackPlanId.HasValue)
                return SubscriptionAccessMode.ReadOnly;
            
            // No fallback = blocked
            return SubscriptionAccessMode.Blocked;
        }
    }
    
    /// <summary>
    /// Is subscription in grace period?
    /// </summary>
    public bool IsInGracePeriod
    {
        get
        {
            if (Plan == null) return false;
            var graceEnd = ExpiryDateUtc.AddDays(Plan.GracePeriodDays);
            return DateTime.UtcNow > ExpiryDateUtc && DateTime.UtcNow <= graceEnd;
        }
    }
    
    /// <summary>
    /// Days remaining in current access mode
    /// </summary>
    public int? DaysRemainingInCurrentMode
    {
        get
        {
            return EffectiveAccessMode switch
            {
                SubscriptionAccessMode.Full => IsLifetime ? null : (int)(ExpiryDateUtc - DateTime.UtcNow).TotalDays,
                SubscriptionAccessMode.GracePeriod => (int)(ExpiryDateUtc.AddDays(Plan?.GracePeriodDays ?? 0) - DateTime.UtcNow).TotalDays,
                SubscriptionAccessMode.ExportOnly => ExportDeadlineUtc.HasValue ? (int)(ExportDeadlineUtc.Value - DateTime.UtcNow).TotalDays : null,
                _ => null
            };
        }
    }
    
    #endregion
    
    #region Navigation
    
    // ... existing navigation ...
    public SubscriptionPlan? FallbackPlan { get; set; }
    public ICollection<SubscriptionEntitlement> Entitlements { get; set; } = new List<SubscriptionEntitlement>();
    
    #endregion
}
```

### 4. Updated SubscriptionPlan Entity

```csharp
public class SubscriptionPlan : AuditEntity<Guid>
{
    // ... existing fields ...
    
    #region NEW: Free Tier & Fallback Configuration
    
    /// <summary>
    /// Is this a free tier plan? (Can be used as fallback)
    /// </summary>
    public bool IsFreeTier { get; set; }
    
    /// <summary>
    /// Access mode when using this plan as fallback
    /// </summary>
    public SubscriptionAccessMode FallbackAccessMode { get; set; } = SubscriptionAccessMode.ReadOnly;
    
    /// <summary>
    /// Days to allow data export after expiry (before full block)
    /// Only applies when no fallback plan is set
    /// </summary>
    [Range(0, 90)]
    public int ExportGraceDays { get; set; } = 30;
    
    /// <summary>
    /// Default fallback plan for subscribers of this plan
    /// When their subscription expires, they get this plan
    /// </summary>
    public Guid? DefaultFallbackPlanId { get; set; }
    
    #endregion
    
    #region NEW: Entitlement Templates
    
    /// <summary>
    /// Default entitlements for subscriptions on this plan (JSON)
    /// Copied to SubscriptionEntitlement records when subscription created
    /// </summary>
    [StringLength(4000)]
    public string? DefaultEntitlements { get; set; }
    
    #endregion
    
    #region Navigation
    
    // ... existing navigation ...
    public SubscriptionPlan? DefaultFallbackPlan { get; set; }
    
    #endregion
}
```

---

## 🔐 TOKEN & LICENSE KEY STRUCTURE

### 5. Complete Token Claims (JWT)

```json
{
  // Standard claims
  "jti": "token-guid",
  "iat": 1701302400,
  "exp": 1732838400,
  
  // Identity
  "company_id": "company-guid",
  "company_name": "Acme Corp",
  "subscription_id": "subscription-guid",
  "plan_id": "plan-guid",
  "plan_name": "Enterprise",
  
  // Access Control (NEW)
  "access_mode": "Full",  // Full | GracePeriod | ReadOnly | ExportOnly | Blocked
  "access_expires_at": "2025-12-31T23:59:59Z",
  "days_remaining": 45,
  "fallback_plan": "Free Tier",  // or null
  
  // Global Operations (NEW)
  "allowed_operations": ["GET", "POST", "PUT", "DELETE", "EXPORT"],
  
  // Entitlements Matrix (NEW - STRUCTURED)
  "entitlements": {
    "projects": [
      {
        "project_id": "erp-guid",
        "project_name": "ERP",
        "grant_type": "FullProject",
        "access_level": "Full",
        "operations": ["GET", "POST", "PUT", "DELETE", "EXPORT"],
        "modules": []  // Empty = all modules in project
      },
      {
        "project_id": "finance-guid",
        "project_name": "Finance",
        "grant_type": "SpecificModules",
        "access_level": "Full",
        "modules": [
          {
            "module_id": "payment-guid",
            "module_name": "Payment",
            "access_level": "Full",
            "operations": ["GET", "POST", "PUT", "DELETE", "EXPORT"],
            "features": ["invoicing", "receipts"]
          }
        ]
      }
    ],
    "standalone_modules": [
      {
        "module_id": "reports-guid",
        "module_name": "Advanced Reports",
        "access_level": "ReadOnly",
        "operations": ["GET", "EXPORT"],
        "features": ["dashboards", "analytics"]
      }
    ],
    "usage_limits": {
      "api_calls_per_month": 10000,
      "storage_mb": 5120,
      "users": 25
    }
  },
  
  // Subscription Status
  "is_trial": false,
  "is_lifetime": false,
  "subscription_active": true,
  
  // Upgrade Info (for showing upgrade prompts)
  "upgrade_available": true,
  "blocked_features": ["Inventory", "Manufacturing"],
  "upgrade_message": "Upgrade to access Inventory and Manufacturing modules"
}
```

### 6. Offline License Key Structure

```csharp
internal class OfflineLicenseData
{
    // Identification
    public Guid CompanyId { get; set; }
    public string CompanyName { get; set; } = string.Empty;
    public Guid SubscriptionId { get; set; }
    public Guid PlanId { get; set; }
    public string PlanName { get; set; } = string.Empty;
    
    // Timestamps
    public DateTime IssuedAtUtc { get; set; }
    public DateTime ExpiryDateUtc { get; set; }
    public DateTime? GraceEndDateUtc { get; set; }
    public DateTime? ExportDeadlineUtc { get; set; }
    
    // Access Control (NEW)
    public SubscriptionAccessMode AccessMode { get; set; }
    public List<string> AllowedOperations { get; set; } = new();
    
    // Fallback Configuration (NEW)
    public bool HasFallbackPlan { get; set; }
    public string? FallbackPlanName { get; set; }
    public SubscriptionAccessMode FallbackAccessMode { get; set; }
    
    // Entitlements Matrix (NEW)
    public EntitlementMatrix Entitlements { get; set; } = new();
    
    // Legacy (for backward compatibility)
    public bool IsTrial { get; set; }
    public List<string> Features { get; set; } = new();
    public List<string> Modules { get; set; } = new();
    
    // Security
    public int Version { get; set; } = 2;
    public string IntegrityHash { get; set; } = string.Empty;
}

public class EntitlementMatrix
{
    public List<ProjectEntitlement> Projects { get; set; } = new();
    public List<ModuleEntitlement> StandaloneModules { get; set; } = new();
    public Dictionary<string, object> UsageLimits { get; set; } = new();
    public List<string> BlockedFeatures { get; set; } = new();
    public string? UpgradeMessage { get; set; }
}

public class ProjectEntitlement
{
    public Guid ProjectId { get; set; }
    public string ProjectName { get; set; } = string.Empty;
    public string GrantType { get; set; } = "Specific";
    public string AccessLevel { get; set; } = "Full";
    public List<string> Operations { get; set; } = new();
    public List<ModuleEntitlement> Modules { get; set; } = new();
}

public class ModuleEntitlement
{
    public Guid ModuleId { get; set; }
    public string ModuleName { get; set; } = string.Empty;
    public string AccessLevel { get; set; } = "Full";
    public List<string> Operations { get; set; } = new();
    public List<string> Features { get; set; } = new();
}
```

---

## 🔄 SUBSCRIPTION LIFECYCLE FLOWS

### 7. Complete Lifecycle State Machine

```
┌────────────────────────────────────────────────────────────────────────────────┐
│                      SUBSCRIPTION LIFECYCLE STATE MACHINE                       │
├────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────┐    Start    ┌─────────┐    Upgrade    ┌──────────────┐            │
│  │ (None)  │────Trial───►│  TRIAL  │──────────────►│    ACTIVE    │◄───┐       │
│  └─────────┘             └────┬────┘               └──────┬───────┘    │       │
│                               │                           │            │       │
│                    Trial Ends │              Subscription │         Renew     │
│                    (No Pay)   │                  Expires  │            │       │
│                               ▼                           ▼            │       │
│                         ┌──────────────────────────────────────┐       │       │
│                         │           GRACE PERIOD               │───────┘       │
│                         │  (X days - Full access, payment due) │               │
│                         └──────────────────┬───────────────────┘               │
│                                            │                                   │
│                               Grace Period │ Ends (No Payment)                 │
│                                            ▼                                   │
│                    ┌───────────────────────────────────────────────────┐       │
│                    │               HAS FALLBACK PLAN?                  │       │
│                    └───────────────────────┬───────────────────────────┘       │
│                           YES              │              NO                   │
│                            ▼               ▼               ▼                   │
│              ┌─────────────────┐     ┌─────────────┐                           │
│              │   FREE TIER     │     │ EXPORT ONLY │                           │
│              │  (ReadOnly)     │     │ (30 days)   │                           │
│              │  - View data    │     │ - Export    │                           │
│              │  - Export       │     │ - Then block│                           │
│              │  - No create    │     └──────┬──────┘                           │
│              │  - No edit      │            │                                  │
│              │  - Indefinite   │    Export Period Ends                         │
│              └────────┬────────┘            │                                  │
│                       │                     ▼                                  │
│                       │            ┌──────────────┐                            │
│               Upgrade │            │   BLOCKED    │                            │
│                       │            │ - No access  │                            │
│                       │            │ - Data held  │                            │
│                       │            │ - Must pay   │                            │
│                       │            └──────────────┘                            │
│                       │                     │                                  │
│                       └─────────► ACTIVE ◄──┘ (Reactivate)                     │
│                                                                                 │
└────────────────────────────────────────────────────────────────────────────────┘
```

### 8. Access Mode Transitions

| From State | Trigger | To State | Entitlements | Operations |
|------------|---------|----------|--------------|------------|
| Trial | Convert | Active | Keep same | Full CRUD |
| Trial | Expire (no fallback) | ExportOnly | Keep same | GET, EXPORT only |
| Trial | Expire (with fallback) | ReadOnly (Free Tier) | Reduce to fallback | GET, EXPORT only |
| Active | Expire | GracePeriod | Keep same | Full CRUD |
| GracePeriod | End (no fallback) | ExportOnly | Keep same | GET, EXPORT only |
| GracePeriod | End (with fallback) | ReadOnly | Reduce to fallback | GET, EXPORT only |
| GracePeriod | Renew | Active | Keep same | Full CRUD |
| ExportOnly | Deadline | Blocked | None | None |
| ExportOnly | Pay | Active | Restore | Full CRUD |
| ReadOnly | Upgrade | Active | Expand | Full CRUD |
| Blocked | Reactivate | Active | Restore | Full CRUD |

---

## 🛠️ SERVICE LAYER DESIGN

### 9. IEntitlementService Interface

```csharp
public interface IEntitlementService
{
    #region Entitlement Management
    
    /// <summary>Get all entitlements for a subscription</summary>
    Task<IEnumerable<SubscriptionEntitlementDto>> GetSubscriptionEntitlementsAsync(Guid subscriptionId);
    
    /// <summary>Grant a new entitlement</summary>
    Task<SubscriptionEntitlementDto> GrantEntitlementAsync(GrantEntitlementRequest request);
    
    /// <summary>Revoke an entitlement</summary>
    Task<bool> RevokeEntitlementAsync(Guid entitlementId, string reason);
    
    /// <summary>Update entitlement access level or operations</summary>
    Task<SubscriptionEntitlementDto> UpdateEntitlementAsync(Guid entitlementId, UpdateEntitlementRequest request);
    
    #endregion
    
    #region Bulk Operations
    
    /// <summary>Copy plan entitlements to subscription (on create)</summary>
    Task CopyPlanEntitlementsToSubscriptionAsync(Guid subscriptionId, Guid planId);
    
    /// <summary>Add upgrade entitlements (additive)</summary>
    Task AddUpgradeEntitlementsAsync(Guid subscriptionId, Guid newPlanId);
    
    /// <summary>Replace all entitlements with new plan</summary>
    Task ReplaceEntitlementsAsync(Guid subscriptionId, Guid newPlanId);
    
    /// <summary>Downgrade to fallback plan</summary>
    Task DowngradeToFallbackAsync(Guid subscriptionId);
    
    #endregion
    
    #region Access Checking (For Online Systems)
    
    /// <summary>Check if subscription has project access</summary>
    Task<AccessCheckResult> CheckProjectAccessAsync(Guid subscriptionId, string projectName);
    
    /// <summary>Check if subscription has module access</summary>
    Task<AccessCheckResult> CheckModuleAccessAsync(Guid subscriptionId, string projectName, string moduleName);
    
    /// <summary>Check if subscription has feature access</summary>
    Task<AccessCheckResult> CheckFeatureAccessAsync(Guid subscriptionId, string moduleName, string featureName);
    
    /// <summary>Check if operation is allowed</summary>
    Task<AccessCheckResult> CheckOperationAsync(Guid subscriptionId, string operation);
    
    #endregion
    
    #region Token/License Generation
    
    /// <summary>Build complete entitlement matrix for token/license</summary>
    Task<EntitlementMatrix> BuildEntitlementMatrixAsync(Guid subscriptionId);
    
    #endregion
}

public class AccessCheckResult
{
    public bool HasAccess { get; set; }
    public string AccessLevel { get; set; } = "None";
    public List<string> AllowedOperations { get; set; } = new();
    public bool UpgradeRequired { get; set; }
    public string? UpgradeMessage { get; set; }
    public string? BlockedReason { get; set; }
}
```

### 10. New Client API Endpoints

```csharp
/// <summary>
/// Check access to a specific resource (Online validation)
/// </summary>
[HttpPost("access/check")]
public async Task<ActionResult<AccessCheckResult>> CheckAccess([FromBody] AccessCheckRequest request)
{
    // request: { project: "ERP", module: "HR", feature: "payroll", operation: "POST" }
    var subscriptionId = _jwtService.GetSubscriptionId(HttpContext.User);
    
    var result = await _entitlementService.CheckAccessAsync(subscriptionId.Value, request);
    
    return Ok(result);
}

/// <summary>
/// Get full entitlement matrix (for client caching)
/// </summary>
[HttpGet("entitlements")]
public async Task<ActionResult<EntitlementMatrix>> GetEntitlements()
{
    var subscriptionId = _jwtService.GetSubscriptionId(HttpContext.User);
    
    var matrix = await _entitlementService.BuildEntitlementMatrixAsync(subscriptionId.Value);
    
    return Ok(matrix);
}
```

---

## 💻 CLIENT APPLICATION INTEGRATION

### 11. For Online Systems (API-Based)

```csharp
// Example: Client ERP Application
public class SynfloxAccessChecker
{
    private readonly HttpClient _http;
    private EntitlementMatrix? _cachedEntitlements;
    private DateTime _cacheExpiry;
    
    /// <summary>
    /// Check access before any operation
    /// </summary>
    public async Task<bool> CanAccessAsync(string project, string module, string operation)
    {
        // Quick check from cache
        if (_cachedEntitlements != null && DateTime.UtcNow < _cacheExpiry)
        {
            return CheckLocalAccess(project, module, operation);
        }
        
        // Call SYNFLOX API
        var response = await _http.PostAsJsonAsync("/api/client/access/check", new
        {
            Project = project,
            Module = module,
            Operation = operation
        });
        
        var result = await response.Content.ReadFromJsonAsync<AccessCheckResult>();
        
        if (!result.HasAccess)
        {
            // Show upgrade prompt
            ShowUpgradeDialog(result.UpgradeMessage);
        }
        
        return result.HasAccess;
    }
    
    /// <summary>
    /// Middleware to enforce access on all requests
    /// </summary>
    public async Task<bool> EnforceAccessMiddleware(HttpContext context)
    {
        var path = context.Request.Path.Value;
        var method = context.Request.Method;
        
        // Map path to project/module
        var (project, module) = MapPathToEntitlement(path);
        
        var canAccess = await CanAccessAsync(project, module, method);
        
        if (!canAccess)
        {
            context.Response.StatusCode = 403;
            await context.Response.WriteAsJsonAsync(new
            {
                Error = "Access Denied",
                Message = "Upgrade required to access this feature",
                UpgradeUrl = "https://synflox.com/upgrade"
            });
            return false;
        }
        
        return true;
    }
}
```

### 12. For Offline Systems (License Key)

```csharp
// Example: Offline ERP with License Key
public class OfflineLicenseValidator
{
    private readonly OfflineLicenseData _license;
    
    public OfflineLicenseValidator(string licenseKey, string publicKey)
    {
        _license = DecryptAndValidate(licenseKey, publicKey);
    }
    
    /// <summary>
    /// Check current access mode
    /// </summary>
    public SubscriptionAccessMode GetAccessMode()
    {
        var now = DateTime.UtcNow;
        
        // Check if expired
        if (now > _license.ExpiryDateUtc)
        {
            // Check grace period
            if (_license.GraceEndDateUtc.HasValue && now <= _license.GraceEndDateUtc)
                return SubscriptionAccessMode.GracePeriod;
            
            // Check export deadline
            if (_license.ExportDeadlineUtc.HasValue && now <= _license.ExportDeadlineUtc)
                return SubscriptionAccessMode.ExportOnly;
            
            // Check fallback
            if (_license.HasFallbackPlan)
                return _license.FallbackAccessMode;
            
            return SubscriptionAccessMode.Blocked;
        }
        
        return SubscriptionAccessMode.Full;
    }
    
    /// <summary>
    /// Check if operation is allowed
    /// </summary>
    public bool CanPerformOperation(string project, string module, string operation)
    {
        var accessMode = GetAccessMode();
        
        // Check global operation permissions based on access mode
        if (accessMode == SubscriptionAccessMode.Blocked)
            return false;
        
        if (accessMode == SubscriptionAccessMode.ExportOnly && operation != "EXPORT")
            return false;
        
        if (accessMode == SubscriptionAccessMode.ReadOnly && operation != "GET" && operation != "EXPORT")
            return false;
        
        // Check project/module entitlement
        var projectEnt = _license.Entitlements.Projects
            .FirstOrDefault(p => p.ProjectName.Equals(project, StringComparison.OrdinalIgnoreCase));
        
        if (projectEnt == null)
            return false;
        
        // Full project access
        if (projectEnt.GrantType == "FullProject")
            return projectEnt.Operations.Contains(operation);
        
        // Specific module access
        var moduleEnt = projectEnt.Modules
            .FirstOrDefault(m => m.ModuleName.Equals(module, StringComparison.OrdinalIgnoreCase));
        
        if (moduleEnt == null)
            return false;
        
        return moduleEnt.Operations.Contains(operation);
    }
    
    /// <summary>
    /// Get message for access denied
    /// </summary>
    public AccessDeniedInfo GetAccessDeniedInfo(string module)
    {
        var accessMode = GetAccessMode();
        
        return accessMode switch
        {
            SubscriptionAccessMode.ExportOnly => new AccessDeniedInfo
            {
                Title = "Export Only Mode",
                Message = $"Your subscription has expired. You can export your data until {_license.ExportDeadlineUtc:MMM dd, yyyy}.",
                Action = "Export Data Now",
                Urgency = "High"
            },
            SubscriptionAccessMode.ReadOnly => new AccessDeniedInfo
            {
                Title = "Read-Only Mode",
                Message = "Your subscription has expired. You can view and export data but cannot make changes.",
                Action = "Upgrade to Continue Editing",
                Urgency = "Medium"
            },
            SubscriptionAccessMode.Blocked => new AccessDeniedInfo
            {
                Title = "Access Blocked",
                Message = "Your subscription has expired and the export period has ended. Please renew to access your data.",
                Action = "Renew Subscription",
                Urgency = "Critical"
            },
            _ => new AccessDeniedInfo
            {
                Title = "Upgrade Required",
                Message = $"Access to {module} requires an upgrade.",
                Action = "View Upgrade Options",
                Urgency = "Low"
            }
        };
    }
}
```

---

## 📋 IMPLEMENTATION PHASES

### Phase 1: Core Infrastructure (4-5 days)

| Day | Task | Files |
|-----|------|-------|
| 1 | Create enums and entities | `EntitlementGrantType.cs`, `EntitlementSource.cs`, `EntitlementAccessLevel.cs`, `SubscriptionAccessMode.cs`, `SubscriptionEntitlement.cs` |
| 1 | Update Subscription entity | Add `AccessMode`, `FallbackPlanId`, `ExportDeadlineUtc`, `Entitlements` |
| 2 | Update SubscriptionPlan entity | Add `IsFreeTier`, `FallbackAccessMode`, `ExportGraceDays`, `DefaultFallbackPlanId` |
| 2 | Create EF configurations | `SubscriptionEntitlementConfiguration.cs`, update `SubscriptionConfiguration.cs` |
| 3 | Create repository | `ISubscriptionEntitlementRepository.cs`, `SubscriptionEntitlementRepository.cs` |
| 3 | Create DTOs | `SubscriptionEntitlementDto.cs`, `GrantEntitlementRequest.cs`, `AccessCheckResult.cs`, `EntitlementMatrix.cs` |
| 4 | Create service | `IEntitlementService.cs`, `EntitlementService.cs` |
| 5 | Database migration | Create tables, indexes, migrate existing subscriptions |

### Phase 2: Access Mode System (2-3 days)

| Day | Task | Files |
|-----|------|-------|
| 6 | Update SubscriptionService | Add access mode transitions on expiry, renewal, etc. |
| 6 | Create background job | `AccessModeBackgroundJob.cs` - auto-transition based on dates |
| 7 | Add fallback plan logic | Auto-downgrade when expired with fallback |
| 7 | Add export deadline logic | Set deadline, track, block after |

### Phase 3: Token & License Redesign (2-3 days)

| Day | Task | Files |
|-----|------|-------|
| 8 | Update ClientJwtService | Include full entitlement matrix in claims |
| 8 | Update LicenseService | Include access mode and entitlements in license key |
| 9 | Add backward compatibility | Version handling for old tokens/licenses |
| 9 | Update validation logic | Validate entitlements, not just expiry |

### Phase 4: API & Controller (2 days)

| Day | Task | Files |
|-----|------|-------|
| 10 | Add access check endpoint | `POST /api/client/access/check` |
| 10 | Add entitlements endpoint | `GET /api/client/entitlements` |
| 10 | Create EntitlementsController | Admin UI for managing entitlements |
| 11 | Update existing endpoints | Add access mode to status responses |

### Phase 5: Frontend Integration (3-4 days)

| Day | Task | Files |
|-----|------|-------|
| 12 | Create entitlement models | `domain/models/entitlement.model.ts` |
| 12 | Create entitlement service | `services/entitlement.service.ts` |
| 13 | Add entitlements tab | Subscription details page - show/manage entitlements |
| 13 | Add grant/revoke dialogs | UI for granting/revoking modules |
| 14 | Add access mode display | Show current mode, countdown, upgrade prompts |
| 15 | Add plan fallback config | Plan create/edit - configure fallback options |

### Phase 6: Testing & Documentation (2 days)

| Day | Task |
|-----|------|
| 16 | E2E testing all scenarios |
| 16 | Test lifecycle transitions |
| 17 | Create client SDK documentation |
| 17 | Update API documentation |

**Total: 15-17 days**

---

## ✅ WHAT THIS SOLVES

| Scenario | Before | After |
|----------|--------|-------|
| Client has ERP but only HR module | ❌ Not possible | ✅ GrantType: SpecificModules |
| Full ERP + Payment from Finance | ❌ Flat list, no context | ✅ Two project entries |
| Upgrade to add Accounting | ❌ Must change plan | ✅ Grant new entitlement |
| After expiry, still view data | ❌ Completely blocked | ✅ ReadOnly mode |
| 30 days to export then block | ❌ Not possible | ✅ ExportOnly mode |
| Free tier fallback | ❌ Not possible | ✅ FallbackPlanId |
| Offline validation | ❌ Honor system | ✅ Enforced in license |
| Module trial (14 days) | ❌ Not possible | ✅ ExpiresAt on entitlement |
| Know what's blocked | ❌ No info | ✅ BlockedFeatures list |

---

## 🎯 DECISION SUMMARY

Based on our discussion, here are the recommended settings:

| Decision | Recommendation | Reason |
|----------|---------------|--------|
| Granularity | Module-level | Balance of control and simplicity |
| Default Plan Behavior | FullProject for included projects | Less configuration needed |
| Upgrade Strategy | Additive | Keep custom grants |
| After Expiry | Fallback to Free Tier | Never hold data hostage |
| Free Tier Access | ReadOnly | View + Export only |
| Export Grace Days | 30 days | Industry standard |
| Token Regeneration | Automatic on change | Always up to date |
| License Key Version | Bump to v2 | New structure |

---

## 🚀 READY FOR IMPLEMENTATION?

This is the complete, enterprise-grade solution. Let me know:

1. ✅ **Approve to start implementation?**
2. ❓ Any changes to the decisions above?
3. ❓ Any scenarios I missed?

Once confirmed, I'll start with **Phase 1: Core Infrastructure**.
