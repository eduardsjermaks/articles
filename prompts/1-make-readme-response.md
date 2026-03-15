codex: 
5.4
medium 

# Project Orientation (Generated)
## 1. What this project is
- Purpose: a sample ASP.NET Core reference application for a single-process, monolithic e-commerce-style web app, with a server-rendered web front end plus an admin experience that uses a separate public API. (README.md:8-10, README.md:25-27, README.md:95-98)
- Who uses it / main use-cases: shoppers can browse catalog items, manage baskets, and place orders through the `Web` app, while administrators can access `/admin` and use JWT-protected catalog-management endpoints exposed by `PublicApi`. (src/Web/Pages/Index.cshtml.cs:7-21, src/Web/Pages/Basket/Index.cshtml.cs:28-66, src/Web/Pages/Basket/Checkout.cshtml.cs:14-68, src/Web/Pages/Admin/Index.cshtml:1-74, src/PublicApi/CatalogItemEndpoints/CreateCatalogItemEndpoint.cs:27-75, README.md:95-98)

## 2. Quick start (developer)
- Prereqs: `.NET` SDK `8.0.x` is pinned in `global.json`; package versions are centrally managed and target `net8.0`; local EF tooling is declared via `dotnet-ef`; Docker and `azd` are optional paths. (global.json:1-6, Directory.Packages.props:2-8, src/Web/.config/dotnet-tools.json:1-12, README.md:47-85, README.md:154-166)
- How to run locally: run `PublicApi` and `Web` together for the full experience; the README calls out `dotnet run` in `src/PublicApi` and `dotnet run --launch-profile Web` in `src/Web`, with the web UI on `https://localhost:5001/` and the API on `https://localhost:5099` by launch settings. (README.md:95-98, src/Web/Properties/launchSettings.json:20-37, src/PublicApi/Properties/launchSettings.json:11-19)
- How to run tests: CI uses `dotnet build ./eShopOnWeb.sln --configuration Release` and `dotnet test ./eShopOnWeb.sln --configuration Release`; test projects are `UnitTests`, `IntegrationTests`, `FunctionalTests`, and `PublicApiIntegrationTests`. (.github/workflows/dotnetcore.yml:18-22, tests/UnitTests/UnitTests.csproj:11-26, tests/IntegrationTests/IntegrationTests.csproj:8-26, tests/FunctionalTests/FunctionalTests.csproj:16-29, tests/PublicApiIntegrationTests/PublicApiIntegrationTests.csproj:21-35)
- Common pitfalls:
- The admin UI depends on `PublicApi`; running only `Web` covers most storefront behavior but not the admin path. (README.md:95-98)
- Local persistence is easy to misread: `Web/appsettings.json` sets `UseOnlyInMemoryDatabase` to `true`, but the README also documents SQL Server migrations for persistent local databases. (src/Web/appsettings.json:6-11, README.md:100-145)
- Docker uses hard-coded SQL credentials and different base URLs than local development. (docker-compose.yml:18-25, src/Web/appsettings.Docker.json:2-9, src/PublicApi/appsettings.Docker.json:2-9)
- Running the apps manually can cause build file-locking issues until the processes are stopped. (README.md:98-98)
- Demo credentials are intentionally visible in the login page. (src/Infrastructure/Identity/AppIdentityDbContextSeed.cs:18-31, src/ApplicationCore/Constants/AuthorizationConstants.cs:7-11, src/Web/Areas/Identity/Pages/Account/Login.cshtml:46-54)

