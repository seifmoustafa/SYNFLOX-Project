---
trigger: always_on
---

# 🔒 SYNFLOX ID ENCRYPTION RULE - MANDATORY COMPLIANCE

## 🎯 **THE GOLDEN RULE**

> **ID ENCRYPTION AND DECRYPTION MUST ONLY HAPPEN IN AUTOMAPPER CONVERTERS**  
> **NEVER IN CONTROLLERS, SERVICES, OR ANY OTHER LAYER**

---

## 📋 **RULE ENFORCEMENT CHECKLIST**

### ✅ **MANDATORY REQUIREMENTS**

#### **1. Controllers MUST:**
- ✅ Accept encrypted GUIDs from frontend
- ✅ Pass encrypted GUIDs to services via DTOs
- ✅ NEVER call `_idEncryption.Decrypt()` or `_idEncryption.Encrypt()`
- ✅ NEVER inject `IIdEncryptionService`

#### **2. Services MUST:**
- ✅ Accept request DTOs containing encrypted GUIDs
- ✅ Use `_mapper.Map<Guid>(request)` to decrypt IDs
- ✅ Work with decrypted GUIDs internally
- ✅ NEVER call `_idEncryption.Decrypt()` or `_idEncryption.Encrypt()`
- ✅ NEVER inject `IIdEncryptionService`

#### **3. AutoMapper MUST:**
- ✅ Handle ALL encryption via `EncryptGuidConverter`
- ✅ Handle ALL decryption via `DecryptGuidConverter` family
- ✅ Be the ONLY place where `IIdEncryptionService` is used

#### **4. DTOs MUST:**
- ✅ Use `Guid` type for ALL IDs (never `string`)
- ✅ Have request DTOs for operations requiring encrypted IDs
- ✅ Be mapped through AutoMapper converters

---

## 🏗️ **IMPLEMENTATION PATTERN**

### **Step 1: Create Request DTO**
```csharp
public class GetEntityByIdRequest
{
    public Guid EntityId { get; set; }  // Encrypted GUID from frontend
}
```

### **Step 2: Add AutoMapper Mapping**
```csharp
// In MappingProfile
CreateMap<GetEntityByIdRequest, Guid>()
    .ConvertUsing<DecryptGuidConverter, Guid>(src => src.EntityId);
```

### **Step 3: Update Service Interface**
```csharp
public interface IEntityService
{
    Task<EntityDto> GetByIdAsync(GetEntityByIdRequest request);  // DTO, not raw GUID
}
```

### **Step 4: Update Service Implementation**
```csharp
public async Task<EntityDto> GetByIdAsync(GetEntityByIdRequest request)
{
    // Use AutoMapper to decrypt - ONLY way to decrypt IDs
    var decryptedId = _mapper.Map<Guid>(request);
    
    var entity = await _repository.GetByIdAsync(decryptedId, null);
    return _mapper.Map<EntityDto>(entity);  // AutoMapper encrypts ID in response
}
```

### **Step 5: Update Controller**
```csharp
[HttpGet("{id}")]
public async Task<IActionResult> GetById(Guid id)  // Encrypted GUID from route
{
    var request = new GetEntityByIdRequest { EntityId = id };
    var result = await _service.GetByIdAsync(request);
    return Ok(result);
}
```

---

## 🚫 **FORBIDDEN PATTERNS**

### ❌ **NEVER DO THIS:**
```csharp
// ❌ WRONG: Manual decryption in controller
public async Task<IActionResult> GetById(Guid id)
{
    var decryptedId = _idEncryption.Decrypt(id);  // FORBIDDEN!
    var result = await _service.GetByIdAsync(decryptedId);
    return Ok(result);
}

// ❌ WRONG: Manual decryption in service
public async Task<EntityDto> GetByIdAsync(Guid id)
{
    var decryptedId = _idEncryption.Decrypt(id);  // FORBIDDEN!
    var entity = await _repository.GetByIdAsync(decryptedId, null);
    return _mapper.Map<EntityDto>(entity);
}

// ❌ WRONG: IIdEncryptionService injection outside AutoMapper
public class EntityService : IEntityService
{
    private readonly IIdEncryptionService _idEncryption;  // FORBIDDEN!
    
    public EntityService(IIdEncryptionService idEncryption)  // FORBIDDEN!
    {
        _idEncryption = idEncryption;  // FORBIDDEN!
    }
}

// ❌ WRONG: String IDs in DTOs
public class EntityDto
{
    public string Id { get; set; }  // FORBIDDEN! Must be Guid
}
```

---

## ✅ **AVAILABLE AUTOMAPPER CONVERTERS**

