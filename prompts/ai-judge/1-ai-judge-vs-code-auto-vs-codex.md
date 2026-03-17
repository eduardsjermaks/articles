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
- Purpose: A sample ASP.NET Core reference app for a single-process (monolithic) e-commerce-style web application, with companion API and admin UI. (README.md)
- Who uses it / use cases: Appears targeted at engineers learning/teaching modern ASP.NET Core architecture patterns and Azure-hosted deployment patterns, not as a production-complete e-commerce product. (README.md)

## 2. Quick start (developer)
- Prereqs: .NET SDK 8 (`global.json`), SQL Server/LocalDB or in-memory DB mode, and optional Docker / Azure Developer CLI (`README.md`, `global.json`).
- How to run locally: Run `src/PublicApi` and `src/Web` together; README states `dotnet run` in `PublicApi` and `dotnet run --launch-profile Web` in `Web`, then browse `https://localhost:5001/`. (README.md)
- How to run tests: CI runs `dotnet build ./eShopOnWeb.sln --configuration Release` and `dotnet test ./eShopOnWeb.sln --configuration Release`; same commands appear to be the baseline quality gate. (`.github/workflows/dotnetcore.yml`)
- Common pitfalls:
  - Admin UI depends on PublicApi; running only `Web` leaves admin functionality incomplete. (README.md)
  - EF migrations are required for persistent SQL usage; README gives `dotnet ef database update` commands. (README.md)
  - Docker config includes explicit SQL SA password strings; treat as sample defaults only. (`docker-compose.yml`, `src/Web/appsettings.Docker.json`, `src/PublicApi/appsettings.Docker.json`)

## 3. Architecture at a glance
- Runtime processes:
  - `Web` ASP.NET Core app (`src/Web/Program.cs`).
  - `PublicApi` ASP.NET Core API app (`src/PublicApi/Program.cs`).
  - `BlazorAdmin` WebAssembly client (`src/BlazorAdmin/Program.cs`), also referenced by `Web` (`src/Web/Web.csproj`).
  - Optional Docker SQL Edge container (`docker-compose.yml`).
- Data stores / queues:
  - Catalog and Identity relational DB contexts via EF Core, configured to SQL Server or in-memory depending on `UseOnlyInMemoryDatabase`. (`src/Infrastructure/Dependencies.cs`, `src/Web/appsettings.json`)
  - Azure SQL + Key Vault path appears used in non-development `Web` startup. (`src/Web/Program.cs`, `infra/main.bicep`)
  - Message brokers/queues: none found in source search for common broker frameworks. (workspace search result)
- ASCII view (based on startup/composition files):

  ```text
  Browser -> Web (Razor/MVC + Blazor host) -> ApplicationCore services -> Infrastructure (EF) -> Catalog/Identity DB
                |                                   ^
                +-> BlazorAdmin (WASM) -> PublicApi-+
  ```
  (src/Web/Program.cs, src/PublicApi/Program.cs, src/Web/Configuration/ConfigureCoreServices.cs, src/Infrastructure/Data/EfRepository.cs)

## 4. Repository map
- Top-level tree (workspace root): `.devcontainer/`, `.github/`, `infra/`, `src/`, `tests/`, solution files, Docker compose files, Azure template file. (workspace root listing)
- `src/` modules: `ApplicationCore`, `Infrastructure`, `Web`, `PublicApi`, `BlazorAdmin`, `BlazorShared`. (`src` listing, `Everything.sln`)
- `tests/` modules: `UnitTests`, `IntegrationTests`, `FunctionalTests`, `PublicApiIntegrationTests`. (`tests` listing, `Everything.sln`)

- Major module: `src/ApplicationCore`
  - Responsibility: Domain entities, interfaces, and domain services (basket/order). (`src/ApplicationCore/Entities`, `src/ApplicationCore/Interfaces`, `src/ApplicationCore/Services`)
  - Main public interfaces: `IRepository<>`, `IReadRepository<>`, `IBasketService`, `IOrderService`, `IUriComposer`, etc. (`src/ApplicationCore/Interfaces`)
  - Key files: `Services/BasketService.cs`, `Services/OrderService.cs`, `CatalogSettings.cs`. (`src/ApplicationCore/Services/BasketService.cs`, `src/ApplicationCore/Services/OrderService.cs`, `src/ApplicationCore/CatalogSettings.cs`)

