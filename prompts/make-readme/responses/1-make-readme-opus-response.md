
10 min


# Project Orientation (Generated)

> **Generated on:** 2026-03-15
> **Confidence model:** FACT = directly observable in code (cited); INFERENCE = deduced from structure; UNKNOWN = no evidence found.
> Every non-trivial claim cites evidence as `(path:lines)` or `(path)`.

---

## 1. What this project is

### Purpose
**FACT:** eShopOnWeb is a reference application demonstrating **Clean Architecture** and **Domain-Driven Design** patterns in ASP.NET Core 8.
Evidence: `README.md`, project structure, `ApplicationCore/` dependency isolation.

### Who uses it / main use-cases
- **FACT:** End-users browse a product catalog, manage a shopping basket, and place orders via a Razor Pages web UI (`src/Web/Pages/`).
- **FACT:** Administrators manage catalog items, brands, and types via a Blazor WebAssembly SPA (`src/BlazorAdmin/`) served from the Web host.
- **FACT:** A REST API (`src/PublicApi/`) exposes catalog CRUD and authentication endpoints, consumed by BlazorAdmin.
- **INFERENCE:** This is a teaching/reference codebase, not a production e-commerce platform — hardcoded demo credentials and TODOs throughout suggest it is intentionally simplified. Evidence: `src/ApplicationCore/Constants/AuthorizationConstants.cs:8,11`.

---

## 2. Quick start (developer)

### Prereqs
- **FACT:** .NET 8 SDK — pinned in `global.json` (root).
- **FACT:** Docker + Docker Compose (optional) for SQL Server — `docker-compose.yml`, `docker-compose.override.yml`.
- **FACT:** No SQL Server required by default: `UseOnlyInMemoryDatabase: true` is the default (`src/Web/appsettings.json`).

### How to run locally

**Option A — In-memory (no DB setup needed):**
```bash
cd src/Web
dotnet run
# Web UI at https://localhost:44315 (or check launchSettings.json)
```
```bash
cd src/PublicApi
dotnet run
# API at https://localhost:5099  (Swagger at /swagger)
```

**Option B — Docker Compose (SQL Server):**
```bash
docker-compose up
# Web: http://localhost:5106
# API: http://localhost:5200
```
FACT: Ports from `docker-compose.override.yml:5106:8080` (web), `5200:8080` (api).

**Demo credentials (seeded at startup):**
- Regular user: `demouser@microsoft.com` / `Pass@word1`
- Admin: `admin@microsoft.com` / `Pass@word1`

FACT: `src/Infrastructure/Identity/AppIdentityDbContextSeed.cs`.

### How to run tests
```bash
dotnet test eShopOnWeb.sln
# or specifically:
dotnet test tests/UnitTests
dotnet test tests/IntegrationTests
dotnet test tests/FunctionalTests
dotnet test tests/PublicApiIntegrationTests
```
FACT: CI workflow runs `dotnet test` on `eShopOnWeb.sln` (`.github/workflows/dotnetcore.yml`).

### Common pitfalls
- **FACT:** `UseOnlyInMemoryDatabase: true` is the default — data does not persist between restarts. Change to `false` and supply a connection string for persistence.
- **FACT:** Docker environment uses hardcoded SA password `@someThingComplicated1234` (`src/Web/appsettings.Docker.json`) — only for local Docker use.
- **FACT:** JWT secret key is hardcoded (`AuthorizationConstants.JWT_SECRET_KEY`) with a TODO to move to env var (`src/ApplicationCore/Constants/AuthorizationConstants.cs:11`). Do not use as-is in any exposed environment.
- **FACT:** BlazorAdmin SPA is served as part of the Web host — it is **not** a standalone deployment.
- **FACT:** Health checks (`/health`) call the PublicApi and expect a specific product name in the response (`src/Web/HealthChecks/ApiHealthCheck.cs`) — if API is not running, health check fails.

---

## 3. Architecture at a glance

### High-level component diagram

```
┌─────────────────────────────────────────────────────┐
│                    Browser                           │
└────────┬────────────────────────┬───────────────────┘
         │ Razor Pages / MVC      │ Blazor WASM (SPA)
         ▼                        ▼
┌─────────────────┐     ┌──────────────────────┐
│  src/Web        │     │  src/BlazorAdmin     │
│  (ASP.NET Core) │     │  (WebAssembly, WASM) │
│  + Blazor Server│     │  mounted on #admin   │
│    hosting WASM │     └──────────┬───────────┘
└────────┬────────┘                │ HTTP (JWT)
         │                         ▼
         │              ┌──────────────────────┐
         │              │  src/PublicApi        │
         │              │  (Minimal API / REST) │
         │              └──────────┬────────────┘
         │                         │
         └──────────┬──────────────┘
                    │ (both depend on)
         ┌──────────▼──────────────┐
         │  src/ApplicationCore    │  ← Domain (no EF)
         └──────────┬──────────────┘
                    │
         ┌──────────▼──────────────┐
         │  src/Infrastructure     │  ← EF, Identity, Email
         └──────────┬──────────────┘
                    │
         ┌──────────▼──────────────┐
         │  SQL Server / In-Memory │  ← CatalogContext
         │  (2 databases)          │    AppIdentityDbContext
         └─────────────────────────┘
```

