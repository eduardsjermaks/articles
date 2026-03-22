# Role
You are acting as an impartial Senior Software Architect and Evaluation Model. You are evaluating two architecture orientation documents for the same repository.

# Context & Constraints
- You do NOT have access to the source code repository. 
- You must evaluate strictly based on the provided text.
- **Bias Warning:** Do NOT prefer verbosity, professional formatting, or "corporate" writing styles. Penalize flowery language and generic definitions.
- **Preference:** Reward technical correctness, internal consistency, cited evidence within the text (e.g., file paths, library names), and rigorous reasoning discipline.

# Pro-Level Directives
- **Chain of Thought First:** Before outputting any scores, provide a detailed step-by-step analysis. Identify internal contradictions, compare specific technical claims, and look for "hallucination markers" (generic claims) versus "grounded markers" (specific file paths/logic flows).
- **Gap Analysis:** Look for the "negative space." What critical architectural domains (e.g., authentication, error handling, CI/CD, data persistence) are completely omitted or glossed over?
- **Implicit vs. Explicit:** Heavily differentiate between what a document implies and what it explicitly proves.
- **Penalty Justification:** If you award a score of 3 or below in any category, you MUST provide a direct quote of the vague, contradictory, or fluffy text that earned the penalty.

# Evaluation Rubric (Score 0–5 per category)
1. **Evidence Grounding:** Does the document cite specific modules, files, or patterns rather than vague generalities?
2. **Structural Accuracy:** Is the internal logic consistent? Do the described components actually fit together logically?
3. **Dependency Mapping:** How well does it identify external integrations, internal coupling, and third-party libraries?
4. **Critical Flow Identification:** Does it map the "Happy Path" of data through the system (Entry point -> Logic -> Storage)?
5. **Migration Insight Quality:** Does it identify technical debt, "gotchas," legacy patterns, or risks that would impact a migration?
6. **Epistemic Discipline:** How does it handle uncertainty? Does it clearly distinguish between known facts ("The system does X") and assumptions ("The system appears to do X")?
7. **Signal-to-Noise Ratio:** Is the document concise and information-dense, or is it filled with filler?

# Required Output Structure

## 1. Initial Analysis & Gap Detection (Your "Thinking" Phase)
- Outline your analysis of both documents.
- Note any contradictions, lack of evidence, or missing critical architectural components.

## 2. Document A Evaluation
- **Scores:** [List 1–7 with 0-5 scores]
- **Justification:** Explain each score. Quote the text to prove why it succeeded, or quote the exact text that caused a penalty (for scores <= 3).

## 3. Document B Evaluation
- **Scores:** [List 1–7 with 0-5 scores]
- **Justification:** Explain each score. Quote the text to prove why it succeeded, or quote the exact text that caused a penalty (for scores <= 3).

## 4. Head-to-Head Comparison
- **Conflict Resolution:** If Doc A and Doc B contradict each other on a technical point, analyze which one provides better logical reasoning or supporting evidence.
- **Depth & Reliability:** Compare the signal-to-noise ratio and epistemic discipline of both.

## 5. Final Verdict
- **The Safer Document for Migration:** Which document would a Lead Engineer trust more for a high-stakes migration?
- **Core Reasoning differences:** The primary technical differentiator between the two.
- **Confidence Level:** (Low / Medium / High) - Explain why.

---
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

---
# Project Orientation (Generated)
## 1. What this project is
- Purpose: `eShopOnWeb` is a sample ASP.NET Core reference application that the root README describes as a single-process, monolithic web app used to demonstrate the patterns from Microsoft's "Architecting Modern Web Applications with ASP.NET Core and Azure" guidance, not a production-complete e-commerce system. (README.md:8-8,18-18,25-27)
- Status: the root README says active development moved to `https://github.com/dotnet/eShop`, and it also points to a community-supported fork, so treat this repository as a learning/reference codebase first. (README.md:3-6)
- Main use-cases: the codebase appears to support storefront browsing/filtering, basket management, checkout, and order history in `Web`, plus catalog administration at `/admin` through the Blazor admin client and `PublicApi`. (src/Web/Pages/Index.cshtml.cs:7-21, src/Web/Pages/Basket/Index.cshtml.cs:28-99, src/Web/Pages/Basket/Checkout.cshtml.cs:14-97, src/Web/Controllers/OrderController.cs:10-43, src/BlazorAdmin/Pages/CatalogItemPage/List.razor:1-67)
- Who uses it: the README explicitly targets .NET developers/newcomers, while the runtime code also distinguishes shopper and administrator paths through authenticated order pages and admin-only catalog pages/endpoints. (README.md:8-8, src/Web/Controllers/OrderController.cs:10-43, src/Web/Pages/Admin/EditCatalogItem.cshtml.cs:11-37, src/BlazorAdmin/Pages/CatalogItemPage/List.razor:1-67, src/PublicApi/CatalogItemEndpoints/CreateCatalogItemEndpoint.cs:27-75)

