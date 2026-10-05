# 🏗️ Production Backend Setup — menuhat

Production setup checklist for the **menuhat .NET Backend API**.

الهدف من الملف ده إننا نجهز الـ Backend بشكل **Production-ready** قبل ما نبدأ أول Feature حقيقية، مع الحفاظ على Architecture بسيطة وواضحة بدون Enterprise complexity غير ضرورية.

---

# Phase 0 — Project Foundation

أول حاجة نثبت أساس المشروع:

* [ ] Create Solution
* [ ] API Project
* [ ] Application Project
* [ ] Domain Project
* [ ] Infrastructure Project
* [ ] Unit Tests
* [ ] Integration Tests
* [ ] Project References
* [ ] `.gitignore`
* [ ] `.editorconfig`
* [ ] `Directory.Build.props`
* [ ] `Directory.Packages.props`
* [ ] Nullable Reference Types
* [ ] Implicit Usings
* [ ] Code Analysis / Analyzers
* [ ] Build verification

الهدف هنا إن الـ Solution نفسها تبقى نظيفة قبل ما نكتب Business Code.

---

# Phase 1 — Architecture

هنا نثبت **الحدود والـ dependencies**.

```text
             ┌─────────────────┐
             │       API       │
             │   Presentation  │
             └────────┬────────┘
                      │
                      ▼
             ┌─────────────────┐
             │   Application   │
             │ Use Cases/Rules │
             └────────┬────────┘
                      │
                      ▼
             ┌─────────────────┐
             │     Domain      │
             │ Business Model  │
             └─────────────────┘

             Infrastructure
                    │
                    └──────► Application / Domain
```

ونحدد من البداية:

* [ ] Dependency Direction
* [ ] Layer Responsibilities
* [ ] Where DTOs live
* [ ] Where Commands / Queries live
* [ ] Where Validators live
* [ ] Where Interfaces live
* [ ] Where EF Core lives
* [ ] Where external services live
* [ ] Where authentication implementation lives

والأهم:

> **Different Responsibility → Separate Appropriate Concern → Clear Boundary → Controlled Dependency → Easier Change**

مش الهدف إننا نزود Layers وInterfaces لمجرد إن المشروع Production.

---

# Phase 2 — Configuration

قبل Database وAuth:

* [ ] `appsettings.json`
* [ ] `appsettings.Development.json`
* [ ] Production configuration strategy
* [ ] Environment Variables
* [ ] Options Pattern
* [ ] Secrets
* [ ] Connection Strings
* [ ] CORS configuration
* [ ] API settings
* [ ] External services configuration

مثلاً:

```text
Configuration
├── Database
├── Authentication
├── JWT
├── CORS
├── Storage
└── External Services
```

والـ secrets **ممنوع** تدخل Git.

---

# Phase 3 — Dependency Injection

نرتب الـ DI registration بشكل واضح:

```csharp
builder.Services
    .AddApplication()
    .AddInfrastructure(configuration);
```

ونحدد:

* [ ] Application DI
* [ ] Infrastructure DI
* [ ] DbContext
* [ ] Repositories / persistence abstractions إن احتجناها
* [ ] External services
* [ ] Validators
* [ ] Authentication services
* [ ] Authorization services

---

# Phase 4 — Database / EF Core

بعد كده:

* [ ] EF Core
* [ ] DbContext
* [ ] Database provider
* [ ] Entity configurations
* [ ] Migrations
* [ ] Database connection
* [ ] Seed strategy
* [ ] Transaction strategy
* [ ] Concurrency strategy
* [ ] Naming conventions
* [ ] Indexes strategy

وهنا كمان نقرر:

**Repository؟ ولا `IAppDbContext`؟**

مش لازم نعمل Repository abstraction لكل Entity بشكل أوتوماتيكي.

---

# Phase 5 — Global Error Handling

دي لازم تتعمل بدري جدًا.

نعمل:

```text
Request
   ↓
Endpoint
   ↓
Application
   ↓
Exception
   ↓
Global Exception Handler
   ↓
Standard Error Response
```

