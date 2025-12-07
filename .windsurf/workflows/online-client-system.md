---
description: 
auto_execution_mode: 3
---

# 🌐 SYNFLOX Online Client System - Complete Implementation Workflow

## 📋 Overview

This workflow implements the **Online Client Licensing System** that mirrors the completeness of the Offline system, with full isolation but shared base classes following SOLID principles.

---

## 🏗️ Architecture Philosophy

### Key Differences from Offline

| Aspect | OFFLINE | ONLINE |
|--------|---------|--------|
| **Token** | Self-contained license key with ALL data | Thin JWT with only identity (no entitlements) |
| **Updates** | Regenerate license key on changes | Real-time via API (no token change needed) |
| **Device Binding** | Hardware fingerprint validation | API-based device registration |
| **Entitlements** | Embedded in encrypted key | Fetched dynamically via `/api/online/entitlements` |
| **Expiry** | Clock tampering detection | Server-side validation |

### Shared Components (SOLID)

```
┌─────────────────────────────────────────────────────────────────┐
│                    SHARED BASE LAYER                            │
├─────────────────────────────────────────────────────────────────┤
│ • IClientLicenseService (base interface)                        │
│ • ClientLicenseContext (company, subscription, plan info)       │
│ • DeviceManagement (bind/unbind/list devices)                   │
│ • EntitlementMatrix (project/module access rights)              │
│ • PlanChangePolicy (immediate vs next-billing-cycle)            │
│ • BillingCycleCalculator (when changes take effect)             │
└─────────────────────────────────────────────────────────────────┘
                    │                    │
         ┌──────────┴────────┐  ┌────────┴──────────┐
         │  OFFLINE SYSTEM   │  │   ONLINE SYSTEM   │
         │                   │  │                   │
         │ • License Key     │  │ • JWT Token       │
         │ • AES-256-GCM     │  │ • API Endpoints   │
         │ • Clock Tampering │  │ • Real-time       │
         │ • Self-contained  │  │ • Dynamic fetch   │
         └───────────────────┘  └───────────────────┘
```

---

## 📊 Billing Cycle Change Policy

### What Changes Apply IMMEDIATELY (Next API Call)

| Change Type | Reason | Example |
|-------------|--------|---------|
| **New modules/features added to plan** | Customer benefit | Added "Reports Module" to Pro Plan |
| **Access level upgrade** | Customer benefit | ReadOnly → Full access |
| **Device limit increase** | Customer benefit | 5 → 10 devices |
| **Security patches** | Critical | New encryption algorithm |
| **Bug fixes** | Critical | Fixed validation bug |
| **Plan name/description** | Cosmetic | "Pro" → "Professional" |

### What Waits Until NEXT BILLING CYCLE

| Change Type | Reason | Example |
|-------------|--------|---------|
| **Price increase** | Contract commitment | $99/mo → $129/mo |
| **Module removal** | Paid for current period | Removed "Advanced Analytics" |
| **Device limit decrease** | Already using devices | 10 → 5 devices |
| **Access level downgrade** | Paid for current access | Full → ReadOnly |
| **Trial conversion to paid** | Grace period | Trial ends, payment starts |
| **Plan downgrade** | Customer initiated | Enterprise → Pro |

### Implementation: `PlanChangeEffectiveDate`

```csharp
public enum ChangeEffectPolicy
{
    Immediate,          // Apply now
    NextBillingCycle,   // Apply at subscription renewal
    GracePeriodEnd,     // Apply after grace period
    Manual              // Admin decides
}

public class PlanChangePolicy
{
    public ChangeEffectPolicy PriceChanges { get; set; } = ChangeEffectPolicy.NextBillingCycle;
    public ChangeEffectPolicy FeatureAdditions { get; set; } = ChangeEffectPolicy.Immediate;
    public ChangeEffectPolicy FeatureRemovals { get; set; } = ChangeEffectPolicy.NextBillingCycle;
    public ChangeEffectPolicy DeviceLimitIncrease { get; set; } = ChangeEffectPolicy.Immediate;
    public ChangeEffectPolicy DeviceLimitDecrease { get; set; } = ChangeEffectPolicy.NextBillingCycle;
    public ChangeEffectPolicy AccessUpgrade { get; set; } = ChangeEffectPolicy.Immediate;
    public ChangeEffectPolicy AccessDowngrade { get; set; } = ChangeEffectPolicy.NextBillingCycle;
}
```

---

## 🔧 Phase 1: Shared Base Classes (SOLID)

### 1.1 Domain Layer - Shared Abstractions

```
Domain/
├── Abstractions/
│   ├── IClientLicenseContext.cs      # Base context interface
│   ├── IDeviceManager.cs             # Device operations interface
│   ├── IEntitlementProvider.cs       # Entitlement fetching interface
│   └── IPlanChangePolicy.cs          # Change policy interface
├── ValueObjects/
│   ├── EntitlementMatrix.cs          # Full entitlement structure
│   ├── DeviceInfo.cs                 # Device fingerprint + metadata
│   ├── BillingCycle.cs               # Start/end dates, period
│   └── ChangeEffectiveDate.cs        # When changes apply
└── Enums/
    ├── ChangeEffectPolicy.cs         # Immediate/NextBilling/etc
    ├── DeviceStatus.cs               # Active/Suspended/Revoked
    └── ClientSystemType.cs           # Online/Offline
```