## 2. Quick start (developer)
- Prereqs: the repo targets .NET SDK `8.0.x`; the Web project includes a local `dotnet-ef` tool manifest; optional local/dev paths include SQL Server or LocalDB, Docker, Azure Developer CLI, or the provided dev container image `mcr.microsoft.com/devcontainers/dotnet:8.0`. (global.json:1-5, src/Web/.config/dotnet-tools.json:1-12, README.md:53-69,100-129,146-165, .devcontainer/devcontainer.json:4-27)
- How to run locally: the README says most storefront functionality works with the Web app alone, but the admin page needs `PublicApi` running too; launch settings put `Web` on `https://localhost:5001` and `PublicApi` on `https://localhost:5099`, and the admin UI route is `/admin`. (README.md:95-96, src/Web/Properties/launchSettings.json:20-37, src/PublicApi/Properties/launchSettings.json:11-19, src/BlazorAdmin/Pages/CatalogItemPage/List.razor:1-2)
- How to run tests: the checked-in GitHub Actions workflow builds and tests `eShopOnWeb.sln` with `dotnet build ./eShopOnWeb.sln --configuration Release` and `dotnet test ./eShopOnWeb.sln --configuration Release`. (.github/workflows/dotnetcore.yml:18-22)
- Common pitfalls:
- `src/Web/appsettings.json` enables `UseOnlyInMemoryDatabase`, but `src/PublicApi/appsettings.json` does not; because `PublicApi` always calls `Infrastructure.Dependencies.ConfigureServices`, the admin path appears to need SQL/LocalDB unless test config or environment-specific config overrides it. (src/Web/appsettings.json:6-10, src/PublicApi/appsettings.json:6-10, src/PublicApi/Program.cs:30-45, src/Infrastructure/Dependencies.cs:11-38)
- Persistent local databases require running EF Core migrations for both `CatalogContext` and `AppIdentityDbContext` from the Web project. (README.md:100-144)
- The README warns that if you run the applications manually you need to stop them before rebuilding or you can hit file-locking issues. (README.md:98-98)
- Both `Web` and `PublicApi` seed catalog and identity data at startup, and the identity seed creates hard-coded demo/admin users from code constants. (src/Web/Program.cs:120-139, src/PublicApi/Program.cs:129-148, src/Infrastructure/Identity/AppIdentityDbContextSeed.cs:18-30, src/ApplicationCore/Constants/AuthorizationConstants.cs:7-11)

## 3. Architecture at a glance
- High-level component diagram:
```text
Browser
  |-- "/" / basket / orders --> Web (MVC + Razor Pages + Identity cookies + health checks)
  |-- "/admin" ---------------> BlazorAdmin WASM served by Web
                                   |
                                   v
                              PublicApi (JWT + admin role)

Web and PublicApi
  -> ApplicationCore (entities, specs, services, repository interfaces)
  -> Infrastructure (EF Core repo, DbContexts, Identity, logging/email adapters)
  -> Catalog DB + Identity DB
```
- Diagram basis: `Web` references `ApplicationCore`, `Infrastructure`, `BlazorAdmin`, and `BlazorShared`; `PublicApi` references `ApplicationCore` and `Infrastructure`; `Web` serves Blazor framework files and maps a fallback file; `BlazorAdmin` is routed at `/admin`; `PublicApi` exposes JWT-protected admin endpoints. (src/Web/Web.csproj:42-47, src/PublicApi/PublicApi.csproj:34-37, src/Web/Program.cs:184-199, src/BlazorAdmin/Pages/CatalogItemPage/List.razor:1-67, src/PublicApi/Program.cs:54-70, src/PublicApi/CatalogItemEndpoints/UpdateCatalogItemEndpoint.cs:25-66)
- Runtime processes: confirmed entrypoints are the `Web` ASP.NET Core app, the `PublicApi` ASP.NET Core app, and the browser-side `BlazorAdmin` WebAssembly client. Workers/schedulers: Unknown; no separate worker entrypoint is evident in the inspected startup files, so inspect future host projects or external repos if that changes. (src/Web/Program.cs:22-202, src/PublicApi/Program.cs:26-179, src/BlazorAdmin/Program.cs:16-40)
- Data stores and queues: `CatalogContext` stores catalog, baskets, orders, order items, and basket items; `AppIdentityDbContext` stores ASP.NET Identity data; local/dev can use SQL Server or in-memory stores depending config. Queues/message brokers: Unknown; none were evident in the inspected startup and infra files. (src/Infrastructure/Data/CatalogContext.cs:9-26, src/Infrastructure/Identity/AppIdentityDbContext.cs:7-20, src/Infrastructure/Dependencies.cs:11-38, src/Web/Program.cs:25-43, infra/main.bicep:47-144)