- Major module: `src/Infrastructure`
  - Responsibility: EF persistence, identity DB, DI database wiring, logging adapter. (`src/Infrastructure/Dependencies.cs`, `src/Infrastructure/Data`, `src/Infrastructure/Logging/LoggerAdapter.cs`)
  - Main public interfaces: concrete `EfRepository<T>` implementing app core repository interfaces. (`src/Infrastructure/Data/EfRepository.cs`)
  - Key files: `Dependencies.cs`, `Data/CatalogContext.cs`, `Data/EfRepository.cs`. (`src/Infrastructure/Dependencies.cs`, `src/Infrastructure/Data/EfRepository.cs`)

- Major module: `src/Web`
  - Responsibility: Main web host (Razor Pages/MVC), auth cookie/identity setup, service composition, health endpoints, DB seeding. (`src/Web/Program.cs`)
  - Main public interfaces: web-facing service abstractions in `Web.Interfaces` plus DI extension methods. (`src/Web/Configuration/ConfigureCoreServices.cs`, `src/Web/Configuration/ConfigureWebServices.cs`)
  - Key files: `Program.cs`, `Pages/Index.cshtml.cs`, `Pages/Basket/Index.cshtml.cs`. (`src/Web/Program.cs`, `src/Web/Pages/Index.cshtml.cs`, `src/Web/Pages/Basket/Index.cshtml.cs`)

- Major module: `src/PublicApi`
  - Responsibility: HTTP API endpoints (minimal endpoint classes + API endpoints package), JWT auth, Swagger, exception middleware, DB seeding. (`src/PublicApi/Program.cs`, `src/PublicApi/README.md`)
  - Main public interfaces: endpoint classes implementing `IEndpoint<...>` and Ardalis endpoint types. (`src/PublicApi/CatalogItemEndpoints/CreateCatalogItemEndpoint.cs`, `src/PublicApi/PublicApi.csproj`)
  - Key files: `Program.cs`, `Middleware/ExceptionMiddleware.cs`, endpoint folders under `CatalogItemEndpoints`, `CatalogTypeEndpoints`, `CatalogBrandEndpoints`, `AuthEndpoints`. (`src/PublicApi/Program.cs`, `src/PublicApi/Middleware/ExceptionMiddleware.cs`)

- Major module: `infra/`
  - Responsibility: Bicep-based Azure provisioning for App Service, App Service Plan, Azure SQL, Key Vault, secret and app setting wiring. (`infra/main.bicep`, `infra/core/host/appservice.bicep`, `infra/core/database/sqlserver/sqlserver.bicep`, `infra/core/security/keyvault.bicep`)
  - Main public interfaces: Bicep module parameters/outputs for connection string keys and Key Vault endpoint. (`infra/main.bicep`)
  - Key files: `main.bicep`, `main.parameters.json`. (`infra/main.bicep`, `infra/main.parameters.json`)

## 5. Critical flows
- Flow: Catalog page request
  - Entrypoint: `OnGet` in home page model. (`src/Web/Pages/Index.cshtml.cs`)
  - Steps: Page model calls `ICatalogViewModelService.GetCatalogItems(...)`, then renders model. (`src/Web/Pages/Index.cshtml.cs`)
  - Side effects: Read-only catalog fetch and view model mapping (implementation appears in web services registrations and catalog services). (`src/Web/Configuration/ConfigureWebServices.cs`)
  - Failure modes: Unknown (no explicit exception handling in this page model). Inspect next: `src/Web/Services/CatalogViewModelService.cs`, `src/Web/Services/CachedCatalogViewModelService.cs`.

- Flow: Add item to basket
  - Entrypoint: Basket page `OnPost`. (`src/Web/Pages/Basket/Index.cshtml.cs`)
  - Steps: Validate product id -> load `CatalogItem` -> resolve cookie/user basket id -> `IBasketService.AddItemToBasket` -> map result to `BasketViewModel`. (`src/Web/Pages/Basket/Index.cshtml.cs`, `src/ApplicationCore/Services/BasketService.cs`)
  - Side effects: Creates basket if needed, updates basket items, persists through repository update/add operations, sets long-lived basket cookie for anonymous user. (`src/ApplicationCore/Services/BasketService.cs`, `src/Web/Pages/Basket/Index.cshtml.cs`)
  - Failure modes: Redirect to home if invalid/missing item; `SetQuantities` can return not found; DB/repository exceptions bubble unless handled upstream. (`src/Web/Pages/Basket/Index.cshtml.cs`, `src/ApplicationCore/Services/BasketService.cs`)