## 3. Architecture at a glance
- High-level component diagram (ASCII is fine)
```text
Browser
  -> Web (Razor Pages/MVC + Identity cookies)
     -> ApplicationCore interfaces/services
     -> Infrastructure (EF Core, Identity, logging, query services)
     -> CatalogContext + AppIdentityDbContext

Admin browser at /admin
  -> BlazorAdmin WebAssembly (served by Web)
  -> UserController issues JWT for signed-in user
  -> PublicApi (JWT auth for admin mutations)
  -> ApplicationCore + Infrastructure
  -> CatalogContext + AppIdentityDbContext
```
Evidence: (src/Web/Program.cs:45-63) (src/Web/Program.cs:96-112) (src/Web/Program.cs:184-199) (src/PublicApi/Program.cs:34-45) (src/PublicApi/Program.cs:54-85) (src/PublicApi/Program.cs:155-176) (src/BlazorAdmin/Program.cs:16-40) (src/Web/Pages/Admin/Index.cshtml:68-71)
- Runtime processes:
- `src/Web` is the main ASP.NET Core process and serves MVC/controllers, Razor Pages, health checks, static files, and the Blazor admin assets. (src/Web/Program.cs:74-89, src/Web/Program.cs:151-199)
- `src/PublicApi` is a second ASP.NET Core process that serves Swagger, controller routes, and minimal API endpoints. (src/PublicApi/Program.cs:84-123, src/PublicApi/Program.cs:155-176)
- `src/BlazorAdmin` is a Blazor WebAssembly client, not a separate server process; it is mounted into the web app’s admin page. (src/BlazorAdmin/BlazorAdmin.csproj:1-16, src/BlazorAdmin/Program.cs:16-40, src/Web/Pages/Admin/Index.cshtml:68-71)
- Data stores and queues:
- Two EF Core contexts are configured: `CatalogContext` for catalog/basket/order data and `AppIdentityDbContext` for ASP.NET Identity data. (src/Infrastructure/Data/CatalogContext.cs:9-26, src/Infrastructure/Identity/AppIdentityDbContext.cs:7-20, src/Infrastructure/Dependencies.cs:19-38)
- Local/Docker deployments point those contexts at LocalDB, in-memory stores, or SQL Server/SQL Edge depending on config. (src/Infrastructure/Dependencies.cs:13-38, src/Web/appsettings.json:6-11, src/Web/appsettings.Docker.json:2-9, docker-compose.yml:18-25)
- Queues/message brokers: Unknown. I did not find queue or broker setup in the inspected startup and deployment files; inspect `infra/` and any external platform config next. (src/Web/Program.cs:22-202, src/PublicApi/Program.cs:26-179, infra/main.bicep:47-144)