FACT: Project reference graph from `.csproj` files:
`Web → ApplicationCore + Infrastructure + BlazorAdmin + BlazorShared`
`PublicApi → ApplicationCore + Infrastructure`
`BlazorAdmin → BlazorShared`
`ApplicationCore → BlazorShared`
`Infrastructure → ApplicationCore`

### Runtime processes
- **Web** — Single ASP.NET Core process hosting: MVC controllers, Razor Pages, Blazor Server circuit, and serving the BlazorAdmin WASM bundle.
- **PublicApi** — Separate ASP.NET Core process. Exposes REST endpoints. Swagger UI at `/swagger`.
- **No background workers / schedulers** — UNKNOWN: no evidence of `IHostedService`, message consumers, or cron-style workers.

### Data stores and queues
- **FACT:** `CatalogContext` (EF Core) — catalog items, baskets, orders (`src/Infrastructure/Data/CatalogContext.cs`).
- **FACT:** `AppIdentityDbContext` (EF Core) — ASP.NET Identity users/roles (`src/Infrastructure/Identity/AppIdentityDbContext.cs`).
- **FACT:** In-memory cache via `IMemoryCache` — used for catalog view model caching and auth revocation (`src/Web/Services/CachedCatalogViewModelService.cs`, `src/Web/Configuration/RevokeAuthenticationEvents.cs`).
- **UNKNOWN:** No message broker (RabbitMQ, Azure Service Bus, etc.) — no evidence found.

---

## 4. Repository map

```
eShopOnWeb/
├── src/
│   ├── ApplicationCore/      # Domain layer — entities, specs, services, interfaces
│   ├── BlazorAdmin/          # Blazor WASM admin SPA
│   ├── BlazorShared/         # DTOs, validators shared between WASM and server
│   ├── Infrastructure/       # EF Core, Identity, email, logging adapter
│   ├── PublicApi/            # REST API (catalog CRUD, auth)
│   └── Web/                  # MVC + Razor Pages UI (also hosts BlazorAdmin)
├── tests/
│   ├── UnitTests/            # xUnit, NSubstitute — domain logic
│   ├── IntegrationTests/     # xUnit — EF in-memory repository tests
│   ├── FunctionalTests/      # xUnit, WebApplicationFactory — end-to-end HTTP
│   └── PublicApiIntegrationTests/ # MSTest, WebApplicationFactory — PublicApi
├── infra/                    # Azure Bicep IaC (App Service, SQL, Key Vault)
├── .github/workflows/        # GitHub Actions CI
│   ├── agents/               # AI agent prompt files (speckit / code-gen)
│   └── prompts/              # AI prompt files
├── docker-compose.yml        # Multi-service local Docker setup
├── Directory.Packages.props  # Central NuGet version management (all versions here)
├── global.json               # .NET SDK version pin
└── azure.yaml                # Azure Developer CLI (azd) config
```

### Module details

#### `src/ApplicationCore/` — Domain layer
**Responsibility:** Core business entities, aggregates, domain services, repository interfaces, and Ardalis Specifications. Zero dependency on EF or ASP.NET Core.
**Key public interfaces:**
- `IRepository<T>`, `IReadRepository<T>` (`Interfaces/`) — repository contracts
- `IBasketService`, `IOrderService` — application service contracts
- `IAppLogger<T>` — logging abstraction

**Key files:**
- [Entities/BasketAggregate/Basket.cs](src/ApplicationCore/Entities/BasketAggregate/Basket.cs) — basket aggregate root
- [Entities/OrderAggregate/Order.cs](src/ApplicationCore/Entities/OrderAggregate/Order.cs) — order aggregate root
- [Entities/CatalogItem.cs](src/ApplicationCore/Entities/CatalogItem.cs) — catalog item aggregate root
- [Services/BasketService.cs](src/ApplicationCore/Services/BasketService.cs) — basket business logic
- [Services/OrderService.cs](src/ApplicationCore/Services/OrderService.cs) — order creation logic
- [Constants/AuthorizationConstants.cs](src/ApplicationCore/Constants/AuthorizationConstants.cs) — JWT key, default password (**hardcoded, see pitfalls**)
- [Specifications/](src/ApplicationCore/Specifications/) — 7 query specification classes

