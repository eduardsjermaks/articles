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