## 4. Repository map
- Top-level directories:
- `.devcontainer`: containerized development environment config for .NET 8. (.devcontainer/devcontainer.json:4-27)
- `.github/workflows`: CI build/test workflow plus a separate code-index workflow. (.github/workflows/dotnetcore.yml:1-22, .github/workflows/richnav.yml:1-24)
- `infra`: Azure Bicep templates for App Service, SQL, and Key Vault deployment. (infra/main.bicep:47-144)
- `src`: application/runtime projects. (src)
- `tests`: unit, integration, functional, and Public API integration test projects. (tests)
- Root solution/config assets: `eShopOnWeb.sln`, `Everything.sln`, `Directory.Packages.props`, `global.json`, `docker-compose.yml`, `azure.yaml`, and `README.md`. (eShopOnWeb.sln), (Everything.sln), (Directory.Packages.props:1-72), (global.json:1-5), (docker-compose.yml:1-24), (azure.yaml:3-8), (README.md:1-165)
- Major modules:
- `src/ApplicationCore`: domain entities and service/repository abstractions for catalog, basket, and order behavior; key public interfaces are `IRepository<T>`, `IReadRepository<T>`, `IBasketService`, and `IOrderService`; key files are the entity folders plus `BasketService`, `OrderService`, and the specification classes. (src/ApplicationCore/ApplicationCore.csproj:1-20, src/ApplicationCore/Interfaces/IRepository.cs:1-7, src/ApplicationCore/Interfaces/IReadRepository.cs:1-7, src/ApplicationCore/Interfaces/IBasketService.cs:1-14, src/ApplicationCore/Interfaces/IOrderService.cs:1-9, src/ApplicationCore/Services/BasketService.cs:11-85, src/ApplicationCore/Services/OrderService.cs:12-52, src/ApplicationCore/Entities)
- `src/Infrastructure`: EF Core, Identity, logging, and email implementations backing `ApplicationCore`; main public implementations are `EfRepository<T>`, `CatalogContext`, `AppIdentityDbContext`, `IdentityTokenClaimService`, `LoggerAdapter<T>`, and `EmailSender`. (src/Infrastructure/Infrastructure.csproj:1-21, src/Infrastructure/Data/EfRepository.cs:1-10, src/Infrastructure/Data/CatalogContext.cs:9-26, src/Infrastructure/Identity/AppIdentityDbContext.cs:7-20, src/Infrastructure/Identity/IdentityTokenClaimService.cs:14-45, src/Infrastructure/Logging/LoggerAdapter.cs:6-22, src/Infrastructure/Services/EmailSender.cs:6-14)
- `src/Web`: the main storefront host; responsibilities include startup/config, MVC/Razor endpoints, basket/order/account flows, Blazor asset hosting, health checks, and UI-specific view-model services such as `IBasketViewModelService`, `ICatalogViewModelService`, and `ICatalogItemViewModelService`. (src/Web/Web.csproj:1-86, src/Web/Program.cs:22-202, src/Web/Interfaces/IBasketViewModelService.cs:6-11, src/Web/Interfaces/ICatalogItemViewModelService.cs:6-9, src/Web/Interfaces/ICatalogViewModelService.cs:8-13, src/Web/Pages, src/Web/Controllers, src/Web/Services)
- `src/PublicApi`: standalone API host that organizes endpoints as individual classes rather than MVC controllers; key files are `Program.cs`, endpoint folders such as `CatalogItemEndpoints` and `AuthEndpoints`, plus `ExceptionMiddleware`. (src/PublicApi/PublicApi.csproj:1-40, src/PublicApi/Program.cs:26-181, src/PublicApi/README.md:1-3, src/PublicApi/CatalogItemEndpoints, src/PublicApi/AuthEndpoints, src/PublicApi/Middleware/ExceptionMiddleware.cs:10-54)
- `src/BlazorAdmin`: browser-side admin UI for catalog management; public-facing pieces are the `/admin` page, `CustomAuthStateProvider`, and client services such as `ICatalogItemService` implementations and `HttpService`. (src/BlazorAdmin/BlazorAdmin.csproj:1-30, src/BlazorAdmin/Program.cs:16-40, src/BlazorAdmin/Pages/CatalogItemPage/List.razor:1-67, src/BlazorAdmin/CustomAuthStateProvider.cs:13-85, src/BlazorAdmin/Services/CatalogItemService.cs:11-95, src/BlazorAdmin/Services/HttpService.cs:11-95)
- `src/BlazorShared`: shared contracts/models used by the admin client and API, including auth/user info and catalog DTO/service interfaces. (src/BlazorShared/BlazorShared.csproj:1-13, src/BlazorShared/Interfaces/ICatalogItemService.cs:7-15, src/BlazorShared/Interfaces/ICatalogLookupDataService.cs:7-10, src/BlazorShared/Authorization/Constants.cs:3-8, src/BlazorShared/Authorization/UserInfo.cs)
- `tests`: `UnitTests` focus on `ApplicationCore` and some Web helpers, `IntegrationTests` target repository behavior with EF InMemory, `FunctionalTests` exercise HTTP flows with `WebApplicationFactory`, and `PublicApiIntegrationTests` target API endpoints with MSTest. (tests/UnitTests/UnitTests.csproj:11-26, tests/IntegrationTests/IntegrationTests.csproj:8-26, tests/FunctionalTests/FunctionalTests.csproj:16-29, tests/PublicApiIntegrationTests/PublicApiIntegrationTests.csproj:21-35)