#### `src/Infrastructure/` — Infrastructure layer
**Responsibility:** EF Core DbContexts, repository implementations (via Ardalis.Specification.EF), ASP.NET Identity, JWT token service, email sender, logging adapter, DB seeding.
**Key files:**
- [Data/CatalogContext.cs](src/Infrastructure/Data/CatalogContext.cs) — main EF DbContext (basket, catalog, orders)
- [Identity/AppIdentityDbContext.cs](src/Infrastructure/Identity/AppIdentityDbContext.cs) — identity EF DbContext
- [Data/EfRepository.cs](src/Infrastructure/Data/EfRepository.cs) — generic EF repository
- [Dependencies.cs](src/Infrastructure/Dependencies.cs) — DI bootstrap (`ConfigureServices`) — switches in-memory vs SQL Server
- [Identity/IdentityTokenClaimService.cs](src/Infrastructure/Identity/IdentityTokenClaimService.cs) — JWT generation (7-day expiry, HMAC-SHA256)
- [Identity/AppIdentityDbContextSeed.cs](src/Infrastructure/Identity/AppIdentityDbContextSeed.cs) — seeds demo/admin users
- [Data/Config/](src/Infrastructure/Data/Config/) — EF fluent configuration per entity

#### `src/Web/` — Web UI
**Responsibility:** Razor Pages UI, MVC controllers, Blazor Server circuit (hosting WASM), cookie auth, health checks, caching layer, MediatR handlers for orders.
**Key files:**
- [Program.cs](src/Web/Program.cs) — startup: DB config, auth, health checks, seeding, routing
- [Pages/](src/Web/Pages/) — Razor Pages (Index, Basket, Checkout, Admin)
- [Controllers/](src/Web/Controllers/) — OrderController, ManageController, UserController
- [Services/CachedCatalogViewModelService.cs](src/Web/Services/CachedCatalogViewModelService.cs) — in-memory cached catalog
- [Configuration/RevokeAuthenticationEvents.cs](src/Web/Configuration/RevokeAuthenticationEvents.cs) — cookie revocation via IMemoryCache
- [HealthChecks/](src/Web/HealthChecks/) — ApiHealthCheck, HomePageHealthCheck
- [Extensions/ServiceCollectionExtensions.cs](src/Web/Extensions/ServiceCollectionExtensions.cs) — `AddCoreServices()`, `AddWebServices()`

#### `src/PublicApi/` — REST API
**Responsibility:** Catalog item/brand/type CRUD, JWT authentication endpoint. Swagger UI. JWT Bearer auth for write operations.
**Key files:**
- [Program.cs](src/PublicApi/Program.cs) — startup: JWT auth, CORS, Swagger, ExceptionMiddleware
- [CatalogItemEndpoints/](src/PublicApi/CatalogItemEndpoints/) — 5 endpoints (List, GetById, Create, Update, Delete)
- [AuthEndpoints/AuthenticateEndpoint.cs](src/PublicApi/AuthEndpoints/AuthenticateEndpoint.cs) — `POST /api/authenticate`
- [Middleware/ExceptionMiddleware.cs](src/PublicApi/Middleware/ExceptionMiddleware.cs) — global error handler
- [MappingProfile.cs](src/PublicApi/MappingProfile.cs) — AutoMapper: entity → DTO

#### `src/BlazorAdmin/` — Admin SPA
**Responsibility:** WebAssembly admin UI for catalog management. Calls PublicApi via HttpClient with JWT token.
**Key files:**
- [Program.cs](src/BlazorAdmin/Program.cs) — WASM entry point, mounts on `#admin`
- [Pages/](src/BlazorAdmin/Pages/) — Blazor page components
- [Services/HttpService.cs](src/BlazorAdmin/Services/HttpService.cs) — HTTP abstraction calling PublicApi

#### `src/BlazorShared/` — Shared models
**Responsibility:** DTOs and FluentValidation validators shared between BlazorAdmin (WASM) and server-side code.

---

## 5. Critical flows

### Flow 1: Unauthenticated catalog browse → add to basket
**Trigger:** User visits home page (`GET /`)

1. `Pages/Index.cshtml.cs:OnGetAsync()` calls `ICatalogViewModelService.GetCatalogItems()` — goes through `CachedCatalogViewModelService` → `CatalogViewModelService` → `IReadRepository<CatalogItem>` → EF query with `CatalogFilterPaginatedSpecification`.
   Evidence: [src/Web/Pages/Index.cshtml.cs](src/Web/Pages/Index.cshtml.cs), [src/Web/Services/CachedCatalogViewModelService.cs](src/Web/Services/CachedCatalogViewModelService.cs)