## 4. Repository map
- Top-level directories:
- `src/`: application code for the web app, public API, shared core, infrastructure, and Blazor admin/shared libraries. (src)
- `tests/`: unit, integration, functional, and API integration test projects. (tests)
- `infra/`: Bicep templates for Azure App Service, SQL Server, and Key Vault. (infra)
- `.github/`: CI workflows plus repo-specific agent/prompt assets. (.github)
- `.devcontainer/`: devcontainer configuration for a `.NET 8` workspace. (.devcontainer)
- Major modules:
- `src/Web`: storefront/UI composition layer. Public interfaces include `IBasketViewModelService` and `ICatalogViewModelService`; key files are `Program.cs`, `Pages/`, `Controllers/`, and `Configuration/`. (src/Web/Program.cs:22-202, src/Web/Interfaces/IBasketViewModelService.cs:6-10, src/Web/Interfaces/ICatalogViewModelService.cs:8-12, src/Web/Pages, src/Web/Controllers, src/Web/Configuration)
- `src/PublicApi`: API surface for authentication and catalog CRUD/list operations. Public interfaces are HTTP endpoints defined in `AuthEndpoints/` and `Catalog*Endpoints/`; key files are `Program.cs`, `Middleware/ExceptionMiddleware.cs`, and the endpoint classes. (src/PublicApi/Program.cs:26-179, src/PublicApi/AuthEndpoints/AuthenticateEndpoint.cs:15-58, src/PublicApi/CatalogItemEndpoints/CatalogItemListPagedEndpoint.cs:18-72, src/PublicApi/CatalogItemEndpoints/CreateCatalogItemEndpoint.cs:18-75, src/PublicApi/Middleware/ExceptionMiddleware.cs:10-53)
- `src/ApplicationCore`: domain/application layer. Public interfaces include `IRepository<>`, `IReadRepository<>`, `IBasketService`, `IOrderService`, `IBasketQueryService`, `ITokenClaimsService`, and `IUriComposer`; key files are `Interfaces/`, `Services/`, and `Specifications/`. (src/ApplicationCore/Interfaces/IRepository.cs, src/ApplicationCore/Interfaces/IReadRepository.cs, src/ApplicationCore/Interfaces/IBasketService.cs:8-14, src/ApplicationCore/Interfaces/IOrderService.cs:6-8, src/ApplicationCore/Interfaces/IBasketQueryService.cs:5-10, src/ApplicationCore/Interfaces/ITokenClaimsService.cs, src/ApplicationCore/Interfaces/IUriComposer.cs, src/ApplicationCore/Services/BasketService.cs:11-85, src/ApplicationCore/Services/OrderService.cs:12-52, src/ApplicationCore/Specifications)
- `src/Infrastructure`: EF Core, Identity, logging, query implementations, migrations, and environment-specific DB wiring. Main public entrypoints are `Dependencies.ConfigureServices`, `CatalogContext`, `AppIdentityDbContext`, `EfRepository<>`, `BasketQueryService`, and `IdentityTokenClaimService`. (src/Infrastructure/Dependencies.cs:9-40, src/Infrastructure/Data/CatalogContext.cs:9-26, src/Infrastructure/Identity/AppIdentityDbContext.cs:7-20, src/Infrastructure/Data/EfRepository.cs, src/Infrastructure/Data/Queries/BasketQueryService.cs:8-30, src/Infrastructure/Identity/IdentityTokenClaimService.cs:14-45)
- `src/BlazorAdmin`: WebAssembly admin client; key files are `Program.cs`, `Pages/`, and `ServicesConfiguration.cs`. (src/BlazorAdmin/BlazorAdmin.csproj:1-30, src/BlazorAdmin/Program.cs:16-40, src/BlazorAdmin/ServicesConfiguration.cs:10-31, src/BlazorAdmin/Pages)
- `src/BlazorShared`: DTOs, auth constants, and validation/models shared between the admin client and API. (src/BlazorShared, src/BlazorShared/Authorization/Constants.cs:3-8)