- Flow: Create catalog item via Public API
  - Entrypoint: `POST api/catalog-items`. (`src/PublicApi/CatalogItemEndpoints/CreateCatalogItemEndpoint.cs`)
  - Steps: Role-restricted endpoint -> duplicate-name check -> create entity -> save -> set default image -> return `201 Created`. (`src/PublicApi/CatalogItemEndpoints/CreateCatalogItemEndpoint.cs`)
  - Side effects: Writes catalog row; updates picture URI; emits created response. (`src/PublicApi/CatalogItemEndpoints/CreateCatalogItemEndpoint.cs`)
  - Failure modes: `DuplicateException` translated to HTTP 409 by exception middleware; other exceptions become HTTP 500. (`src/PublicApi/CatalogItemEndpoints/CreateCatalogItemEndpoint.cs`, `src/PublicApi/Middleware/ExceptionMiddleware.cs`)

- Flow: Startup seeding
  - Entrypoint: app startup after build in both `Web` and `PublicApi`. (`src/Web/Program.cs`, `src/PublicApi/Program.cs`)
  - Steps: Create scope -> resolve contexts/user managers -> call `CatalogContextSeed.SeedAsync` and `AppIdentityDbContextSeed.SeedAsync`. (`src/Web/Program.cs`, `src/PublicApi/Program.cs`)
  - Side effects: Inserts seed data and default users/roles if needed. (`src/Web/Program.cs`, `src/PublicApi/Program.cs`)
  - Failure modes: Exceptions are logged; startup continues after logging. (`src/Web/Program.cs`, `src/PublicApi/Program.cs`)

## 6. Dependencies
- Internal modules:
  - `Web` references `ApplicationCore`, `Infrastructure`, `BlazorAdmin`, `BlazorShared`. (`src/Web/Web.csproj`)
  - `PublicApi` references `ApplicationCore` and `Infrastructure`. (`src/PublicApi/PublicApi.csproj`)
  - `Infrastructure` references `ApplicationCore`. (`src/Infrastructure/Infrastructure.csproj`)
- External services:
  - SQL Server / LocalDB and EF Core in local modes. (`src/Infrastructure/Dependencies.cs`, `src/Web/appsettings.json`, `src/PublicApi/appsettings.json`)
  - Azure SQL, Azure Key Vault, Azure App Service in infra/prod path. (`infra/main.bicep`, `infra/core/database/sqlserver/sqlserver.bicep`, `infra/core/security/keyvault.bicep`, `src/Web/Program.cs`)
  - Docker SQL Edge service for containerized local setup. (`docker-compose.yml`)
- Build / deploy tools:
  - .NET SDK 8.x (`global.json`).
  - GitHub Actions build+test workflow (`.github/workflows/dotnetcore.yml`).
  - Azure Developer CLI project descriptor (`azure.yaml`).

## 7. Configuration & environments
- Config files:
  - App settings per app and environment variants (`appsettings.json`, `.Development`, `.Docker`) for both `Web` and `PublicApi`. (`src/Web/appsettings.json`, `src/Web/appsettings.Development.json`, `src/Web/appsettings.Docker.json`, `src/PublicApi/appsettings.json`, `src/PublicApi/appsettings.Development.json`, `src/PublicApi/appsettings.Docker.json`)
- Env vars:
  - Docker: `ASPNETCORE_ENVIRONMENT=Docker`, `ASPNETCORE_URLS=http://+:8080`. (`docker-compose.override.yml`)
  - Azure/prod path expects `AZURE_KEY_VAULT_ENDPOINT`, `AZURE_SQL_CATALOG_CONNECTION_STRING_KEY`, `AZURE_SQL_IDENTITY_CONNECTION_STRING_KEY`. (`src/Web/Program.cs`, `infra/main.bicep`)
- Secrets:
  - User secrets configured in `Web` and `PublicApi` projects (`UserSecretsId`). (`src/Web/Web.csproj`, `src/PublicApi/PublicApi.csproj`)
  - Bicep stores SQL/admin/app credentials in Key Vault and outputs key names. (`infra/core/database/sqlserver/sqlserver.bicep`, `infra/main.parameters.json`)
  - Sample code contains explicit default auth/password constants marked TODO-not-for-production. (`src/ApplicationCore/Constants/AuthorizationConstants.cs`)
  - Docker and Docker appsettings include explicit SA password string (sample risk). (`docker-compose.yml`, `src/Web/appsettings.Docker.json`, `src/PublicApi/appsettings.Docker.json`)