## 5. Critical flows
- **Catalog browse and filter flow**
- Trigger/entrypoint: `GET /` reaches `IndexModel.OnGet`. (src/Web/Pages/Index.cshtml.cs:18-21)
- Steps: `IndexModel` calls `ICatalogViewModelService.GetCatalogItems`; Web registers `CachedCatalogViewModelService` for that interface; the cached service uses `IMemoryCache` keys from `CacheHelpers`; the underlying `CatalogViewModelService` builds filter/pagination specifications, queries repositories, loads brand/type lookups, and composes picture URIs. (src/Web/Pages/Index.cshtml.cs:18-21, src/Web/Configuration/ConfigureWebServices.cs:11-18, src/Web/Services/CachedCatalogViewModelService.cs:10-49, src/Web/Extensions/CacheHelpers.cs:5-23, src/Web/Services/CatalogViewModelService.cs:40-110)
- Side effects: this path populates in-memory caches with a 30-second sliding expiration for catalog items, brands, and types. (src/Web/Services/CachedCatalogViewModelService.cs:22-49, src/Web/Extensions/CacheHelpers.cs:5-23)
- Failure modes: stale reads are possible for up to 30 seconds because the cache is time-based, and no page-specific exception handling is evident in the `IndexModel` or catalog view-model services inspected. (src/Web/Pages/Index.cshtml.cs:18-21, src/Web/Services/CachedCatalogViewModelService.cs:31-49, src/Web/Services/CatalogViewModelService.cs:40-110)
- **Basket, login transfer, checkout, and order creation**
- Trigger/entrypoint: basket add/update happens in `Pages/Basket/Index`, anonymous-to-user basket transfer happens after successful login, and order creation happens in `Pages/Basket/Checkout`. (src/Web/Pages/Basket/Index.cshtml.cs:28-99, src/Web/Areas/Identity/Pages/Account/Login.cshtml.cs:68-118, src/Web/Pages/Basket/Checkout.cshtml.cs:39-68)
- Steps: the basket page resolves the current user from auth or a cookie, loads the selected catalog item, and calls `IBasketService.AddItemToBasket`; login transfers a GUID-backed anonymous basket to the authenticated user; checkout updates quantities, calls `IOrderService.CreateOrderAsync`, and then deletes the basket. (src/Web/Pages/Basket/Index.cshtml.cs:33-99, src/ApplicationCore/Services/BasketService.cs:23-84, src/Web/Areas/Identity/Pages/Account/Login.cshtml.cs:80-118, src/Web/Pages/Basket/Checkout.cshtml.cs:44-68, src/ApplicationCore/Services/OrderService.cs:30-52)
- Side effects: the basket flow can create a long-lived basket cookie, write basket/order rows, and delete the basket after order creation. (src/Web/Pages/Basket/Index.cshtml.cs:68-99, src/ApplicationCore/Services/BasketService.cs:23-84, src/ApplicationCore/Services/OrderService.cs:30-52, src/Web/Pages/Basket/Checkout.cshtml.cs:55-59)
- Failure modes: checkout hard-codes the shipping address in the page model, empty baskets redirect back to `/Basket/Index`, and order creation plus basket deletion happen in separate calls with no transaction boundary evident in this path. (src/Web/Pages/Basket/Checkout.cshtml.cs:55-64, src/ApplicationCore/Services/OrderService.cs:30-52)
- **Admin authentication and catalog CRUD**
- Trigger/entrypoint: the admin UI route is `/admin`; auth state comes from `CustomAuthStateProvider`, and create/update/delete requests go to `PublicApi` endpoints. (src/BlazorAdmin/Pages/CatalogItemPage/List.razor:1-67, src/BlazorAdmin/CustomAuthStateProvider.cs:31-85, src/PublicApi/CatalogItemEndpoints/CreateCatalogItemEndpoint.cs:27-75, src/PublicApi/CatalogItemEndpoints/UpdateCatalogItemEndpoint.cs:25-66, src/PublicApi/CatalogItemEndpoints/DeleteCatalogItemEndpoint.cs:18-40)
- Steps: the admin client calls Web's `UserController` to get current-user claims plus a JWT, stores that token in the `HttpClient` default headers, then uses `CatalogItemService`/`HttpService` to call `PublicApi`; `PublicApi` enforces admin role + JWT on catalog mutations and persists through `IRepository<CatalogItem>`. (src/BlazorAdmin/CustomAuthStateProvider.cs:50-84, src/Web/Controllers/UserController.cs:35-104, src/BlazorAdmin/Services/CatalogItemService.cs:29-95, src/BlazorAdmin/Services/HttpService.cs:25-95, src/PublicApi/CatalogItemEndpoints/CreateCatalogItemEndpoint.cs:27-75, src/PublicApi/CatalogItemEndpoints/UpdateCatalogItemEndpoint.cs:25-66, src/PublicApi/CatalogItemEndpoints/DeleteCatalogItemEndpoint.cs:18-40)
- Side effects: the admin client caches lookup data and item lists in browser local storage for one minute, and new catalog items are forced onto a default placeholder image instead of uploading arbitrary files. (src/BlazorAdmin/Services/CachedCatalogLookupDataServiceDecorator .cs:29-50, src/BlazorAdmin/Services/CachedCatalogItemServiceDecorator.cs:27-112, src/PublicApi/CatalogItemEndpoints/CreateCatalogItemEndpoint.cs:53-60)
- Failure modes: `HttpService` returns `null` on many non-success responses and only shows a generic `"Error"` toast for failed `PUT`s, the paged list endpoint adds an artificial one-second delay, and duplicate catalog names raise `DuplicateException` which the API middleware maps to HTTP 409. (src/BlazorAdmin/Services/HttpService.cs:25-95, src/PublicApi/CatalogItemEndpoints/CatalogItemListPagedEndpoint.cs:40-71, src/PublicApi/CatalogItemEndpoints/CreateCatalogItemEndpoint.cs:43-48, src/PublicApi/Middleware/ExceptionMiddleware.cs:31-53)
- **Startup/bootstrap, migrations, and seed data**
- Trigger/entrypoint: both `Web` and `PublicApi` run seeding during startup after building the app. (src/Web/Program.cs:116-139, src/PublicApi/Program.cs:125-148)
- Steps: startup selects local or prod DB wiring, resolves the DB contexts and identity managers, runs EF migrations when the DB provider is SQL Server, seeds catalog brands/types/items, creates the administrator role, and creates `demouser@microsoft.com` plus `admin@microsoft.com`. (src/Web/Program.cs:25-43, src/PublicApi/Program.cs:34-45, src/Infrastructure/Data/CatalogContextSeed.cs:12-47, src/Infrastructure/Identity/AppIdentityDbContextSeed.cs:10-30)
- Side effects: schema migrations, sample catalog inserts, seeded users/roles, and console/application logs on startup. (src/Infrastructure/Data/CatalogContextSeed.cs:19-47, src/Infrastructure/Identity/AppIdentityDbContextSeed.cs:13-30, src/Web/Program.cs:118-139, src/PublicApi/Program.cs:127-148)
- Failure modes: both apps attempt startup seeding, and `CatalogContextSeed` recursively retries on failure but then unconditionally rethrows after the recursive call, so transient startup DB issues deserve careful review. (src/Web/Program.cs:120-139, src/PublicApi/Program.cs:129-148, src/Infrastructure/Data/CatalogContextSeed.cs:48-56)