## 5. Critical flows
- Flow: Catalog browse/filter
- Trigger/entrypoint: `GET /` hits `src/Web/Pages/Index.cshtml.cs`, which asks `ICatalogViewModelService` for paged catalog data. (src/Web/Pages/Index.cshtml.cs:7-21)
- Steps: `CachedCatalogViewModelService` builds cache keys and delegates misses to `CatalogViewModelService`; `CatalogViewModelService` applies Ardalis specifications, queries item/brand/type repositories, and maps entities to UI view models with composed picture URLs. (src/Web/Services/CachedCatalogViewModelService.cs:10-49, src/Web/Services/CatalogViewModelService.cs:40-110)
- Side effects: in-memory cache entries for catalog pages, brands, and types. (src/Web/Services/CachedCatalogViewModelService.cs:22-49)
- Failure modes: this path uses in-memory sliding-expiration cache entries, and no external cache is configured in `Web`. (src/Web/Services/CachedCatalogViewModelService.cs:24-47, src/Web/Program.cs:65-66)
- Flow: Add item to basket
- Trigger/entrypoint: `POST /basket/index` calls `IndexModel.OnPost`. (src/Web/Pages/Basket/Index.cshtml.cs:33-53)
- Steps: the page model loads the catalog item, resolves an authenticated username or long-lived basket cookie, calls `IBasketService.AddItemToBasket`, then maps the resulting basket to a view model. `BasketService` loads or creates a basket via `BasketWithItemsSpecification`, mutates it, and updates the repository. (src/Web/Pages/Basket/Index.cshtml.cs:40-52, src/Web/Pages/Basket/Index.cshtml.cs:68-99, src/ApplicationCore/Services/BasketService.cs:23-38, src/Web/Services/BasketViewModelService.cs:28-39)
- Side effects: basket rows are inserted/updated in `CatalogContext`; anonymous users receive a persistent cookie. (src/ApplicationCore/Services/BasketService.cs:29-37, src/Web/Pages/Basket/Index.cshtml.cs:93-98, src/Infrastructure/Data/CatalogContext.cs:14-20)
- Failure modes: invalid or missing product IDs redirect back to the catalog; basket identity depends on cookie integrity for anonymous users. (src/Web/Pages/Basket/Index.cshtml.cs:35-44, src/Web/Pages/Basket/Index.cshtml.cs:79-98)
- Flow: Checkout/order creation
- Trigger/entrypoint: authorized `POST /basket/checkout` calls `CheckoutModel.OnPost`. (src/Web/Pages/Basket/Checkout.cshtml.cs:14-18, src/Web/Pages/Basket/Checkout.cshtml.cs:44-68, src/Web/Program.cs:81-84)
- Steps: the page model reloads the basket, updates quantities, calls `IOrderService.CreateOrderAsync`, then deletes the basket. `OrderService` validates the basket, loads current catalog items, snapshots item identity/name/picture into `CatalogItemOrdered`, builds an `Order`, and persists it. (src/Web/Pages/Basket/Checkout.cshtml.cs:46-59, src/ApplicationCore/Services/OrderService.cs:30-52, src/ApplicationCore/Services/BasketService.cs:47-64)
- Side effects: an order is inserted, the basket is deleted, and a warning is logged if checkout is attempted with an empty basket. (src/ApplicationCore/Services/OrderService.cs:49-52, src/ApplicationCore/Services/BasketService.cs:40-45, src/Web/Pages/Basket/Checkout.cshtml.cs:60-67)
- Failure modes: empty baskets redirect back to the basket page; shipping address is currently hard-coded, so address capture is not represented in this flow. (src/Web/Pages/Basket/Checkout.cshtml.cs:57-64)
- Flow: Admin catalog mutation
- Trigger/entrypoint: the signed-in web session calls `UserController.GetCurrentUser` to obtain a JWT, then the admin client calls `PublicApi` endpoints such as `POST api/catalog-items`. (src/Web/Controllers/UserController.cs:35-40, src/Web/Controllers/UserController.cs:60-104, src/PublicApi/CatalogItemEndpoints/CreateCatalogItemEndpoint.cs:27-75)
- Steps: `IdentityTokenClaimService` issues a JWT from the signed-in Identity user and roles; `PublicApi` validates bearer tokens; admin-only endpoints create/update/delete catalog items through the generic repository. (src/Infrastructure/Identity/IdentityTokenClaimService.cs:23-45, src/PublicApi/Program.cs:54-70, src/PublicApi/CatalogItemEndpoints/CreateCatalogItemEndpoint.cs:29-75, src/PublicApi/CatalogItemEndpoints/UpdateCatalogItemEndpoint.cs:27-66)
- Side effects: catalog records change; duplicate names throw `DuplicateException`, which the API middleware converts to `409 Conflict`. (src/PublicApi/CatalogItemEndpoints/CreateCatalogItemEndpoint.cs:43-48, src/PublicApi/Middleware/ExceptionMiddleware.cs:31-53)
- Failure modes: non-admin tokens are forbidden in tests; mutation image upload is intentionally disabled due to a noted security concern. (tests/PublicApiIntegrationTests/CatalogItemEndpoints/CreateCatalogItemEndpointTest.cs:23-52, src/PublicApi/CatalogItemEndpoints/CreateCatalogItemEndpoint.cs:53-60)