مثلاً API يرجع شكل موحد:

```json
{
  "type": "validation_error",
  "message": "One or more validation errors occurred.",
  "errors": {
    "name": [
      "Name is required."
    ]
  }
}
```

ونحدد:

* [ ] Exception handling
* [ ] ProblemDetails
* [ ] Validation errors
* [ ] Not Found
* [ ] Conflict
* [ ] Unauthorized
* [ ] Forbidden
* [ ] Unexpected exceptions

---

# Phase 6 — Logging & Observability

هنا ندخل Production فعلاً.

* [ ] Structured Logging
* [ ] Log levels
* [ ] Request logging
* [ ] Error logging
* [ ] Correlation ID / Trace ID
* [ ] User ID in logs where appropriate
* [ ] Environment-aware logging
* [ ] Sensitive data protection
* [ ] Log retention strategy

## What should we log?

مش:

```text
Everything
```

لكن:

```text
Business-important events
Errors
Warnings
Infrastructure failures
Security-relevant events
Performance problems
```

وممنوع نحط:

```text
Password
JWT
Refresh Token
Credit Card
Sensitive personal data
```

في الـ logs.

---

# Phase 7 — Validation

نحدد validation strategy:

```text
API Input
   ↓
Validation
   ↓
Application
   ↓
Domain Rules
```

والفرق مهم جدًا:

## Input Validation

مثلاً:

```text
Name is required
Price must be greater than 0
```

## Business Rule

مثلاً في menuhat:

```text
Order can only be placed from an Active Restaurant.
```

دي **مش مجرد validation**؛ دي Business Rule.

---

# Phase 8 — Authentication

بعد الأساسيات:

* [ ] Authentication
* [ ] JWT / Cookie strategy حسب النظام
* [ ] Access Token
* [ ] Refresh Token
* [ ] Token expiration
* [ ] Password hashing
* [ ] Password policies
* [ ] Authentication middleware
* [ ] Current User abstraction
* [ ] Security considerations

مثلاً:

```text
Request
   ↓
Authentication
   ↓
Who is this user?
   ↓
Authorization
   ↓
Can this user perform this action?
```

---

# Phase 9 — Authorization

ودي مهمة جدًا في menuhat.

عندك مثلاً:

```text
Admin
Restaurant Owner
Customer
```

لكن مش كفاية نقول:

```csharp
[Authorize(Roles = "RestaurantOwner")]
```

لأن عندك Ownership rules.

مثلاً:

> Restaurant Owner can edit **their own restaurant only**.

دي Authorization + Business Rule / Resource Ownership.

فنحدد من البداية:

* [ ] Roles
* [ ] Policies
* [ ] Permissions
* [ ] Resource ownership
* [ ] Admin access
* [ ] Restaurant Owner access
* [ ] Customer access

---

# Phase 10 — API Standards

قبل أول Feature:

* [ ] Routing conventions
* [ ] HTTP status codes
* [ ] Request DTOs
* [ ] Response DTOs
* [ ] Pagination
* [ ] Filtering
* [ ] Sorting
* [ ] API versioning strategy
* [ ] Swagger / OpenAPI
* [ ] Consistent response/error format

---

# Phase 11 — Health & Reliability

مهمة جدًا في Production وممكن تتنسى.

* [ ] Health Checks
* [ ] Database health check
* [ ] External service health checks
* [ ] Readiness
* [ ] Liveness
* [ ] Timeout strategy
* [ ] CancellationToken
* [ ] Retry strategy where appropriate
* [ ] Rate limiting
* [ ] Request size limits

---

# Phase 12 — Security

نعمل Security baseline قبل production:

* [ ] HTTPS
* [ ] CORS
* [ ] Authentication
* [ ] Authorization
* [ ] Secrets management
* [ ] Input validation
* [ ] SQL Injection protection
* [ ] Sensitive logging protection
* [ ] Rate limiting
* [ ] Security headers where applicable
* [ ] File upload restrictions
* [ ] Password security
* [ ] Token security

---

# Phase 13 — Testing Foundation