## 6. Dependencies
- Internal module dependencies:
- `Web` depends on `ApplicationCore`, `BlazorAdmin`, `BlazorShared`, and `Infrastructure`. (src/Web/Web.csproj:42-47)
- `PublicApi` depends on `ApplicationCore` and `Infrastructure`. (src/PublicApi/PublicApi.csproj:34-37)
- `Infrastructure` depends on `ApplicationCore`. (src/Infrastructure/Infrastructure.csproj:15-17)
- `ApplicationCore` depends on `BlazorShared` at the project level, so the build graph already crosses from core logic into the shared client-contract package. (src/ApplicationCore/ApplicationCore.csproj:16-18)
- `BlazorAdmin` depends on `BlazorShared`. (src/BlazorAdmin/BlazorAdmin.csproj:14-16)
- External services:
- Databases: LocalDB/SQL Server in local config, SQL Server or Azure SQL in Docker/Azure paths, and optional in-memory databases for some environments/tests. (src/Web/appsettings.json:6-10, src/PublicApi/appsettings.json:6-10, docker-compose.yml:18-24, infra/main.bicep:76-105, src/Infrastructure/Dependencies.cs:11-38)
- Secrets/config backing: Azure Key Vault is used by the non-development Web startup path, and the Azure infra templates provision/access that vault. (src/Web/Program.cs:29-42, infra/main.bicep:47-74,108-144)
- Identity/auth: ASP.NET Core Identity backs the Web app and also issues JWTs used by `PublicApi` and `BlazorAdmin`. (src/Web/Program.cs:45-58, src/PublicApi/Program.cs:36-70, src/Infrastructure/Identity/IdentityTokenClaimService.cs:23-45)
- Outbound third-party APIs: Unknown; no external HTTP integrations were evident beyond internal health-check calls and Azure Key Vault usage. Inspect future service classes or infrastructure if new integrations are introduced. (src/Web/HealthChecks/ApiHealthCheck.cs:19-33, src/Web/HealthChecks/HomePageHealthCheck.cs:18-34, src/Web/Program.cs:29-42)
- Build/deploy dependencies:
- Package/build tooling is centrally versioned in `Directory.Packages.props`, targets `net8.0`, and uses EF Core, ASP.NET Core Identity, MediatR, AutoMapper, Swagger, Blazored.LocalStorage, and test tooling such as xUnit, MSTest, NSubstitute, and coverlet. (Directory.Packages.props:3-71)
- Container support exists via `docker-compose.yml` plus Dockerfiles in `src/Web` and `src/PublicApi`. (docker-compose.yml:3-24, docker-compose.override.yml:2-20, src/Web/Dockerfile:10-27, src/PublicApi/Dockerfile:3-25)
- Azure deployment support exists via `azure.yaml` and `infra/main.bicep`, which target App Service, Key Vault, SQL databases, and an App Service plan. (azure.yaml:3-8, infra/main.bicep:47-144)