## 6. Dependencies
- Internal module dependencies:
- `Web` references `ApplicationCore`, `Infrastructure`, `BlazorAdmin`, and `BlazorShared`. (src/Web/Web.csproj:42-47)
- `PublicApi` references `ApplicationCore` and `Infrastructure`. (src/PublicApi/PublicApi.csproj:34-37)
- `Infrastructure` references `ApplicationCore`; `ApplicationCore` references `BlazorShared`. (src/Infrastructure/Infrastructure.csproj:15-17, src/ApplicationCore/ApplicationCore.csproj:16-18)
- External services:
- SQL Server / LocalDB / SQL Edge are the only concrete external runtime services I found in repo-managed config. (src/Infrastructure/Dependencies.cs:19-38, src/Web/appsettings.json:6-10, docker-compose.yml:18-25)
- Azure Key Vault is used in non-development `Web` environments to resolve connection strings by key name. (src/Web/Program.cs:29-42)
- Third-party HTTP APIs/message brokers: Unknown. Inspect `infra/`, environment variables, and any external platform configuration if this repo is part of a larger system. (infra/main.bicep:47-144)
- Build/deploy dependencies:
- Build uses `.NET 8`, central package management, GitHub Actions, and optional devcontainers. (global.json:1-6, Directory.Packages.props:2-71, .github/workflows/dotnetcore.yml:1-22, .devcontainer/devcontainer.json:4-27)
- Deployment options in-repo are Docker Compose and `azd`/Bicep for Azure App Service + SQL Server + Key Vault. (docker-compose.yml:3-25, docker-compose.override.yml:2-20, azure.yaml:3-8, infra/main.bicep:47-144)

## 7. Configuration & environments
- Where config lives:
- App config is in `src/Web/appsettings*.json` and `src/PublicApi/appsettings*.json`; local launch URLs are in each project’s `Properties/launchSettings.json`. (src/Web/appsettings.json:1-21, src/Web/appsettings.Docker.json:1-17, src/PublicApi/appsettings.json:1-20, src/PublicApi/appsettings.Docker.json:1-17, src/Web/Properties/launchSettings.json:10-38, src/PublicApi/Properties/launchSettings.json:2-37)
- Infrastructure config for Azure lives in `azure.yaml` and `infra/*.bicep`. (azure.yaml:3-8, infra/main.bicep:1-144)
- Environment variables:
- `Web` consumes `AZURE_KEY_VAULT_ENDPOINT`, `AZURE_SQL_CATALOG_CONNECTION_STRING_KEY`, and `AZURE_SQL_IDENTITY_CONNECTION_STRING_KEY` in non-development environments; both apps also call `AddEnvironmentVariables()`. (src/Web/Program.cs:31-42, src/Web/Program.cs:61-63, src/PublicApi/Program.cs:86-86, infra/main.bicep:59-64)
- Docker sets `ASPNETCORE_ENVIRONMENT=Docker` and `ASPNETCORE_URLS=http://+:8080` for both runtime apps. (docker-compose.override.yml:3-20)
- Secrets handling:
- `Web` production-style config pulls secrets from Azure Key Vault. (src/Web/Program.cs:29-42, infra/core/security/keyvault.bicep:7-25)
- `Web` and `PublicApi` both declare `UserSecretsId`, but the repo also contains hard-coded demo credentials and Docker SQL passwords, so secrets handling is mixed by environment. (src/Web/Web.csproj:3-9, src/PublicApi/PublicApi.csproj:3-9, src/ApplicationCore/Constants/AuthorizationConstants.cs:7-11, docker-compose.yml:22-24)
- Local vs prod differences:
- `Web` explicitly branches between development/Docker and non-development startup: local/Docker uses `Infrastructure.Dependencies.ConfigureServices`, while non-development loads Key Vault-backed SQL connection strings with retry enabled. (src/Web/Program.cs:25-42)
- Azure provisioning appears to deploy only the `web` service in `azure.yaml`; local docs still require both `Web` and `PublicApi` for the full app. (azure.yaml:3-8, README.md:95-98)

