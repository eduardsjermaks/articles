
10 min


# Project Orientation (Generated)

> **Generated**: 2026-03-01 | **Repo**: eShopOnWeb | **Framework**: ASP.NET Core 8.0  
> **Epistemic labels**: FACT = directly observed in code, INFERENCE = deduced from structure, UNKNOWN = no evidence found.

---

## 1. What this project is

**Purpose** (FACT): A reference/sample application by Microsoft demonstrating a monolithic ("single-process") web application built with ASP.NET Core, following Clean Architecture principles. It is the companion code for the free eBook *"Architecting Modern Web Applications with ASP.NET Core and Azure"* ([README.md:20-24](README.md#L20-L24)).

**Who uses it / main use-cases** (FACT):
- Developers learning Clean Architecture with .NET ([README.md:7-8](README.md#L7-L8)).
- The domain is a fictional e-commerce catalog ("eShop") — browse products, add to basket, checkout as an order.
- There is an admin panel (Blazor WebAssembly) for managing catalog items.
- This is **not** a production eCommerce system; it deliberately omits features a real store would need ([README.md:33-34](README.md#L33-L34)).

> **Note**: Active development has moved to <https://github.com/dotnet/eShop>. This repo is effectively in maintenance/reference mode ([README.md:3](README.md#L3)).

---

## 2. Quick start (developer)

### Prerequisites

| Requirement | Evidence |
|---|---|
| .NET SDK 8.0.x | [global.json](global.json) `"version": "8.0.x"` |
| SQL Server LocalDB **or** set `UseOnlyInMemoryDatabase: true` | [src/Web/appsettings.json:11](src/Web/appsettings.json#L11), [src/Infrastructure/Dependencies.cs:15-18](src/Infrastructure/Dependencies.cs#L15-L18) |
| (Optional) Docker Desktop | [docker-compose.yml](docker-compose.yml) |
| (Optional) Azure Developer CLI (`azd`) for Azure deployment | [README.md:50-88](README.md#L50-L88) |

### How to run locally

**Option A — In-memory database (quickest, default)**

The default config has `UseOnlyInMemoryDatabase: true` ([src/Web/appsettings.json:11](src/Web/appsettings.json#L11)):

```bash
# Terminal 1 — PublicApi (needed for Blazor Admin)
cd src/PublicApi
dotnet run

# Terminal 2 — Web app
cd src/Web
dotnet run --launch-profile Web
```

Browse to `https://localhost:5001/`. Admin: `https://localhost:5001/admin`.

**Option B — Docker Compose**

```bash
docker-compose build
docker-compose up
```


Web at `http://localhost:5106`, API at `http://localhost:5200` ([docker-compose.override.yml:7-8](docker-compose.override.yml#L7-L8), [docker-compose.override.yml:14-15](docker-compose.override.yml#L14-L15)).

**Option C — SQL Server (LocalDB)**

Set `"UseOnlyInMemoryDatabase": false` in `appsettings.json`, then run EF migrations ([README.md:119-133](README.md#L119-L133)):

```bash
cd src/Web
dotnet ef database update -c catalogcontext -p ../Infrastructure/Infrastructure.csproj -s Web.csproj
dotnet ef database update -c appidentitydbcontext -p ../Infrastructure/Infrastructure.csproj -s Web.csproj
dotnet run
```

### How to run tests

```bash
dotnet test ./eShopOnWeb.sln --configuration Release
```

CI does exactly this ([.github/workflows/dotnetcore.yml:20-22](.github/workflows/dotnetcore.yml#L20-L22)).

### Common pitfalls

1. **Must run both Web and PublicApi** for Blazor Admin to work — the Admin panel calls the PublicApi over HTTP ([README.md:94-96](README.md#L94-L96)).
2. **File locking** — if you run `dotnet run` and then try to build, you'll get lock errors; stop the app first ([README.md:97](README.md#L97)).
3. **Docker login issues** — try with a new Incognito browser window ([README.md:161](README.md#L161)).
4. **Default credentials**: `demouser@microsoft.com` / `Pass@word1` and `admin@microsoft.com` / `Pass@word1` — seeded at startup ([src/Infrastructure/Identity/AppIdentityDbContextSeed.cs](src/Infrastructure/Identity/AppIdentityDbContextSeed.cs)).

---

## 3. Architecture at a glance

### High-level component diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Browser                                      │
│   ┌────────────────┐         ┌──────────────────────────┐           │
│   │   MVC/Razor    │         │  Blazor WASM (Admin)     │           │
│   │   Pages UI     │         │  (BlazorAdmin project)   │           │
│   └───────┬────────┘         └────────────┬─────────────┘           │
└───────────│────────────────────────────────│─────────────────────────┘
            │ HTTPS :5001                    │ HTTPS (via PublicApi)
            ▼                                ▼
┌───────────────────────┐      ┌──────────────────────────┐
│      Web (MVC)        │      │     PublicApi (REST)     │
│  Razor Pages + Ctrlrs │      │  Minimal API Endpoints   │
│  + Blazor Server      │      │  + Ardalis ApiEndpoints  │
│  (src/Web)            │      │  (src/PublicApi)          │
│                       │      │  Swagger @ /swagger       │
└─────────┬─────────────┘      └────────────┬─────────────┘
          │                                  │
          ▼                                  ▼
┌──────────────────────────────────────────────────────────┐
│             ApplicationCore (Domain Layer)                │
│  Entities (DDD aggregates) │ Interfaces │ Services       │
│  Specifications (Ardalis)  │ Extensions │ Constants      │
│  (src/ApplicationCore)                                   │
└────────────────────────────────┬─────────────────────────┘
                                 │
                                 ▼
┌──────────────────────────────────────────────────────────┐
│             Infrastructure (Persistence Layer)            │
│  EF Core DbContexts │ EfRepository<T>  │ Identity       │
│  Seed data           │ Logging adapter  │ Email stub     │
│  (src/Infrastructure)                                    │
└────────────────────────────────┬─────────────────────────┘
                                 │
              ┌──────────────────┴──────────────────┐
              ▼                                     ▼
┌──────────────────────┐          ┌──────────────────────────┐
│  CatalogDb (SQL)     │          │  IdentityDb (SQL)        │
│  Baskets, Orders,    │          │  ASP.NET Identity tables  │
│  CatalogItems, etc.  │          │                          │
└──────────────────────┘          └──────────────────────────┘
```

### Runtime processes

| Process | Project | Port (local) | Purpose |
|---|---|---|---|
| Web MVC + Blazor Server Host | `src/Web` | 5001 (HTTPS) | Storefront UI, Razor Pages, serves Blazor WASM admin assets |
| PublicApi | `src/PublicApi` | 5099 (HTTPS) | REST API consumed by Blazor Admin |
| BlazorAdmin | `src/BlazorAdmin` | N/A (WASM, runs in browser) | Admin SPA for catalog CRUD, hosted by Web |

(FACT: Two server processes, configured in [docker-compose.yml](docker-compose.yml) and confirmed in [README.md:94](README.md#L94).)

### Data stores

| Store | Technology | Evidence |
|---|---|---|
| Catalog & Orders DB | SQL Server (or InMemory for dev) | [src/Infrastructure/Data/CatalogContext.cs](src/Infrastructure/Data/CatalogContext.cs) |
| Identity DB | SQL Server (or InMemory for dev) | [src/Infrastructure/Identity/AppIdentityDbContext.cs](src/Infrastructure/Identity/AppIdentityDbContext.cs) |

(FACT: No message brokers, caches (only `IMemoryCache` in-process), or external queues observed.)

---

## 4. Repository map

### Top-level directories

| Directory/File | Purpose |
|---|---|
| `src/ApplicationCore/` | Domain layer — entities, interfaces, services, specifications. Zero infrastructure dependencies. |
| `src/Infrastructure/` | Persistence (EF Core), Identity, logging adapter, email stub. Depends on ApplicationCore. |
| `src/Web/` | ASP.NET Core MVC app — controllers, Razor Pages, view models, MediatR handlers, health checks. Hosts Blazor WASM. |
| `src/PublicApi/` | ASP.NET Core REST API — Minimal API + Ardalis ApiEndpoints, Swagger, JWT auth. |
| `src/BlazorAdmin/` | Blazor WebAssembly SPA — admin UI for catalog management. |
| `src/BlazorShared/` | Shared DTOs/models between BlazorAdmin and PublicApi. |
| `tests/UnitTests/` | xUnit unit tests (entities, services, MediatR handlers). |
| `tests/IntegrationTests/` | xUnit integration tests (EF in-memory repositories). |
| `tests/FunctionalTests/` | xUnit functional tests (WebApplicationFactory-based HTTP tests). |
| `tests/PublicApiIntegrationTests/` | MSTest integration tests for PublicApi endpoints. |
| `infra/` | Azure Bicep templates for `azd` deployment. |
| `.github/workflows/` | CI: build + test on push/PR. |

### Key modules detail

#### ApplicationCore (`src/ApplicationCore/`)

- **Responsibility**: Pure domain model — no framework dependencies beyond `Ardalis.*` and `System.Text.Json`.
- **Key files**:
  - [Entities/CatalogItem.cs](src/ApplicationCore/Entities/CatalogItem.cs) — product entity with `IAggregateRoot`
  - [Entities/BasketAggregate/Basket.cs](src/ApplicationCore/Entities/BasketAggregate/Basket.cs) — shopping basket aggregate
  - [Entities/OrderAggregate/Order.cs](src/ApplicationCore/Entities/OrderAggregate/Order.cs) — order aggregate with value objects (Address, CatalogItemOrdered)
  - [Interfaces/IRepository.cs](src/ApplicationCore/Interfaces/IRepository.cs) — generic repository (extends `Ardalis.Specification.IRepositoryBase<T>`)
  - [Services/BasketService.cs](src/ApplicationCore/Services/BasketService.cs) — add-to-basket, transfer basket, set quantities
  - [Services/OrderService.cs](src/ApplicationCore/Services/OrderService.cs) — create order from basket
  - [Specifications/](src/ApplicationCore/Specifications/) — Ardalis Specification pattern for query encapsulation

#### Infrastructure (`src/Infrastructure/`)

- **Responsibility**: EF Core persistence, ASP.NET Identity, cross-cutting concerns.
- **Key files**:
  - [Data/CatalogContext.cs](src/Infrastructure/Data/CatalogContext.cs) — DbContext for Baskets, Orders, CatalogItems, etc.
  - [Data/EfRepository.cs](src/Infrastructure/Data/EfRepository.cs) — generic repository implementation
  - [Data/CatalogContextSeed.cs](src/Infrastructure/Data/CatalogContextSeed.cs) — seeds brands, types, and 12 catalog items
  - [Identity/AppIdentityDbContext.cs](src/Infrastructure/Identity/AppIdentityDbContext.cs) — Identity DbContext
  - [Identity/AppIdentityDbContextSeed.cs](src/Infrastructure/Identity/AppIdentityDbContextSeed.cs) — seeds demo + admin users
  - [Dependencies.cs](src/Infrastructure/Dependencies.cs) — DB provider switching (InMemory vs. SQL Server)

#### Web (`src/Web/`)

- **Responsibility**: Customer-facing UI, health checks, Blazor hosting.
- **Key files**:
  - [Program.cs](src/Web/Program.cs) — app builder, DI, middleware pipeline, health checks, DB seeding
  - [Pages/Basket/Index.cshtml.cs](src/Web/Pages/Basket/Index.cshtml.cs) — add to basket, update quantities
  - [Pages/Basket/Checkout.cshtml.cs](src/Web/Pages/Basket/Checkout.cshtml.cs) — checkout flow (create order)
  - [Controllers/OrderController.cs](src/Web/Controllers/OrderController.cs) — order list/detail (uses MediatR)
  - [Controllers/ManageController.cs](src/Web/Controllers/ManageController.cs) — account management (461 lines)
  - [Configuration/ConfigureCoreServices.cs](src/Web/Configuration/ConfigureCoreServices.cs) — DI registration for domain services
  - [Configuration/ConfigureWebServices.cs](src/Web/Configuration/ConfigureWebServices.cs) — DI registration for web services + MediatR
  - [Services/CachedCatalogViewModelService.cs](src/Web/Services/CachedCatalogViewModelService.cs) — caching decorator
  - [HealthChecks/ApiHealthCheck.cs](src/Web/HealthChecks/ApiHealthCheck.cs), [HomePageHealthCheck.cs](src/Web/HealthChecks/HomePageHealthCheck.cs)

#### PublicApi (`src/PublicApi/`)

- **Responsibility**: REST API for Blazor Admin and potential external consumers.
- **Key files**:
  - [Program.cs](src/PublicApi/Program.cs) — JWT auth, Swagger, CORS, DI, DB seeding
  - [CatalogItemEndpoints/](src/PublicApi/CatalogItemEndpoints/) — CRUD endpoints for catalog items
  - [AuthEndpoints/AuthenticateEndpoint.cs](src/PublicApi/AuthEndpoints/AuthenticateEndpoint.cs) — JWT token issuance
  - [Middleware/ExceptionMiddleware.cs](src/PublicApi/Middleware/ExceptionMiddleware.cs) — global error handler

#### BlazorAdmin (`src/BlazorAdmin/`)

- **Responsibility**: Admin SPA (Blazor WASM) for catalog management.
- **Key files**:
  - [Program.cs](src/BlazorAdmin/Program.cs) — WASM host builder, HTTP client setup
  - [ServicesConfiguration.cs](src/BlazorAdmin/ServicesConfiguration.cs) — DI for catalog services + cache decorators

---

## 5. Critical flows

### Flow 1: Browse Catalog (Home Page)

- **Trigger**: `GET /` → [src/Web/Pages/Index.cshtml.cs](src/Web/Pages/Index.cshtml.cs)
- **Steps**:
  1. `CachedCatalogViewModelService.GetCatalogItems()` ([src/Web/Services/CachedCatalogViewModelService.cs](src/Web/Services/CachedCatalogViewModelService.cs)) checks `IMemoryCache`.
  2. On cache miss → `CatalogViewModelService.GetCatalogItems()` ([src/Web/Services/CatalogViewModelService.cs](src/Web/Services/CatalogViewModelService.cs)) uses `IReadRepository<CatalogItem>` with `CatalogFilterPaginatedSpecification`.
  3. EF Core query → `CatalogContext.CatalogItems` → SQL Server / InMemory.
- **Side effects**: None (read-only).
- **Failure modes**: DB connection failure → 500 error (no circuit-breaker observed).

### Flow 2: Add Item to Basket

- **Trigger**: `POST /Basket` → [src/Web/Pages/Basket/Index.cshtml.cs:36-55](src/Web/Pages/Basket/Index.cshtml.cs#L36-L55)
- **Steps**:
  1. Look up `CatalogItem` from repository to get price (prevents price tampering).
  2. `BasketService.AddItemToBasket(username, itemId, price)` ([src/ApplicationCore/Services/BasketService.cs](src/ApplicationCore/Services/BasketService.cs)).
  3. Finds existing basket by `BuyerId` or creates new one.
  4. Calls `basket.AddItem()` → persists via `IRepository<Basket>.UpdateAsync()`.
- **Side effects**: Basket row created/updated in DB.
- **Failure modes**: If catalog item doesn't exist, redirects to home. DB write failure → unhandled 500.

### Flow 3: Checkout (Create Order)

- **Trigger**: `POST /Basket/Checkout` → [src/Web/Pages/Basket/Checkout.cshtml.cs:48-66](src/Web/Pages/Basket/Checkout.cshtml.cs#L48-L66)
- **Steps**:
  1. `BasketService.SetQuantities()` — updates basket quantities.
  2. `OrderService.CreateOrderAsync(basketId, address)` ([src/ApplicationCore/Services/OrderService.cs](src/ApplicationCore/Services/OrderService.cs)):
     - Loads basket with items via specification.
     - Guards against empty basket ([src/ApplicationCore/Extensions/GuardExtensions.cs](src/ApplicationCore/Extensions/GuardExtensions.cs)).
     - Loads each `CatalogItem` for current price snapshots.
     - Creates `Order` aggregate with `OrderItem`s → persists.
  3. `BasketService.DeleteBasketAsync()` — clears basket.
- **Side effects**: Order persisted; basket deleted. **No email sent** (email sender is a stub — [src/Infrastructure/Services/EmailSender.cs:12](src/Infrastructure/Services/EmailSender.cs#L12)).
- **Failure modes**: 
  - Empty basket → `EmptyBasketOnCheckoutException` → redirect to Basket page.
  - **RISK**: No transaction wrapping order creation + basket deletion — if delete fails, order exists but basket remains (INFERENCE — no explicit `TransactionScope` observed).
  - Address is hardcoded: `"123 Main St.", "Kent", "OH"` ([src/Web/Pages/Basket/Checkout.cshtml.cs:61](src/Web/Pages/Basket/Checkout.cshtml.cs#L61)). This is a sample app limitation.

### Flow 4: Blazor Admin — Edit Catalog Item

- **Trigger**: Admin user navigates to `/admin` (Blazor WASM).
- **Steps**:
  1. BlazorAdmin authenticates via `POST /api/authenticate` → [src/PublicApi/AuthEndpoints/AuthenticateEndpoint.cs](src/PublicApi/AuthEndpoints/AuthenticateEndpoint.cs).
  2. JWT stored in browser `localStorage` ([src/BlazorAdmin/Program.cs:39-48](src/BlazorAdmin/Program.cs#L39-L48)).
  3. CRUD calls to PublicApi → `CatalogItemEndpoints` → `EfRepository<CatalogItem>`.
- **Side effects**: Catalog item updated in DB.
- **Failure modes**: JWT expired/invalid → 401. CORS misconfigured → request blocked.

---

## 6. Dependencies

### Internal module dependencies

```
BlazorAdmin ──► BlazorShared
     │
     └──► (HTTP calls to PublicApi at runtime)

Web ──► ApplicationCore
Web ──► Infrastructure
Web ──► BlazorAdmin (hosts WASM assets)
Web ──► BlazorShared

PublicApi ──► ApplicationCore
PublicApi ──► Infrastructure

Infrastructure ──► ApplicationCore

ApplicationCore ──► BlazorShared (for shared models)
```

(FACT: Confirmed from `.csproj` `<ProjectReference>` elements in [src/Web/Web.csproj:42-46](src/Web/Web.csproj#L42-L46), [src/PublicApi/PublicApi.csproj:34-36](src/PublicApi/PublicApi.csproj#L34-L36), [src/Infrastructure/Infrastructure.csproj:15](src/Infrastructure/Infrastructure.csproj#L15), [src/ApplicationCore/ApplicationCore.csproj:16](src/ApplicationCore/ApplicationCore.csproj#L16).)

**Note on `ApplicationCore ──► BlazorShared`** (INFERENCE: This is a design smell — the domain layer references a UI-shared models library. See Section 10.)

### External services

| Service | Usage | Evidence |
|---|---|---|
| SQL Server / LocalDB | Primary data store | [src/Infrastructure/Dependencies.cs:34-35](src/Infrastructure/Dependencies.cs#L34-L35) |
| Azure SQL (prod) | Production DB via Key Vault | [src/Web/Program.cs:32-43](src/Web/Program.cs#L32-L43) |
| Azure Key Vault (prod) | Secrets (connection strings) | [src/Web/Program.cs:31-32](src/Web/Program.cs#L31-L32) |
| Email (stub) | Not wired — `Task.CompletedTask` | [src/Infrastructure/Services/EmailSender.cs:12](src/Infrastructure/Services/EmailSender.cs#L12) |

### Build/deploy dependencies

| Tool | Purpose | Evidence |
|---|---|---|
| .NET SDK 8.0.x | Build + run | [global.json](global.json) |
| Central package management | Version pinning | [Directory.Packages.props](Directory.Packages.props) |
| GitHub Actions | CI: build + test | [.github/workflows/dotnetcore.yml](.github/workflows/dotnetcore.yml) |
| Azure Developer CLI (`azd`) | Azure deployment | [azure.yaml](azure.yaml), [infra/main.bicep](infra/main.bicep) |
| Docker / docker-compose | Containerized local & CI runs | [docker-compose.yml](docker-compose.yml) |
| LibMan | Client-side library management | [src/Web/libman.json](src/Web/libman.json) |

### Key NuGet packages

| Package | Version | Purpose | Evidence |
|---|---|---|---|
| Ardalis.Specification | 7.0.0 | Specification pattern for queries | [Directory.Packages.props:15](Directory.Packages.props#L15) |
| Ardalis.GuardClauses | 4.0.1 | Guard clauses (input validation) | [Directory.Packages.props:12](Directory.Packages.props#L12) |
| Ardalis.ApiEndpoints | 4.1.0 | Endpoint-per-action pattern | [Directory.Packages.props:11](Directory.Packages.props#L11) |
| MediatR | 12.0.1 | Mediator pattern (CQRS-lite) | [Directory.Packages.props:24](Directory.Packages.props#L24) |
| AutoMapper | 12.0.1 | Object-object mapping (PublicApi) | [Directory.Packages.props:19](Directory.Packages.props#L19) |
| EF Core (SQL Server) | 8.0.2 | ORM | [Directory.Packages.props:42](Directory.Packages.props#L42) |
| Swashbuckle | 6.5.0 | Swagger/OpenAPI | [Directory.Packages.props:55](Directory.Packages.props#L55) |

---

## 7. Configuration & environments

### Where config lives

| File | Scope |
|---|---|
| [src/Web/appsettings.json](src/Web/appsettings.json) | Web defaults (in-memory DB on by default) |
| [src/Web/appsettings.Docker.json](src/Web/appsettings.Docker.json) | Docker overrides (SQL Server via `sqlserver:1433`) |
| [src/Web/appsettings.Development.json](src/Web/appsettings.Development.json) | Development overrides |
| [src/PublicApi/appsettings.json](src/PublicApi/appsettings.json) | PublicApi defaults |
| [src/PublicApi/appsettings.Docker.json](src/PublicApi/appsettings.Docker.json) | PublicApi Docker overrides |
| [infra/main.bicep](infra/main.bicep) | Azure infrastructure-as-code |
| [infra/main.parameters.json](infra/main.parameters.json) | Bicep parameters |

### Key configuration values

| Key | Purpose | Where |
|---|---|---|
| `UseOnlyInMemoryDatabase` | Toggle in-memory vs SQL Server | [src/Web/appsettings.json:11](src/Web/appsettings.json#L11), [src/Infrastructure/Dependencies.cs:15](src/Infrastructure/Dependencies.cs#L15) |
| `ConnectionStrings:CatalogConnection` | Catalog DB connection string | [src/Web/appsettings.json:8](src/Web/appsettings.json#L8) |
| `ConnectionStrings:IdentityConnection` | Identity DB connection string | [src/Web/appsettings.json:9](src/Web/appsettings.json#L9) |
| `baseUrls:apiBase` | PublicApi base URL | [src/Web/appsettings.json:3](src/Web/appsettings.json#L3) |
| `baseUrls:webBase` | Web base URL (for CORS) | [src/Web/appsettings.json:4](src/Web/appsettings.json#L4) |
| `CatalogBaseUrl` | Override for catalog image URLs | [src/Web/appsettings.json:12](src/Web/appsettings.json#L12) |

### Secrets handling

- **Local dev**: ASP.NET Core User Secrets supported (UserSecretsId in [src/Web/Web.csproj:7](src/Web/Web.csproj#L7) and [src/PublicApi/PublicApi.csproj:5](src/PublicApi/PublicApi.csproj#L5)).
- **Production (Azure)**: Azure Key Vault loaded via `ChainedTokenCredential` ([src/Web/Program.cs:31-32](src/Web/Program.cs#L31-L32)).
- **Docker**: Passwords in plaintext in `appsettings.Docker.json` and `docker-compose.yml` — `SA_PASSWORD=@someThingComplicated1234` ([docker-compose.yml:23](docker-compose.yml#L23)).
- **WARNING**: JWT secret key and default password are hardcoded constants ([src/ApplicationCore/Constants/AuthorizationConstants.cs:6-11](src/ApplicationCore/Constants/AuthorizationConstants.cs#L6-L11)). The source code itself has `// TODO: Don't use this in production` and `// TODO: Change this to an environment variable`.

### Local vs. prod differences

| Aspect | Local (Development/Docker) | Production (Azure) |
|---|---|---|
| DB provider | InMemory or LocalDB / Docker SQL | Azure SQL via Key Vault | 
| DB config | [src/Infrastructure/Dependencies.cs](src/Infrastructure/Dependencies.cs) | [src/Web/Program.cs:30-43](src/Web/Program.cs#L30-L43) |
| Auth secrets | Hardcoded constants | Same hardcoded constants (RISK) |
| Error pages | `UseDeveloperExceptionPage()` | `UseExceptionHandler("/Error")` + HSTS |

---

## 8. Observability

### Logging

- **Framework**: `Microsoft.Extensions.Logging` with console provider ([src/Web/Program.cs:23](src/Web/Program.cs#L23), [src/PublicApi/Program.cs:33](src/PublicApi/Program.cs#L33)).
- **Abstraction**: `IAppLogger<T>` interface in ApplicationCore ([src/ApplicationCore/Interfaces/IAppLogger.cs](src/ApplicationCore/Interfaces/IAppLogger.cs)) → `LoggerAdapter<T>` in Infrastructure ([src/Infrastructure/Logging/LoggerAdapter.cs](src/Infrastructure/Logging/LoggerAdapter.cs)).
- **Levels configured in appsettings**: Default `Warning`, downgraded to `Debug`/`Information` in Docker ([src/Web/appsettings.Docker.json:9-13](src/Web/appsettings.Docker.json#L9-L13)).
- (FACT: Informational logs present at startup for seeding and launching — [src/Web/Program.cs:119-120](src/Web/Program.cs#L119-L120), [src/Web/Program.cs:200](src/Web/Program.cs#L200).)

### Metrics

UNKNOWN — No metrics libraries (Prometheus, OpenTelemetry, Application Insights SDK) observed in `Directory.Packages.props` or `Program.cs` files.

### Tracing

UNKNOWN — No distributed tracing (OpenTelemetry, Application Insights) observed.

### Health checks

- **Two health checks registered** ([src/Web/Program.cs:91-93](src/Web/Program.cs#L91-L93)):
  - `ApiHealthCheck` — calls `/api/catalog-items` and checks for product name in response ([src/Web/HealthChecks/ApiHealthCheck.cs](src/Web/HealthChecks/ApiHealthCheck.cs)).
  - `HomePageHealthCheck` — calls home page and checks for product name ([src/Web/HealthChecks/HomePageHealthCheck.cs](src/Web/HealthChecks/HomePageHealthCheck.cs)).
- **Endpoints**: `/health` (all checks), `/home_page_health_check`, `/api_health_check` ([src/Web/Program.cs:155-168](src/Web/Program.cs#L155-L168), [src/Web/Program.cs:192-193](src/Web/Program.cs#L192-L193)).
- (INFERENCE: Health checks rely on content matching — fragile if seed data changes.)

### Alerts

UNKNOWN — No alerting configuration found in the repository.

---

## 9. Testing & quality gates

### Test projects

| Project | Framework | Type | Key areas tested |
|---|---|---|---|
| `tests/UnitTests/` | xUnit + NSubstitute | Unit | Entity behavior (Basket, Order), services (BasketService), specifications, MediatR handlers |
| `tests/IntegrationTests/` | xUnit + EF InMemory | Integration | Repository operations (Basket, Order) |
| `tests/FunctionalTests/` | xUnit + WebApplicationFactory | Functional/E2E | Controller HTTP responses, Razor page rendering, auth endpoints |
| `tests/PublicApiIntegrationTests/` | MSTest + WebApplicationFactory | Integration | PublicApi endpoint responses, pagination, concurrency stress test |

### CI pipeline

(FACT: [.github/workflows/dotnetcore.yml](.github/workflows/dotnetcore.yml)):
- Triggers: push, pull_request, workflow_dispatch.
- Steps: checkout → setup .NET 8.0.x → `dotnet build` → `dotnet test` (Release config).
- **No code coverage reporting in CI** (coverage settings file exists: [CodeCoverage.runsettings](CodeCoverage.runsettings) but is not referenced in the workflow).

### Coverage gaps / risks

| Gap | Evidence | Risk |
|---|---|---|
| **No tests for PublicApi CRUD** mutations beyond basic list/get | Only `Create` and `Delete` test files present under `tests/PublicApiIntegrationTests/` — UNKNOWN if fully implemented | Medium |
| **ManageController (461 lines)** has no dedicated tests | No test file found targeting `ManageController` | Medium |
| **BlazorAdmin has no tests** | No test project referencing `BlazorAdmin` found | Low (admin-only) |
| **Mixed test frameworks** (xUnit + MSTest) | `tests/PublicApiIntegrationTests/` uses MSTest ([Directory.Packages.props:66-67](Directory.Packages.props#L66-L67)) while all others use xUnit | Low (maintenance friction) |
| **Email sending untested** (it's a stub) | [src/Infrastructure/Services/EmailSender.cs](src/Infrastructure/Services/EmailSender.cs) | Low (no-op) |

---

## 10. Migration hotspots / tech debt (evidence-based)

### Hotspot 1: Hardcoded secrets in source code

- **Evidence**: [src/ApplicationCore/Constants/AuthorizationConstants.cs:6-11](src/ApplicationCore/Constants/AuthorizationConstants.cs#L6-L11) — `AUTH_KEY`, `DEFAULT_PASSWORD`, `JWT_SECRET_KEY` as `const string`.
- **Risk type**: Configuration risk | Security
- **Confidence**: High
- **Impact**: If deployed as-is, JWT tokens can be forged by anyone who reads the source.
- **Suggested fix**: Move to environment variables or Azure Key Vault. Facade: read from `IConfiguration` at startup.
- **Migration risk**: Low — replace constants with config lookups; existing tests pass secrets via config.

### Hotspot 2: `ApplicationCore` depends on `BlazorShared`

- **Evidence**: [src/ApplicationCore/ApplicationCore.csproj:16](src/ApplicationCore/ApplicationCore.csproj#L16) — `<ProjectReference Include="..\BlazorShared\BlazorShared.csproj" />`
- **Risk type**: Coupling | Mixed concerns
- **Confidence**: High
- **Impact**: The domain layer (which should have zero UI dependencies) references a UI-shared library. This creates a coupling violation in Clean Architecture.
- **Suggested refactoring**: Extract shared DTOs used by ApplicationCore into a separate `SharedKernel` project, or move the relevant types into ApplicationCore directly.
- **Migration risk**: Medium — requires moving `CatalogSettings` or shared models and updating all consumers.

### Hotspot 3: `ManageController` god class (461 lines)

- **Evidence**: [src/Web/Controllers/ManageController.cs](src/Web/Controllers/ManageController.cs) — 461 lines, handles account management, 2FA, password changes, etc.
- **Risk type**: Mixed concerns | Lack of tests
- **Confidence**: High
- **Impact**: Difficult to test, maintain, or extend. No dedicated test coverage found.
- **Suggested refactoring**: Split into focused controllers (PasswordController, TwoFactorController, etc.) or migrate to MediatR handlers like OrderController does.
- **Migration risk**: Medium — many views may reference this controller's action names.

### Hotspot 4: `IMemoryCache` for auth revocation in multi-host scenarios

- **Evidence**: [src/Web/Configuration/RevokeAuthenticationEvents.cs:11](src/Web/Configuration/RevokeAuthenticationEvents.cs#L11) — `//TODO : replace IMemoryCache with a distributed cache if you are in multi-host scenario`. Same TODO at [src/Web/Areas/Identity/Pages/Account/Logout.cshtml.cs:13](src/Web/Areas/Identity/Pages/Account/Logout.cshtml.cs#L13).
- **Risk type**: Hidden side-effects
- **Confidence**: Medium (only matters if scaling horizontally)
- **Impact**: In a multi-instance deployment, cookie revocation would not propagate across instances.
- **Suggested fix**: Replace `IMemoryCache` with `IDistributedCache` (Redis).
- **Migration risk**: Low — interface is similar.

### Hotspot 5: Duplicate DB seeding in both Web and PublicApi

- **Evidence**: Both [src/Web/Program.cs:122-136](src/Web/Program.cs#L122-L136) and [src/PublicApi/Program.cs:135-149](src/PublicApi/Program.cs#L135-L149) run `CatalogContextSeed.SeedAsync()` and `AppIdentityDbContextSeed.SeedAsync()` at startup.
- **Risk type**: Coupling | Hidden side-effects
- **Confidence**: High
- **Impact**: Running both processes against the same database could cause seed race conditions. Seeding logic is duplicated rather than centralized.
- **Suggested fix**: Move seeding to a separate migration/CLI tool or make it truly idempotent with concurrency guards.
- **Migration risk**: Low.

### Hotspot 6: No transaction around checkout (order creation + basket deletion)

- **Evidence**: [src/Web/Pages/Basket/Checkout.cshtml.cs:60-62](src/Web/Pages/Basket/Checkout.cshtml.cs#L60-L62) — `CreateOrderAsync` then `DeleteBasketAsync` are separate calls with no explicit transaction.
- **Risk type**: Hidden side-effects
- **Confidence**: Medium (INFERENCE — EF `SaveChanges` in each repository call is a separate transaction)
- **Impact**: If `DeleteBasketAsync` fails after order is created, basket persists as a phantom.
- **Suggested fix**: Wrap in a `TransactionScope` or a Unit of Work pattern.
- **Migration risk**: Low.

### Hotspot 7: Email sender is a stub

- **Evidence**: [src/Infrastructure/Services/EmailSender.cs:12](src/Infrastructure/Services/EmailSender.cs#L12) — `// TODO: Wire this up to actual email sending logic via SendGrid, local SMTP, etc.`
- **Risk type**: Missing feature
- **Confidence**: High
- **Impact**: No order confirmations or notifications are sent.
- **Suggested fix**: Implement with SendGrid or similar; interface is already clean (`IEmailSender`).
- **Migration risk**: Low — plug-in implementation.

### Hotspot 8: Hardcoded shipping address in checkout

- **Evidence**: [src/Web/Pages/Basket/Checkout.cshtml.cs:61](src/Web/Pages/Basket/Checkout.cshtml.cs#L61) — `new Address("123 Main St.", "Kent", "OH", "United States", "44240")`.
- **Risk type**: Missing feature
- **Confidence**: High
- **Impact**: Sample limitation — no real address collection from user.
- **Migration risk**: Low.

---

## 11. Open questions

- **How is the PublicApi secured in production?** JWT secret is a hardcoded constant. Is there a production override mechanism beyond the source code? → Inspect Azure Key Vault config in deployed environment.
- **Is there any APM / monitoring in production?** No Application Insights or OpenTelemetry SDK observed. → Check Azure App Service configuration and Bicep outputs.
- **What happens with concurrent basket modifications?** No optimistic concurrency tokens observed on `Basket` entity. → Inspect EF migrations and entity configurations for `RowVersion`/`ConcurrencyCheck`.
- **Is the `Everything.sln` used?** Referenced in [.github/workflows/richnav.yml](.github/workflows/richnav.yml) for code indexing. Does it include additional projects? → Open and inspect.
- **What does the `.devcontainer/` setup look like?** Referenced in [README.md:150](README.md#L150) but not explored. → Inspect `.devcontainer/` directory.
- **Are EF migrations checked in?** `src/Infrastructure/Data/Migrations/` folder exists as an empty include in `.csproj`. → Inspect for migration files.
- **BlazorAdmin cache duration**: `// TODO: Get Default Cache Duration from Config` in [src/BlazorAdmin/CustomAuthStateProvider.cs:15](src/BlazorAdmin/CustomAuthStateProvider.cs#L15). What's the current hardcoded value?

---

## 12. First 90 minutes checklist

1. **[5 min]** Read this document end-to-end.
2. **[5 min]** Read the original [README.md](README.md) for additional context and the eBook link.
3. **[10 min]** Run the app locally with in-memory DB:
   ```bash
   # Terminal 1
   cd src/PublicApi && dotnet run
   # Terminal 2
   cd src/Web && dotnet run --launch-profile Web
   ```
   Browse `https://localhost:5001/`, add items to basket, go through checkout.
4. **[5 min]** Log in as `admin@microsoft.com` / `Pass@word1`, visit `/admin` to see the Blazor Admin panel.
5. **[10 min]** Trace the checkout flow in code: start at [src/Web/Pages/Basket/Checkout.cshtml.cs](src/Web/Pages/Basket/Checkout.cshtml.cs) → `OrderService.CreateOrderAsync` → `CatalogContext`.
6. **[10 min]** Read the DI setup: [src/Web/Configuration/ConfigureCoreServices.cs](src/Web/Configuration/ConfigureCoreServices.cs) and [ConfigureWebServices.cs](src/Web/Configuration/ConfigureWebServices.cs). Understand how services are wired.
7. **[10 min]** Read the domain model: [src/ApplicationCore/Entities/](src/ApplicationCore/Entities/) — focus on `Basket`, `Order`, and `CatalogItem` aggregates.
8. **[10 min]** Run the tests: `dotnet test ./eShopOnWeb.sln` — note which pass and how fast.
9. **[10 min]** Explore the PublicApi via Swagger: run PublicApi, navigate to `https://localhost:5099/swagger`.
10. **[5 min]** Look at the CI pipeline: [.github/workflows/dotnetcore.yml](.github/workflows/dotnetcore.yml).
11. **[10 min]** Skim the infrastructure: [infra/main.bicep](infra/main.bicep) for Azure deployment topology, [docker-compose.yml](docker-compose.yml) for local multi-container setup.

At this point you should be able to confidently navigate the codebase, understand the request lifecycle, and identify where to make changes for any feature work. Consult Section 10 (tech debt) before proposing architectural changes.