2. User clicks "Add to Cart" → `POST /Basket` (anonymous — basket identified by cookie claim `BasketService.cs:GetOrCreateBasketForUser()`).
   Evidence: [src/Web/Pages/Basket/Index.cshtml.cs](src/Web/Pages/Basket/Index.cshtml.cs), [src/ApplicationCore/Services/BasketService.cs](src/ApplicationCore/Services/BasketService.cs)

3. `BasketService.AddItemToBasket()` loads or creates basket via `IRepository<Basket>`, adds `BasketItem`, saves.

**Side effects:** New `Basket` + `BasketItem` row written to `CatalogContext`.
**Failure modes:** In-memory: no persistence across restart. SQL Server: transaction on `CatalogContext`. No stock check — **INFERENCE:** no inventory guard.

---

### Flow 2: Login → basket transfer
**Trigger:** User logs in via `POST /Account/Login`

1. `Areas/Identity/Pages/Account/Login.cshtml.cs:OnPostAsync()` calls `SignInManager.PasswordSignInAsync()`.
2. On success → calls `BasketService.TransferBasketAsync(anonymousId, userName)`.
   Evidence: [src/Web/Areas/Identity/Pages/Account/Login.cshtml.cs](src/Web/Areas/Identity/Pages/Account/Login.cshtml.cs), [src/ApplicationCore/Services/BasketService.cs](src/ApplicationCore/Services/BasketService.cs)
3. `TransferBasketAsync` loads both anonymous and user baskets, merges items, deletes anonymous basket.

**Side effects:** Anonymous basket deleted; user basket updated in DB.
**Failure modes:** If transfer fails mid-way, items could be lost (no transaction boundary visible in application code — relies on EF SaveChanges per operation).

---

### Flow 3: Checkout → order creation
**Trigger:** `POST /Basket/Checkout` (requires authentication)

1. `Pages/Basket/Checkout.cshtml.cs:OnPost()` calls:
   - `BasketService.SetQuantities()` — updates basket item quantities
   - `OrderService.CreateOrderAsync(basketId, address)` — creates order
   - `BasketService.DeleteBasketAsync()` — clears basket
   Evidence: [src/Web/Pages/Basket/Checkout.cshtml.cs](src/Web/Pages/Basket/Checkout.cshtml.cs)

2. `OrderService.CreateOrderAsync()`:
   - Loads basket with items via `BasketWithItemsSpecification`
   - Fetches current `CatalogItem` prices via `CatalogItemsSpecification` (re-reads price at checkout time)
   - Constructs `Order` aggregate with `OrderItem`s
   - Saves via `IRepository<Order>`
   Evidence: [src/ApplicationCore/Services/OrderService.cs](src/ApplicationCore/Services/OrderService.cs)

3. **FACT (hardcoded):** Checkout page uses a hardcoded `Address` ("123 Main St., Kent, OH") — shipping address input from user is NOT implemented.
   Evidence: [src/Web/Pages/Basket/Checkout.cshtml.cs](src/Web/Pages/Basket/Checkout.cshtml.cs)

**Side effects:** New `Order` + `OrderItem` rows in `CatalogContext`. Basket deleted.
**Failure modes:** No stock reservation / inventory check. Price re-fetch from DB protects against stale basket prices. No payment processing. If `DeleteBasketAsync` fails after order creation, user sees an error but order may have been created.

---

### Flow 4: Admin catalog update via PublicApi
**Trigger:** BlazorAdmin SPA calls `PUT /api/catalog-items` with JWT Bearer token

1. `UpdateCatalogItemEndpoint.HandleAsync()` — validates JWT (`Administrators` role required)
2. Loads item via `IRepository<CatalogItem>`, calls `item.UpdateDetails()`, `UpdatePrice()`, `UpdateBrand()`, etc.
3. Saves via `IRepository<CatalogItem>.UpdateAsync()`
4. Returns 200 with updated `CatalogItemDto`
   Evidence: [src/PublicApi/CatalogItemEndpoints/UpdateCatalogItemEndpoint.cs](src/PublicApi/CatalogItemEndpoints/UpdateCatalogItemEndpoint.cs)

**Side effects:** `CatalogItem` updated in `CatalogContext`. Cache in Web process is **not** invalidated by this call (cross-process `IMemoryCache`) — **stale cache risk**.
**Failure modes:** Stale catalog cache in Web after API update (different processes, in-memory cache). JWT expiry = 7 days (long-lived tokens, no refresh mechanism visible).

---

### Flow 5: PublicApi authentication (JWT issuance)
**Trigger:** `POST /api/authenticate`

