# ProductCatalogApp - Azure Cloud Modernization Assessment

**Assessment Date:** January 12, 2026  
**Assessor:** GitHub Copilot Modernization Agent  
**Target Platform:** Azure Cloud  
**Application:** ProductCatalogApp

---

## Executive Summary

The ProductCatalogApp is a .NET Framework 4.8.1 application built with ASP.NET MVC 5, utilizing WCF services and MSMQ for asynchronous order processing. The application demonstrates a clean separation of concerns but relies heavily on legacy Windows-specific technologies that are incompatible with modern cloud deployments.

**Cloud Readiness Score: 2/10** ⚠️

**Verdict:** Moderate complexity migration requiring significant modernization effort. The application is currently **not cloud-ready** and requires a comprehensive replatform and refactoring approach to successfully deploy on Azure.

**Estimated Effort:** 8-12 weeks with a team of 2-3 developers

---

## Current Architecture

### Technology Stack

| Component | Technology | Version |
|-----------|-----------|---------|
| **Framework** | .NET Framework | 4.8.1 |
| **Web Framework** | ASP.NET MVC | 5.2.9 |
| **Service Layer** | Windows Communication Foundation (WCF) | - |
| **Messaging** | Microsoft Message Queuing (MSMQ) | - |
| **Data Storage** | In-Memory (Static Repository) | - |
| **Session State** | In-Process | - |
| **Frontend** | Bootstrap, jQuery | 5.2.3, 3.7.0 |

### Application Components

#### 1. **ProductCatalog** (Web Application)
- **Type:** ASP.NET MVC 5 Web Application
- **Purpose:** Customer-facing web interface for browsing products, managing shopping cart, and placing orders
- **Features:**
  - Product catalog browsing
  - Shopping cart management
  - Order submission via MSMQ
  - Session-based cart storage
- **Dependencies:** ProductServiceLibrary (via WCF)

#### 2. **ProductServiceLibrary** (WCF Service)
- **Type:** WCF Service Library
- **Purpose:** Backend service providing product management operations
- **Protocol:** SOAP/XML over WCF
- **Features:**
  - CRUD operations for products
  - Category management
  - Product search and filtering
  - Price range queries
- **Data Access:** In-memory repository with static seed data

#### 3. **OrderProcessor** (Console Application)
- **Type:** Console Application / Background Processor
- **Purpose:** Asynchronous order processing from MSMQ queue
- **Features:**
  - Polls MSMQ queue for new orders
  - Simulates payment validation
  - Inventory updates
  - Shipping label generation
  - Email confirmations

### Communication Patterns

```
┌─────────────────┐         WCF (SOAP)        ┌──────────────────────┐
│  ProductCatalog │◄──────────────────────────►│ ProductServiceLibrary│
│   (Web App)     │                            │   (WCF Service)      │
└─────────────────┘                            └──────────────────────┘
        │                                                 │
        │ MSMQ Queue                                     │
        │ (System.Messaging)                             │
        ▼                                                 ▼
┌─────────────────┐                          ┌──────────────────────┐
│ OrderProcessor  │                          │   In-Memory Store    │
│ (Console App)   │                          │  (Static Repository) │
└─────────────────┘                          └──────────────────────┘
```

---

## Legacy Patterns Identified

### 🔴 Critical Issues

#### 1. **WCF (Windows Communication Foundation)**
- **Severity:** High
- **Impact:** Not supported in .NET Core/.NET 6+
- **Blocker:** Yes - prevents cloud migration
- **Modern Alternative:** ASP.NET Core Web API with REST or gRPC
- **Migration Effort:** Medium (2-3 weeks)

**Why This Matters:** WCF was designed for .NET Framework and Windows environments. It's not part of .NET Core/modern .NET, making it incompatible with cross-platform and cloud-optimized deployments.

#### 2. **MSMQ (Microsoft Message Queuing)**
- **Severity:** High
- **Impact:** Windows-specific, not cloud-native
- **Blocker:** Yes - requires Azure infrastructure
- **Modern Alternative:** Azure Service Bus, Azure Queue Storage, Azure Event Grid
- **Migration Effort:** Medium (1-2 weeks)

**Why This Matters:** MSMQ is a Windows Server feature that doesn't exist in Azure PaaS services. Cloud-native messaging services provide better scalability, reliability, and cross-platform support.

