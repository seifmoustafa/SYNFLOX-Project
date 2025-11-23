---
description: SYNFLOX Complete Implementation Roadmap - Dependency-Based Workflow
auto_execution_mode: 3
---

# 🚀 SYNFLOX IMPLEMENTATION ROADMAP

**Complete dependency-based workflow for remaining features**

---

## 📊 ARCHITECTURE & DEPENDENCIES

### Entity Relationship Structure:
```
FOUNDATION LAYER (Independent)
├── Companies (DONE)
├── Projects (Product definitions) - COMPLETE
└── Modules (Feature definitions) - COMPLETE
        ↓
PRODUCT CATALOG LAYER - COMPLETE
└── Plans (Depends on: Projects + Modules) - COMPLETE
    - Plan → PlanProjects (many-to-many)
    - Plan → PlanModules (many-to-many)
    - Plan → PlanPrices (one-to-many)
        ↓
CUSTOMER INSTANCES LAYER 
└── Subscriptions (Depends on: Company + Plans)
    - Lifecycle: trial, active, suspended, expired
    - Operations: renew, upgrade, cancel, extend
    - Tracks usage and analytics
        ↓
OFFLINE ACCESS LAYER
└── 🟡 License Keys (Depends on: Subscriptions)
    - Generate/regenerate encrypted keys (AES-256)
    - Validate keys (client apps)
    - Revoke keys
```

---

## 🎯 PHASE 1: FOUNDATION - PROJECTS & MODULES

**Priority:** ⭐⭐⭐⭐⭐ CRITICAL (Everything depends on this)
**Time:** 2-3 days
**Dependencies:** None (Independent)

### Step 1.1: Build Projects Module
**What:** Product/application definitions (e.g., "CRM System", "ERP Suite")

**Backend Status:** ✅ Complete (ProjectsController, IProjectService)

**Frontend Tasks:**
1. Create `domain/models/project.model.ts`:
   - Project class with validation
   - CreateProjectRequest
   - UpdateProjectRequest
   - ProjectsResponse interface

2. Create `domain/mappers/project.mapper.ts`:
   - fromJson, toJson conversions
   - handleApiResponse for pagination

3. Create `services/project.service.ts`:
   - IProjectService interface
   - ProjectService implementation
   - CRUD operations

4. Add API endpoints to `config/api-endpoints.ts`:
   - PROJECTS_GET_ALL
   - PROJECTS_GET_BY_ID
   - PROJECTS_CREATE
   - PROJECTS_UPDATE
   - PROJECTS_DELETE

5. Register service in `providers/service-provider.tsx`

6. Create `viewmodels/project-viewmodel.tsx`:
   - useGenericCrudViewModel integration
   - Column definitions
   - Form fields (name, description, version, isActive)
   - Action handlers

7. Create `views/project-view.tsx` and `app/projects/page.tsx`

8. Add translations to `locales/en.ts` and `locales/ar.ts`

**UI Features:**
- List projects with pagination & search
- Create/Edit/Delete projects
- Status badges (Active/Inactive)
- View project details
- Link to associated modules

---

### Step 1.2: Build Modules Module
**What:** Feature definitions within projects (e.g., "Reports", "Analytics")

**Backend Status:** ✅ Complete (ModulesController, IModuleService)

**Frontend Tasks:**
1. Create `domain/models/module.model.ts`:
   - Module class with validation
   - CreateModuleRequest
   - UpdateModuleRequest
   - ModulesResponse interface

2. Create `domain/mappers/module.mapper.ts`

3. Create `services/module.service.ts`:
   - CRUD operations
   - Get modules by project

4. Add API endpoints to config

5. Register service in provider

6. Create `viewmodels/module-viewmodel.tsx`:
   - Column definitions
   - Form fields (name, description, key, isActive)
   - Action handlers

7. Create `views/module-view.tsx` and `app/modules/page.tsx`

8. Add translations