## 8. Observability
- Logging:
- Both runtime apps add console logging at startup. (src/Web/Program.cs:22-23, src/PublicApi/Program.cs:30-32)
- Application code uses `ILogger` and an `IAppLogger` wrapper implemented by `LoggerAdapter<T>`. (src/Web/Services/CatalogViewModelService.cs:20-24, src/Web/Pages/Basket/Checkout.cshtml.cs:22-35, src/Infrastructure/Logging/LoggerAdapter.cs:6-22)
- Metrics:
- Unknown. I found health checks, but no explicit metrics pipeline or metrics backend registration in inspected startup files. Inspect external Azure resources or additional packages if metrics are expected. (src/Web/Program.cs:85-89, src/Web/Program.cs:151-168, Directory.Packages.props:10-71)
- Tracing:
- Unknown. I did not find OpenTelemetry or Application Insights setup in inspected startup/config files. Inspect Azure resources and untracked environment-specific config next. (src/Web/Program.cs:22-202, src/PublicApi/Program.cs:26-179, Directory.Packages.props:10-71)
- Alerts (if present):
- Unknown. No alerting rules or monitor resources were visible in `infra/` or GitHub workflows; inspect Azure subscription resources outside this repo next. (infra/main.bicep:47-144, .github/workflows/dotnetcore.yml:1-22)
- Health/error reporting:
- `Web` exposes `/health`, `home_page_health_check`, and `api_health_check`; the checks make live HTTP calls and look for expected page content. (src/Web/Program.cs:151-168, src/Web/Program.cs:196-197, src/Web/HealthChecks/HomePageHealthCheck.cs:18-34, src/Web/HealthChecks/ApiHealthCheck.cs:19-33)
- `PublicApi` uses `ExceptionMiddleware` to turn exceptions into JSON error responses and uses Swagger for interactive inspection. (src/PublicApi/Middleware/ExceptionMiddleware.cs:19-53, src/PublicApi/Program.cs:165-173)

## 9. Testing & quality gates
- Types of tests present:
- Unit tests use xUnit and NSubstitute against `ApplicationCore` and some `Web` handlers. (tests/UnitTests/UnitTests.csproj:11-26)
- Integration tests use xUnit with EF Core in-memory repositories against `Infrastructure` services. (tests/IntegrationTests/IntegrationTests.csproj:8-26, tests/IntegrationTests/Repositories/BasketRepositoryTests/SetQuantities.cs:19-39)
- Functional tests use `WebApplicationFactory` plus in-memory contexts against both `Web` and `PublicApi`. (tests/FunctionalTests/FunctionalTests.csproj:16-29, tests/FunctionalTests/Web/WebTestFixture.cs:19-55, tests/FunctionalTests/PublicApi/ApiTestFixture.cs:16-42)
- `PublicApiIntegrationTests` use MSTest and `WebApplicationFactory<Program>` against the API. (tests/PublicApiIntegrationTests/PublicApiIntegrationTests.csproj:21-35, tests/PublicApiIntegrationTests/ProgramTest.cs:7-25)
- How they run in CI:
- GitHub Actions builds and tests `eShopOnWeb.sln` on `ubuntu-latest` with `.NET 8`. (.github/workflows/dotnetcore.yml:5-22)
- A code coverage settings file exists, but I did not find it referenced in the CI workflow. (CodeCoverage.runsettings:1-106, .github/workflows/dotnetcore.yml:18-22)
- Coverage gaps / risks:
- Checkout happy-path coverage exists, but the production checkout path still contains a hard-coded shipping address, so address validation and persistence behavior are underrepresented. (tests/FunctionalTests/Web/Pages/Basket/CheckoutTest.cs:19-67, src/Web/Pages/Basket/Checkout.cshtml.cs:55-58)
- API auth functional coverage appears incomplete: a functional `AuthenticateEndpoint` test file exists but is entirely commented out. (tests/FunctionalTests/PublicApi/AuthEndpoints/AuthenticateEndpoint.cs:1-43)
- Local verification status in this review: Unknown; the documented and CI-backed test path is `dotnet test ./eShopOnWeb.sln --configuration Release`. (.github/workflows/dotnetcore.yml:18-22)