#### 3. **In-Memory Data Storage**
- **Severity:** Critical
- **Impact:** Data loss on restart, no scalability
- **Blocker:** Yes - unsuitable for production
- **Modern Alternative:** Azure SQL Database, Cosmos DB
- **Migration Effort:** Medium (1-2 weeks)

**Why This Matters:** Static in-memory storage loses all data on application restart and cannot scale horizontally. Production applications require persistent, scalable data storage.

#### 4. **.NET Framework 4.8.1**
- **Severity:** High
- **Impact:** Windows-only, limited cloud optimization
- **Blocker:** Yes - limits deployment options
- **Modern Alternative:** .NET 8 or .NET 9
- **Migration Effort:** High (3-4 weeks)

**Why This Matters:** .NET Framework is Windows-only and doesn't receive performance improvements for cloud workloads. Modern .NET offers cross-platform support, better performance, and cloud optimization.

### 🟡 Medium Priority Issues

#### 5. **In-Process Session State**
- **Severity:** Medium
- **Impact:** Cannot scale horizontally in cloud
- **Modern Alternative:** Azure Cache for Redis (distributed sessions)
- **Migration Effort:** Low (1 week)

#### 6. **No Authentication/Authorization**
- **Severity:** High (Security)
- **Impact:** Security vulnerability
- **Modern Alternative:** Azure AD B2C, Microsoft Identity Platform
- **Migration Effort:** Medium (1-2 weeks)

#### 7. **No Logging/Monitoring**
- **Severity:** Medium
- **Impact:** Poor observability
- **Modern Alternative:** Application Insights, Structured Logging
- **Migration Effort:** Low (3-5 days)

#### 8. **ASP.NET MVC 5**
- **Severity:** Medium
- **Impact:** Legacy framework
- **Modern Alternative:** ASP.NET Core MVC
- **Migration Effort:** High (included in framework migration)

---

## Complexity Assessment

**Overall Complexity: 6/10**

### Complexity Breakdown

| Area | Score (1-10) | Reasoning |
|------|--------------|-----------|
| **Code Complexity** | 3 | Simple business logic, clear structure |
| **Architecture Complexity** | 7 | WCF and MSMQ require significant changes |
| **Data Complexity** | 2 | No existing database to migrate |
| **Integration Complexity** | 8 | WCF and MSMQ migrations are complex |
| **Infrastructure Complexity** | 7 | Multiple Azure services needed |

### Complexity Factors