## 7. Configuration & environments
- Where config lives:
- Runtime app settings live in `src/Web/appsettings*.json`, `src/PublicApi/appsettings*.json`, and `src/BlazorAdmin/wwwroot/appsettings*.json`. (src/Web/appsettings.json:1-21, src/Web/appsettings.Development.json:1-13, src/Web/appsettings.Docker.json:1-17, src/PublicApi/appsettings.json:1-20, src/PublicApi/appsettings.Development.json:1-13, src/PublicApi/appsettings.Docker.json:1-17, src/BlazorAdmin/wwwroot/appsettings.json:1-14)
- Local launch behavior lives in each app's `launchSettings.json`. (src/Web/Properties/launchSettings.json:1-39, src/PublicApi/Properties/launchSettings.json:1-47, src/BlazorAdmin/Properties/launchSettings.json:1-30)
- Build/deploy configuration lives in `Directory.Packages.props`, `global.json`, `docker-compose*.yml`, `azure.yaml`, and `infra/main.bicep`. (Directory.Packages.props:1-72, global.json:1-5, docker-compose.yml:1-24, docker-compose.override.yml:1-20, azure.yaml:1-8, infra/main.bicep:1-144)
- Environment variables:
- Docker override sets `ASPNETCORE_ENVIRONMENT=Docker` and `ASPNETCORE_URLS=http://+:8080` for both server apps. (docker-compose.override.yml:3-20)
- Non-development Web startup reads `AZURE_KEY_VAULT_ENDPOINT`, `AZURE_SQL_CATALOG_CONNECTION_STRING_KEY`, and `AZURE_SQL_IDENTITY_CONNECTION_STRING_KEY`, and the Azure Bicep template provisions those settings. (src/Web/Program.cs:29-42, infra/main.bicep:59-63,135-144)
- Azure parameters also reference `AZURE_ENV_NAME`, `AZURE_LOCATION`, `AZURE_PRINCIPAL_ID`, and Key Vault-backed password generation. (infra/main.parameters.json:4-19)
- Secrets handling:
- Local user-secret IDs are configured for `Web` and `PublicApi`. (src/Web/Web.csproj:3-9, src/PublicApi/PublicApi.csproj:3-9)
- Azure deployment stores SQL/admin passwords in Key Vault and grants the web app identity access. (README.md:89-91, infra/main.bicep:67-74,108-144, infra/main.parameters.json:14-19)
- Security-sensitive values are also checked into source today: the Docker compose file contains a hard-coded SQL `SA_PASSWORD`, the auth constants contain a default password and JWT secret, and `src/Web` contains a checked-in data-protection key whose XML warns the master key is unencrypted. (docker-compose.yml:22-24, src/ApplicationCore/Constants/AuthorizationConstants.cs:7-11, src/Web/key-768c1632-cf7b-41a9-bb7a-bff228ae8fba.xml:10-13)
- Local vs prod differences:
- In `Development` and `Docker`, `Web` calls `Infrastructure.Dependencies.ConfigureServices`, which chooses in-memory or connection-string-backed DBs from config; in non-development Web startup, the app instead loads Key Vault and configures SQL Server with retry-on-failure. (src/Web/Program.cs:25-43, src/Infrastructure/Dependencies.cs:11-38)
- `PublicApi` always calls `Infrastructure.Dependencies.ConfigureServices`, so its local behavior is config-driven rather than split by a custom prod branch. (src/PublicApi/Program.cs:30-45, src/Infrastructure/Dependencies.cs:11-38)
- Base URLs differ across local and Docker configs, especially for the admin client talking to `PublicApi`. (src/Web/appsettings.json:2-11, src/Web/appsettings.Docker.json:2-9, src/PublicApi/appsettings.json:2-10, src/PublicApi/appsettings.Docker.json:2-9, src/BlazorAdmin/wwwroot/appsettings.json:2-5)

## 8. Observability
- Logging: both server apps add console logging; `Web` and `PublicApi` log startup/seeding events; `ApplicationCore`/`Infrastructure` expose `IAppLogger<T>` through `LoggerAdapter<T>`; the Blazor admin client also logs auth and catalog fetch activity. (src/Web/Program.cs:22-24,118-139,169-201, src/PublicApi/Program.cs:31-32,127-179, src/Infrastructure/Logging/LoggerAdapter.cs:6-22, src/BlazorAdmin/CustomAuthStateProvider.cs:54-61, src/BlazorAdmin/Services/CatalogItemService.cs:61-80)
- Health checks: `Web` registers `/health`, `home_page_health_check`, and `api_health_check`, with custom health checks that probe the storefront and `PublicApi`. (src/Web/Program.cs:85-89,151-168,194-199, src/Web/HealthChecks/HomePageHealthCheck.cs:18-34, src/Web/HealthChecks/ApiHealthCheck.cs:19-33)
- Metrics: Unknown; no metrics libraries/exporters were evident in the inspected startup files. Inspect Azure/App Service monitoring config or any separate observability repo next. (src/Web/Program.cs:22-202, src/PublicApi/Program.cs:26-179, infra/main.bicep:47-144)
- Tracing: Unknown; no tracing setup was evident in the inspected startup files. Inspect future OpenTelemetry/Application Insights wiring if it exists outside this repo. (src/Web/Program.cs:22-202, src/PublicApi/Program.cs:26-179)
- Alerts: Unknown; the checked-in Azure infra provisions App Service, Key Vault, SQL databases, and an App Service plan, but no alerting resources were evident here. Inspect Azure Monitor/App Insights or another infra repository next. (infra/main.bicep:47-144)