### 1.2 Application Layer - Shared DTOs

```
Application/DTOs/
├── ClientLicense/
│   ├── ClientLicenseContextDto.cs    # Common context
│   ├── EntitlementMatrixDto.cs       # Full entitlements
│   ├── DeviceBindingDto.cs           # Device binding info
│   ├── PlanChangePolicyDto.cs        # Change policies
│   └── BillingCycleDto.cs            # Billing info
└── Common/
    └── IdRequestDtos.cs              # ✅ Already created
```

### 1.3 Infrastructure Layer - Shared Services

```
Infrastructure/Services/
├── Shared/
│   ├── DeviceManagementService.cs    # Shared device logic
│   ├── EntitlementService.cs         # Shared entitlement logic
│   ├── PlanChangePolicyService.cs    # Change policy evaluation
│   └── BillingCycleService.cs        # Billing calculations
```

---

## 🔧 Phase 2: Online Client Domain

### 2.1 Entity: OnlineClientToken

```csharp
public class OnlineClientToken : AuditEntity<Guid>
{
    // Identity
    public Guid CompanyId { get; set; }
    public Guid SubscriptionId { get; set; }
    
    // Token (thin - no entitlements)
    public required string TokenHash { get; set; }  // SHA-256
    public DateTime IssuedAtUtc { get; set; }
    public DateTime ExpiresAtUtc { get; set; }
    public ClientTokenStatus Status { get; set; }
    
    // Auto-refresh on subscription changes (no manual regeneration needed!)
    public bool AutoRefreshEnabled { get; set; } = true;
    
    // Device tracking (optional, for limited-device plans)
    public int? MaxDevices { get; set; }  // Null = unlimited
    
    // Usage tracking
    public DateTime? LastUsedAtUtc { get; set; }
    public int UsageCount { get; set; }
    public string? LastUsedFromIp { get; set; }
    
    // Navigation
    public virtual Company Company { get; set; }
    public virtual Subscription Subscription { get; set; }
    public virtual ICollection<OnlineDeviceBinding> BoundDevices { get; set; }
}
```

### 2.2 Entity: OnlineDeviceBinding

```csharp
public class OnlineDeviceBinding : AuditEntity<Guid>
{
    public Guid TokenId { get; set; }
    public Guid SubscriptionId { get; set; }
    
    // Device info
    public required string DeviceFingerprint { get; set; }
    public string? DeviceName { get; set; }
    public string? DeviceType { get; set; }  // Desktop, Server, VM, etc.
    
    // Status
    public DeviceStatus Status { get; set; } = DeviceStatus.Active;
    public DateTime FirstSeenAtUtc { get; set; }
    public DateTime? LastSeenAtUtc { get; set; }
    public string? LastIpAddress { get; set; }
    
    // Navigation
    public virtual OnlineClientToken Token { get; set; }
    public virtual Subscription Subscription { get; set; }
}
```

### 2.3 Entity: SubscriptionChangeLog (Billing Cycle Tracking)

```csharp
public class SubscriptionChangeLog : AuditEntity<Guid>
{
    public Guid SubscriptionId { get; set; }
    public Guid? PlanId { get; set; }
    
    // What changed
    public required string ChangeType { get; set; }  // PriceChange, FeatureAdded, etc.
    public required string ChangeDescription { get; set; }
    public string? OldValue { get; set; }
    public string? NewValue { get; set; }
    
    // When it takes effect
    public ChangeEffectPolicy EffectPolicy { get; set; }
    public DateTime EffectiveDateUtc { get; set; }
    public bool IsApplied { get; set; }
    public DateTime? AppliedAtUtc { get; set; }
    
    // Navigation
    public virtual Subscription Subscription { get; set; }
    public virtual SubscriptionPlan? Plan { get; set; }
}
```

---

## 🔧 Phase 3: Online Client API

### 3.1 Controller: OnlineClientController

