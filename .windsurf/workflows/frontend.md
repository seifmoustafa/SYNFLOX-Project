---
description: workflow frontend
auto_execution_mode: 3
---

# 🚀 SYNFLOX Frontend - Feature Development Workflow

> **Step-by-step guide for building features with Clean Architecture**  
> **Last Updated:** November 16, 2025

---

## 📋 **MANDATORY Development Order**

```
✅ STEP 1: Domain Model     → domain/models/[feature].model.ts
✅ STEP 2: Mapper           → domain/mappers/[feature].mapper.ts  
✅ STEP 3: Service          → services/[feature].service.ts
✅ STEP 4: ViewModel        → viewmodels/[feature]-viewmodel.tsx
✅ STEP 5: View             → views/[feature]-view.tsx
✅ STEP 6: Page             → app/[feature]/page.tsx
✅ STEP 7: Registration     → Update exports & providers
✅ STEP 8: Localization     → Add i18n keys (en.ts + ar.ts)
```

**Why this order?** Each layer depends on the previous one. Bottom-up prevents missing dependencies.

---

## 📦 **STEP 1: Domain Model**

**File:** `domain/models/[feature].model.ts`

### **Three Classes Required:**

```typescript
// 1️⃣ DATA INTERFACE (API shape)
export interface [Feature]Data {
  id: string;  // Encrypted GUID from backend
  name: string;
  isActive: boolean;
  createdTimestamp: string;
  updatedTimestamp?: string;
}

// 2️⃣ DOMAIN MODEL (Business logic)
export class [Feature] {
  public readonly id: string;     // ⭐ All fields readonly
  public readonly name: string;
  
  constructor(data: [Feature]Data) {
    this.id = data.id;
    this.name = data.name;
  }
  
  // ⭐ Business logic in getters
  get displayName(): string {
    return this.name;
  }
  
  get status(): "Active" | "Inactive" {
    return this.isActive ? "Active" : "Inactive";
  }
  
  // ⭐ Immutability
  update(updates: Partial<[Feature]Data>): [Feature] {
    return new [Feature]({ ...this, ...updates });
  }
}

// 3️⃣ CREATE REQUEST
export class Create[Feature]Request {
  public readonly name: string;
  
  constructor(data: { name: string }) {
    this.name = data.name;
  }
  
  get isValid(): boolean {
    return !!(this.name && this.name.trim().length > 0);
  }
}

// 4️⃣ UPDATE REQUEST
export class Update[Feature]Request {
  public readonly id: string;
  public readonly name?: string;
  
  get isValid(): boolean {
    return !!this.id;
  }
}
```

**Checklist:**
- [ ] All fields `readonly` (immutable)
- [ ] Business logic in getters
- [ ] `update()` method
- [ ] `Create[Feature]Request` class
- [ ] `Update[Feature]Request` class
- [ ] `isValid` validation

---

## 🔄 **STEP 2: Mapper**

**File:** `domain/mappers/[feature].mapper.ts`

```typescript
export class [Feature]Mapper {
  // ⭐ API → Domain
  static fromJson(json: any): [Feature] {
    return new [Feature]({
      id: json.id || '',
      name: json.name || '',
      isActive: json.isActive ?? true,
    });
  }
  
  // ⭐ Domain → API
  static toJson(feature: [Feature]): any {
    return {
      id: feature.id,
      name: feature.name,
    };
  }
  
  // ⭐ Create Request → API
  static createRequestToJson(request: Create[Feature]Request): any {
    return {
      name: request.name,
    };
  }
  
  // ⭐ Update Request → API (only changed fields)
  static updateRequestToJson(request: Update[Feature]Request): any {
    const json: any = {};
    if (request.name !== undefined) json.name = request.name;
    return json;
  }
  
  // ⭐ Handle API response
  static handleApiResponse(response: any): { data: [Feature][]; pagination: any } {
    // Handle multiple formats from backend
    if (response?.data && Array.isArray(response.data)) {
      return {
        data: response.data.map(item => this.fromJson(item)),
        pagination: response.pagination || {}
      };
    }
    return { data: [], pagination: {} };
  }
}
```

**Checklist:**
- [ ] `fromJson()` - API to Domain
- [ ] `toJson()` - Domain to API
- [ ] `createRequestToJson()`
- [ ] `updateRequestToJson()` - Only changed fields
- [ ] `handleApiResponse()` - Handle pagination

---

## ⚙️ **STEP 3: Service**

**File:** `services/[feature].service.ts`