- Local vs prod differences:
  - Dev/Docker uses `Infrastructure.Dependencies.ConfigureServices` (in-memory or configured SQL connection strings). (`src/Web/Program.cs`, `src/Infrastructure/Dependencies.cs`)
  - Non-development `Web` path loads Key Vault and SQL connection string keys via Azure identity credential chain. (`src/Web/Program.cs`)

## 8. Observability
- Logging:
  - Console logging enabled in `Web` and `PublicApi` startup. (`src/Web/Program.cs`, `src/PublicApi/Program.cs`)
  - App-level logger abstraction (`IAppLogger` -> `LoggerAdapter`). (`src/ApplicationCore/Interfaces/IAppLogger.cs`, `src/Infrastructure/Logging/LoggerAdapter.cs`)
  - Per-environment log levels configured in appsettings. (`src/Web/appsettings*.json`, `src/PublicApi/appsettings*.json`)
- Metrics:
  - Unknown.
  - Inspect next: `src/Web/Program.cs`, `src/PublicApi/Program.cs`, `infra/core/host/appservice.bicep`, and any `OpenTelemetry`/`ApplicationInsights` setup files if added later.
- Tracing:
  - Unknown.
  - Inspect next: `src/Web/Program.cs`, `src/PublicApi/Program.cs`, and project/package files for tracing SDK packages.
- Alerts:
  - Unknown.
  - Inspect next: `infra/` for monitor/alert modules and deployment pipelines under `.github/workflows`.
- Health checks:
  - `Web` registers health checks and maps `/health`, `home_page_health_check`, and `api_health_check`. (`src/Web/Program.cs`)
  - Health probes validate expected content from homepage/API responses. (`src/Web/HealthChecks/HomePageHealthCheck.cs`, `src/Web/HealthChecks/ApiHealthCheck.cs`)

## 9. Testing & quality gates
- Test types present:
  - Unit tests (`tests/UnitTests`) with xUnit and NSubstitute. (`tests/UnitTests/UnitTests.csproj`)
  - Integration tests (`tests/IntegrationTests`) using EF in-memory context patterns. (`tests/IntegrationTests/IntegrationTests.csproj`, `tests/IntegrationTests/Repositories/OrderRepositoryTests/GetByIdWithItemsAsync.cs`)
  - Functional tests (`tests/FunctionalTests`) using `Microsoft.AspNetCore.Mvc.Testing`. (`tests/FunctionalTests/FunctionalTests.csproj`, `tests/FunctionalTests/Web/Pages/HomePageOnGet.cs`)
  - Public API integration tests (`tests/PublicApiIntegrationTests`) using MSTest + `WebApplicationFactory`. (`tests/PublicApiIntegrationTests/PublicApiIntegrationTests.csproj`, `tests/PublicApiIntegrationTests/ProgramTest.cs`)
- CI execution:
  - GitHub Actions workflow runs build and full solution tests on push/PR/workflow_dispatch. (`.github/workflows/dotnetcore.yml`)
- Gaps / risks:
  - No explicit performance/load/security test pipeline found in workspace files reviewed. Unknown. Inspect next: `.github/workflows/`, any docs under `docs/` (if added later), and test directories for non-unit suites.

## 10. Migration hotspots / tech debt (evidence-based)
- Hotspot: Large startup composition roots (`Web` and `PublicApi` `Program.cs`)
  - why risky: Many responsibilities (auth, DB wiring, seeding, middleware, health checks, Swagger) concentrated in single files can increase change coupling. (`src/Web/Program.cs`, `src/PublicApi/Program.cs`)
  - evidence: Extensive inline service wiring + middleware + seeding logic in startup files. (`src/Web/Program.cs`, `src/PublicApi/Program.cs`)
  - suggested seam: Extract startup responsibilities into focused extension classes per concern (auth, persistence, observability, seeding).
  - suggested facade: `IHostBootstrapper`-style composition wrapper or grouped extension modules under `Configuration/`.
  - migration risk: Medium

- Hotspot: Embedded sample secrets / security TODO constants
  - why risky: Hard-coded credentials/secrets and TODO comments can leak into non-sample deployments. (`src/ApplicationCore/Constants/AuthorizationConstants.cs`, `docker-compose.yml`)
  - evidence: `JWT_SECRET_KEY`, `DEFAULT_PASSWORD`, and SQL SA password literals are present. (`src/ApplicationCore/Constants/AuthorizationConstants.cs`, `docker-compose.yml`, `src/Web/appsettings.Docker.json`, `src/PublicApi/appsettings.Docker.json`)
  - suggested seam: Centralize secret providers via environment + Key Vault abstractions.
  - suggested facade: `ISecretProvider` / options-binding facade with validation at startup.
  - migration risk: High