1. `AuthenticateEndpoint.HandleAsync()` calls `SignInManager.PasswordSignInAsync()`
2. On success, calls `ITokenClaimService.GetTokenAsync(userName)` → `IdentityTokenClaimService` generates HMAC-SHA256 JWT with username + roles, 7-day expiry.
3. Returns `{ token: "..." }`
   Evidence: [src/PublicApi/AuthEndpoints/AuthenticateEndpoint.cs](src/PublicApi/AuthEndpoints/AuthenticateEndpoint.cs), [src/Infrastructure/Identity/IdentityTokenClaimService.cs](src/Infrastructure/Identity/IdentityTokenClaimService.cs)

**Side effects:** None (stateless JWT).
**Failure modes:** JWT secret is hardcoded constant. No token refresh. No revocation mechanism for JWT (only cookie auth has revocation in Web).

---

## 6. Dependencies

### Internal module dependencies (dependency graph)
```
BlazorAdmin ──► BlazorShared
ApplicationCore ──► BlazorShared (shared DTOs)
Infrastructure ──► ApplicationCore
Web ──► ApplicationCore + Infrastructure + BlazorAdmin + BlazorShared
PublicApi ──► ApplicationCore + Infrastructure
```
FACT: From `.csproj` `<ProjectReference>` elements.

### External services
| Service | Purpose | Evidence |
|---|---|---|
| SQL Server | Persistence (catalog + identity) | `src/Infrastructure/Dependencies.cs` |
| Azure SQL (prod) | Same, cloud-hosted | `infra/main.bicep` |
| Azure Key Vault (prod) | Connection string secrets | `src/Web/Program.cs:31-32` |
| Azure App Service (prod) | Web hosting | `infra/main.bicep`, `azure.yaml` |
| **Email** | UNKNOWN — `IEmailSender` registered but implementation unclear | `src/Web/Extensions/ServiceCollectionExtensions.cs` |

### Build/deploy dependencies
| Tool | Version | Purpose |
|---|---|---|
| .NET SDK 8.0 | `global.json` | Build + run |
| Docker + Compose | n/a | Local SQL Server |
| Azure Developer CLI (`azd`) | n/a | `azure.yaml` — `azd up` deploys to Azure |
| Azure Bicep | `infra/main.bicep` | IaC for App Service, SQL, Key Vault |
| GitHub Actions | `.github/workflows/dotnetcore.yml` | CI: build + test |
| Dependabot | `.github/dependabot.yml` | Automated package updates |

### Key NuGet packages
| Package | Role |
|---|---|
| `Ardalis.Specification.EntityFrameworkCore` 7.0.0 | Repository pattern over EF Core |
| `Ardalis.GuardClauses` 4.0.1 | Input validation in domain |
| `Ardalis.Result` 7.0.0 | Result type for operations |
| `MediatR` 12.0.1 | CQRS for order queries in Web |
| `AutoMapper` 12.0.1 | Entity→DTO mapping in PublicApi |
| `FluentValidation` 11.9.0 | Validation in BlazorShared |
| `MinimalApi.Endpoint` 1.3.0 | Endpoint pattern in PublicApi |
| `Blazored.LocalStorage` 4.5.0 | Client-side brand/type caching in BlazorAdmin |
| `Azure.Identity` 1.10.4 | Azure Key Vault credential chain |

FACT: `Directory.Packages.props` (root) — all versions centrally managed.

---

## 7. Configuration & environments

### Where config lives
- **`src/Web/appsettings.json`** — default (in-memory DB, base URLs)
- **`src/Web/appsettings.Development.json`** — dev overrides (log level)
- **`src/Web/appsettings.Docker.json`** — Docker SQL Server connection strings
- **`src/PublicApi/appsettings*.json`** — same pattern for API
- **`src/BlazorAdmin/wwwroot/appsettings*.json`** — client-side Blazor config (served as static file)
- **User Secrets** (dev): `UserSecretsId` in `Web.csproj:7` and `PublicApi.csproj:5`
- **Azure Key Vault** (prod): injected via `builder.Configuration.AddAzureKeyVault()` (`src/Web/Program.cs:31`)

### Key environment variables (production)
| Variable | Purpose | Evidence |
|---|---|---|
| `AZURE_KEY_VAULT_ENDPOINT` | Key Vault URL for secret retrieval | `src/Web/Program.cs:31` |
| `AZURE_SQL_CATALOG_CONNECTION_STRING_KEY` | Key Vault key name for catalog DB | `infra/main.bicep` |
| `AZURE_SQL_IDENTITY_CONNECTION_STRING_KEY` | Key Vault key name for identity DB | `infra/main.bicep` |
| `ASPNETCORE_ENVIRONMENT` | `Docker` triggers Docker-specific appsettings | `docker-compose.override.yml` |