✅ **Simplifying Factors:**
- Clean code structure and separation of concerns
- Simple business logic without complex rules
- No existing database migrations required
- Modern frontend libraries already in use
- Small codebase (~17 C# files)

⚠️ **Complicating Factors:**
- Complete framework migration (.NET Framework → .NET 8)
- WCF to REST API conversion required
- MSMQ to Azure Service Bus migration
- No existing test coverage
- Session state redesign needed
- Authentication/authorization from scratch

---

## Recommended Migration Strategy

### Strategy: **Replatform and Refactor**

**Approach:** Incremental migration with parallel running during transition

### Migration Phases

#### **Phase 1: Foundation Setup** (2 weeks)

**Objectives:**
- Establish new .NET 8 project structure
- Set up Azure infrastructure
- Implement data persistence layer

**Tasks:**
1. Create new .NET 8 ASP.NET Core solution
2. Provision Azure resources:
   - Azure App Service (S1 tier)
   - Azure SQL Database (Basic tier)
   - Azure Service Bus (Standard tier)
   - Azure Cache for Redis (C0 tier)
3. Set up Azure Application Insights
4. Design and implement database schema with Entity Framework Core
5. Migrate data models and DTOs
6. Create CI/CD pipeline with GitHub Actions
7. Set up development and staging environments

**Deliverables:**
- ✅ New .NET 8 solution structure
- ✅ Azure resources provisioned
- ✅ Database schema implemented
- ✅ CI/CD pipeline operational

---

#### **Phase 2: Backend Modernization** (3-4 weeks)

**Objectives:**
- Replace WCF with REST API
- Implement Azure Service Bus
- Migrate business logic

**Tasks:**
1. **REST API Development:**
   - Design RESTful API endpoints matching WCF operations
   - Implement ASP.NET Core Web API controllers
   - Add Swagger/OpenAPI documentation
   - Implement proper HTTP status codes and error handling

2. **Data Layer:**
   - Implement Entity Framework Core repository pattern
   - Create database migrations
   - Seed initial product data
   - Add data validation and constraints

3. **Messaging Migration:**
   - Replace MSMQ with Azure Service Bus
   - Implement order queue sender in web app
   - Create Azure Function or background service for order processing
   - Add dead-letter queue handling

4. **Cross-Cutting Concerns:**
   - Implement structured logging with Serilog
   - Add Application Insights telemetry
   - Implement dependency injection
   - Add health check endpoints

**Deliverables:**
- ✅ RESTful API replacing WCF service
- ✅ Entity Framework Core data access
- ✅ Azure Service Bus integration
- ✅ Order processing service
- ✅ Comprehensive logging

---

#### **Phase 3: Frontend Migration** (2-3 weeks)

**Objectives:**
- Migrate ASP.NET MVC 5 to ASP.NET Core MVC
- Update views and client-side code
- Implement distributed sessions

**Tasks:**
1. **View Migration:**
   - Convert Razor views to ASP.NET Core syntax
   - Update tag helpers and HTML helpers
   - Migrate bundling and minification
   - Update static file handling

2. **Controller Migration:**
   - Port MVC controllers to ASP.NET Core
   - Update action filters and attributes
   - Implement model binding and validation
   - Add anti-forgery token handling

3. **Session Management:**
   - Configure Azure Cache for Redis
   - Implement distributed session state
   - Update session-based cart logic
   - Add session timeout handling

4. **API Integration:**
   - Replace WCF client calls with HTTP client
   - Implement proper error handling
   - Add retry policies with Polly
   - Update AJAX calls to use new API

**Deliverables:**
- ✅ ASP.NET Core MVC application
- ✅ Migrated Razor views
- ✅ Distributed session state
- ✅ REST API integration

---

#### **Phase 4: Security & Production Readiness** (2-3 weeks)

**Objectives:**
- Implement authentication and authorization
- Harden security
- Optimize performance

**Tasks:**
1. **Authentication:**
   - Integrate Azure AD B2C for customer authentication
   - Implement login/logout flows
   - Add user registration and profile management
   - Implement "Remember Me" functionality

2. **Authorization:**
   - Define authorization policies
   - Implement role-based access control (RBAC)
   - Protect sensitive endpoints
   - Add claims-based authorization

3. **Security Hardening:**
   - Implement HTTPS enforcement
   - Add security headers (HSTS, CSP, X-Frame-Options)
   - Enable CORS policies
   - Implement rate limiting
   - Add input validation and sanitization
   - Configure Content Security Policy

4. **Performance Optimization:**
   - Implement response caching
   - Add output caching for static content
   - Configure CDN for static files (Azure Front Door)
   - Optimize database queries
   - Add query result caching
   - Implement compression

5. **Monitoring & Diagnostics:**
   - Configure Application Insights dashboards
   - Set up alerts for errors and performance
   - Implement custom telemetry
   - Add availability tests
   - Configure log aggregation

**Deliverables:**
- ✅ Azure AD B2C authentication
- ✅ Authorization policies
- ✅ Security hardening complete
- ✅ Performance optimizations
- ✅ Monitoring infrastructure

---

#### **Phase 5: Testing & Deployment** (2 weeks)

**Objectives:**
- Comprehensive testing
- Production deployment
- Validation and monitoring

**Tasks:**
1. **Testing:**
   - Unit tests for business logic
   - Integration tests for API endpoints
   - End-to-end UI tests
   - Load testing with Azure Load Testing
   - Security testing and vulnerability scanning
   - User acceptance testing (UAT)

2. **Deployment:**
   - Configure production App Service
   - Set up deployment slots for blue/green deployment
   - Execute database migrations
   - Configure application settings and secrets (Key Vault)
   - Deploy to production
   - DNS cutover

3. **Post-Deployment:**
   - Monitor Application Insights for errors
   - Validate all functionality
   - Performance baseline establishment
   - Documentation updates
   - Team handoff and training

**Deliverables:**
- ✅ Comprehensive test suite
- ✅ Production deployment
- ✅ Monitoring and alerts
- ✅ Documentation
- ✅ Knowledge transfer

---

## Target Azure Architecture

```
                           ┌─────────────────────┐
                           │   Azure Front Door  │
                           │  (CDN, WAF, GSLB)   │
                           └──────────┬──────────┘
                                      │
                           ┌──────────▼──────────┐
                           │  Azure App Service  │
                           │  (ASP.NET Core MVC) │
                           └──────────┬──────────┘
                                      │
                    ┌─────────────────┼─────────────────┐
                    │                 │                 │
         ┌──────────▼─────────┐      │      ┌─────────▼────────┐
         │  Azure SQL Database│      │      │ Azure Cache for  │
         │  (Product/Orders)  │      │      │   Redis (Session)│
         └────────────────────┘      │      └──────────────────┘
                                     │
                          ┌──────────▼──────────┐
                          │ Azure Service Bus   │
                          │   (Order Queue)     │
                          └──────────┬──────────┘
                                     │
                          ┌──────────▼──────────┐
                          │  Azure Functions    │
                          │  (Order Processor)  │
                          └─────────────────────┘
                                     │
                    ┌────────────────┼────────────────┐
                    │                │                │
         ┌──────────▼─────────┐     │     ┌─────────▼────────┐
         │ Azure Blob Storage │     │     │  Application     │
         │ (Product Images)   │     │     │   Insights       │
         └────────────────────┘     │     │  (Monitoring)    │
                                    │     └──────────────────┘
                         ┌──────────▼──────────┐
                         │   Azure AD B2C      │
                         │  (Authentication)   │
                         └─────────────────────┘
```

### Azure Services Mapping

| Current Technology | Azure Service | Purpose |
|-------------------|---------------|---------|
| IIS / ASP.NET MVC | **Azure App Service** | Web application hosting |
| In-Memory Storage | **Azure SQL Database** | Persistent data storage |
| MSMQ | **Azure Service Bus** | Message queuing |
| OrderProcessor Console | **Azure Functions** | Serverless background processing |
| In-Process Session | **Azure Cache for Redis** | Distributed session state |
| - | **Azure Blob Storage** | Product images, static files |
| - | **Application Insights** | APM and monitoring |
| - | **Azure AD B2C** | User authentication |
| - | **Azure Front Door** | CDN, WAF, global routing |
| - | **Azure Key Vault** | Secrets management |

---

## Cost Estimation

### Required Azure Services (Minimum)

| Service | SKU | Monthly Cost | Annual Cost |
|---------|-----|--------------|-------------|
| Azure App Service | S1 (Standard) | $73 | $876 |
| Azure SQL Database | Basic | $5 | $60 |
| Azure Service Bus | Basic | $10 | $120 |
| Azure Cache for Redis | C0 (250 MB) | $17 | $204 |
| **Total (Minimum)** | | **~$105/month** | **~$1,260/year** |

### Recommended Configuration (Production)

| Service | SKU | Monthly Cost | Annual Cost |
|---------|-----|--------------|-------------|
| Azure App Service | S2 (2 cores) | $146 | $1,752 |
| Azure SQL Database | S0 (10 DTUs) | $15 | $180 |
| Azure Service Bus | Standard | $10 + usage | $120+ |
| Azure Cache for Redis | C1 (1 GB) | $75 | $900 |
| Application Insights | Pay-as-you-go | $20 | $240 |
| Azure Functions | Consumption | $10 | $120 |
| Azure Blob Storage | Standard LRS | $5 | $60 |
| Azure AD B2C | Free (up to 50k MAU) | $0 | $0 |
| Azure Front Door | Standard | $35 + bandwidth | $420+ |
| **Total (Recommended)** | | **~$316/month** | **~$3,792/year** |

### Optional Enhancements

| Service | Purpose | Est. Monthly Cost |
|---------|---------|-------------------|
| Azure API Management | API gateway, versioning, rate limiting | $50+ |
| Azure Container Apps | Alternative to App Service for containers | $20-100 |
| Azure DevOps | CI/CD pipelines (alternative to GitHub Actions) | $0-50 |
| Azure Monitor | Advanced monitoring and alerting | $10-50 |

**Notes:**
- Costs are estimates based on East US region
- Actual costs vary by usage, data transfer, and region
- Dev/Test environments would be additional
- Bandwidth and transaction costs not included

---

## Migration Effort Estimation

### Timeline: 8-12 Weeks

| Phase | Duration | Team Size | Effort (Person-Days) |
|-------|----------|-----------|----------------------|
| Phase 1: Foundation Setup | 2 weeks | 2-3 devs | 20-30 days |
| Phase 2: Backend Modernization | 3-4 weeks | 2-3 devs | 30-48 days |
| Phase 3: Frontend Migration | 2-3 weeks | 2 devs | 20-30 days |
| Phase 4: Security & Production | 2-3 weeks | 2-3 devs | 20-36 days |
| Phase 5: Testing & Deployment | 2 weeks | 2-3 devs | 20-30 days |
| **Total** | **11-14 weeks** | **2-3 devs** | **110-174 days** |

### Breakdown by Activity

| Activity | Estimated Effort |
|----------|------------------|
| .NET Framework to .NET 8 migration | 3-4 weeks |
| WCF to REST API conversion | 2-3 weeks |
| MSMQ to Azure Service Bus | 1-2 weeks |
| Database implementation | 1-2 weeks |
| Authentication/Authorization | 1-2 weeks |
| Testing (all types) | 2-3 weeks |
| Infrastructure setup | 1 week |
| Documentation & handoff | 1 week |

### Team Composition

**Recommended Team:**
- 1 Senior .NET Developer (lead, architecture decisions)
- 1-2 .NET Developers (implementation)
- 1 DevOps Engineer (part-time, Azure infrastructure)
- 1 QA Engineer (part-time, testing)

---

## Risks and Mitigation Strategies

### High-Priority Risks

| Risk | Severity | Impact | Mitigation Strategy |
|------|----------|--------|---------------------|
| **No existing test coverage** | High | Difficult to validate migration correctness | Create comprehensive test suite during migration; use tools like NCrunch or dotCover for coverage |
| **WCF service contracts complexity** | Medium | Hidden dependencies may emerge | Thoroughly analyze WCF WSDL; create contract tests; map all operations to REST endpoints |
| **Session state migration** | Medium | May affect user experience | Implement gradual rollout; test thoroughly; add session migration logic |
| **Learning curve for team** | Medium | Slower initial progress | Provide Azure training; pair programming; dedicated learning time |
| **Data loss during migration** | Low | In-memory data is temporary | Export seed data; plan data migration strategy; minimal impact as no production data |

### Technical Risks

1. **Performance Degradation**
   - **Risk:** New architecture may have different performance characteristics
   - **Mitigation:** Performance testing at each phase; benchmarking; optimization before go-live

2. **Azure Service Quotas**
   - **Risk:** Default Azure quotas may be insufficient
   - **Mitigation:** Request quota increases early; monitor usage; plan for scaling

3. **Breaking Changes**
   - **Risk:** API changes may break existing integrations
   - **Mitigation:** Version APIs; maintain backward compatibility period; communication plan

4. **Security Vulnerabilities**
   - **Risk:** New attack surfaces with cloud deployment
   - **Mitigation:** Security reviews at each phase; penetration testing; follow Azure Security Benchmark

---

## Immediate Recommendations

### Critical Actions (Start Immediately)

1. **Set up Azure Subscription and Resource Groups**
   - Create dedicated subscription or resource group
   - Set up cost alerts and budgets
   - Configure RBAC for team access

2. **Create New .NET 8 Project Structure**
   - Initialize new solution with modern patterns
   - Set up dependency injection framework
   - Establish coding standards and guidelines

3. **Design REST API Contracts**
   - Map WCF operations to REST endpoints
   - Design DTOs and request/response models
   - Create OpenAPI/Swagger specifications

4. **Implement Database Schema**
   - Design normalized database schema
   - Create Entity Framework Core models
   - Set up migrations
   - Plan data seeding strategy

### Short-Term Actions (1-2 Weeks)

5. **Replace WCF with REST API**
   - Highest priority blocker
   - Enables cloud deployment
   - Provides foundation for frontend migration

6. **Set up CI/CD Pipeline**
   - GitHub Actions for build and test
   - Automated deployment to staging
   - Infrastructure as Code (Bicep or Terraform)

7. **Implement Basic Authentication**
   - Azure AD B2C integration
   - Secure sensitive endpoints
   - Foundation for authorization

### Medium-Term Actions (3-4 Weeks)

8. **Migrate to Azure Service Bus**
   - Replace MSMQ queues
   - Implement order processing with Azure Functions
   - Add retry policies and dead-letter handling

9. **Implement Distributed Caching**
   - Set up Azure Cache for Redis
   - Migrate session state
   - Add caching for product queries

10. **Add Monitoring and Logging**
    - Configure Application Insights
    - Implement structured logging
    - Set up alerts and dashboards

---

## Success Criteria

### Technical Success Metrics

- ✅ 100% functionality parity with existing application
- ✅ API response times < 200ms (p95)
- ✅ Zero data loss during migration
- ✅ 99.9% availability SLA
- ✅ All security scans pass without critical issues
- ✅ Unit test coverage > 70%
- ✅ Load testing supports 100 concurrent users

### Business Success Metrics

- ✅ Zero customer-facing downtime during migration
- ✅ Monthly hosting costs within $200-400 budget
- ✅ Deployment time reduced to < 15 minutes
- ✅ Team trained and comfortable with new stack
- ✅ Documentation complete and up-to-date

---

## Long-Term Modernization Opportunities

Beyond the initial migration, consider these enhancements:

### Architecture Evolution

1. **Microservices Architecture**
   - Decompose monolith into product service, order service, and cart service
   - Enables independent scaling and deployment
   - Better team autonomy

2. **Event-Driven Architecture**
   - Use Azure Event Grid for cross-service events
   - Implement CQRS for read/write separation
   - Enable real-time features

3. **API Gateway**
   - Azure API Management for centralized API governance
   - Rate limiting, throttling, and quotas
   - API versioning and documentation portal

### Technology Enhancements

4. **Containerization**
   - Package application in Docker containers
   - Deploy to Azure Kubernetes Service (AKS)
   - Better portability and scaling

5. **Frontend Modernization**
   - Consider Blazor for rich interactive UI
   - Or migrate to React/Angular SPA with Web API backend
   - Progressive Web App (PWA) capabilities

6. **Advanced Data Solutions**
   - Azure Cosmos DB for global distribution
   - Azure Cognitive Search for advanced product search
   - Azure Synapse for analytics and reporting

---

## Conclusion

The ProductCatalogApp represents a typical .NET Framework application requiring comprehensive modernization for Azure Cloud deployment. While the application has clean architecture and simple business logic, the reliance on legacy technologies (WCF, MSMQ, in-memory storage) creates significant blockers for cloud migration.

### Key Takeaways

✅ **What's Working:**
- Clean code structure and separation of concerns
- Modern frontend libraries
- Simple, understandable business logic
- No complex external dependencies

⚠️ **What Needs Attention:**
- Complete framework migration to .NET 8
- WCF to REST API conversion (critical blocker)
- MSMQ to Azure Service Bus migration (critical blocker)
- Persistent database implementation
- Authentication and security layer
- Session state modernization

### Recommended Next Steps

1. **This Week:** Set up Azure resources and new .NET 8 project
2. **Week 2-4:** Implement REST API and database layer
3. **Week 5-7:** Migrate frontend to ASP.NET Core MVC
4. **Week 8-10:** Implement security and production readiness
5. **Week 11-12:** Testing, deployment, and validation

With proper planning, adequate resources, and following the phased approach outlined in this assessment, the ProductCatalogApp can be successfully modernized and deployed to Azure Cloud within the 8-12 week timeframe.

---

## Appendix

### Resources

- [.NET Framework to .NET 8 Migration Guide](https://learn.microsoft.com/en-us/dotnet/core/porting/)
- [ASP.NET to ASP.NET Core Migration](https://learn.microsoft.com/en-us/aspnet/core/migration/)
- [Azure App Service Documentation](https://learn.microsoft.com/en-us/azure/app-service/)
- [Azure Service Bus Documentation](https://learn.microsoft.com/en-us/azure/service-bus-messaging/)
- [Azure Well-Architected Framework](https://learn.microsoft.com/en-us/azure/well-architected/)

### Tools

- [.NET Upgrade Assistant](https://dotnet.microsoft.com/en-us/platform/upgrade-assistant)
- [Azure Migrate](https://azure.microsoft.com/en-us/products/azure-migrate/)
- [Try-Convert Tool](https://github.com/dotnet/try-convert)
- [CoreWCF](https://github.com/CoreWCF/CoreWCF) - WCF for .NET Core (if REST API migration is not feasible)

---

**Document Version:** 1.0  
**Last Updated:** January 12, 2026  
**Prepared By:** GitHub Copilot Modernization Agent