```typescript
export interface I[Feature]Service {
  get[Features](params?: any): Promise<{ data: [Feature][]; pagination: any }>;
  get[Feature]ById(id: string): Promise<[Feature]>;
  create[Feature](data: Create[Feature]Request): Promise<[Feature]>;
  update[Feature](id: string, data: Update[Feature]Request): Promise<[Feature]>;
  delete[Feature](id: string): Promise<void>;
}

export class [Feature]Service implements I[Feature]Service {
  constructor(
    private apiService: IApiService,
    private notificationService: INotificationService
  ) {}
  
  async get[Features](params?: any) {
    const response = await this.apiService.get(API_ENDPOINTS.[FEATURES]_GET_ALL, params);
    return [Feature]Mapper.handleApiResponse(response);  // ⭐ Always use mapper
  }
  
  async create[Feature](data: Create[Feature]Request) {
    const json = [Feature]Mapper.createRequestToJson(data);  // ⭐ Mapper
    const response = await this.apiService.post(API_ENDPOINTS.[FEATURES]_CREATE, json);
    this.notificationService.success(response?.message);  // ⭐ Notification
    return [Feature]Mapper.fromJson(response?.data);
  }
  
  // ... update, delete similar
}
```

**Checklist:**
- [ ] Interface + Implementation
- [ ] All CRUD methods
- [ ] ALWAYS use mapper
- [ ] Show success notifications

---

## 🎛️ **STEP 4: ViewModel**

**File:** `viewmodels/[feature]-viewmodel.tsx`

```typescript
"use client";

export function use[Feature]ViewModel() {
  const { [feature]Service } = useServices();
  const { t } = useI18n();
  
  // ⭐ Generic CRUD handles 90% of logic
  const vm = useGenericCrudViewModel<[Feature], Create[Feature]Request, Update[Feature]Request>({
    getData: [feature]Service.get[Features].bind([feature]Service),
    create: [feature]Service.create[Feature].bind([feature]Service),
    update: [feature]Service.update[Feature].bind([feature]Service),
    delete: [feature]Service.delete[Feature].bind([feature]Service),
  }, {
    itemTypeName: t("[feature].item"),
    getItemDisplayName: (item) => item.displayName,
  });
  
  // ⭐ UI Configuration
  const config: CrudConfig<[Feature]> = useMemo(() => ({
    titleKey: "[feature].title",
    columns: [
      { key: "name", label: t("[feature].name"), render: (_, item) => item.name },
    ],
    createFields: [
      { name: "name", label: t("[feature].name"), type: "text", required: true },
    ],
    editFields: [
      { name: "name", label: t("[feature].name"), type: "text", required: true },
      { name: "id", type: "hidden", required: true },
    ],
    editInitialValues: (item) => ({ name: item.name, id: item.id }),
    getActions: (vm, t, handleDelete) => [
      { label: t("common.edit"), onClick: (item) => vm.openEditModal(item) },
      { label: t("common.delete"), onClick: handleDelete, isDeleteAction: true },
    ],
  }), [t, vm]);
  
  return { vm, config };
}
```

**Checklist:**
- [ ] Use `useGenericCrudViewModel`
- [ ] Define `columns`
- [ ] Define `createFields`
- [ ] Define `editFields`
- [ ] Define `editInitialValues`
- [ ] Define `getActions`

---

## 🎨 **STEP 5: View**

**File:** `views/[feature]-view.tsx`

```typescript
"use client";

export function [Feature]View() {
  const { vm, config } = use[Feature]ViewModel();
  
  return <GenericCrudView viewModel={vm} config={config} />;
}
```

**Checklist:**
- [ ] Client component
- [ ] Use ViewModel hook
- [ ] Render GenericCrudView

---

## 📄 **STEP 6: Page**

**File:** `app/[features]/page.tsx`

```typescript
import { DashboardLayout } from "@/components/layout/dashboard-layout";
import { [Feature]View } from "@/views/[feature]-view";

export default async function [Features]Page() {
  return (
    <DashboardLayout>
      <[Feature]View />
    </DashboardLayout>
  );
}
```

---

## 📝 **STEP 7: Registration**

### **7.1: Export Domain**
**`domain/index.ts`:**
```typescript
export * from './models/[feature].model';
export * from './mappers/[feature].mapper';
```

### **7.2: Register Service**
**`providers/service-provider.tsx`:**
```typescript
const [feature]Service = new [Feature]Service(apiService, notificationService);
const services = { ..., [feature]Service };
```

### **7.3: API Endpoints**
**`config/api-endpoints.ts`:**
```typescript
[FEATURES]_GET_ALL: "/[features]",
[FEATURES]_CREATE: "/[features]",
// ...
```

---

## 🌍 **STEP 8: Localization**

**`locales/en.ts` & `locales/ar.ts`:**
```typescript
[feature]: {
  title: "[Features] Management",
  item: "[Feature]",
  name: "Name",
  status: { active: "Active", inactive: "Inactive" },
}
```

---

## ✅ **Testing Checklist**

- [ ] List items with pagination
- [ ] Search/filter works
- [ ] Create new item
- [ ] Edit existing item
- [ ] Delete with confirmation
- [ ] Loading states
- [ ] Error messages
- [ ] Success notifications
- [ ] English translations
- [ ] Arabic translations (RTL)

---

**🎯 Follow this workflow for EVERY new feature!**