### Secrets handling
- **Development:** Hardcoded in `AuthorizationConstants.cs` (JWT key, default password) with TODO comments. User Secrets for connection strings.
- **Production:** Azure Key Vault via `ChainedTokenCredential(AzureDeveloperCliCredential, DefaultAzureCredential)`. Evidence: `src/Web/Program.cs:25-43`.
- **RISK:** JWT secret key (`JWT_SECRET_KEY`) is hardcoded in ApplicationCore — it is not pulled from Key Vault or user secrets. Evidence: `src/ApplicationCore/Constants/AuthorizationConstants.cs:11`.

### Local vs prod differences
| Concern | Local/Dev | Docker | Production |
|---|---|---|---|
| Database | In-memory EF | SQL Server (docker) | Azure SQL |
| Secrets | Hardcoded constants | Same | Azure Key Vault |
| Auth cookie | HTTP allowed | HTTP allowed | HTTPS enforced (FACT: `ConfigureCookieSettings.cs`) |
| Key Vault | Not used | Not used | Required |

---

## 8. Observability

### Logging
- **FACT:** `IAppLogger<T>` abstraction defined in `ApplicationCore/Interfaces/`. Implemented by `LoggerAdapter<T>` wrapping `Microsoft.Extensions.Logging.ILogger<T>` (`src/Infrastructure/Logging/LoggerAdapter.cs`).
- **INFERENCE:** Standard ASP.NET Core logging pipeline — console, debug providers by default. No structured logging library (Serilog, NLog) visible.
- **UNKNOWN:** No log aggregation service (Application Insights, Datadog, Seq) configured in code. Check `appsettings.json` for `ApplicationInsights` key.

### Metrics
- **UNKNOWN:** No `IMetrics`, Prometheus, or OpenTelemetry metrics registration found.

### Tracing
- **UNKNOWN:** No distributed tracing (OpenTelemetry, Application Insights tracing) found in code.

### Health checks
- **FACT:** `GET /health` — JSON health report (`src/Web/Program.cs:86-89`)
- **FACT:** `GET /home_page_health_check` — checks Web home page HTML
- **FACT:** `GET /api_health_check` — calls PublicApi catalog endpoint, checks for known product string
- Evidence: [src/Web/HealthChecks/](src/Web/HealthChecks/), [src/Web/Program.cs](src/Web/Program.cs)

### Alerts
- **UNKNOWN:** No alerting configuration found in this repository.

---

## 9. Testing & quality gates

### Types of tests present

| Project | Framework | Type | Count (approx) | Key focus |
|---|---|---|---|---|
| `tests/UnitTests` | xUnit + NSubstitute | Unit | ~15 files | Domain services, specifications, MediatR handlers, Web extensions |
| `tests/IntegrationTests` | xUnit + InMemory EF | Integration | ~5 files | EF repositories (basket, order) with in-memory DB |
| `tests/FunctionalTests` | xUnit + `WebApplicationFactory` | Functional/E2E | ~10 files | HTTP-level tests for Web pages and PublicApi |
| `tests/PublicApiIntegrationTests` | MSTest + `WebApplicationFactory` | Integration | ~5 files | PublicApi endpoint tests; code coverage via coverlet |

FACT: Project structures from `.csproj` references and directory listings.

### How tests run in CI
- **FACT:** GitHub Actions (`.github/workflows/dotnetcore.yml`) runs `dotnet build` then `dotnet test` on `eShopOnWeb.sln` on every push/PR to any branch.
- **FACT:** Tests use `UseOnlyInMemoryDatabase: true` by default — no SQL Server required in CI.
- **FACT:** Code coverage collected via `coverlet.collector` in `PublicApiIntegrationTests.csproj` with `CodeCoverage.runsettings` at root.

### Coverage gaps / risks
- **FACT (gap):** Basket `Checkout.cshtml.cs` uses a **hardcoded address** — the checkout flow is not meaningfully testable end-to-end without real address input.
- **FACT (gap):** Auth token revocation (`RevokeAuthenticationEvents`) relies on `IMemoryCache` — behavior in multi-instance deployments is explicitly called out as a known gap in the comment (`src/Web/Configuration/RevokeAuthenticationEvents.cs`).
- **INFERENCE (gap):** BlazorAdmin has no visible test coverage (no test project targeting it).
- **INFERENCE (gap):** Error paths (failed order creation, basket transfer failure) have limited test coverage based on file inventory.
- **UNKNOWN:** No mutation testing, no contract tests for PublicApi.

---

## 10. Migration hotspots / tech debt (evidence-based)

---