## 10. Migration hotspots / tech debt (evidence-based)
- Hotspot: secrets and demo auth are hard-coded in source and Docker config, including `DEFAULT_PASSWORD`, `JWT_SECRET_KEY`, and the Docker SQL password. Why it matters: this is safe for a sample, but it is the highest-risk area to carry forward into real environments. Suggested seam: `ISecretSettingsProvider` with methods like `GetJwtSigningKey()` and `GetBootstrapCredentials()`. Test strategy: unit-test config binding plus startup smoke tests per environment. Migration risk: High. (src/ApplicationCore/Constants/AuthorizationConstants.cs:5-11, src/Infrastructure/Identity/AppIdentityDbContextSeed.cs:18-31, docker-compose.yml:22-24)
- Hotspot: checkout orchestration is split across the Razor Page and application services, and the current shipping address is hard-coded in the page model. Why it matters: feature work around addresses, payments, and order confirmation will land in UI code unless this flow is centralized. Suggested seam: `ICheckoutOrchestrator.CheckoutAsync(userName, basketId, items, shippingAddress)`. Test strategy: unit tests around orchestration decisions plus functional tests for checkout success/failure. Migration risk: Medium. (src/Web/Pages/Basket/Checkout.cshtml.cs:44-68, src/ApplicationCore/Services/OrderService.cs:30-52, src/ApplicationCore/Services/BasketService.cs:47-64)
- Hotspot: session revocation uses `IMemoryCache` and includes a TODO noting it should become distributed for multi-host scenarios. Why it matters: logout revocation appears host-local today, which is a scaling boundary. Suggested seam: `IRevokedSessionStore` with `RevokeAsync` and `IsRevokedAsync`. Test strategy: unit tests for revocation semantics and multi-node integration tests once backed by a distributed cache. Migration risk: Medium. (src/Web/Configuration/RevokeAuthenticationEvents.cs:11-33, src/Web/Configuration/ConfigureCookieSettings.cs:22-37, src/Web/Controllers/UserController.cs:45-57)
- Hotspot: catalog administration logic is spread across the Blazor admin client, `UserController` token issuance, and multiple `PublicApi` endpoint classes. Why it matters: the admin surface is already a natural extraction candidate, but auth/session coupling crosses app boundaries. Suggested seam: `ICatalogAdminService` with `ListPageAsync`, `CreateAsync`, `UpdateAsync`, and `DeleteAsync`. Test strategy: API contract tests plus admin UI smoke tests using admin and non-admin tokens. Migration risk: Medium. (src/Web/Controllers/UserController.cs:35-104, src/PublicApi/CatalogItemEndpoints/CatalogItemListPagedEndpoint.cs:29-72, src/PublicApi/CatalogItemEndpoints/CreateCatalogItemEndpoint.cs:27-75, src/PublicApi/CatalogItemEndpoints/UpdateCatalogItemEndpoint.cs:25-66, src/BlazorAdmin/Pages/CatalogItemPage/List.razor:1-2)
- Hotspot: `CatalogItemListPagedEndpoint` contains an explicit `Task.Delay(1000)`. Why it matters: it introduces artificial latency into a catalog read path and can distort perceived runtime behavior and tests. Suggested seam: remove the delay behind a temporary diagnostic abstraction if needed (`ICatalogLatencySimulator`). Test strategy: API response-time assertion or unit tests that verify the handler does not intentionally wait. Migration risk: Low. (src/PublicApi/CatalogItemEndpoints/CatalogItemListPagedEndpoint.cs:40-42)
- Candidate boundaries for extraction:
- Catalog read/query boundary: facade `ICatalogQueryService.GetPageAsync(pageSize, pageIndex, brandId, typeId)` backed by the existing spec/repository stack. Start with characterization tests around `CatalogViewModelService` and `CatalogItemListPagedEndpoint`. Risk: Low. (src/Web/Services/CatalogViewModelService.cs:40-110, src/PublicApi/CatalogItemEndpoints/CatalogItemListPagedEndpoint.cs:40-72)
- Order history/query boundary: facade `IOrderQueries.GetMyOrdersAsync(userName)` and `GetOrderDetailsAsync(userName, orderId)` backed by the existing MediatR handlers. Start with handler unit tests and controller view-model assertions. Risk: Low. (src/Web/Controllers/OrderController.cs:22-43, src/Web/Features/MyOrders/GetMyOrdersHandler.cs:18-31, src/Web/Features/OrderDetails/GetOrderDetailsHandler.cs:18-44)
- Identity/token boundary: facade `IUserSessionService.GetCurrentUserAsync()`, `LogoutAsync()`, and `CreateApiTokenAsync(userName)`. Start with controller tests around `UserController` and API auth integration tests. Risk: Medium. (src/Web/Controllers/UserController.cs:35-104, src/Infrastructure/Identity/IdentityTokenClaimService.cs:23-45, src/PublicApi/AuthEndpoints/AuthenticateEndpoint.cs:36-58)

