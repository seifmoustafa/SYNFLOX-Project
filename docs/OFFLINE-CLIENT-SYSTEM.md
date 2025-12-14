# 🔌 SYNFLOX Offline Client System - Complete Business Documentation

> **Version**: 2.0  
> **Last Updated**: December 2025  
> **Audience**: Developers, System Architects, Business Analysts

---

## 📋 Table of Contents

1. [System Overview](#system-overview)
2. [Offline vs Online Comparison](#offline-vs-online-comparison)
3. [License Key Architecture](#license-key-architecture)
4. [Complete Business Flow](#complete-business-flow)
5. [Device Binding & Machine Fingerprinting](#device-binding--machine-fingerprinting)
6. [License Validation](#license-validation)
7. [Clock Tampering Detection](#clock-tampering-detection)
8. [Company Admin Portal](#company-admin-portal)
9. [Flowcharts](#flowcharts)
10. [Security Deep Dive](#security-deep-dive)

---

## 🎯 System Overview

### What is the Offline System?

The SYNFLOX Offline System enables software licensing for **air-gapped environments** - systems with no internet connectivity. Unlike the Online system that fetches entitlements via API, the Offline system embeds ALL licensing information directly in an encrypted license key.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     OFFLINE LICENSE PHILOSOPHY                          │
│                                                                          │
│  License Key Contains EVERYTHING:                                       │
│  ┌──────────────────────────────────────────────────────────────┐       │
│  │  Company Information                                          │       │
│  │  ├── company_id                                              │       │
│  │  ├── company_name                                            │       │
│  │  │                                                            │       │
│  │  Subscription Details                                         │       │
│  │  ├── subscription_id                                         │       │
│  │  ├── plan_name                                               │       │
│  │  ├── start_date                                              │       │
│  │  ├── expiry_date                                             │       │
│  │  │                                                            │       │
│  │  Entitlements (FULL MATRIX)                                  │       │
│  │  ├── projects[] with access levels                           │       │
│  │  ├── modules[] with permissions                              │       │
│  │  ├── features[]                                              │       │
│  │  │                                                            │       │
│  │  Device Limits                                                │       │
│  │  ├── max_devices                                             │       │
│  │  ├── concurrent_access_mode                                   │       │
│  │  │                                                            │       │
│  │  Security                                                     │       │
│  │  ├── machine_binding_required                                │       │
│  │  ├── signature (HMAC-SHA256)                                 │       │
│  │  └── version                                                  │       │
│  └──────────────────────────────────────────────────────────────┘       │
│                                                                          │
│  Encrypted with: AES-256-CBC                                            │
│  Signed with: HMAC-SHA256                                               │
│                                                                          │
│  Key Difference from Online:                                            │
│  ❌ Cannot update without regenerating key                              │
│  ❌ Cannot revoke instantly (must distribute new key)                   │
│  ✓ Works completely offline                                             │
│  ✓ No API calls needed                                                  │
│  ✓ Tamper-proof (cryptographic signature)                               │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### Use Cases

| Scenario | Why Offline? |
|----------|--------------|
| Military/Government | Air-gapped networks, security requirements |
| Industrial/Manufacturing | Factory floor with no internet |
| Medical/Healthcare | Isolated medical devices |
| Remote Locations | Mining, oil rigs, ships |
| High-Security Finance | Disconnected trading systems |

---

## ⚖️ Offline vs Online Comparison

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    OFFLINE vs ONLINE COMPARISON                         │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Feature                │ ONLINE              │ OFFLINE                 │
│  ──────────────────────┼─────────────────────┼─────────────────────────│
│  Internet Required     │ YES (always)        │ NO (never)              │
│  Token/Key Contents    │ Identity only       │ Full entitlements       │
│  Entitlement Updates   │ Instant via API     │ Regenerate key          │
│  Revocation            │ Instant             │ Distribute new key      │
│  Device Tracking       │ Real-time           │ Local only              │
│  Clock Tampering       │ Server validates    │ Local detection         │
│  Key Size              │ ~500 bytes          │ ~2000+ bytes            │
│  Security Model        │ Server-authoritative│ Key-authoritative       │
│                                                                          │
│  When to Use ONLINE:                                                    │
│  ✓ Web applications                                                     │
│  ✓ Cloud-connected desktop apps                                         │
│  ✓ Frequent entitlement changes needed                                  │
│  ✓ Real-time usage monitoring required                                  │
│                                                                          │
│  When to Use OFFLINE:                                                   │
│  ✓ Air-gapped environments                                              │
│  ✓ Unreliable internet connectivity                                     │
│  ✓ High-security requirements                                           │
│  ✓ Compliance requirements (no external calls)                          │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 🔐 License Key Architecture

### Key Structure

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    LICENSE KEY STRUCTURE                                │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Raw License Data (JSON):                                               │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  {                                                              │     │
│  │    "version": 1,                                               │     │
│  │    "company": {                                                │     │
│  │      "id": "guid",                                             │     │
│  │      "name": "Acme Corp",                                      │     │
│  │      "timezone": "America/New_York"                            │     │
│  │    },                                                           │     │
│  │    "subscription": {                                           │     │
│  │      "id": "guid",                                             │     │
│  │      "plan_name": "Enterprise",                                │     │
│  │      "start_date": "2025-01-01T00:00:00Z",                     │     │
│  │      "expiry_date": "2025-12-31T23:59:59Z",                    │     │
│  │      "is_lifetime": false,                                     │     │
│  │      "is_trial": false                                         │     │
│  │    },                                                           │     │
│  │    "entitlements": {                                           │     │
│  │      "access_mode": "Full",                                    │     │
│  │      "projects": [...],                                        │     │
│  │      "modules": [...],                                         │     │
│  │      "features": [...],                                        │     │
│  │      "custom_features": ["24/7 Support", "SLA 1h"]             │     │
│  │    },                                                           │     │
│  │    "devices": {                                                │     │
│  │      "max_devices": 50,                                        │     │
│  │      "concurrent_mode": "LimitedConcurrent",                   │     │
│  │      "max_concurrent": 25,                                     │     │
│  │      "require_machine_binding": true,                          │     │
│  │      "hardware_change_tolerance": 2                            │     │
│  │    },                                                           │     │
│  │    "issued_at": "2025-01-01T00:00:00Z",                        │     │
│  │    "signature": "base64_hmac_sha256_signature"                 │     │
│  │  }                                                              │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
│  Encryption Process:                                                    │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  1. JSON → UTF-8 bytes                                         │     │
│  │  2. Compress (GZip)                                            │     │
│  │  3. Sign (HMAC-SHA256 with secret key)                         │     │
│  │  4. Encrypt (AES-256-CBC with derived key)                     │     │
│  │  5. Base64 encode                                              │     │
│  │  6. Add version prefix: "SYN1-{base64_data}"                   │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
│  Final Key Format:                                                      │
│  SYN1-eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjb21wYW55X2lkI...        │
│  └──┬─┘└────────────────────────────────────────────────────────┘       │
│   Version                    Encrypted Payload                          │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### Key Storage in Database

The license key is stored in the `Subscription` entity:

```csharp
public class Subscription
{
    // ... other fields ...
    
    /// <summary>
    /// Encrypted offline license key for this specific subscription
    /// Contains: CompanyId, PlanId, ExpiryDate, Features, Modules, Signature
    /// Used by client applications for offline validation
    /// </summary>
    [StringLength(2000)]
    public string? OfflineLicenseKey { get; set; }

    /// <summary>
    /// When the offline license key was generated (UTC)
    /// </summary>
    public DateTime? LicenseKeyGeneratedAt { get; set; }

    /// <summary>
    /// Version of the license key format for future compatibility
    /// </summary>
    public int LicenseKeyVersion { get; set; } = 1;
}
```

---

## 📊 Complete Business Flow

### Phase 1: Setup (Same as Online)

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    STEP 1-5: SAME AS ONLINE SYSTEM                      │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  1. Create Company                                                      │
│  2. Create CompanyAdmin (for device management)                         │
│  3. Create SubscriptionPlan with Entitlements                          │
│  4. Configure PlanEntitlements (Projects/Modules/Permissions)          │
│  5. Create Subscription (Company + Plan + Time Window)                 │
│                                                                          │
│  These steps are identical for both Online and Offline systems.         │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### Phase 2: License Key Generation

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    STEP 6: GENERATE OFFLINE LICENSE KEY                 │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Who Can Generate:                                                      │
│  • SYNFLOX Admin (always)                                               │
│  • CompanyAdmin (if CanGenerateLicenses = true)                        │
│                                                                          │
│  Generation Process:                                                    │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  1. Load Subscription + Company + Plan + Entitlements          │     │
│  │  2. Build license payload (JSON structure above)                │     │
│  │  3. Sign with HMAC-SHA256 (server secret key)                  │     │
│  │  4. Encrypt with AES-256-CBC (derived from master key)         │     │
│  │  5. Base64 encode with version prefix                          │     │
│  │  6. Store in Subscription.OfflineLicenseKey                    │     │
│  │  7. Update LicenseKeyGeneratedAt timestamp                     │     │
│  │  8. Return key to admin for distribution                       │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
│  Key Distribution Options:                                              │
│  • Download as .lic file                                                │
│  • Copy to clipboard                                                    │
│  • Email to company contact                                             │
│  • QR code for mobile scanning                                          │
│                                                                          │
│  ⚠️ IMPORTANT: Key must be regenerated when:                           │
│  • Plan changes (upgrade/downgrade)                                     │
│  • Entitlements modified                                                │
│  • Expiry date extended                                                 │
│  • Device limits changed                                                │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### Phase 3: Client Application Installation

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    STEP 7: CLIENT INSTALLATION & ACTIVATION             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  First-Time Activation:                                                 │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  1. User installs client application                           │     │
│  │  2. App prompts for license key input                          │     │
│  │  3. User pastes/loads license key                              │     │
│  │  4. App validates key:                                          │     │
│  │     a. Decrypt with embedded public key                        │     │
│  │     b. Verify HMAC signature                                    │     │
│  │     c. Check expiry date                                        │     │
│  │     d. Validate version compatibility                           │     │
│  │  5. If machine binding required:                                │     │
│  │     a. Generate machine fingerprint                             │     │
│  │     b. Compare with bound devices (if any)                     │     │
│  │     c. Register new device if allowed                          │     │
│  │  6. Store key securely in local encrypted storage              │     │
│  │  7. App launches with entitled features                        │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
│  Subsequent Launches:                                                   │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  1. Load key from local storage                                 │     │
│  │  2. Validate key (decrypt, verify, check expiry)               │     │
│  │  3. Check machine fingerprint (if binding required)            │     │
│  │  4. Check for clock tampering                                   │     │
│  │  5. Launch app or show error                                    │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 🖥️ Device Binding & Machine Fingerprinting

### Machine Fingerprint Generation

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    MACHINE FINGERPRINT COMPONENTS                       │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Hardware Components Used:                                              │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  Component        │ How Obtained              │ Stability     │     │
│  │  ─────────────────┼───────────────────────────┼───────────────│     │
│  │  CPU ID           │ WMI/sysctl                │ Very High     │     │
│  │  Motherboard SN   │ WMI/DMI                   │ Very High     │     │
│  │  Primary Disk SN  │ WMI/ioreg                 │ Medium*       │     │
│  │  Primary MAC      │ Network interfaces        │ Medium**      │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
│  * Disk can be replaced                                                 │
│  ** Network adapter can be changed                                      │
│                                                                          │
│  Fingerprint Algorithm:                                                 │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  fingerprint = SHA256(                                          │     │
│  │    CPU_ID + "|" +                                               │     │
│  │    Motherboard_Serial + "|" +                                   │     │
│  │    Disk_Serial + "|" +                                          │     │
│  │    MAC_Address                                                   │     │
│  │  )                                                               │     │
│  │                                                                  │     │
│  │  Result: 64-character hex string                                │     │
│  │  Example: "a1b2c3d4e5f6..."                                     │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### Hardware Change Tolerance

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    HARDWARE CHANGE TOLERANCE                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Problem: Users replace hardware (new disk, new RAM, new NIC)           │
│  Solution: Allow partial fingerprint changes                            │
│                                                                          │
│  Plan Configuration:                                                    │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  HardwareChangeTolerance: 2                                     │     │
│  │  • 0 = Any change invalidates activation                        │     │
│  │  • 1 = Allow 1 component change                                 │     │
│  │  • 2 = Allow 2 component changes (recommended)                  │     │
│  │  • 3 = Allow 3 component changes (lenient)                      │     │
│  │  • 4 = Allow all components to change (only CPU must match)    │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
│  Change Detection Logic:                                                │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  On each validation:                                            │     │
│  │  1. Get current fingerprint components                          │     │
│  │  2. Compare with stored activation:                             │     │
│  │     • CPU matches? ✓ or ✗                                       │     │
│  │     • Motherboard matches? ✓ or ✗                               │     │
│  │     • Disk matches? ✓ or ✗                                      │     │
│  │     • MAC matches? ✓ or ✗                                       │     │
│  │  3. Count mismatches                                            │     │
│  │  4. If mismatches <= HardwareChangeTolerance:                   │     │
│  │     • Update stored fingerprint                                 │     │
│  │     • Increment HardwareChangeCount                             │     │
│  │     • Log change                                                 │     │
│  │     • Allow access                                              │     │
│  │  5. If mismatches > tolerance:                                  │     │
│  │     • Deny access                                                │     │
│  │     • Require re-activation                                     │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
│  Example:                                                               │
│  • Original: CPU=A, MB=B, Disk=C, MAC=D                                │
│  • Current:  CPU=A, MB=B, Disk=E, MAC=F (2 changes)                    │
│  • Tolerance=2: ✓ Allowed (update stored fingerprint)                  │
│  • Tolerance=1: ✗ Denied (exceeds tolerance)                           │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### LicenseActivation Entity

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    LICENSE ACTIVATION ENTITY                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Tracks each device activation:                                         │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  LicenseActivation                                              │     │
│  │  │                                                              │     │
│  │  │ Identity:                                                    │     │
│  │  ├── SubscriptionId                                            │     │
│  │  ├── CompanyId (denormalized)                                   │     │
│  │  ├── MachineHash (SHA256 fingerprint)                          │     │
│  │  │                                                              │     │
│  │  │ Device Info:                                                 │     │
│  │  ├── DeviceName: "John's Laptop"                               │     │
│  │  ├── OperatingSystem: "Windows 11 Pro 22H2"                    │     │
│  │  ├── CpuId: "Intel i9-13900K"                                  │     │
│  │  ├── MotherboardSerial: "ABC123"                               │     │
│  │  ├── DiskSerial: "DEF456"                                      │     │
│  │  ├── MacAddress: "00:1A:2B:3C:4D:5E"                           │     │
│  │  │                                                              │     │
│  │  │ Status:                                                      │     │
│  │  ├── IsActive: true                                            │     │
│  │  ├── ActivatedAtUtc: 2025-01-01                                │     │
│  │  ├── LastSeenAtUtc: 2025-01-15                                 │     │
│  │  ├── LastIpAddress: "192.168.1.100"                            │     │
│  │  ├── ValidationCount: 150                                       │     │
│  │  │                                                              │     │
│  │  │ Hardware Changes:                                            │     │
│  │  ├── HardwareChangeCount: 1                                    │     │
│  │  ├── LastHardwareChangeAtUtc: 2025-01-10                       │     │
│  │  │                                                              │     │
│  │  │ Concurrent Access:                                           │     │
│  │  ├── IsCurrentlyActive: true                                   │     │
│  │  ├── CurrentSessionId: "sess-123"                              │     │
│  │  ├── SessionStartedAtUtc: 2025-01-15 09:00                     │     │
│  │  │                                                              │     │
│  │  │ Admin Device:                                                │     │
│  │  ├── IsAdminDevice: false                                      │     │
│  │  └── AdminId: null                                             │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
│  ⚠️ IMPORTANT: Admin devices are NEVER counted in device limits!       │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## ✅ License Validation

### Validation Steps

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    LICENSE VALIDATION PROCESS                           │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Step 1: Decrypt Key                                                    │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  Input: "SYN1-eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."         │     │
│  │  1. Extract version prefix ("SYN1")                             │     │
│  │  2. Base64 decode payload                                       │     │
│  │  3. Decrypt with AES-256-CBC                                    │     │
│  │  4. Decompress (GZip)                                           │     │
│  │  5. Parse JSON                                                  │     │
│  │  Output: License data object                                    │     │
│  │                                                                  │     │
│  │  Possible Errors:                                               │     │
│  │  • InvalidFormat: Bad version prefix                            │     │
│  │  • DecryptionFailed: Wrong key or corrupted data               │     │
│  │  • MalformedData: Invalid JSON structure                        │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
│  Step 2: Verify Signature                                               │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  1. Extract signature from license data                         │     │
│  │  2. Compute HMAC-SHA256 of data (without signature)            │     │
│  │  3. Compare computed vs stored signature                        │     │
│  │                                                                  │     │
│  │  Possible Errors:                                               │     │
│  │  • SignatureInvalid: Tampering detected!                        │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
│  Step 3: Check Expiry                                                   │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  1. Get current system time                                     │     │
│  │  2. Compare with expiry_date from license                       │     │
│  │  3. Check for clock tampering (see next section)                │     │
│  │                                                                  │     │
│  │  Possible Errors:                                               │     │
│  │  • Expired: License has expired                                 │     │
│  │  • ClockTampering: System clock manipulation detected           │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
│  Step 4: Validate Machine Binding                                       │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  If machine_binding_required = true:                            │     │
│  │  1. Generate current machine fingerprint                        │     │
│  │  2. Load stored activation for this subscription                │     │
│  │  3. Compare fingerprints (with tolerance)                       │     │
│  │  4. Update or reject based on tolerance                         │     │
│  │                                                                  │     │
│  │  Possible Errors:                                               │     │
│  │  • MachineNotBound: Device not activated                        │     │
│  │  • HardwareChanged: Too many hardware changes                   │     │
│  │  • DeviceLimitReached: Max devices already bound                │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
│  Step 5: Check Concurrent Access                                        │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  If ConcurrentAccessMode != Unlimited:                          │     │
│  │  1. Count currently active devices (recent heartbeat)          │     │
│  │  2. Compare with MaxConcurrentDevices                           │     │
│  │  3. Allow, queue, or reject new session                         │     │
│  │                                                                  │     │
│  │  Note: For offline, this is LOCAL enforcement only!            │     │
│  │  Each device tracks its own state without server sync.          │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
│  Final Result:                                                          │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  ValidationResult {                                             │     │
│  │    Status: Valid | Invalid | Expired | HardwareChanged | ...   │     │
│  │    Entitlements: { projects, modules, features }                │     │
│  │    ExpiryDate: DateTime                                         │     │
│  │    DaysRemaining: int                                           │     │
│  │    ErrorMessage: string?                                        │     │
│  │  }                                                               │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### Validation Status Codes

| Status | Code | Description |
|--------|------|-------------|
| `Valid` | 0 | License is valid, access granted |
| `InvalidFormat` | 1 | License key format is wrong |
| `DecryptionFailed` | 2 | Could not decrypt (wrong key/corrupted) |
| `SignatureInvalid` | 3 | Tampering detected |
| `Expired` | 4 | License has expired |
| `NotYetValid` | 5 | Start date is in the future |
| `ClockTampering` | 6 | System clock manipulation detected |
| `MachineNotBound` | 7 | Device not activated |
| `HardwareChanged` | 8 | Too many hardware changes |
| `DeviceLimitReached` | 9 | Max devices already bound |
| `ConcurrentLimitReached` | 10 | Too many active sessions |
| `Suspended` | 11 | Subscription suspended |
| `Revoked` | 12 | License explicitly revoked |

---

## ⏰ Clock Tampering Detection

### Why It Matters

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    CLOCK TAMPERING THREAT                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Without clock tampering detection:                                     │
│  • User sets system clock to Jan 1, 2020                               │
│  • License with expiry Dec 31, 2025 never expires                      │
│  • User gets perpetual access without paying                            │
│                                                                          │
│  Solution: Multiple timestamp tracking                                  │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### Detection Mechanisms

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    CLOCK TAMPERING DETECTION                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Mechanism 1: Last Known Good Time                                      │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  On each validation:                                            │     │
│  │  1. Store current time locally (encrypted)                      │     │
│  │  2. On next validation, compare:                                │     │
│  │     • If current_time < stored_time by more than 1 hour:       │     │
│  │       → Clock was set backwards! 🚨                             │     │
│  │  3. Allow grace period for timezone changes                     │     │
│  │                                                                  │     │
│  │  Storage: %APPDATA%\SYNFLOX\.timestamp (encrypted)              │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
│  Mechanism 2: License Issue Time Check                                  │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  If current_time < license.issued_at:                           │     │
│  │  → Clock is before license was even created! 🚨                 │     │
│  │                                                                  │     │
│  │  This catches obvious tampering (setting clock to 2010)         │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
│  Mechanism 3: Build Time Anchor                                         │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  Embed build timestamp in client application                    │     │
│  │  If current_time < build_time:                                  │     │
│  │  → Clock is before app was built! 🚨                            │     │
│  │                                                                  │     │
│  │  This is a hard anchor that can't be bypassed without          │     │
│  │  modifying the application binary.                              │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
│  Mechanism 4: File System Timestamps (Advanced)                         │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  Check timestamps of:                                           │     │
│  │  • System files (ntdll.dll, kernel32.dll)                      │     │
│  │  • Recent Windows Update folders                                │     │
│  │  • Application installation folder                              │     │
│  │                                                                  │     │
│  │  If current_time < recent_system_file_time:                    │     │
│  │  → Suspicious! System files are newer than "now" 🚨            │     │
│  │                                                                  │     │
│  │  Note: Can be bypassed by sophisticated users                   │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
│  Response to Tampering:                                                 │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  Options (configurable per plan):                               │     │
│  │  • Block: Deny access completely                                │     │
│  │  • Warn: Show warning, allow access                             │     │
│  │  • Grace: Allow X hours of grace time                           │     │
│  │  • Log: Log incident for later audit                            │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 👨‍💼 Company Admin Portal

### Admin Capabilities for Offline Systems

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    COMPANY ADMIN PORTAL - OFFLINE FEATURES              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Dashboard View (when online):                                          │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  • Active subscriptions and their status                        │     │
│  │  • License key generation/regeneration                          │     │
│  │  • Device activations list                                       │     │
│  │  • Hardware change events                                        │     │
│  │  • Replacement requests (pending/approved/denied)               │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
│  Device Management:                                                     │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  View Devices:                                                  │     │
│  │  ├── Device Name                                                │     │
│  │  ├── Machine Hash (partial, for security)                      │     │
│  │  ├── OS Info                                                    │     │
│  │  ├── Activated At                                               │     │
│  │  ├── Last Seen                                                  │     │
│  │  ├── Hardware Changes                                           │     │
│  │  └── Status (Active/Inactive/Deactivated)                      │     │
│  │                                                                  │     │
│  │  Actions:                                                       │     │
│  │  ├── Deactivate Device (free up slot)                          │     │
│  │  ├── Approve Replacement Request                                │     │
│  │  ├── Deny Replacement Request                                   │     │
│  │  └── View Device Details                                        │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
│  License Key Operations:                                                │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  Generate New Key:                                              │     │
│  │  • Required after plan changes                                  │     │
│  │  • Required after entitlement updates                           │     │
│  │  • Required after expiry extension                              │     │
│  │                                                                  │     │
│  │  Download Key:                                                  │     │
│  │  • .lic file format                                             │     │
│  │  • Copy to clipboard                                            │     │
│  │  • QR code display                                              │     │
│  │                                                                  │     │
│  │  View Key History:                                              │     │
│  │  • Previous keys generated                                      │     │
│  │  • Who generated                                                 │     │
│  │  • When generated                                               │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
│  Permission Flags (set by SYNFLOX admin):                              │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  CanManageDevices: true         → Bind/unbind devices          │     │
│  │  CanViewSubscriptions: true     → View subscription info       │     │
│  │  CanApproveReplacements: true   → Handle replacement requests  │     │
│  │  CanGenerateLicenses: true      → Generate offline keys        │     │
│  │  CanViewUsageReports: true      → View analytics               │     │
│  │  CanModifySessionSettings: true → Change own settings          │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### Device Replacement Flow

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    DEVICE REPLACEMENT REQUEST FLOW                      │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  When device limit reached and user tries to activate new device:       │
│                                                                          │
│  Client App:                                                            │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  1. Validate license → DeviceLimitReached                       │     │
│  │  2. Show message: "Device limit reached"                        │     │
│  │  3. Offer options:                                               │     │
│  │     a. Contact admin (show admin email)                         │     │
│  │     b. Submit replacement request (if online)                   │     │
│  │     c. Enter admin token (offline replacement)                  │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
│  DeviceReplacementPolicy Options:                                       │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  AdminApproval (default, most secure):                          │     │
│  │  • Request goes to pending queue                                │     │
│  │  • Admin reviews in portal                                      │     │
│  │  • Admin approves/denies with reason                            │     │
│  │  • If approved, old device deactivated, new device activated   │     │
│  │                                                                  │     │
│  │  AutoReplaceOldest:                                             │     │
│  │  • System automatically deactivates oldest device               │     │
│  │  • New device activated immediately                             │     │
│  │  • Notification sent to admin                                   │     │
│  │                                                                  │     │
│  │  AutoReplaceLeastActive:                                        │     │
│  │  • System deactivates device with longest inactivity           │     │
│  │  • Uses LastSeenAtUtc to determine                              │     │
│  │  • New device activated immediately                             │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
│  Offline Replacement Token:                                             │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  For air-gapped systems that can't submit online requests:     │     │
│  │                                                                  │     │
│  │  1. User calls/emails admin with device info                    │     │
│  │  2. Admin generates OfflineLicenseAdminToken via portal         │     │
│  │  3. Token contains:                                              │     │
│  │     • Old device hash to deactivate                             │     │
│  │     • New device hash to activate                               │     │
│  │     • Expiry (24 hours typical)                                 │     │
│  │     • Signature                                                  │     │
│  │  4. Admin provides token to user (phone, email, etc.)           │     │
│  │  5. User enters token in client app                             │     │
│  │  6. Client validates and performs swap locally                  │     │
│  │  7. Next time online, sync the change to server                 │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 📈 Flowcharts

### Complete Offline License Flow

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    COMPLETE OFFLINE LICENSE FLOW                        │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  SYNFLOX ADMIN SIDE                          CLIENT SIDE                │
│  ════════════════════                        ═══════════                │
│                                                                          │
│  Create Company                                                         │
│       │                                                                  │
│       ▼                                                                  │
│  Create Subscription                                                    │
│       │                                                                  │
│       ▼                                                                  │
│  Generate License Key ─────────────────────► Receive Key               │
│       │                                           │                     │
│       │                                           ▼                     │
│  Store in Subscription                       Install App               │
│       │                                           │                     │
│       │                                           ▼                     │
│       │                                      Enter License Key          │
│       │                                           │                     │
│       │                                           ▼                     │
│       │                                  ┌─────────────────┐            │
│       │                                  │ Decrypt & Verify │            │
│       │                                  └────────┬────────┘            │
│       │                                           │                     │
│       │                                      ┌────┴────┐               │
│       │                                      │ Valid?  │               │
│       │                                      └────┬────┘               │
│       │                                     NO   │  YES                │
│       │                                          │    │                │
│       │                                          ▼    ▼                │
│       │                                  ┌──────────┐ ┌──────────────┐ │
│       │                                  │Show Error│ │Bind Machine  │ │
│       │                                  └──────────┘ └──────┬───────┘ │
│       │                                                      │         │
│       │                                                      ▼         │
│       │                                              ┌──────────────┐  │
│       │                                              │ Store Local  │  │
│       │                                              │ Activation   │  │
│       │                                              └──────┬───────┘  │
│       │                                                      │         │
│       │                                                      ▼         │
│       │                                              ┌──────────────┐  │
│       │                                              │  Load        │  │
│       │                                              │  Entitlements│  │
│       │                                              └──────┬───────┘  │
│       │                                                      │         │
│       │                                                      ▼         │
│       │                                              ┌──────────────┐  │
│       │                                              │  Build Menu  │  │
│       │                                              │  Start App   │  │
│       │                                              └──────────────┘  │
│       │                                                                 │
│  [Plan Change / Extension]                                             │
│       │                                                                 │
│       ▼                                                                 │
│  Regenerate License Key ────────────────────► Update Key in App       │
│                                                      │                 │
│                                                      ▼                 │
│                                              Re-validate & Continue   │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### Validation Decision Tree

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    VALIDATION DECISION TREE                             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Start Validation                                                       │
│       │                                                                  │
│       ▼                                                                  │
│  ┌────────────────────┐                                                 │
│  │ Can decrypt key?   │──NO──► InvalidFormat / DecryptionFailed        │
│  └─────────┬──────────┘                                                 │
│            │ YES                                                        │
│            ▼                                                            │
│  ┌────────────────────┐                                                 │
│  │ Signature valid?   │──NO──► SignatureInvalid (TAMPERING!)           │
│  └─────────┬──────────┘                                                 │
│            │ YES                                                        │
│            ▼                                                            │
│  ┌────────────────────┐                                                 │
│  │ Clock tampered?    │──YES─► ClockTampering                          │
│  └─────────┬──────────┘                                                 │
│            │ NO                                                         │
│            ▼                                                            │
│  ┌────────────────────┐                                                 │
│  │ Before start date? │──YES─► NotYetValid                             │
│  └─────────┬──────────┘                                                 │
│            │ NO                                                         │
│            ▼                                                            │
│  ┌────────────────────┐                                                 │
│  │ After expiry date? │──YES─► Expired                                 │
│  └─────────┬──────────┘                                                 │
│            │ NO                                                         │
│            ▼                                                            │
│  ┌────────────────────┐                                                 │
│  │ Machine binding?   │──NO──► Skip binding check                      │
│  └─────────┬──────────┘                                                 │
│            │ YES                                                        │
│            ▼                                                            │
│  ┌────────────────────┐                                                 │
│  │ Device activated?  │──NO──┬─► ┌────────────────────┐                │
│  └─────────┬──────────┘      │   │ Device limit OK?   │                │
│            │ YES             │   └─────────┬──────────┘                │
│            │                 │         YES │  NO                       │
│            │                 │             │    │                      │
│            │                 │             ▼    ▼                      │
│            │                 │         Activate  DeviceLimitReached    │
│            │                 │         Device                          │
│            │                 │             │                           │
│            │                 └─────────────┘                           │
│            ▼                                                            │
│  ┌────────────────────┐                                                 │
│  │ HW changes OK?     │──NO──► HardwareChanged                         │
│  └─────────┬──────────┘                                                 │
│            │ YES                                                        │
│            ▼                                                            │
│  ┌────────────────────┐                                                 │
│  │ Concurrent OK?     │──NO──► ConcurrentLimitReached                  │
│  └─────────┬──────────┘                                                 │
│            │ YES                                                        │
│            ▼                                                            │
│       ✅ VALID                                                         │
│       Return entitlements                                               │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 🔒 Security Deep Dive

### Encryption Details

| Component | Algorithm | Key Size | Purpose |
|-----------|-----------|----------|---------|
| Encryption | AES-256-CBC | 256 bits | Data confidentiality |
| Signature | HMAC-SHA256 | 256 bits | Data integrity |
| Key Derivation | PBKDF2 | N/A | Derive encryption key from master |
| Fingerprint | SHA-256 | 256 bits | Machine identification |

### Key Management

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    KEY MANAGEMENT ARCHITECTURE                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Server Side (SYNFLOX):                                                 │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  Master Encryption Key                                          │     │
│  │  ├── Stored in: Azure Key Vault / AWS KMS / HSM                │     │
│  │  ├── Access: Admin API only                                     │     │
│  │  └── Rotation: Yearly (with re-generation of all keys)         │     │
│  │                                                                  │     │
│  │  Signing Key (HMAC)                                              │     │
│  │  ├── Stored in: Same as master key                              │     │
│  │  └── Used for: License signature                                │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
│  Client Side (Application):                                             │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  Public Verification Key                                        │     │
│  │  ├── Embedded in: Application binary                            │     │
│  │  ├── Used for: Decryption, signature verification               │     │
│  │  └── Cannot generate new licenses (one-way)                    │     │
│  │                                                                  │     │
│  │  Local Storage Encryption                                       │     │
│  │  ├── Key derived from: Machine fingerprint + app secret        │     │
│  │  └── Used for: Local license cache, timestamp storage          │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### Attack Vectors & Mitigations

| Attack | Description | Mitigation |
|--------|-------------|------------|
| Key Sharing | Users share license key | Machine binding, device limits |
| Key Modification | Editing license to extend expiry | HMAC signature verification |
| Clock Tampering | Setting system clock back | Multiple timestamp checks |
| Binary Patching | Modifying validation code | Code obfuscation, integrity checks |
| Memory Editing | Runtime license modification | Memory encryption, anti-debug |
| Virtual Machine Cloning | Clone VM with activated license | VM detection, unique fingerprint |

---

## 📝 Quick Reference

### Key Entities

| Entity | Purpose |
|--------|---------|
| `Subscription.OfflineLicenseKey` | The encrypted license key |
| `Subscription.LicenseKeyGeneratedAt` | When key was generated |
| `LicenseActivation` | Device activation record |
| `DeviceReplacementRequest` | Pending device swap requests |
| `OfflineLicenseAdminToken` | Token for offline device swaps |

### Key Settings (from Plan)

| Setting | Description | Default |
|---------|-------------|---------|
| `MaxDevices` | Total devices allowed | 1 |
| `RequireMachineBinding` | Bind to hardware | false |
| `HardwareChangeTolerance` | Allowed component changes | 2 |
| `DeviceReplacementPolicy` | How to handle full capacity | AdminApproval |
| `ConcurrentAccessMode` | Simultaneous access control | Unlimited |

### Validation Status Codes

| Code | Status | Action |
|------|--------|--------|
| 0 | Valid | Grant access |
| 1-3 | Format/Decrypt/Signature | Show error, request new key |
| 4 | Expired | Show renewal prompt |
| 5 | NotYetValid | Show start date |
| 6 | ClockTampering | Block, show warning |
| 7-9 | Device issues | Show device management options |
| 10 | ConcurrentLimit | Queue or kick oldest |
| 11-12 | Suspended/Revoked | Contact SYNFLOX |

---

> **Related Document**: [ONLINE-CLIENT-SYSTEM.md](./ONLINE-CLIENT-SYSTEM.md) - Documentation for online/connected systems