**UI Features:**
- List modules with pagination & search
- Create/Edit/Delete modules
- Status badges
- View module details
- Link to associated projects

**Child Page:** Create `/projects/[id]/modules/page.tsx` to view modules for a specific project

---

## 🎯 PHASE 2: PRODUCT CATALOG - SUBSCRIPTION PLANS

**Priority:** ⭐⭐⭐⭐⭐ HIGH (Core business model)
**Time:** 4-5 days
**Dependencies:** ✅ Projects, ✅ Modules

### Step 2.1: Build Plans Module (Basic)
**What:** Commercial offerings (Basic, Pro, Enterprise, Lifetime)

**Backend Status:** ✅ Complete (PlansController, ISubscriptionPlanService)

**Frontend Tasks:**
1. Create `domain/models/subscription-plan.model.ts`:
   - SubscriptionPlan class
   - CreatePlanRequest
   - UpdatePlanRequest
   - PlanDurationType enum (Monthly, Yearly, Lifetime, etc.)
   - UpgradePolicy enum
   - PlansResponse interface

2. Create `domain/mappers/subscription-plan.mapper.ts`

3. Create `services/subscription-plan.service.ts`:
   - CRUD operations
   - Get plan details (with projects/modules)

4. Add API endpoints

5. Register service

6. Create `viewmodels/subscription-plan-viewmodel.tsx`:
   - Column definitions (name, duration, price, status)
   - Form fields (basic info)
   - Action handlers

7. Create multi-step plan creation wizard

8. Create `views/subscription-plan-view.tsx`

9. Create `app/plans/page.tsx`

10. Add translations (50+ keys)

**UI Features (Complex):**
- List plans with filters (duration type, status)
- Status badges (Active/Inactive)
- Price display

---

### Step 2.2: Build Advanced Plan Features

**Multi-Step Plan Creation Wizard:**
1. **Step 1: Basic Info**
   - Name, description
   - Duration type (dropdown)
   - Is active checkbox

2. **Step 2: Pricing**
   - Add multiple price tiers
   - Currency, amount, billing cycle
   - Table view of all prices

3. **Step 3: Projects Selection**
   - Multi-select dropdown/checkboxes
   - Show selected projects

4. **Step 4: Modules Selection**
   - Group by project
   - Multi-select modules per project
   - Visual representation

5. **Step 5: Features**
   - Custom features list (add/remove)
   - Text input for each feature

6. **Step 6: Settings**
   - Allow trial (yes/no)
   - Trial duration (days)
   - Auto-renew (yes/no)
   - Upgrade policy (dropdown)
   - Grace period (days)

**Plan Details Page (`/plans/[id]/page.tsx`):**
- Overview section (name, description, duration, status)
- Pricing tiers table
- Included projects (cards with icons)
- Included modules (grouped by project)
- Custom features list
- Settings summary
- Actions (Edit, Delete, Duplicate)

**Child Pages:**
- `/plans/[id]/subscriptions/page.tsx` - All subscriptions using this plan
- `/plans/[id]/analytics/page.tsx` - Usage analytics

---

## 🎯 PHASE 3: CUSTOMER INSTANCES - SUBSCRIPTIONS

**Priority:** ⭐⭐⭐⭐⭐ CRITICAL (Core operations)
**Time:** 5-6 days
**Dependencies:** ✅ Companies, ✅ Plans

### Step 3.1: Build Subscriptions Module (Basic CRUD)

**Backend Status:** ✅ Complete (SubscriptionsController - 15+ operations!)

**Frontend Tasks:**
1. Create `domain/models/subscription.model.ts`:
   - Subscription class
   - CreateSubscriptionRequest
   - UpdateSubscriptionRequest
   - SubscriptionStatus enum
   - SubscriptionsResponse interface

2. Create `domain/mappers/subscription.mapper.ts`

3. Create `services/subscription.service.ts`:
   - Basic CRUD
   - Get by company
   - Get active subscription
   - Get subscription status

