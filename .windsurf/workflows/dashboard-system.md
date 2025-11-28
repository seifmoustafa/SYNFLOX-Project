---
description: SYNFLOX Dashboard System - Complete Implementation Workflow
---

# 🚀 SYNFLOX Dashboard System Implementation

## Overview
A comprehensive, multi-page dashboard system with role-based access for the SYNFLOX Central Licensing System.

---

## 📊 Dashboard Pages Structure

| Page | Route | Purpose |
|------|-------|---------|
| **Home Overview** | `/` | Quick snapshot of entire system |
| **Companies** | `/dashboard/companies` | Company metrics & insights |
| **Subscriptions** | `/dashboard/subscriptions` | Subscription lifecycle analytics |
| **Revenue** | `/dashboard/revenue` | Financial insights & projections |
| **Activity** | `/dashboard/activity` | Admin activity & audit trail |
| **Alerts** | `/dashboard/alerts` | System health & action items |
| **Analytics** | `/dashboard/analytics` | Advanced insights & predictions |

---

## 🔐 Role-Based Access

| Dashboard | SuperAdmin | Admin | Custom AdminType |
|-----------|------------|-------|------------------|
| Home Overview | ✅ | ✅ | ✅ (filtered) |
| Companies | ✅ | ✅ | Configurable |
| Subscriptions | ✅ | ✅ | Configurable |
| Revenue | ✅ | ❌ | Configurable |
| Activity | ✅ | ⚠️ Own only | Configurable |
| Alerts | ✅ | ✅ | Configurable |
| Analytics | ✅ | ❌ | Configurable |

---

## 📋 Implementation Phases

### Phase 1: Core Infrastructure (Backend)
// turbo
1. Create folder: `Application/DTOs/Dashboard/`

2. Create `Application/DTOs/Dashboard/OverviewDashboardDto.cs`:
   - TotalCompanies, ActiveCompanies, CompaniesWithoutSubscription
   - TotalSubscriptions, ActiveSubscriptions, TrialSubscriptions
   - ExpiringSoon (7 days), ExpiringThisMonth (30 days)
   - TotalAdmins, ActiveAdmins
   - RecentActivity list (last 10 actions)
   - QuickStats (growth percentages)

3. Create `Application/DTOs/Dashboard/CompaniesDashboardDto.cs`:
   - StatusDistribution (Active, Inactive, Suspended, AtRisk counts)
   - GrowthData (companies created per day/week/month)
   - TopCompanies (by subscription value)
   - CompaniesNeedingAttention list
   - SubscriptionCoverage percentage

4. Create `Application/DTOs/Dashboard/SubscriptionsDashboardDto.cs`:
   - StatusDistribution (Active, Trial, Expired, Suspended, Cancelled)
   - ByPlan (count and revenue per plan)
   - ExpiryTimeline (this week, this month, next 3 months)
   - LifecycleMetrics (new, renewed, upgraded, churned this period)
   - TrialConversionRate
   - AverageDuration

5. Create `Application/DTOs/Dashboard/RevenueDashboardDto.cs`:
   - MRR (Monthly Recurring Revenue)
   - ARR (Annual Recurring Revenue)
   - ARPC (Average Revenue Per Customer)
   - RevenueByPlan list
   - RevenueByCurrency list
   - GrowthRate (month over month)
   - Projections (30-day forecast)

6. Create `Application/DTOs/Dashboard/ActivityDashboardDto.cs`:
   - TotalLogins, TodayLogins
   - TotalActions, TodayActions
   - TopAdmins list (by actions)
   - RecentActivity list (last 50)
   - ActivityByType (company, subscription, admin actions)

7. Create `Application/DTOs/Dashboard/AlertsDashboardDto.cs`:
   - CriticalAlerts list (expiring today)
   - HighPriorityAlerts list (expiring this week)
   - MediumPriorityAlerts list (expiring this month)
   - LowPriorityAlerts list (inactive admins, etc.)
   - TotalAlertCount

8. Create `Application/Services Interfaces/IDashboardService.cs`:
   - GetOverviewAsync()
   - GetCompaniesDashboardAsync()
   - GetSubscriptionsDashboardAsync()
   - GetRevenueDashboardAsync()
   - GetActivityDashboardAsync()
   - GetAlertsDashboardAsync()