## 9. Testing & quality gates
- Types of tests present:
- Unit tests use xUnit and NSubstitute and reference `ApplicationCore` plus `Web`. (tests/UnitTests/UnitTests.csproj:11-26, tests/UnitTests/ApplicationCore/Services/BasketServiceTests/AddItemToBasket.cs:18-45)
- Integration tests use xUnit plus EF Core InMemory and reference `Infrastructure`. (tests/IntegrationTests/IntegrationTests.csproj:8-26, tests/IntegrationTests/Repositories/OrderRepositoryTests/GetById.cs:21-40)
- Functional tests use `WebApplicationFactory`, xUnit, and EF Core InMemory to exercise Web/PublicApi HTTP flows. (tests/FunctionalTests/FunctionalTests.csproj:16-29, tests/FunctionalTests/Web/WebTestFixture.cs:19-55, tests/FunctionalTests/PublicApi/ApiTestFixture.cs:16-42, tests/FunctionalTests/Web/Pages/Basket/CheckoutTest.cs:19-67)
- Public API integration tests use MSTest, `WebApplicationFactory<Program>`, and `coverlet.collector`. (tests/PublicApiIntegrationTests/PublicApiIntegrationTests.csproj:21-35, tests/PublicApiIntegrationTests/ProgramTest.cs:7-25, tests/PublicApiIntegrationTests/AuthEndpoints/CreateCatalogItemEndpointTest.cs:23-52)
- How they run in CI: GitHub Actions builds and tests the solution on every `push`, `pull_request`, and manual dispatch. (.github/workflows/dotnetcore.yml:1-22)
- Coverage gaps / risks:
- Many repository/functional fixtures replace real DB providers with EF Core InMemory, so SQL-specific behavior, transaction boundaries, and retry/migration paths are only lightly represented. (tests/IntegrationTests/IntegrationTests.csproj:8-26, tests/FunctionalTests/Web/WebTestFixture.cs:24-52, tests/FunctionalTests/PublicApi/ApiTestFixture.cs:21-39)
- There is no dedicated `BlazorAdmin` test project under `tests`, so admin correctness appears to rely mostly on API tests and runtime/manual verification. (tests, src/BlazorAdmin)
- Coverage tooling exists, but the checked-in CI workflow does not pass coverage arguments or enforce thresholds. (CodeCoverage.runsettings:1-106, tests/PublicApiIntegrationTests/PublicApiIntegrationTests.csproj:21-29, .github/workflows/dotnetcore.yml:18-22)
- Test stack consistency is mixed: xUnit is used in most projects, while `PublicApiIntegrationTests` uses MSTest. (tests/UnitTests/UnitTests.csproj:11-20, tests/IntegrationTests/IntegrationTests.csproj:8-20, tests/FunctionalTests/FunctionalTests.csproj:16-23, tests/PublicApiIntegrationTests/PublicApiIntegrationTests.csproj:21-29)

## 10. Migration hotspots / tech debt (evidence-based)
- Checkout orchestration is page-driven and split across multiple writes: `CheckoutModel` updates quantities, creates an order with a hard-coded address, and deletes the basket in separate calls, while `OrderService` itself only adds the order. Suggested seam: introduce an `ICheckoutService.PlaceOrder(basketId, shippingAddress, userName)` facade that owns validation, persistence, and transaction boundaries. Test strategy: unit-test orchestration with mocked repositories plus integration tests against a real SQL provider for rollback/consistency. Migration risk: High. (src/Web/Pages/Basket/Checkout.cshtml.cs:44-68, src/ApplicationCore/Services/OrderService.cs:30-52)
- Auth/session management is split across cookie auth in `Web`, JWT auth in `PublicApi`, token minting in `UserController`, browser caching in `CustomAuthStateProvider`, and hard-coded auth secrets/default credentials in source. Suggested seam: centralize this behind an `IAdminSessionService` or `IIdentityGateway` that issues tokens, revokes sessions, and validates configuration at startup. Test strategy: end-to-end auth flows with `WebApplicationFactory`, role-based API tests, and config-binding/startup-validation tests. Migration risk: High. (src/Web/Program.cs:45-58,189-191, src/Web/Configuration/ConfigureCookieSettings.cs:13-39, src/Web/Configuration/RevokeAuthenticationEvents.cs:11-34, src/PublicApi/Program.cs:54-70, src/Web/Controllers/UserController.cs:35-104, src/BlazorAdmin/CustomAuthStateProvider.cs:50-84, src/ApplicationCore/Constants/AuthorizationConstants.cs:7-11)
- Catalog admin writes currently have two paths: legacy Razor Pages in `Web` directly update the repository, while the `/admin` Blazor UI uses `PublicApi` CRUD endpoints. Suggested seam: converge on one `ICatalogAdminService` facade, ideally backed by `PublicApi` or an application-service layer, then retire the duplicate path. Test strategy: contract tests for CRUD DTOs plus smoke tests for whichever UI remains. Migration risk: Medium. (src/Web/Pages/Admin/EditCatalogItem.cshtml.cs:14-37, src/Web/Services/CatalogItemViewModelService.cs:9-27, src/BlazorAdmin/Pages/CatalogItemPage/List.razor:1-67, src/PublicApi/CatalogItemEndpoints/CreateCatalogItemEndpoint.cs:27-75, src/PublicApi/CatalogItemEndpoints/UpdateCatalogItemEndpoint.cs:25-66, src/PublicApi/CatalogItemEndpoints/DeleteCatalogItemEndpoint.cs:18-40)
- Startup initialization is duplicated and fragile: both server apps seed on startup, and `CatalogContextSeed` retries recursively but then rethrows after the recursive call. Suggested seam: move initialization into an `IDatabaseInitializer.InitializeAsync()` component with explicit retry policy and host-level ownership so only one process is responsible. Test strategy: integration tests that simulate transient DB failures and assert idempotent startup. Migration risk: Medium. (src/Web/Program.cs:120-139, src/PublicApi/Program.cs:129-148, src/Infrastructure/Data/CatalogContextSeed.cs:12-56, src/Infrastructure/Identity/AppIdentityDbContextSeed.cs:10-30)
- The admin client/API error contract is weak: `HttpService` often returns `null`, `PUT` failures only show a generic toast, and the paged catalog endpoint injects a one-second delay. Suggested seam: wrap API access behind a typed `ICatalogApiClient` returning a result object with status/error metadata, then remove artificial delay behind a feature flag or test-only path. Test strategy: API contract tests for non-200 responses and UI tests for error rendering. Migration risk: Medium. (src/BlazorAdmin/Services/HttpService.cs:25-95, src/PublicApi/CatalogItemEndpoints/CatalogItemListPagedEndpoint.cs:40-71, src/PublicApi/Middleware/ExceptionMiddleware.cs:31-53)
- Build-graph boundaries are already blurred: `ApplicationCore` references `BlazorShared`, and `Web` directly references the `BlazorAdmin` client project to host its assets. Suggested seam: extract an explicit contracts package such as `Eshop.Contracts` and keep UI hosts depending outward on that package instead of inward on each other. Test strategy: compile-time dependency checks plus smoke tests for hosted `/admin` assets during the split. Migration risk: Medium. (src/ApplicationCore/ApplicationCore.csproj:16-18, src/Web/Web.csproj:42-47, src/BlazorAdmin/BlazorAdmin.csproj:14-16)