## 11. Open questions
- How is `PublicApi` deployed in production Azure? The local docs require it for admin features, but `azure.yaml` only declares the `web` service. Inspect any external deployment pipeline or Azure resources next. (README.md:95-98, azure.yaml:3-8)
- Are metrics, tracing, and alerting configured outside this repo? I found console logging and health checks, but no explicit telemetry/alert resources here. Inspect Azure Monitor/Application Insights resources next. (src/Web/Program.cs:22-23, src/Web/Program.cs:85-89, src/PublicApi/Program.cs:30-32, infra/main.bicep:47-144)
- Is this intended to remain a sample-only auth model? Source includes demo users, a hard-coded default password, and a TODO to move the JWT secret to environment variables. Inspect roadmap/issues or downstream forks next. (src/ApplicationCore/Constants/AuthorizationConstants.cs:7-11, src/Infrastructure/Identity/AppIdentityDbContextSeed.cs:18-31)
- What is the intended production basket/session revocation store? The current implementation explicitly calls out the lack of distributed cache support. Inspect infrastructure decisions or open issues next. (src/Web/Configuration/RevokeAuthenticationEvents.cs:11-33)
- Are there non-repo compliance/privacy controls for user/order data? I found Identity, orders, and addresses in persistence code, but no explicit PII/data-retention policy in repo. Inspect external policy docs and deployment configuration next. (src/Infrastructure/Data/CatalogContext.cs:14-20, src/ApplicationCore/Services/OrderService.cs:41-51)

## 12. First 90 minutes checklist
- Open `README.md`, `src/Web/Program.cs`, and `src/PublicApi/Program.cs` first to understand the two-process local dev story and environment branching. (README.md:95-166, src/Web/Program.cs:22-202, src/PublicApi/Program.cs:26-179)
- Run `Web` and `PublicApi` locally on their default ports, then verify `/`, `/admin`, and `/swagger`. (src/Web/Properties/launchSettings.json:20-37, src/PublicApi/Properties/launchSettings.json:11-19, README.md:95-98)
- Decide whether you want in-memory or SQL-backed local data before doing feature work, and apply the documented EF migration steps if you need persistence. (src/Web/appsettings.json:6-11, README.md:100-145, src/Infrastructure/Dependencies.cs:13-38)
- Trace one storefront flow end-to-end: `Index` -> basket page -> `BasketService` -> `CatalogContext`. (src/Web/Pages/Index.cshtml.cs:18-20, src/Web/Pages/Basket/Index.cshtml.cs:33-52, src/ApplicationCore/Services/BasketService.cs:23-38, src/Infrastructure/Data/CatalogContext.cs:14-20)
- Trace one admin flow end-to-end: `/admin` -> `UserController` token -> `PublicApi` catalog endpoint. (src/Web/Pages/Admin/Index.cshtml:68-71, src/Web/Controllers/UserController.cs:35-104, src/PublicApi/CatalogItemEndpoints/CreateCatalogItemEndpoint.cs:27-75)
- Skim `ApplicationCore/Interfaces` and `ApplicationCore/Specifications` before changing data access; they are the main abstraction boundary used by both apps. (src/ApplicationCore/Interfaces, src/ApplicationCore/Specifications)
- Review the test projects that match your area and prefer adding characterization tests before refactoring hotspot code. (tests/UnitTests, tests/IntegrationTests, tests/FunctionalTests, tests/PublicApiIntegrationTests)
- Treat auth secrets, checkout orchestration, and session revocation as high-attention areas if you are moving this code beyond sample usage. (src/ApplicationCore/Constants/AuthorizationConstants.cs:7-11, src/Web/Pages/Basket/Checkout.cshtml.cs:55-58, src/Web/Configuration/RevokeAuthenticationEvents.cs:11-33)