Route: `/api/online/`

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/auth/validate` | POST | Validate token (returns company/subscription context) |
| `/entitlements` | GET | Get current entitlement matrix |
| `/entitlements/version` | GET | Get entitlement version (for cache checking) |
| `/subscription/status` | GET | Get subscription status |
| `/subscription/billing` | GET | Get billing cycle info |
| `/devices/register` | POST | Register a new device |
| `/devices/list` | GET | List bound devices |
| `/devices/unregister` | DELETE | Unregister a device |
| `/devices/limit` | GET | Get device limit info |
| `/plan/features` | GET | Get plan features |
| `/plan/pending-changes` | GET | Get changes waiting for next billing cycle |
| `/health` | GET | Health check |

### 3.2 Authentication: OnlineClientAuthMiddleware

- Extract JWT from Authorization header
- Validate signature (different key from admin JWT)
- Extract claims: company_id, subscription_id, token_id
- Check token status in database
- Set HttpContext.User with claims

### 3.3 Real-time Updates (SignalR Hub)

```csharp
public interface IOnlineClientHub
{
    Task EntitlementsUpdated(string subscriptionId, int newVersion);
    Task PlanChanged(string subscriptionId, PlanChangeNotification change);
    Task DeviceLimitReached(string subscriptionId, int limit);
    Task SubscriptionStatusChanged(string subscriptionId, string newStatus);
}
```

---

## 🔧 Phase 4: Plan Change Event System

### 4.1 Event Types

```csharp
public enum PlanChangeEventType
{
    // Immediate
    ModuleAdded,
    ModuleRemoved,
    FeatureAdded,
    FeatureRemoved,
    AccessLevelChanged,
    DeviceLimitChanged,
    
    // Next Billing Cycle
    PriceChanged,
    PlanDowngraded,
    PlanUpgraded,
    
    // Manual
    CustomChange
}
```

### 4.2 Background Job: ApplyPendingChangesJob

- Runs daily at midnight UTC
- Finds subscriptions where `BillingCycleEndUtc <= UtcNow`
- Applies pending changes from `SubscriptionChangeLog`
- Notifies clients via SignalR
- Sends email notifications

### 4.3 Event Handlers

```csharp
public class PlanChangedEventHandler : INotificationHandler<PlanChangedEvent>
{
    public async Task Handle(PlanChangedEvent notification)
    {
        // 1. Determine effect policy for each change
        // 2. For Immediate: Apply now, increment version
        // 3. For NextBillingCycle: Create SubscriptionChangeLog entry
        // 4. Notify affected subscriptions via SignalR
        // 5. Update entitlement version
    }
}
```

---

## 🔧 Phase 5: Admin UI Integration

### 5.1 Company Details → "Online Tokens" Tab

- Generate online token for subscription
- View token status and usage
- View bound devices
- Manage device limits

### 5.2 Subscription Details → "Online Access" Tab

- Token status
- Entitlements (inherited from plan)
- Bound devices
- Pending changes (waiting for next billing)

### 5.3 Plan Edit → "Change Policies" Section

- Configure which changes are immediate vs next-billing
- Set notification preferences
- Override default policies

---

## 🔧 Phase 6: Testing & Migration

### 6.1 Test Scenarios

1. **Token validation** - Valid/expired/revoked tokens
2. **Entitlement fetch** - Correct plan entitlements returned
3. **Device binding** - Respect device limits
4. **Plan change - Immediate** - Module added, immediately available
5. **Plan change - Next billing** - Price change, takes effect at renewal
6. **Real-time updates** - SignalR notifications work
7. **Billing cycle rollover** - Pending changes applied correctly

### 6.2 Migration from Old System

- Not needed (old system was never built, we deleted empty files)

---

## 📊 Progress Tracker

### Phase 1: Shared Base Classes
- [ ] 1.1 Domain abstractions and interfaces
- [ ] 1.2 Shared DTOs
- [ ] 1.3 Shared services

### Phase 2: Online Client Domain
- [ ] 2.1 OnlineClientToken entity
- [ ] 2.2 OnlineDeviceBinding entity  
- [ ] 2.3 SubscriptionChangeLog entity
- [ ] 2.4 Repository interfaces
- [ ] 2.5 EF configurations

### Phase 3: Online Client API
- [ ] 3.1 OnlineClientController
- [ ] 3.2 Authentication middleware
- [ ] 3.3 OnlineClientService
- [ ] 3.4 JWT service for online tokens

### Phase 4: Plan Change Event System
- [ ] 4.1 Event types and handlers
- [ ] 4.2 ApplyPendingChangesJob
- [ ] 4.3 SignalR hub integration

### Phase 5: Admin UI Integration
- [ ] 5.1 Company "Online Tokens" tab
- [ ] 5.2 Subscription "Online Access" tab
- [ ] 5.3 Plan "Change Policies" section

### Phase 6: Testing & Documentation
- [ ] 6.1 Unit tests
- [ ] 6.2 Integration tests
- [ ] 6.3 API documentation

---

## ⏱️ Estimated Timeline

| Phase | Duration | Dependencies |
|-------|----------|--------------|
| Phase 1 | 1-2 days | None |
| Phase 2 | 2-3 days | Phase 1 |
| Phase 3 | 3-4 days | Phase 2 |
| Phase 4 | 2-3 days | Phase 3 |
| Phase 5 | 3-4 days | Phase 3 |
| Phase 6 | 1-2 days | All phases |

**Total: 12-18 days**

---

## 🎯 Success Criteria

1. ✅ Online system completely isolated from offline
2. ✅ Shared base classes follow SOLID principles
3. ✅ Plan changes respect billing cycle policies
4. ✅ No token regeneration needed for plan updates
5. ✅ Real-time notifications via SignalR
6. ✅ Device binding with configurable limits
7. ✅ Full admin UI integration