## 11. Open questions
- How is `PublicApi` hosted in Azure/production? The local README says admin needs it, but `azure.yaml` only declares one `web` service. Inspect deployment pipeline/app-service configuration next. (README.md:95-96, azure.yaml:3-8, infra/main.bicep:47-65)
- Are metrics, distributed tracing, and alerts configured outside this repo? No evidence was found in the inspected startup files or Bicep templates. Inspect Azure Monitor/Application Insights resources or another infra repo next. (src/Web/Program.cs:22-202, src/PublicApi/Program.cs:26-179, infra/main.bicep:47-144)
- Is the default local DB split intentional? `Web` defaults to in-memory storage, while `PublicApi` defaults to connection-string-backed DBs. Inspect team onboarding docs or environment overrides next. (src/Web/appsettings.json:6-10, src/PublicApi/appsettings.json:6-10, src/Infrastructure/Dependencies.cs:11-38)
- Is the `Cancel` order link intentional? `MyOrders.cshtml` renders a cancel action link, but no matching `OrderController.Cancel` action was found in the inspected controller file. Inspect git history or backlog items next. (src/Web/Views/Order/MyOrders.cshtml:28-32, src/Web/Controllers/OrderController.cs:10-43)
- Was the checked-in data-protection key intentional? The XML says the master key is unencrypted. Inspect deployment guidance, git history, and secret-scanning policy next. (src/Web/key-768c1632-cf7b-41a9-bb7a-bff228ae8fba.xml:10-13)

## 12. First 90 minutes checklist
- Read the root README for project intent and local run modes, then skim this generated orientation for the current code map. (README.md:8-27,47-165)
- Open the three startup files first: `src/Web/Program.cs`, `src/PublicApi/Program.cs`, and `src/BlazorAdmin/Program.cs`. They define the actual runtime entrypoints and most cross-project wiring. (src/Web/Program.cs:22-202, src/PublicApi/Program.cs:26-181, src/BlazorAdmin/Program.cs:16-40)
- Trace one domain flow through `ApplicationCore` and `Infrastructure`: `BasketService`, `OrderService`, `EfRepository<T>`, `CatalogContext`, and the relevant specifications are the fastest way to understand business logic vs persistence. (src/ApplicationCore/Services/BasketService.cs:11-85, src/ApplicationCore/Services/OrderService.cs:12-52, src/Infrastructure/Data/EfRepository.cs:6-10, src/Infrastructure/Data/CatalogContext.cs:9-26, src/ApplicationCore/Specifications)
- Decide whether you need storefront-only work or admin/API work; if you need admin, plan to run both `Web` and `PublicApi` and verify `/admin` plus `/swagger`. (README.md:95-96, src/Web/Properties/launchSettings.json:20-37, src/PublicApi/Properties/launchSettings.json:11-19, src/BlazorAdmin/Pages/CatalogItemPage/List.razor:1-2)
- Decide whether to stay on the default in-memory store or switch to SQL/LocalDB; if you switch, restore the EF tool and run the two DB migration commands from the README. (src/Web/appsettings.json:6-10, src/Web/.config/dotnet-tools.json:1-12, README.md:115-144)
- Run the solution test command the CI workflow uses so your local branch matches the repository quality gate. (.github/workflows/dotnetcore.yml:18-22)
- If your task touches checkout, auth, or admin catalog management, read the hotspot files before coding because those areas already have duplicated paths or fragile boundaries. (src/Web/Pages/Basket/Checkout.cshtml.cs:44-68, src/Web/Controllers/UserController.cs:35-104, src/BlazorAdmin/Pages/CatalogItemPage/List.razor:1-67, src/PublicApi/CatalogItemEndpoints)
- If your task touches deployment or secrets, inspect `azure.yaml`, `infra/main.bicep`, `docker-compose.yml`, and the auth/key files before making assumptions about environment parity or secret handling. (azure.yaml:3-8, infra/main.bicep:47-144, docker-compose.yml:18-24, src/ApplicationCore/Constants/AuthorizationConstants.cs:7-11, src/Web/key-768c1632-cf7b-41a9-bb7a-bff228ae8fba.xml:10-13)