4. Add API endpoints (20+ endpoints!)

5. Register service

6. Create `viewmodels/subscription-viewmodel.tsx`:
   - Column definitions (company, plan, status, dates)
   - Form fields
   - Status badges with colors

7. Create `views/subscription-view.tsx`

8. Create `app/subscriptions/page.tsx`

9. Add translations (50+ keys)

**UI Features:**
- List subscriptions with advanced filters
- Search by company, plan, status
- Filter by: Active, Expired, Suspended, Trial
- Status badges (Active=green, Expired=red, Suspended=yellow)
- Expiry date warnings

---

### Step 3.2: Build Subscription Lifecycle Operations

**Add to service (15 operations):**
- Renew subscription
- Upgrade subscription (3 modes: FullReplace, Prorated, Deferred)
- Cancel subscription
- Suspend/Resume subscription
- Pause/Unpause subscription
- Stop trial (convert to paid)
- Extend expiry date
- Reactivate expired subscription

**Upgrade Modal (Complex):**
- Select new plan (dropdown)
- Select upgrade mode:
  - FullReplace: Switch immediately
  - Prorated: Credit remaining time
  - Deferred: Switch at next renewal
- Show cost calculation
- Confirm button

**Action Buttons (Context-Aware):**
- Renew (if active, near expiry)
- Upgrade (if active)
- Cancel (if active)
- Suspend (if active)
- Resume (if suspended)
- Pause (if active)
- Unpause (if paused)
- Extend (always available)
- Stop Trial (if in trial)
- Reactivate (if expired)

---

### Step 3.3: Build Subscription Details Page

**Create `/subscriptions/[id]/page.tsx`:**

**Overview Section:**
- Company name (link to company)
- Plan name (link to plan details)
- Status badge with visual indicator
- Start date, expiry date
- Next renewal date (if auto-renew)
- Trial status (if applicable)
- Days remaining indicator

**Actions Section:**
- Action buttons (context-aware)
- Each action opens confirmation modal
- Some actions open input modals (extend date, reason for cancel)

**License Key Section:**
- Generate License Key button
- View key (masked initially, show on click)
- Copy to clipboard button
- Regenerate button (with warning)
- Revoke button (with confirmation)
- Last generated date
- Key status indicator

**History Timeline:**
- Vertical timeline of all changes
- Each event shows:
  - Action type (Created, Renewed, Upgraded, Suspended, etc.)
  - Admin who made the change
  - Timestamp
  - Reason/notes
- Filter by event type
- Pagination for large histories

**Analytics Section:**
- Usage statistics (if available)
- Activity graphs
- Custom metrics
- Time-based charts

---

### Step 3.4: Build Subscription Child Pages

**Create `/companies/[id]/subscriptions/page.tsx`:**
- List all subscriptions for a company
- Quick create subscription for this company
- Active subscription highlighted
- Historical subscriptions

**Create `/companies/[id]/subscriptions/active/page.tsx`:**
- Show only active subscription
- If no active subscription, show "No active subscription" message
- Quick action to create one

**Create `/plans/[id]/subscriptions/page.tsx`:**
- List all subscriptions using this plan
- Statistics (total, active, expired)
- Revenue metrics
- Conversion rates

---

## 🎯 PHASE 4: OFFLINE ACCESS - LICENSE KEYS

**Priority:** ⭐⭐⭐⭐ HIGH (Client integration)
**Time:** 2-3 days
**Dependencies:** ✅ Subscriptions

### Step 4.1: Build License Keys Module

**Backend Status:** ✅ Complete (LicenseController)

**Frontend Tasks:**
1. Create `domain/models/license-key.model.ts`:
   - LicenseKey class
   - GenerateLicenseRequest
   - ValidateLicenseRequest
   - ValidationResponse interface

2. Create `domain/mappers/license-key.mapper.ts`