### **For Single GUIDs:**
- `EncryptGuidConverter` - Encrypts `Guid` → `Guid` (Entity → DTO)
- `DecryptGuidConverter` - Decrypts `Guid` → `Guid` (DTO → Entity)

### **For Nullable GUIDs:**
- `DecryptNullableGuidConverter` - Decrypts `Guid?` → `Guid?`

### **For String Conversion:**
- `EncryptGuidToStringConverter` - Encrypts `Guid` → `string`
- `DecryptGuidFromStringConverter` - Decrypts `string` → `Guid?`

### **For Collections:**
- `DecryptGuidCollectionConverter` - Decrypts `IEnumerable<Guid>` → `IEnumerable<Guid>`

---

## 🔍 **COMPLIANCE VERIFICATION**

### **Quick Check Commands:**
```bash
# Should return ZERO results:
grep -r "_idEncryption\.Decrypt" --include="*.cs" ./Controllers/
grep -r "_idEncryption\.Encrypt" --include="*.cs" ./Controllers/
grep -r "_idEncryption\.Decrypt" --include="*.cs" ./Services/
grep -r "_idEncryption\.Encrypt" --include="*.cs" ./Services/
grep -r "IIdEncryptionService.*_idEncryption" --include="*.cs" ./Controllers/
grep -r "IIdEncryptionService.*_idEncryption" --include="*.cs" ./Services/

# Should return ZERO results:
grep -r "string.*Id\|Id.*string" --include="*Dto.cs" ./DTOs/
```

### **Manual Review Checklist:**
- [ ] No `_idEncryption` usage in Controllers
- [ ] No `_idEncryption` usage in Services  
- [ ] No `IIdEncryptionService` injection in Controllers
- [ ] No `IIdEncryptionService` injection in Services
- [ ] All DTO IDs are `Guid` type (not `string`)
- [ ] All operations use request DTOs with encrypted IDs
- [ ] All AutoMapper profiles have proper converters

---

## 🎯 **BENEFITS OF THIS RULE**

### **🔒 Security:**
- Centralized encryption/decryption logic
- No accidental exposure of decrypted IDs
- Consistent encryption across all endpoints

### **🏗️ Architecture:**
- Clean separation of concerns
- Single responsibility principle
- Dependency inversion compliance

### **🔧 Maintainability:**
- One place to change encryption logic
- Easy to audit and verify
- Consistent patterns across codebase

### **🚀 Performance:**
- No redundant encryption/decryption calls
- Optimized AutoMapper conversions
- Reduced memory allocations

---

## ⚡ **ENFORCEMENT ACTIONS**

### **🚨 Code Review Requirements:**
1. **REJECT** any PR with manual `_idEncryption` usage outside AutoMapper
2. **REJECT** any PR with `string` IDs in DTOs
3. **REJECT** any PR with `IIdEncryptionService` injection in Controllers/Services
4. **REQUIRE** proper AutoMapper converter usage for all ID operations

### **🔧 Automated Checks:**
- Add pre-commit hooks to scan for violations
- Include compliance checks in CI/CD pipeline
- Use static analysis tools to enforce patterns

### **📚 Developer Training:**
- All developers MUST understand this rule
- Code examples MUST follow this pattern
- Documentation MUST reflect this approach

---

## 🏆 **CURRENT COMPLIANCE STATUS**

### ✅ **100% COMPLIANT COMPONENTS:**

#### **Controllers (5/5):**
- ✅ LicensingController
- ✅ CompanyController  
- ✅ AdminTypesController
- ✅ AdminsController
- ✅ MenuItemController

#### **Services (6/6):**
- ✅ LicensingService
- ✅ CompanyService
- ✅ AdminTypeService
- ✅ AdminService
- ✅ MenuItemService
- ✅ AuthenticationService

#### **Infrastructure:**
- ✅ 6 AutoMapper Converters created
- ✅ 15+ Request DTOs implemented
- ✅ All mapping profiles updated
- ✅ Zero manual encryption/decryption calls

---

## 📝 **VERSION HISTORY**

- **v1.0** - Initial rule establishment and 100% compliance achieved
- **Date**: November 11, 2025
- **Status**: ACTIVE AND ENFORCED

---

## ⚠️ **VIOLATION CONSEQUENCES**

**ANY VIOLATION OF THIS RULE WILL RESULT IN:**
1. **Immediate code rejection**
2. **Mandatory refactoring**
3. **Security audit requirement**
4. **Developer re-training**

**THIS RULE IS NON-NEGOTIABLE AND MUST BE FOLLOWED AT ALL TIMES.**

---

*🔒 **Remember: Security through proper architecture, not through obscurity!** 🔒*