### Hotspot 1: Hardcoded JWT secret key in ApplicationCore
**Risk type:** Configuration risk | Security
**Confidence:** High
**Evidence:** `src/ApplicationCore/Constants/AuthorizationConstants.cs:11` — `JWT_SECRET_KEY = "SecretKeyOfDoomThatMustBeAMinimumNumberOfBytes"` with `// TODO: change this to an environment variable in your real app`
**Why it matters:** JWT secret is baked into the domain layer assembly. Any deployment using this default is trivially forgeable. ApplicationCore should have zero awareness of infrastructure secrets.
**Suggested seam:** Move to `IOptions<JwtSettings>` injected into `IdentityTokenClaimService`. Backed by user secrets (dev) / Key Vault (prod).
**Migration risk:** Low — contained change, good test coverage of auth flow.

---

### Hotspot 2: Cross-process cache invalidation (IMemoryCache)
**Risk type:** Hidden side-effects | Coupling
**Confidence:** High
**Evidence:**
- `src/Web/Services/CachedCatalogViewModelService.cs` — caches catalog in Web process memory
- `src/Web/Configuration/RevokeAuthenticationEvents.cs` — stores revocation tokens in Web process memory with explicit comment: "replace with a distributed cache in a multi-host scenario"
- `src/PublicApi/` — catalog updates go through PublicApi (separate process); Web cache is never invalidated
**Why it matters:** Any catalog update via the API will not be reflected in the Web UI until cache expires or Web restarts.
**Suggested seam:** Replace `IMemoryCache` with `IDistributedCache` + Redis. Cache invalidation can be event-driven or use cache tags.
**Migration risk:** Medium — requires Redis infrastructure; test coverage for cache behavior is thin.

---

### Hotspot 3: Hardcoded checkout address
**Risk type:** Mixed concerns | Lack of tests
**Confidence:** High
**Evidence:** `src/Web/Pages/Basket/Checkout.cshtml.cs` — `new Address("123 Main St.", "Kent", "OH", "United States", "44240")`
**Why it matters:** Checkout is a critical user flow with no address input. The feature is deliberately simplified but blocks real-world use and makes functional tests for this flow meaningless.
**Suggested seam:** Add `CheckoutViewModel` with address fields; wire to existing `Address` value object.
**Migration risk:** Low — isolated to `Checkout.cshtml.cs` and `Order.cs` aggregate.

---

### Hotspot 4: Dual API patterns (MinimalApi.Endpoint vs Ardalis.ApiEndpoints)
**Risk type:** Mixed concerns | Coupling
**Confidence:** Medium
**Evidence:**
- `src/PublicApi/AuthEndpoints/AuthenticateEndpoint.cs` — uses `Ardalis.ApiEndpoints` (`EndpointBaseAsync`)
- `src/PublicApi/CatalogItemEndpoints/*.cs` — uses `MinimalApi.Endpoint` pattern
- Both `Ardalis.ApiEndpoints` 4.1.0 AND `MinimalApi.Endpoint` 1.3.0 present in `Directory.Packages.props`
**Why it matters:** Two different endpoint registration patterns in one project increases cognitive load and inconsistent error handling.
**Suggested seam:** Standardize on one pattern (MinimalApi.Endpoint is more idiomatic for .NET 8 Minimal APIs).
**Migration risk:** Low — isolated to PublicApi.

---

### Hotspot 5: ApplicationCore depending on BlazorShared (UI DTOs in domain layer)
**Risk type:** Coupling | Mixed concerns
**Confidence:** Medium
**Evidence:** `src/ApplicationCore/ApplicationCore.csproj` has `<ProjectReference>` to `BlazorShared`
**Why it matters:** Domain layer (ApplicationCore) should not know about UI-layer DTOs (BlazorShared). This inverts the dependency direction and means the domain layer is coupled to Blazor package versions.
**Suggested seam:** Move shared types to ApplicationCore or a new `Shared` project with no Blazor dependency. BlazorShared maps from/to those types.
**Migration risk:** Medium — requires identifying which BlazorShared types are referenced by ApplicationCore and extracting them.

---

### Hotspot 6: No inventory / stock management
**Risk type:** Hidden side-effects | Missing domain concept
**Confidence:** High
**Evidence:** `src/ApplicationCore/Services/OrderService.cs` — creates order from basket with no stock check. `CatalogItem` entity has no `StockQuantity` or equivalent field.
**Why it matters:** Any quantity can be ordered regardless of availability. This is likely intentional for the reference app but is a significant gap for any real-world use.
**Suggested seam:** Add `StockOnHand` to `CatalogItem`. Add `IStockService` with guard in `OrderService.CreateOrderAsync()`.
**Migration risk:** High — requires schema migration + domain logic addition.

---