3. Create `services/license-key.service.ts`:
   - Generate license key
   - Regenerate license key
   - Validate license key
   - Get company license keys
   - Revoke license key
   - Check if subscription has valid key

4. Add API endpoints

5. Register service

6. Integrate into subscription details page (already planned in Phase 3)

7. Create standalone license management page (optional)

8. Add translations

**UI Features:**

**Integrated into Subscription Page:**
- License Key section (already described in Phase 3.3)
- Generate, view, copy, regenerate, revoke

**Standalone License Validation Tool (`/licenses/validate/page.tsx`):**
- Input field for license key
- Validate button
- Show validation result:
  - Valid/Invalid badge
  - If valid, show:
    - Subscription ID
    - Company name
    - Plan name
    - Expiry date
    - Status
  - If invalid, show error message

**License Management Page (`/licenses/page.tsx`):**
- List all license keys
- Filter by company, subscription, status
- Search by key (partial)
- View key details
- Revoke keys
- Expiry warnings

**Child Pages:**
- `/companies/[id]/licenses/page.tsx` - All licenses for a company

---

## 🎯 PHASE 5: INTEGRATION & POLISH

**Priority:** ⭐⭐⭐ MEDIUM (UX improvements)
**Time:** 2-3 days

### Step 5.1: Enhance Company Page

**Add tabs to Company Details:**
1. Overview tab (existing data)
2. Subscriptions tab:
   - List all subscriptions
   - Active subscription highlighted
   - Quick actions
3. Licenses tab:
   - List all license keys
   - Key status indicators
4. Analytics tab:
   - Usage metrics
   - Revenue contribution
   - Subscription history chart

---

### Step 5.2: Enhance Dashboard

**Add new widgets:**
1. Active Subscriptions Count (with trend)
2. Expiring Soon Warning (list subscriptions expiring in 30 days)
3. Revenue Metrics by Plan (pie chart)
4. Most Popular Plans (bar chart)
5. Trial Conversion Rate (percentage)
6. License Usage Statistics
7. Recent Subscription Activities (timeline)

---

### Step 5.3: Build Cross-Module Linking

**Add navigation links:**
- Company page → Subscriptions
- Company page → Licenses
- Plan page → Subscriptions
- Subscription page → Company
- Subscription page → Plan
- Subscription page → License
- Project page → Modules
- Plan page → Projects
- Plan page → Modules

**Breadcrumb Navigation:**
- All detail pages should have breadcrumbs
- Example: Home > Companies > Acme Corp > Subscriptions > Pro Plan

---

### Step 5.4: Add Advanced Filters

**Build reusable filter components:**
1. Date Range Picker (for filtering by date ranges)
2. Multi-Select Filter (for status, plan type, etc.)
3. Search Input with Debounce
4. Saved Filter Presets (allow users to save common filters)

**Apply to all list pages:**
- Companies
- Projects
- Modules
- Plans
- Subscriptions
- Licenses

---

## 📈 IMPLEMENTATION TIMELINE

### Week 1: Foundation
- **Day 1-2:** Projects Module
  - Domain model, mapper, service
  - ViewModel, view, page
  - Translations
- **Day 3:** Modules Module
  - Same structure as Projects
  - Add project-module child page

### Week 2: Product Catalog
- **Day 1-2:** Plans Module (Basic CRUD)
  - Domain model, mapper, service
  - Basic list and forms
- **Day 3-4:** Plans Module (Advanced)
  - Multi-step wizard
  - Plan details page
  - Project/module associations
- **Day 5:** Plans Module (Polish)
  - Child pages
  - Styling
  - Testing

### Week 3: Customer Instances
- **Day 1-2:** Subscriptions (Basic)
  - Domain model, mapper, service
  - List view with filters
  - Basic CRUD
- **Day 3-4:** Subscriptions (Lifecycle)
  - All 15 operations
  - Action buttons
  - Modals (upgrade, extend, etc.)