مش لازم نكتب 500 test من البداية.

لكن نجهز البنية:

```text
Unit Tests
Integration Tests
```

## Unit Tests

Business logic:

```text
Given
When
Then
```

## Integration Tests

مثلاً:

```text
HTTP
 ↓
Endpoint
 ↓
Application
 ↓
Database
```

ونبدأ نختبر الـ critical workflows.

---

# Phase 14 — Documentation

ودي مهمة جدًا في مشروع menuhat خصوصًا إن موضوع الـ roles والـ workflows محتاج يبقى واضح.

نعمل:

```text
docs/
├── 01-architecture.md
├── 02-domain-rules.md
├── 03-authentication-authorization.md
├── 04-api-conventions.md
├── 05-database.md
├── 06-error-handling.md
├── 07-logging.md
├── 08-testing.md
└── 09-deployment.md
```

وبالذات:

```text
Register Restaurant
Approve Restaurant
Create Menu
Place Order
Manage Order
Review Restaurant
```

كل Workflow يبقى له rules واضحة.

---

# Phase 15 — CI/CD

آخر جزء من الـ foundation:

* [ ] Build pipeline
* [ ] Test pipeline
* [ ] Formatting / analyzers
* [ ] Migration strategy
* [ ] Environment configuration
* [ ] Deployment
* [ ] Production configuration
* [ ] Health check after deployment
* [ ] Rollback strategy

---

# 🎯 Final Production Setup Order

دي الـ checklist الأساسية اللي هنمشي عليها:

```text
01. Solution & Projects
02. Project References
03. Architecture Boundaries
04. Coding Standards
05. Configuration
06. Dependency Injection
07. Database / EF Core
08. Global Error Handling
09. Logging
10. Validation
11. Authentication
12. Authorization
13. API Conventions
14. Health Checks
15. Security
16. Testing Foundation
17. Documentation
18. CI/CD
19. First Feature
```

**وبعد رقم 18 فقط** نبدأ أول Feature حقيقية.

---

# ⚠️ Production-Ready ≠ Enterprise Ceremony

مش كل حاجة في القائمة لازم تتعمل بأقصى تعقيد من أول يوم.

إحنا عايزين:

> **Production-ready foundation, not unnecessary complexity.**

مثلاً ممكن نجهز Health Checks من البداية، لكن مش لازم نبني distributed tracing كامل لو المشروع لسه Monolith.

وممكن نبدأ Authorization بسيط، لكن لازم الـ boundaries تكون واضحة من البداية.

وممكن نستخدم `IAppDbContext` بدل ما نعمل Repository لكل Entity، لو ده أنسب للـ use cases.

الهدف هو بناء Architecture تخدم المشروع، مش Architecture لمجرد إن شكلها Enterprise.

---

# 🧭 Next Step — Production Readiness Checklist

بالنسبة لـ **menuhat**، الخطوة التالية هي إننا نحول الـ roadmap دي إلى **Production Readiness Checklist فعلية** ونمشي عليها بند بند.

لكل بند هنحدد:

```text
What
Why
Where
Package
Folder
Implementation
Done
```

وبالتالي كل مرحلة هتكون واضحة، ومفيش حاجة أساسية هتتنسى أثناء بناء الـ Backend.

---

# Definition of Done

قبل ما نبدأ أول Feature كبيرة، لازم يكون عندنا:

* [ ] Solution builds successfully
* [ ] Project dependencies are intentional
* [ ] Architecture boundaries are clear
* [ ] Configuration is environment-aware
* [ ] Secrets are protected
* [ ] DI is organized
* [ ] Database infrastructure works
* [ ] Migrations work
* [ ] Global error handling works
* [ ] Logging works
* [ ] Validation strategy is established
* [ ] Authentication strategy is established
* [ ] Authorization strategy is established
* [ ] API conventions are established
* [ ] Health checks are available
* [ ] Security baseline is established
* [ ] Unit tests are configured
* [ ] Integration tests are configured
* [ ] Core domain rules are documented
* [ ] CI/CD foundation is available

**After that → First Feature.**