- Hotspot: Over-permissive SQL firewall rule in IaC
  - why risky: Wide IP range allows broad direct access and can violate stricter enterprise posture. (`infra/core/database/sqlserver/sqlserver.bicep`)
  - evidence: Firewall rule from `0.0.0.1` to `255.255.255.254`. (`infra/core/database/sqlserver/sqlserver.bicep`)
  - suggested seam: Separate local-dev connectivity from production network policy.
  - suggested facade: Network policy module/parameter set (private endpoints, restricted IP lists).
  - migration risk: High

- Hotspot: Potentially large/complex controller
  - why risky: `ManageController` shows many logging points and likely broad account-management scope; high edit conflict and regression potential.
  - evidence: Multiple user-account operations logged across distant line locations. (`src/Web/Controllers/ManageController.cs`)
  - suggested seam: Split by user profile/password/2FA concerns.
  - suggested facade: `IAccountManagementService` with dedicated sub-services.
  - migration risk: Medium

- Hotspot: Cyclic dependencies
  - why risky: Unknown.
  - evidence: Unknown.
  - suggested seam: Run dependency graph analysis before major modularization.
  - suggested facade: Strict layer references (`ApplicationCore` -> none, `Infrastructure` -> `ApplicationCore`, hosts -> both).
  - migration risk: Unknown

## 11. Open questions
- Is there a formal production architecture document beyond sample README narrative? Unknown.
  - Inspect next: `README.md`, `.specify/`, or any architecture ADR/doc folders if added.
- Which production authentication/identity provider model is intended (beyond default Identity/JWT sample constants)? Unknown.
  - Inspect next: `src/Web/Controllers/UserController.cs`, `src/PublicApi/AuthEndpoints/*`, deployment/environment docs.
- Are observability metrics/traces/alerts intentionally omitted or configured externally (Azure portal/manual)? Unknown.
  - Inspect next: `infra/` monitor-related modules, `.github/workflows`, and operations docs.
- Is `BlazorAdmin` expected to be deployed independently or only hosted through `Web` static assets/pathing? Unknown.
  - Inspect next: `src/Web/Program.cs`, `src/BlazorAdmin/Program.cs`, deployment manifests.
- Are there mandated code quality thresholds (coverage %, static analysis rules) in CI? Unknown.
  - Inspect next: `.github/workflows/*`, `CodeCoverage.runsettings`, repo policy docs.

## 12. First 90 minutes checklist
- Open `README.md` and confirm local run mode (multi-process vs Docker) and migration setup commands. (`README.md`)
- Build and test baseline: `dotnet build ./eShopOnWeb.sln --configuration Release` and `dotnet test ./eShopOnWeb.sln --configuration Release`. (`.github/workflows/dotnetcore.yml`)
- Trace startup paths in order: `src/Web/Program.cs`, `src/PublicApi/Program.cs`, `src/BlazorAdmin/Program.cs`.
- Review DI/service boundaries: `src/Web/Configuration/ConfigureCoreServices.cs`, `src/Web/Configuration/ConfigureWebServices.cs`, `src/Infrastructure/Dependencies.cs`.
- Walk one user flow end-to-end: basket add/update (`src/Web/Pages/Basket/Index.cshtml.cs` -> `src/ApplicationCore/Services/BasketService.cs` -> `src/Infrastructure/Data/EfRepository.cs`).
- Walk one API flow end-to-end: create catalog item (`src/PublicApi/CatalogItemEndpoints/CreateCatalogItemEndpoint.cs` -> `src/PublicApi/Middleware/ExceptionMiddleware.cs`).
- Review environment configs and secrets handling (`src/Web/appsettings*.json`, `src/PublicApi/appsettings*.json`, `docker-compose*.yml`, `src/ApplicationCore/Constants/AuthorizationConstants.cs`).
- Review deployment IaC for cloud assumptions and security posture (`infra/main.bicep`, `infra/core/database/sqlserver/sqlserver.bicep`, `infra/core/security/keyvault.bicep`).
- Identify your first safe seam before feature work (recommended: extract startup concern modules from `Program.cs` files). (`src/Web/Program.cs`, `src/PublicApi/Program.cs`)