9. Create `Infrastructure/Services/DashboardService.cs`:
   - Implement all interface methods
   - Use proper EF Core queries with Include/ThenInclude
   - Add caching for expensive queries (optional)

10. Create `WebAPI/Controllers/DashboardController.cs`:
    - GET /api/dashboard/overview
    - GET /api/dashboard/companies
    - GET /api/dashboard/subscriptions
    - GET /api/dashboard/revenue (SuperAdmin only)
    - GET /api/dashboard/activity
    - GET /api/dashboard/alerts

11. Register service in `InfrastructureServiceRegistration.cs`

12. Build and test backend endpoints

---

### Phase 2: Frontend Infrastructure
1. Create `config/api-endpoints.ts` - Add dashboard endpoints:
   ```typescript
   DASHBOARD: {
     OVERVIEW: "/dashboard/overview",
     COMPANIES: "/dashboard/companies",
     SUBSCRIPTIONS: "/dashboard/subscriptions",
     REVENUE: "/dashboard/revenue",
     ACTIVITY: "/dashboard/activity",
     ALERTS: "/dashboard/alerts",
   }
   ```

2. Create domain models in `domain/models/dashboard/`:
   - overview.model.ts
   - companies-dashboard.model.ts
   - subscriptions-dashboard.model.ts
   - revenue-dashboard.model.ts
   - activity-dashboard.model.ts
   - alerts-dashboard.model.ts

3. Create mappers in `domain/mappers/dashboard/`:
   - One mapper per model

4. Create `services/dashboard.service.ts`:
   - getOverview()
   - getCompaniesDashboard()
   - getSubscriptionsDashboard()
   - getRevenueDashboard()
   - getActivityDashboard()
   - getAlertsDashboard()

5. Register service in `providers/service-provider.tsx`

6. Export from `domain/index.ts`

---

### Phase 3: Home Overview Page
1. Create `viewmodels/dashboard/overview-viewmodel.tsx`

2. Create `views/dashboard/overview-view.tsx`:
   - 4 KPI cards (Companies, Subscriptions, Revenue, Alerts)
   - Quick action buttons
   - Recent activity feed
   - Mini charts (status pie, growth line)

3. Update `app/page.tsx` to use OverviewView

4. Add translations in `locales/en.ts` and `locales/ar.ts`

---

### Phase 4: Companies Dashboard Page
1. Create `app/dashboard/companies/page.tsx`

2. Create `viewmodels/dashboard/companies-dashboard-viewmodel.tsx`

3. Create `views/dashboard/companies-dashboard-view.tsx`:
   - Status distribution chart (pie/doughnut)
   - Growth over time chart (line)
   - Top companies table
   - Companies needing attention list
   - Subscription coverage gauge

4. Add navigation link in sidebar

5. Add translations

---

### Phase 5: Subscriptions Dashboard Page
1. Create `app/dashboard/subscriptions/page.tsx`

2. Create `viewmodels/dashboard/subscriptions-dashboard-viewmodel.tsx`

3. Create `views/dashboard/subscriptions-dashboard-view.tsx`:
   - Status distribution chart
   - By plan chart
   - Expiry timeline chart
   - Lifecycle metrics cards
   - Trial conversion rate

4. Add navigation link in sidebar

5. Add translations

---

### Phase 6: Revenue Dashboard Page
1. Create `app/dashboard/revenue/page.tsx`

2. Create `viewmodels/dashboard/revenue-dashboard-viewmodel.tsx`

3. Create `views/dashboard/revenue-dashboard-view.tsx`:
   - MRR/ARR cards
   - ARPC card
   - Revenue by plan chart
   - Revenue trend chart
   - Projections section

4. Add navigation link (SuperAdmin only)

5. Add translations

---

### Phase 7: Activity Dashboard Page
1. Create `app/dashboard/activity/page.tsx`

2. Create `viewmodels/dashboard/activity-dashboard-viewmodel.tsx`

3. Create `views/dashboard/activity-dashboard-view.tsx`:
   - Login stats cards
   - Action stats cards
   - Top admins leaderboard
   - Activity timeline
   - Activity by type chart

4. Add navigation link

5. Add translations

---

### Phase 8: Alerts Dashboard Page
1. Create `app/dashboard/alerts/page.tsx`