### Hotspot 7: Long-lived JWT tokens with no refresh
**Risk type:** Security | Configuration risk
**Confidence:** High
**Evidence:** `src/Infrastructure/Identity/IdentityTokenClaimService.cs` — `expires: DateTime.UtcNow.AddDays(7)`. No refresh token endpoint visible in PublicApi endpoints.
**Why it matters:** 7-day tokens with no revocation mechanism. If a token is compromised, it remains valid for up to 7 days.
**Suggested seam:** Add refresh token pattern or reduce expiry + add sliding refresh endpoint.
**Migration risk:** Medium.

---

## 11. Open questions

1. **What does `IEmailSender` actually do?** `src/Web/Extensions/ServiceCollectionExtensions.cs` registers it — is it a no-op, SMTP, or SendGrid? Check `Infrastructure/Services/`.
2. **Is there any Azure Application Insights integration?** Not visible in code or appsettings. If this is deployed to App Service, AI may be auto-injected by Azure — check `APPLICATIONINSIGHTS_CONNECTION_STRING` env var in Azure portal.
3. **What is in `.github/agents/` and `.github/prompts/`?** These appear to be speckit/AI code-generation prompt files. What workflow uses them? Are they live or experimental?
4. **Why does `Everything.sln` exist alongside `eShopOnWeb.sln`?** Is there a difference in project inclusion? Check which projects are in each solution.
5. **Are there EF migrations?** No `Migrations/` folder found in Infrastructure. DB is created from model at startup (`EnsureCreated` or seeding). Confirm behavior in production (schema upgrades would drop and recreate with in-memory, but SQL Server may differ).
6. **How is BlazorAdmin deployed?** It is served as static files from the Web host via `Microsoft.AspNetCore.Components.WebAssembly.Server`. Confirm this is the sole deployment mechanism — there is no standalone WASM deployment.
7. **Multi-host deployment:** `RevokeAuthenticationEvents` explicitly notes IMemoryCache is not multi-host safe. Is this deployed behind a load balancer? If so, session stickiness or distributed cache is required.
8. **Data protection key storage:** `src/Web/key-768c1632-cf7b-41a9-bb7a-bff228ae8fba.xml` is a data protection key committed to the repo. This is a security concern in real deployments — is this intentional for the reference app?
9. **`BasketQueryService` bypasses Specification pattern** — uses direct EF queries. Is this intentional for performance? Check `src/Infrastructure/Data/Queries/BasketQueryService.cs`.
10. **No `UNKNOWN` resolved for background jobs:** Confirm there are no hosted services by searching for `IHostedService` or `BackgroundService` implementations.

---

## 12. First 90 minutes checklist

- [ ] **Clone and build:** `dotnet build eShopOnWeb.sln` — confirm zero errors.
- [ ] **Run the Web app:** `cd src/Web && dotnet run` — open browser, confirm catalog page loads with seeded data (in-memory, no SQL needed).
- [ ] **Run the PublicApi:** `cd src/PublicApi && dotnet run` — open `/swagger`, confirm endpoints are listed.
- [ ] **Log in as demo user** (`demouser@microsoft.com` / `Pass@word1`), add item to basket, place an order — walk the critical checkout flow.
- [ ] **Log in as admin** (`admin@microsoft.com` / `Pass@word1`), open the Admin panel — confirm BlazorAdmin SPA loads and catalog CRUD works.
- [ ] **Run all tests:** `dotnet test eShopOnWeb.sln` — confirm all pass.
- [ ] **Read `src/ApplicationCore/`** — understand the entity model (Basket, Order, CatalogItem aggregates) before touching anything else. This is the heart of the domain.
- [ ] **Read `src/Infrastructure/Dependencies.cs`** — understand the DB switching logic (`UseOnlyInMemoryDatabase`).
- [ ] **Read `src/Web/Program.cs`** — understand startup: environment branching, auth setup, seeding order.
- [ ] **Read `src/PublicApi/Program.cs`** — understand JWT auth setup and CORS config.
- [ ] **Inspect `src/ApplicationCore/Constants/AuthorizationConstants.cs`** — note all hardcoded secrets. Understand they must be replaced before any real deployment.
- [ ] **Try Docker Compose:** `docker-compose up` — confirm Web + API + SQL Server all start and inter-operate. Compare behavior to in-memory mode.
- [ ] **Read `infra/main.bicep`** — understand what gets created in Azure (`azd up`). Note: B1 App Service SKU, two Azure SQL databases, Key Vault.
- [ ] **Check open questions above** (section 11) — resolve the email sender and EF migrations questions before modifying data layer.
- [ ] **Read the existing test structure** (`tests/`) — understand what is covered and what the test data builders produce (`tests/UnitTests/Builders/`) before writing new tests.
