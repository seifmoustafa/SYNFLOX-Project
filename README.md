# SYNFLOX - Enterprise Central Licensing System

<div align="center">

![SYNFLOX Logo](https://via.placeholder.com/200x80/4F46E5/FFFFFF?text=SYNFLOX)

**The Ultimate Solution for Centralized Software License Management**

[![.NET](https://img.shields.io/badge/.NET-8.0-512BD4)](https://dotnet.microsoft.com/)
[![Next.js](https://img.shields.io/badge/Next.js-14-000000)](https://nextjs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.0-3178C6)](https://www.typescriptlang.org/)
[![Clean Architecture](https://img.shields.io/badge/Architecture-Clean-green)](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
[![License](https://img.shields.io/badge/License-Enterprise-blue)](LICENSE)

</div>

---

## 🌟 What is SYNFLOX?

**SYNFLOX** is an enterprise-grade **Central Licensing System** that revolutionizes how software vendors manage and control licenses across multiple products. Instead of each application handling its own licensing, SYNFLOX provides a unified, secure, and scalable solution for centralized license management.

### The Business Problem We Solve

Software vendors distributing multiple enterprise products (ERP, CRM, POS, HR, Inventory systems) face these challenges:

- **Fragmented License Management**: Each product manages its own licenses
- **No Unified Control**: Difficult to manage customer subscriptions across products
- **Security Vulnerabilities**: Inconsistent license validation mechanisms
- **Operational Overhead**: Multiple systems to maintain and monitor
- **Customer Experience**: Confusing licensing across different products

### The SYNFLOX Solution

SYNFLOX transforms this chaos into a **unified, secure, and efficient licensing ecosystem**:

```
┌─────────────────────────────────────────────────────────────┐
│                    SYNFLOX ECOSYSTEM                        │
│                                                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│  │   ERP       │    │    CRM      │    │    POS      │     │
│  │  System     │    │   System    │    │   System    │     │
│  └─────┬───────┘    └─────┬───────┘    └─────┬───────┘     │
│        │                  │                  │             │
│        └──────────────────┼──────────────────┘             │
│                           │                                │
│  ┌─────────────────────────▼─────────────────────────┐     │
│  │            SYNFLOX CENTRAL CORE                   │     │
│  │  • License Validation  • Subscription Control    │     │
│  │  • Key Management     • Multi-Tenant Support     │     │
│  │  • Audit Trails       • Real-time Monitoring     │     │
│  └─────────────────────────────────────────────────────┘     │
│                                                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│  │ Inventory   │    │  HR System  │    │  Accounting │     │
│  │  System     │    │             │    │   System    │     │
│  └─────────────┘    └─────────────┘    └─────────────┘     │
└─────────────────────────────────────────────────────────────┘
```

---

## 🏢 Business Value Proposition

### For Software Vendors
- **Unified License Control**: Single source of truth for all product licenses
- **Reduced Development Costs**: No need to build licensing into each product
- **Enhanced Security**: Enterprise-grade encryption and validation
- **Scalable Architecture**: Handle thousands of customers and products
- **Compliance Ready**: Built-in audit trails and reporting
- **Faster Time-to-Market**: Focus on product features, not licensing infrastructure

### For Enterprise Customers
- **Simplified Management**: One system to manage all software licenses
- **Transparent Licensing**: Clear visibility into subscription status
- **Flexible Deployment**: Support for both online and offline environments
- **Reliable Service**: 99.9% uptime with enterprise-grade infrastructure
- **Multi-Language Support**: Arabic and English interfaces

---

## 🚀 Core Business Functions

### 1. **Centralized License Management**
```
Customer Journey:
Registration → Activation → Monitoring → Renewal → Support

┌─────────────┐   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐
│   Company   │──▶│ Subscription│──▶│   License   │──▶│  Validation │
│ Registration│   │ Activation  │   │ Generation  │   │ & Monitoring│
└─────────────┘   └─────────────┘   └─────────────┘   └─────────────┘
```

**Key Features:**
- **Multi-Tenant Architecture**: Manage thousands of customer companies
- **Subscription Lifecycle**: Complete control from activation to expiration
- **Real-Time Status**: Instant license validation for all connected products
- **Flexible Licensing Models**: Support various subscription types and durations

### 2. **Dual Integration Model**

#### **Online Systems Integration**
```http
External Product → GET /api/licensing/{companyId}/status → SYNFLOX
                                                              ↓
External Product ← { status: "Active", expiryDate: "..." } ← SYNFLOX
```

**Perfect for:**
- Cloud-based applications
- Always-connected systems
- Real-time license validation
- Centralized monitoring and control

#### **Offline Systems Integration**
```
License Key Generation → Secure Distribution → Local Validation
         ↓                       ↓                    ↓
   AES-256 Encrypted      Installation in      Tamper-Proof
   HMAC Signed Keys       Offline System       Validation
```

**Perfect for:**
- Air-gapped environments
- Industrial systems
- Remote locations
- Security-critical applications

### 3. **Administrative Control Center**

The SYNFLOX Admin Dashboard provides complete control over:

- **Company Management**: Customer onboarding, contact management, subscription tracking
- **License Operations**: Activation, suspension, extension, key generation
- **User Administration**: Role-based access control, audit trails
- **System Monitoring**: Real-time status, performance metrics, error tracking
- **Reporting & Analytics**: Comprehensive insights into license usage and trends

---

## 🏗️ System Architecture

SYNFLOX is built on **Clean Architecture** principles with complete separation between backend and frontend:

### Backend (.NET Clean Architecture)
```
┌─────────────────────────────────────────────────────────┐
│                    WebAPI Layer                          │
│  REST Controllers • JWT Auth • Localization • CORS     │
└─────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│                 Application Layer                        │
│  Business Services • DTOs • AutoMapper • Validation    │
└─────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│                   Domain Layer                           │
│  Entities • Business Rules • Interfaces • Enums        │
└─────────────────────────────────────────────────────────┘
                            ↑
┌─────────────────────────────────────────────────────────┐
│               Infrastructure Layer                       │
│  EF Core • Repositories • External Services • Auth     │
└─────────────────────────────────────────────────────────┘
```

### Frontend (Next.js Clean Architecture)
```
┌─────────────────────────────────────────────────────────┐
│                     Pages Layer                          │
│  Next.js App Router • Server Components • Layouts      │
└─────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│                     Views Layer                          │
│  UI Components • User Interactions • Presentation      │
└─────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│                  ViewModels Layer                        │
│  Business Logic • State Management • MVVM Pattern      │
└─────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│                   Services Layer                         │
│  API Communication • Error Handling • Notifications    │
└─────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│                   Domain Layer                           │
│  Business Models • Mappers • Validation • Rules        │
└─────────────────────────────────────────────────────────┘
```

---

## 🔒 Enterprise Security

### Multi-Layer Security Architecture

#### **1. Authentication & Authorization**
- **JWT-based Authentication**: Secure token-based access control
- **Role-based Authorization**: SuperAdmin and Admin roles with granular permissions
- **Session Management**: Automatic token refresh and secure logout
- **Multi-Factor Authentication**: Ready for 2FA implementation

#### **2. Data Protection**
- **ID Encryption**: All entity IDs encrypted at API boundaries
- **Database Security**: Encrypted connections, parameterized queries
- **Audit Trails**: Complete tracking of all system changes
- **Soft Delete**: Data retention with logical deletion

#### **3. License Key Security**
```
Security Layers:
┌─────────────────────────────────────────────────────────┐
│  Base64 Encoding (Transport)                            │
│  ┌─────────────────────────────────────────────────────┐ │
│  │  AES-256 Encryption (Content Protection)           │ │
│  │  ┌─────────────────────────────────────────────────┐ │ │
│  │  │  HMAC SHA256 Signature (Tamper Protection)     │ │ │
│  │  │  ┌─────────────────────────────────────────────┐ │ │ │
│  │  │  │  Clock Tampering Detection (Time Security) │ │ │ │
│  │  │  └─────────────────────────────────────────────┘ │ │ │
│  │  └─────────────────────────────────────────────────┘ │ │
│  └─────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────┘
```

---

## 🌍 Multi-Language & Cultural Support

### Complete Internationalization
- **Languages**: English (LTR) and Arabic (RTL)
- **UI Adaptation**: Complete interface transformation for RTL languages
- **Cultural Formatting**: Date, time, and number formatting per locale
- **Font Integration**: Optimized typography for each language
- **Content Translation**: All user-facing content localized

### Language Detection Priority
1. **Accept-Language Header** (HTTP standard)
2. **X-Language Header** (explicit control)
3. **Query Parameter** (?lang=ar)
4. **User Preferences** (stored settings)

---

## 📊 Technology Stack

### Backend Technologies
- **.NET 8.0**: Latest LTS framework
- **ASP.NET Core**: Web API framework
- **Entity Framework Core**: ORM with SQL Server/Oracle support
- **AutoMapper**: Object-to-object mapping
- **JWT**: Authentication and authorization
- **Serilog**: Structured logging
- **Redis**: Distributed caching (optional)

### Frontend Technologies
- **Next.js 14**: React framework with App Router
- **TypeScript**: Type-safe development
- **Tailwind CSS**: Utility-first CSS framework
- **Shadcn/UI**: Modern UI component library
- **React Hook Form**: Form management
- **Zustand**: State management
- **React Query**: Server state management

### Database & Infrastructure
- **SQL Server**: Primary database (production)
- **Oracle**: Secondary database support
- **Docker**: Containerization support
- **Redis**: Caching and session storage
- **Swagger/OpenAPI**: API documentation

---

## 🚀 Getting Started

### Prerequisites
- **.NET 8.0 SDK**
- **Node.js 18+**
- **SQL Server** (LocalDB, Express, or Full)
- **Git**

### Quick Setup

#### 1. **Clone the Repository**
```bash
git clone <repository-url>
cd SYNFLOX-Project
```

#### 2. **Backend Setup**
```bash
cd SYNFLOX
dotnet restore
dotnet ef database update --project WebAPI
dotnet run --project WebAPI
```

#### 3. **Frontend Setup**
```bash
cd synflox-frontend
npm install
npm run dev
```

#### 4. **Access the System**
- **Backend API**: `https://localhost:5001/swagger`
- **Frontend Dashboard**: `http://localhost:3000`
- **Login**: Username: `superadmin`, Password: `password`

### Docker Deployment
```bash
# Backend
cd SYNFLOX
docker build -t synflox-api .
docker run -p 5000:80 synflox-api

# Frontend
cd synflox-frontend
docker build -t synflox-frontend .
docker run -p 3000:3000 synflox-frontend
```

---

## 📁 Project Structure

```
SYNFLOX-Project/
├── SYNFLOX/                    # Backend (.NET Clean Architecture)
│   ├── Domain/                 # Core business logic
│   ├── Application/            # Use cases and DTOs
│   ├── Infrastructure/         # External concerns
│   ├── WebAPI/                 # REST API controllers
│   └── README.md               # Backend documentation
│
├── synflox-frontend/           # Frontend (Next.js Clean Architecture)
│   ├── app/                    # Next.js App Router pages
│   ├── components/             # UI components
│   ├── views/                  # Feature views
│   ├── viewmodels/             # Business logic
│   ├── services/               # API communication
│   ├── domain/                 # Domain models
│   ├── providers/              # React contexts
│   ├── locales/                # Internationalization
│   └── README.md               # Frontend documentation
│
├── docs/                       # Additional documentation
├── docker-compose.yml          # Multi-container deployment
└── README.md                   # This file
```

---

## 🎯 Use Cases & Integration Examples

### ERP System Integration
```csharp
// Online validation in ERP system
public async Task<bool> ValidateLicense(Guid companyId)
{
    var client = new HttpClient();
    var response = await client.GetAsync(
        $"https://synflox-api.com/api/licensing/{companyId}/status");
    
    var result = await response.Content.ReadFromJsonAsync<CompanyStatusResponse>();
    
    return result.Status == "Active";
}
```

### POS System (Offline)
```csharp
// Offline validation in POS system
public bool ValidateOfflineLicense(string licenseKey)
{
    var request = new ValidateLicenseKeyRequest { LicenseKey = licenseKey };
    var response = await _httpClient.PostAsJsonAsync(
        "https://synflox-api.com/api/licensing/validate-key", request);
    
    var result = await response.Content.ReadFromJsonAsync<LicenseKeyValidationResponse>();
    
    return result.IsValid && !result.ClockTampered;
}
```

### Multi-Language API Usage
```javascript
// JavaScript integration with language support
const validateLicense = async (companyId, language = 'en') => {
    const response = await fetch(
        `https://synflox-api.com/api/licensing/${companyId}/status`,
        {
            headers: {
                'Accept-Language': language,
                'X-Language': language
            }
        }
    );
    
    const data = await response.json();
    return data.data.status === 'Active';
};
```

---

## 🔮 Future Roadmap

### Phase 1: Core Enhancement
- [ ] **Advanced Analytics**: Comprehensive reporting and dashboards
- [ ] **API Rate Limiting**: Advanced throttling and quotas
- [ ] **Webhook System**: Real-time notifications for license events
- [ ] **Bulk Operations**: Mass license management capabilities

### Phase 2: Enterprise Features
- [ ] **Multi-Factor Authentication**: Enhanced security
- [ ] **Advanced Audit Logs**: Comprehensive activity tracking
- [ ] **Custom Branding**: White-label support for vendors
- [ ] **Advanced Reporting**: Custom report builder

### Phase 3: Platform Expansion
- [ ] **Mobile App**: iOS/Android admin applications
- [ ] **API Gateway**: Advanced API management
- [ ] **Microservices**: Service decomposition for scale
- [ ] **Cloud Native**: Kubernetes deployment support

### Phase 4: AI & Automation
- [ ] **Predictive Analytics**: License usage forecasting
- [ ] **Automated Renewals**: Smart renewal recommendations
- [ ] **Anomaly Detection**: Suspicious activity monitoring
- [ ] **Chatbot Support**: AI-powered customer support

---

## 🤝 Contributing

We welcome contributions from the community! Please read our [Contributing Guidelines](CONTRIBUTING.md) for details on:

- Code of Conduct
- Development Process
- Pull Request Process
- Issue Reporting
- Feature Requests

---

## 📄 License

This project is licensed under the Enterprise License - see the [LICENSE](LICENSE) file for details.

---

## 🆘 Support

### Documentation
- **Backend API**: [SYNFLOX/README.md](SYNFLOX/README.md)
- **Frontend Dashboard**: [synflox-frontend/README.md](synflox-frontend/README.md)
- **API Documentation**: Available via Swagger UI

### Community
- **Issues**: [GitHub Issues](https://github.com/your-org/synflox/issues)
- **Discussions**: [GitHub Discussions](https://github.com/your-org/synflox/discussions)
- **Email**: support@synflox.com

### Enterprise Support
For enterprise customers, we provide:
- 24/7 technical support
- Dedicated account management
- Custom integration assistance
- Priority bug fixes and feature requests

---

<div align="center">

**SYNFLOX** - Revolutionizing Enterprise License Management

[![Website](https://img.shields.io/badge/Website-synflox.com-blue)](https://synflox.com)
[![Documentation](https://img.shields.io/badge/Docs-docs.synflox.com-green)](https://docs.synflox.com)
[![Support](https://img.shields.io/badge/Support-support@synflox.com-red)](mailto:support@synflox.com)

*Built with ❤️ for the enterprise software community*

**Version 1.0** | **Last Updated: 2025**

</div>