2. Create `viewmodels/dashboard/alerts-dashboard-viewmodel.tsx`

3. Create `views/dashboard/alerts-dashboard-view.tsx`:
   - Critical alerts section (red)
   - High priority alerts (orange)
   - Medium priority alerts (yellow)
   - Low priority alerts (blue)
   - Alert actions (dismiss, snooze, take action)

4. Add navigation link

5. Add translations

---

### Phase 9: Polish & Testing
1. Test all endpoints with real data
2. Verify role-based access works
3. Test RTL (Arabic) layout
4. Performance optimization
5. Add loading skeletons
6. Add error states

---

## 📁 File Structure Reference

### Backend
```
SYNFLOX/
├── Application/
│   ├── DTOs/Dashboard/
│   │   ├── OverviewDashboardDto.cs
│   │   ├── CompaniesDashboardDto.cs
│   │   ├── SubscriptionsDashboardDto.cs
│   │   ├── RevenueDashboardDto.cs
│   │   ├── ActivityDashboardDto.cs
│   │   └── AlertsDashboardDto.cs
│   └── Services Interfaces/
│       └── IDashboardService.cs
├── Infrastructure/
│   └── Services/
│       └── DashboardService.cs
└── WebAPI/
    └── Controllers/
        └── DashboardController.cs
```

### Frontend
```
synflox-frontend/
├── app/
│   ├── page.tsx (Home Overview)
│   └── dashboard/
│       ├── companies/page.tsx
│       ├── subscriptions/page.tsx
│       ├── revenue/page.tsx
│       ├── activity/page.tsx
│       └── alerts/page.tsx
├── views/dashboard/
│   ├── overview-view.tsx
│   ├── companies-dashboard-view.tsx
│   ├── subscriptions-dashboard-view.tsx
│   ├── revenue-dashboard-view.tsx
│   ├── activity-dashboard-view.tsx
│   └── alerts-dashboard-view.tsx
├── viewmodels/dashboard/
│   ├── overview-viewmodel.tsx
│   ├── companies-dashboard-viewmodel.tsx
│   ├── subscriptions-dashboard-viewmodel.tsx
│   ├── revenue-dashboard-viewmodel.tsx
│   ├── activity-dashboard-viewmodel.tsx
│   └── alerts-dashboard-viewmodel.tsx
├── domain/
│   ├── models/dashboard/
│   │   ├── overview.model.ts
│   │   ├── companies-dashboard.model.ts
│   │   ├── subscriptions-dashboard.model.ts
│   │   ├── revenue-dashboard.model.ts
│   │   ├── activity-dashboard.model.ts
│   │   └── alerts-dashboard.model.ts
│   └── mappers/dashboard/
│       ├── overview.mapper.ts
│       └── ... (one per model)
└── services/
    └── dashboard.service.ts
```

---

## 📊 Business Logic Reference

### Company Status Logic
- **Active**: Has at least one Active or Trial subscription
- **Inactive**: No subscription OR all subscriptions expired/cancelled
- **Suspended**: Has a suspended subscription
- **At Risk**: Active subscription expiring within 30 days

### Subscription Status Values
- Active, Trial, Expired, Suspended, Cancelled, Paused

### Revenue Calculations
- **MRR**: Sum of all active subscription monthly prices
- **ARR**: MRR × 12
- **ARPC**: MRR / Number of active companies

### Alert Priority
- 🔴 **Critical**: Expiring today, payment failed
- 🟠 **High**: Expiring within 7 days, suspended
- 🟡 **Medium**: Expiring within 30 days, no subscription
- 🟢 **Low**: Inactive admins, optimization suggestions

---

## ✅ Completion Checklist

- [ ] Phase 1: Backend Infrastructure
- [ ] Phase 2: Frontend Infrastructure
- [ ] Phase 3: Home Overview Page
- [ ] Phase 4: Companies Dashboard
- [ ] Phase 5: Subscriptions Dashboard
- [ ] Phase 6: Revenue Dashboard
- [ ] Phase 7: Activity Dashboard
- [ ] Phase 8: Alerts Dashboard
- [ ] Phase 9: Polish & Testing

---

## 🚀 Start Command

To begin implementation, say: "Start Phase 1 of dashboard system"
