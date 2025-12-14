# 🌐 SYNFLOX Online Client System - Complete Business Documentation

> **Version**: 2.0  
> **Last Updated**: December 2025  
> **Audience**: Developers, System Architects, Business Analysts

---

## 📋 Table of Contents

1. [System Overview](#system-overview)
2. [Entity Hierarchy](#entity-hierarchy)
3. [Complete Business Flow](#complete-business-flow)
4. [Entitlement System](#entitlement-system)
5. [Device Management](#device-management)
6. [Access Modes & Lifecycle](#access-modes--lifecycle)
7. [API Endpoints](#api-endpoints)
8. [Menu Visibility & UI Control](#menu-visibility--ui-control)
9. [Security Considerations](#security-considerations)
10. [Flowcharts](#flowcharts)

---

## 🎯 System Overview

### What is SYNFLOX?

SYNFLOX is a **Central Licensing System** that manages software licenses for enterprise products. It supports two client modes:

| Mode | Description | Use Case |
|------|-------------|----------|
| **Online** | Thin JWT token + API-based entitlement fetching | Web apps, cloud-connected desktop apps |
| **Offline** | Self-contained encrypted license key | Air-gapped systems, no internet required |

**This document focuses on the ONLINE system.**

### Key Principle: Thin Token Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     ONLINE TOKEN PHILOSOPHY                             │
│                                                                          │
│  Token Contains ONLY:                                                   │
│  ┌──────────────────┐                                                   │
│  │  company_id      │                                                   │
│  │  subscription_id │   ◄── Identity Only! No entitlements embedded    │
│  │  token_id        │                                                   │
│  └──────────────────┘                                                   │
│                                                                          │
│  Benefits:                                                              │
│  ✓ No token regeneration when plan changes                             │
│  ✓ Entitlements always fresh from API                                  │
│  ✓ Real-time access control updates                                    │
│  ✓ Smaller token size                                                   │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 🏗️ Entity Hierarchy

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         ENTITY RELATIONSHIPS                            │
│                                                                          │
│  Company (Tenant)                                                       │
│  ├── CompanyAdmin (1:1) ─────────── Client portal login                │
│  │       └── CompanyAdminSession    Session management                  │
│  │                                                                       │
│  └── Subscriptions (1:N)                                                │
│          │                                                               │
│          ├── Plan (N:1) ─────────── SubscriptionPlan                   │
│          │       ├── PlanEntitlements (1:N) ── Access rights            │
│          │       ├── PlanPrices (1:N)                                   │
│          │       ├── AccessTimeWindows (1:N)                            │
│          │       └── ParentPlan (self-ref) ── Inheritance               │
│          │                                                               │
│          ├── OnlineClientTokens (1:N) ─────── API access tokens        │
│          │       └── OnlineDeviceBindings (1:N)                         │
│          │                                                               │
│          └── SubscriptionHistory (1:N) ────── Audit trail              │
│                                                                          │
│  Projects (Products)                                                     │
│  ├── Modules (N:N via ProjectModule)                                    │
│  └── Features (JSON array)                                              │
│                                                                          │
│  Modules (Features)                                                      │
│  ├── Projects (N:N via ProjectModule)                                   │
│  └── Features (JSON array)                                              │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 📊 Complete Business Flow

### Phase 1: Setup by SYNFLOX Admin

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    STEP 1: CREATE COMPANY                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  SYNFLOX Admin creates Company:                                         │
│  ┌────────────────────────────────────────┐                             │
│  │  Company                               │                             │
│  │  ├── Name: "Acme Corporation"         │                             │
│  │  ├── ContactEmail: "admin@acme.com"   │                             │
│  │  ├── ContactPhone: "+1-555-0100"      │                             │
│  │  ├── Address: "123 Business St"       │                             │
│  │  ├── TimezoneId: "America/New_York"   │                             │
│  │  └── IsActive: true                   │                             │
│  └────────────────────────────────────────┘                             │
│                                                                          │
│  Triggers:                                                              │
│  ✓ Company created in database                                         │
│  ✓ Email sent to ContactEmail (optional)                               │
│  ✓ Company can now receive subscriptions                               │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    STEP 2: CREATE COMPANY ADMIN                         │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  SYNFLOX Admin creates CompanyAdmin for device management:              │
│  ┌────────────────────────────────────────┐                             │
│  │  CompanyAdmin                          │                             │
│  │  ├── Username: "acme-admin"           │                             │
│  │  ├── PasswordHash: [hashed]           │                             │
│  │  ├── DisplayName: "IT Administrator" │                             │
│  │  ├── Email: "it@acme.com"            │                             │
│  │  ├── Phone: "+1-555-0101"            │                             │
│  │  │                                    │                             │
│  │  │ Permissions:                       │                             │
│  │  ├── CanManageDevices: true          │ ◄── Bind/unbind devices     │
│  │  ├── CanViewSubscriptions: true      │ ◄── View subscription info  │
│  │  ├── CanApproveReplacements: true    │ ◄── Approve device swaps    │
│  │  ├── CanGenerateLicenses: true       │ ◄── Generate offline keys   │
│  │  ├── CanViewUsageReports: true       │ ◄── View analytics          │
│  │  └── CanModifySessionSettings: true  │ ◄── Change own settings     │
│  └────────────────────────────────────────┘                             │
│                                                                          │
│  Session Policies:                                                      │
│  • SingleSession: Only one active session at a time                    │
│  • MultiSession: Multiple sessions allowed                             │
│  • SessionTimeout: 8 hours default                                      │
│  • InactivityTimeout: 30 minutes default                                │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    STEP 3: CREATE SUBSCRIPTION PLAN                     │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  SubscriptionPlan defines WHAT a customer can access:                   │
│  ┌────────────────────────────────────────┐                             │
│  │  SubscriptionPlan: "Enterprise"        │                             │
│  │  │                                      │                             │
│  │  │ Basic Info:                          │                             │
│  │  ├── Name: "Enterprise Plan"           │                             │
│  │  ├── Description: "Full access..."     │                             │
│  │  ├── DurationType: Yearly              │                             │
│  │  ├── AllowTrial: true                  │                             │
│  │  ├── TrialDurationDays: 14             │                             │
│  │  ├── AutoRenew: true                   │                             │
│  │  ├── GracePeriodDays: 7                │                             │
│  │  │                                      │                             │
│  │  │ Device Limits:                       │                             │
│  │  ├── MaxDevices: 50                    │ ◄── Total devices allowed  │
│  │  ├── MaxConcurrentDevices: 25          │ ◄── Simultaneous active    │
│  │  ├── ConcurrentAccessMode: Limited    │                             │
│  │  ├── DeviceAdmissionMode: Open         │ ◄── Self-register OK       │
│  │  ├── DeviceReplacementPolicy: Auto    │                             │
│  │  │                                      │                             │
│  │  │ Entitlement Config:                  │                             │
│  │  ├── ShowLockedModulesInMenu: true    │ ◄── Show upgrades in UI    │
│  │  ├── LockedItemStyle: "greyed_with_lock"│                            │
│  │  ├── IsFreeTier: false                 │                             │
│  │  └── FallbackAccessMode: ReadOnly      │ ◄── When expired           │
│  └────────────────────────────────────────┘                             │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    STEP 4: CONFIGURE PLAN ENTITLEMENTS                  │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  PlanEntitlement defines access to Projects and Modules:                │
│                                                                          │
│  Enterprise Plan Entitlements:                                          │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │ Project: "ERP System"                   AccessLevel: Full     │     │
│  │   ├── Module: "Inventory"               AccessLevel: Full     │     │
│  │   ├── Module: "Sales"                   AccessLevel: Full     │     │
│  │   ├── Module: "Purchasing"              AccessLevel: Full     │     │
│  │   └── Module: "Analytics" [OVERRIDE]    AccessLevel: ReadOnly │     │
│  │                                                                │     │
│  │ Project: "CRM System"                   AccessLevel: Full     │     │
│  │   ├── Module: "Contacts"                AccessLevel: Full     │     │
│  │   ├── Module: "Leads"                   AccessLevel: Full     │     │
│  │   └── Module: "Reports"                 AccessLevel: Full     │     │
│  │                                                                │     │
│  │ Module: "AI Assistant" [STANDALONE]     AccessLevel: Full     │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
│  Permission Flags per Entitlement:                                      │
│  • CanCreate: true/false                                                │
│  • CanRead: true/false                                                  │
│  • CanUpdate: true/false                                                │
│  • CanDelete: true/false                                                │
│  • CanExport: true/false                                                │
│  • DisplayInMenu: true/false ◄── Hide/show in client UI                │
│  • Features: "reports,analytics,advanced-search"                       │
│                                                                          │
│  Inheritance:                                                           │
│  • Module inherits from parent Project UNLESS IsOverride = true         │
│  • Child plans inherit from ParentPlan                                  │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    STEP 5: CREATE SUBSCRIPTION                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Subscription = Company + Plan + Time Window:                           │
│  ┌────────────────────────────────────────┐                             │
│  │  Subscription                          │                             │
│  │  ├── CompanyId: [Acme Corp]           │                             │
│  │  ├── PlanId: [Enterprise Plan]        │                             │
│  │  ├── StartDateUtc: 2025-01-01         │                             │
│  │  ├── ExpiryDateUtc: 2025-12-31        │                             │
│  │  ├── IsActive: true                   │                             │
│  │  ├── IsTrial: false                   │                             │
│  │  ├── AutoRenew: true                  │                             │
│  │  │                                    │                             │
│  │  │ Access Control:                    │                             │
│  │  ├── AccessMode: Full                 │ ◄── Current access state   │
│  │  ├── EntitlementsVersion: 1           │ ◄── Cache invalidation     │
│  │  │                                    │                             │
│  │  │ Device Overrides (optional):       │                             │
│  │  ├── MaxDevicesOverride: 100          │ ◄── Custom limit           │
│  │  └── DeviceAdmissionModeOverride: null│ ◄── Use plan default       │
│  └────────────────────────────────────────┘                             │
│                                                                          │
│  Triggers:                                                              │
│  ✓ Subscription created                                                 │
│  ✓ Email sent to company                                                │
│  ✓ Company can now generate online tokens                               │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### Phase 2: Token Generation & Client Access

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    STEP 6: GENERATE ONLINE TOKEN                        │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  CompanyAdmin or SYNFLOX Admin generates token:                         │
│  ┌────────────────────────────────────────┐                             │
│  │  OnlineClientToken                     │                             │
│  │  ├── Name: "Production API Token"     │                             │
│  │  ├── TokenHash: [SHA256 of JWT]       │ ◄── We never store JWT     │
│  │  ├── CompanyId: [Acme Corp]           │                             │
│  │  ├── SubscriptionId: [Enterprise Sub] │                             │
│  │  ├── IssuedAtUtc: 2025-01-01          │                             │
│  │  ├── ExpiresAtUtc: 2026-01-01         │ ◄── Can match subscription │
│  │  ├── Status: Active                   │                             │
│  │  ├── AutoRefreshEnabled: true         │ ◄── Token stays valid      │
│  │  └── MaxDevices: null                 │ ◄── Use plan default       │
│  └────────────────────────────────────────┘                             │
│                                                                          │
│  JWT Token Contents (THIN):                                             │
│  {                                                                       │
│    "token_id": "abc123",                                                │
│    "company_id": "...",                                                 │
│    "subscription_id": "...",                                            │
│    "iat": 1704067200,                                                   │
│    "exp": 1735689600                                                    │
│  }                                                                       │
│                                                                          │
│  ❌ NO entitlements in token!                                           │
│  ❌ NO plan details in token!                                           │
│  ❌ NO device list in token!                                            │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    STEP 7: CLIENT APPLICATION FLOW                      │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Client App Startup:                                                    │
│                                                                          │
│  1. Validate Token:                                                     │
│     POST /api/online/auth/validate                                      │
│     Header: Authorization: Bearer <jwt_token>                           │
│     Body: { "device_fingerprint": "...", "device_name": "..." }         │
│     Response: { valid: true, subscription_status: "Active" }            │
│                                                                          │
│  2. Register Device (if new):                                           │
│     POST /api/online/devices/register                                   │
│     Body: { "fingerprint": "...", "name": "...", "type": "Desktop" }    │
│     Response: { device_id: "...", status: "Active" }                    │
│                                                                          │
│  3. Fetch Entitlements:                                                 │
│     GET /api/online/entitlements                                        │
│     Response: Complete entitlement matrix (see below)                   │
│                                                                          │
│  4. Cache Entitlements:                                                 │
│     • Store locally for 24 hours                                        │
│     • Track EntitlementsVersion header                                  │
│     • Refresh when version changes                                      │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 🔐 Entitlement System

### Entitlement Matrix Response

```json
{
  "subscription": {
    "id": "sub-123",
    "company_id": "company-456",
    "plan_name": "Enterprise",
    "access_mode": "Full",
    "expires_at": "2025-12-31T23:59:59Z",
    "is_trial": false,
    "days_remaining": 365
  },
  "entitlements_version": 3,
  "projects": [
    {
      "id": "proj-erp",
      "name": "ERP System",
      "access_level": "Full",
      "display_in_menu": true,
      "modules": [
        {
          "id": "mod-inventory",
          "name": "Inventory",
          "access_level": "Full",
          "can_create": true,
          "can_read": true,
          "can_update": true,
          "can_delete": true,
          "can_export": true,
          "display_in_menu": true,
          "features": ["reports", "barcode-scanner"]
        },
        {
          "id": "mod-analytics",
          "name": "Analytics",
          "access_level": "ReadOnly",
          "can_create": false,
          "can_read": true,
          "can_update": false,
          "can_delete": false,
          "can_export": true,
          "display_in_menu": true,
          "is_override": true
        }
      ]
    }
  ],
  "standalone_modules": [
    {
      "id": "mod-ai",
      "name": "AI Assistant",
      "access_level": "Full",
      "can_create": true,
      "can_read": true,
      "can_update": true,
      "can_delete": true,
      "can_export": true,
      "display_in_menu": true
    }
  ],
  "locked_items": [
    {
      "id": "proj-hr",
      "name": "HR System",
      "type": "Project",
      "display_style": "greyed_with_lock",
      "upgrade_plan": "Ultimate"
    }
  ],
  "custom_features": ["24/7 Support", "SLA 1h", "Priority Queue"],
  "device_limits": {
    "max_devices": 50,
    "max_concurrent": 25,
    "current_devices": 12,
    "current_active": 8
  }
}
```

### Access Level Hierarchy

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    ENTITLEMENT ACCESS LEVELS                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Full (1)          All CRUD operations allowed                          │
│      │             CanCreate, CanRead, CanUpdate, CanDelete, CanExport  │
│      │                                                                   │
│      ▼                                                                   │
│  ReadOnly (2)      View + Export only                                   │
│      │             CanRead, CanExport                                    │
│      │             (For expired with fallback plan)                      │
│      │                                                                   │
│      ▼                                                                   │
│  ExportOnly (3)    Export data before block                             │
│      │             CanExport only                                        │
│      │             (Last chance to get data out)                        │
│      │                                                                   │
│      ▼                                                                   │
│  Blocked (4)       No access whatsoever                                 │
│                    Upgrade required                                      │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 📱 Device Management

### Device Admission Modes

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    DEVICE ADMISSION MODES                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Open (1)                                                               │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  Any device can self-register up to MaxDevices                 │     │
│  │  When limit reached → New devices blocked                       │     │
│  │  Best for: Self-service, small teams                           │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
│  AdminOnly (2)                                                          │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  CompanyAdmin must explicitly bind each device                  │     │
│  │  Devices cannot self-register                                   │     │
│  │  Best for: High-security, strict control                        │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
│  AutoWithQueue (3)                                                      │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  Auto-register up to limit                                      │     │
│  │  When limit reached → Queue for admin approval                  │     │
│  │  Best for: Balanced convenience + oversight                     │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
│  HybridAutoAdmin (4)                                                    │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  First N devices auto-register (MaxAutoAdmitDevices)            │     │
│  │  Device N+1 onwards require admin approval                      │     │
│  │  Best for: Core team + occasional guests                        │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### Concurrent Access Modes

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    CONCURRENT ACCESS MODES                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  SingleDevice (1)      Only ONE device active at a time                │
│                        New login kicks previous device                  │
│                                                                          │
│  LimitedConcurrent (2) Up to N devices simultaneously                  │
│                        Uses MaxConcurrentDevices setting                │
│                                                                          │
│  Unlimited (3)         All allowed devices can access                   │
│                        No concurrent restrictions                       │
│                                                                          │
│  TimeBasedUnlimited (4) All devices, but only during time windows      │
│                         Uses AccessTimeWindow records                   │
│                                                                          │
│  TimeBasedLimited (5)  N devices during time windows                   │
│                        Combines concurrent + time restrictions          │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 🔄 Access Modes & Lifecycle

### Subscription Lifecycle

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    SUBSCRIPTION LIFECYCLE                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Created                                                                │
│     │                                                                    │
│     ▼                                                                    │
│  ┌─────────┐                                                            │
│  │ Active  │ ◄───────────────────────────────────────┐                 │
│  │ (Full)  │                                          │                 │
│  └────┬────┘                                          │                 │
│       │                                               │                 │
│       │ Expires                                       │ Renew/Upgrade   │
│       ▼                                               │                 │
│  ┌─────────────┐                                      │                 │
│  │ GracePeriod │ ── X days (from plan.GracePeriodDays)│                 │
│  │ (GracePeriod)│                                     │                 │
│  └──────┬──────┘                                      │                 │
│         │                                             │                 │
│         │ Grace expires                               │                 │
│         ▼                                             │                 │
│  ┌────────────────┐                                   │                 │
│  │ Has Fallback?  │                                   │                 │
│  └───────┬────────┘                                   │                 │
│      YES │    NO                                      │                 │
│          │     │                                      │                 │
│          ▼     ▼                                      │                 │
│  ┌──────────┐ ┌────────────┐                          │                 │
│  │ ReadOnly │ │ ExportOnly │ ── 30 days (ExportGraceDays)               │
│  │ (Fallback│ │            │                          │                 │
│  │  Plan)   │ └─────┬──────┘                          │                 │
│  └──────────┘       │                                 │                 │
│                     │ Export deadline                 │                 │
│                     ▼                                 │                 │
│                ┌─────────┐                            │                 │
│                │ Blocked │ ───── Upgrade Required ────┘                 │
│                │         │                                              │
│                └─────────┘                                              │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### Special States

| State | Description | User Experience |
|-------|-------------|-----------------|
| **IsTrial** | Trial period active | Full access, countdown shown |
| **IsPaused** | Timer frozen | Full access, no expiry countdown |
| **AutoRenew** | Will renew automatically | Notification before renewal |
| **IsLifetime** | Never expires | Permanent access |

---

## 🌐 API Endpoints

### Online Client API

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/online/auth/validate` | POST | Validate token, register device |
| `/api/online/entitlements` | GET | Get complete entitlement matrix |
| `/api/online/subscription/status` | GET | Get subscription status |
| `/api/online/devices/register` | POST | Register new device |
| `/api/online/devices/list` | GET | List all bound devices |
| `/api/online/devices/unregister` | DELETE | Remove device binding |
| `/api/online/plan/pending-changes` | GET | Changes waiting for next billing |

### Response Headers

| Header | Description |
|--------|-------------|
| `X-Entitlements-Version` | Current entitlements version |
| `X-Subscription-Status` | Active, GracePeriod, Blocked, etc. |
| `X-Days-Remaining` | Days until expiry |

---

## 🎨 Menu Visibility & UI Control

### Plan Configuration for UI

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    MENU VISIBILITY CONTROL                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Plan Settings:                                                         │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  ShowLockedModulesInMenu: true/false                           │     │
│  │  ├── true:  Show modules not in plan as locked (marketing)     │     │
│  │  └── false: Hide modules not in plan completely                │     │
│  │                                                                 │     │
│  │  LockedItemStyle: "greyed_with_lock" | "hidden" |              │     │
│  │                   "upgrade_badge" | "separate_section"          │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
│  Entitlement Settings:                                                  │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  DisplayInMenu: true/false                                      │     │
│  │  ├── true:  Show in navigation menu                             │     │
│  │  └── false: Hide from menu (but API access may still work)     │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
│  Client UI Implementation:                                              │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  1. Fetch entitlements from API                                 │     │
│  │  2. Build menu from projects/modules with DisplayInMenu=true   │     │
│  │  3. Add locked_items based on LockedItemStyle:                 │     │
│  │     • greyed_with_lock: Show greyed with 🔒 icon               │     │
│  │     • upgrade_badge: Show with "Upgrade" badge                  │     │
│  │     • separate_section: "Available with Upgrade" section        │     │
│  │     • hidden: Don't show at all                                 │     │
│  │  4. Check AccessLevel before allowing actions                   │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 🔒 Security Considerations

### Token Security

| Aspect | Implementation |
|--------|----------------|
| Token Storage | Never store JWT in database, only SHA-256 hash |
| Token Expiry | Configurable, can match subscription or custom |
| Auto-Refresh | Token stays valid as long as subscription active |
| Revocation | Immediate via Status = Revoked |

### Device Security

| Aspect | Implementation |
|--------|----------------|
| Fingerprint | Client-generated device fingerprint |
| IP Tracking | Last known IP logged |
| Usage Analytics | API call count, last seen time |
| Force Disconnect | Admin can kick devices remotely |

### Data Security

| Aspect | Implementation |
|--------|----------------|
| ID Encryption | All IDs encrypted in API responses |
| HTTPS Only | All communication encrypted |
| Rate Limiting | Prevent abuse |
| Audit Logging | All actions logged |

---

## 📈 Flowcharts

### Client Application Startup Flow

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    CLIENT STARTUP SEQUENCE                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  App Launch                                                             │
│      │                                                                   │
│      ▼                                                                   │
│  ┌─────────────────┐                                                    │
│  │ Has saved token?│                                                    │
│  └────────┬────────┘                                                    │
│       NO  │  YES                                                        │
│           │    │                                                        │
│           ▼    ▼                                                        │
│  ┌──────────┐ ┌────────────────┐                                        │
│  │ Show     │ │ POST /validate │                                        │
│  │ Login UI │ └───────┬────────┘                                        │
│  └──────────┘         │                                                 │
│                       ▼                                                 │
│              ┌─────────────────┐                                        │
│              │ Token valid?    │                                        │
│              └────────┬────────┘                                        │
│                   NO  │  YES                                            │
│                       │    │                                            │
│                       ▼    ▼                                            │
│              ┌──────────┐ ┌──────────────────┐                          │
│              │ Clear    │ │ Device registered?│                         │
│              │ token,   │ └────────┬─────────┘                          │
│              │ re-login │      NO  │  YES                               │
│              └──────────┘          │    │                               │
│                                    ▼    ▼                               │
│                        ┌───────────────┐ ┌─────────────────┐            │
│                        │ POST /devices │ │ GET /entitlements│            │
│                        │ /register     │ └────────┬────────┘            │
│                        └───────┬───────┘          │                     │
│                                │                  │                     │
│                                ▼                  ▼                     │
│                        ┌──────────────┐  ┌─────────────────┐            │
│                        │ Approved?    │  │ Build Menu &    │            │
│                        │              │  │ Start App       │            │
│                        └──────┬───────┘  └─────────────────┘            │
│                           NO  │  YES                                    │
│                               │    │                                    │
│                               ▼    ▼                                    │
│                        ┌──────────────┐                                 │
│                        │ Show "Pending│                                 │
│                        │ Approval" UI │                                 │
│                        └──────────────┘                                 │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### Entitlement Check Flow

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    ENTITLEMENT CHECK FLOW                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  User clicks menu item / performs action                                │
│      │                                                                   │
│      ▼                                                                   │
│  ┌─────────────────────────────┐                                        │
│  │ Check Subscription.AccessMode│                                        │
│  └─────────────┬───────────────┘                                        │
│                │                                                         │
│      ┌─────────┴─────────┬─────────────┬──────────────┐                 │
│      ▼                   ▼             ▼              ▼                 │
│  ┌────────┐         ┌──────────┐  ┌──────────┐  ┌──────────┐           │
│  │ Full   │         │GracePeriod│  │ ReadOnly │  │ Blocked  │           │
│  └───┬────┘         └────┬─────┘  └────┬─────┘  └────┬─────┘           │
│      │                   │             │             │                  │
│      │                   │             │             ▼                  │
│      │                   │             │      ┌──────────────┐          │
│      │                   │             │      │ Show Upgrade │          │
│      │                   │             │      │ Required UI  │          │
│      │                   │             │      └──────────────┘          │
│      │                   │             │                                │
│      ▼                   ▼             ▼                                │
│  ┌─────────────────────────────────────────────────────────┐            │
│  │ Find entitlement for Project/Module                      │            │
│  └────────────────────────┬────────────────────────────────┘            │
│                           │                                             │
│               ┌───────────┴───────────┐                                 │
│               │ Check AccessLevel     │                                 │
│               │ & Permission Flags    │                                 │
│               └───────────┬───────────┘                                 │
│                           │                                             │
│      ┌────────┬───────────┼───────────┬────────┐                       │
│      ▼        ▼           ▼           ▼        ▼                       │
│   Create    Read       Update      Delete   Export                     │
│   CanCreate CanRead    CanUpdate   CanDelete CanExport                 │
│      │        │           │           │        │                       │
│      ▼        ▼           ▼           ▼        ▼                       │
│   Allow/   Allow/      Allow/      Allow/   Allow/                     │
│   Deny     Deny        Deny        Deny     Deny                       │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 📝 Quick Reference

### Key Tables

| Entity | Purpose |
|--------|---------|
| `Company` | Tenant/customer organization |
| `CompanyAdmin` | Client-side admin for device management |
| `SubscriptionPlan` | Product offering definition |
| `PlanEntitlement` | Access rights per plan |
| `Subscription` | Company's active plan instance |
| `OnlineClientToken` | API access token |
| `OnlineDeviceBinding` | Registered device |

### Key Enums

| Enum | Values |
|------|--------|
| `SubscriptionAccessMode` | None, Full, GracePeriod, ReadOnly, ExportOnly, Blocked |
| `EntitlementAccessLevel` | None, Full, ReadOnly, ExportOnly, Blocked |
| `DeviceAdmissionMode` | Open, AdminOnly, AutoWithQueue, HybridAutoAdmin |
| `ConcurrentAccessMode` | SingleDevice, LimitedConcurrent, Unlimited, TimeBasedUnlimited, TimeBasedLimited |

### Key Settings

| Setting | Description | Default |
|---------|-------------|---------|
| `MaxDevices` | Total devices allowed | 1 |
| `MaxConcurrentDevices` | Simultaneous active devices | 0 (use MaxDevices) |
| `GracePeriodDays` | Days after expiry before restriction | 7 |
| `ExportGraceDays` | Days to export after blocked | 30 |
| `DeviceHeartbeatTimeoutMinutes` | Inactivity threshold | 30 |

---

> **Next Document**: [OFFLINE-CLIENT-SYSTEM.md](./OFFLINE-CLIENT-SYSTEM.md) - Complete documentation for offline/air-gapped systems