- **Day 5:** Subscriptions (Details)
  - Subscription details page
  - History timeline
  - Analytics

### Week 4: Offline Access & Polish
- **Day 1-2:** License Keys
  - Integration into subscription page
  - Validation tool
  - License management
- **Day 3-4:** Integration
  - Company enhancements
  - Dashboard widgets
  - Cross-linking
- **Day 5:** Testing & Polish
  - End-to-end testing
  - Bug fixes
  - Documentation

---

## 🎨 UI/UX PATTERNS

### Multi-Step Wizards
- Progress indicator at top
- Next/Previous/Save buttons
- Validation on each step
- Can save as draft

### Detail Pages Layout
```
┌─────────────────────────────────────────┐
│ Header (Title + Actions)                │
├─────────────────────────────────────────┤
│ Tab Navigation                           │
├─────────────────────────────────────────┤
│                                          │
│ Content Area                             │
│ - Cards for sections                     │
│ - Tables for lists                       │
│ - Charts for analytics                   │
│                                          │
└─────────────────────────────────────────┘
```

### Status Badges
- Active: Green
- Expired: Red
- Suspended: Yellow/Orange
- Paused: Blue
- Trial: Purple
- Cancelled: Gray

### Action Buttons
- Primary action: Filled button
- Secondary actions: Outline buttons
- Destructive actions: Red outline
- Contextual: Show/hide based on state

### Data Visualization
- Timeline: Vertical for history
- Charts: Line for trends, Pie for distribution, Bar for comparison
- Progress bars: For trial periods, days remaining
- Badges: For status indicators

---

## ✅ SUCCESS CRITERIA

Each phase is complete when:
- [ ] All domain models created with validation
- [ ] All mappers handle API responses correctly
- [ ] All services implement interface completely
- [ ] All ViewModels configure GenericCrudView properly
- [ ] All Views render without errors
- [ ] All Pages route correctly
- [ ] All translations added (EN + AR)
- [ ] All child pages created and linked
- [ ] Manual testing passed for CRUD operations
- [ ] Manual testing passed for special operations
- [ ] Cross-module links work correctly
- [ ] No TypeScript errors
- [ ] No runtime errors
- [ ] Responsive design verified
- [ ] RTL (Arabic) layout verified

---

## 🚫 CRITICAL: FOLLOW THIS ORDER!

**❌ DO NOT:**
- Start with Subscriptions before Plans are done
- Start with Plans before Projects/Modules are done
- Start with Licenses before Subscriptions are done
- Skip any phase

**✅ ALWAYS:**
- Complete Phase 1 before Phase 2
- Complete Phase 2 before Phase 3
- Complete Phase 3 before Phase 4
- Test each phase before moving to next

---

## 📊 PROGRESS TRACKING

- [ ] **Phase 1: Projects & Modules** (2-3 days)
  - [ ] 1.1: Projects Module
  - [ ] 1.2: Modules Module

- [ ] **Phase 2: Subscription Plans** (4-5 days)
  - [ ] 2.1: Basic Plans CRUD
  - [ ] 2.2: Advanced Features (wizard, details, associations)

- [ ] **Phase 3: Subscriptions** (5-6 days)
  - [ ] 3.1: Basic CRUD
  - [ ] 3.2: Lifecycle Operations
  - [ ] 3.3: Details Page
  - [ ] 3.4: Child Pages

- [ ] **Phase 4: License Keys** (2-3 days)
  - [ ] 4.1: Integration & Management

- [ ] **Phase 5: Integration & Polish** (2-3 days)
  - [ ] 5.1: Company Enhancement
  - [ ] 5.2: Dashboard Enhancement
  - [ ] 5.3: Cross-Linking
  - [ ] 5.4: Advanced Filters

**TOTAL TIME: 15-20 days** 🚀

---

**🎯 CURRENT STATUS: Ready to start Phase 1 - Projects Module**