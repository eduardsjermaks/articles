# Project Orientation (Generated)

> Evidence-based first-pass orientation. Claims are tagged **FACT** (directly observable), **INFERENCE** (deduced from structure), **UNKNOWN** (no evidence found). Structural claims with fewer than 2 independent references are marked **Tentative**. Generated 2026-07-06 against commit `42f2a9ac53`.

---

## 1. What this project is

- **Purpose** — **FACT**: This is **nopCommerce**, an open-source ASP.NET Core eCommerce platform (shopping cart). The web entrypoint assembly is `Nop.Web` (`src/Presentation/Nop.Web/Program.cs:8-51`), and the product self-describes as "free and open-source eCommerce solution" (`README.md:1-4`).
- **Runtime target** — **FACT**: Targets **.NET 10 SDK** (`global.json:2-6`, `Dockerfile:2`, `.github/workflows/dotnet.yml:22` → `dotnet-version: 10.0.x`).
  - ⚠️ **Discrepancy (FACT)**: The human-authored `README.md:11` still states "nopCommerce runs on .NET 9". The build/CI config says .NET 10. Treat the README prose as stale; trust `global.json`/`Dockerfile`/CI.
- **Who uses it / main use-cases** — **INFERENCE**: Store operators (admin area under `src/Presentation/Nop.Web/Areas/Admin`) and shoppers (public controllers under `src/Presentation/Nop.Web/Controllers`). Domains present in code include Catalog, Orders, Customers, Payments, Shipping, Tax, Discounts (`src/Libraries/Nop.Services/*`). Marketplace/plugin extensibility is a first-class concern (see §3, §4).

---

## 2. Quick start (developer)

- **Prereqs** — **FACT**:
  - .NET 10 SDK `10.0.100` (`global.json:2-6`, `rollForward: latestFeature`).
  - A supported database: **MS SQL Server**, **PostgreSQL**, or **MySQL** (`src/Libraries/Nop.Data/DataProviders/` contains `MsSqlDataProvider.cs`, `PostgreSqlDataProvider.cs`, `MySqlDataProvider.cs`).
  - Node/npm for front-end asset bundling — **INFERENCE**: `src/Presentation/Nop.Web/package.json` + `gulpfile.js` present.
- **How to run locally**:
  - **FACT (Docker)**: `docker-compose.yml` builds the app and starts an MS SQL 2019 container (`docker-compose.yml:1-18`); app is exposed on port 80. Alternate compose files exist for MySQL (`mysql-docker-compose.yml`) and PostgreSQL (`postgresql-docker-compose.yml`).
  - **INFERENCE (dotnet)**: Run `dotnet run` in `src/Presentation/Nop.Web`. On first launch the app is **not yet installed** — a web installer collects the DB connection string and writes `~/App_Data/dataSettings.json` (`src/Libraries/Nop.Data/NopDataSettingsDefaults.cs:16`; installer code under `src/Presentation/Nop.Web/Infrastructure/Installation` and `src/Libraries/Nop.Services/Installation`). **Tentative** on the exact install UX — inspect `src/Presentation/Nop.Web/Controllers/InstallController*` next.
- **How to run tests** — **FACT**: `dotnet test src` (this is exactly what CI runs, `.github/workflows/dotnet.yml:30`). Single test project: `src/Tests/Nop.Tests` (NUnit + Moq, see §9). Tests run against **in-memory SQLite** (`src/Tests/Nop.Tests/SqLiteNopDataProvider.cs`), so **no external DB is needed for the test suite** — **INFERENCE** from the provider's presence and `TestDataProviderManager.cs`.
- **Common pitfalls**:
  - **FACT**: There is **no committed `appsettings.json`**; a search of `src/Presentation/Nop.Web` for `appsettings*.json` returns nothing. `Program.cs:14` loads it as *optional* (`AddJsonFile(..., optional: true, ...)`). App config largely comes from `App_Data/appSettings.json` created at install time — **INFERENCE** (see §7). First run without completing installation will redirect to the installer.
  - **INFERENCE**: The DI container is switchable between Autofac and the built-in provider via `CommonConfig.UseAutofac` (`Program.cs:26-39`); a wrong assumption about which container is active can bite when debugging registration.

---

## 3. Architecture at a glance

**FACT** — This is a **modular monolith**: a single ASP.NET Core web process (`Nop.Web`) with a layered library structure and a plugin system. There is no evidence of separate microservices or a message broker (see §6).

```
                         ┌──────────────────────────────────────────┐
   Browser / API  ─────► │  Nop.Web  (ASP.NET Core, single process)  │
   client               │   Program.cs → WebApplication              │
                         │   ┌─────────────────────────────────────┐ │
                         │   │ Controllers / Areas/Admin / Views   │ │  Presentation
                         │   │ Factories (view-model assembly)     │ │
                         │   └─────────────────────────────────────┘ │
                         │   ┌─────────────────────────────────────┐ │
                         │   │ Nop.Web.Framework (MVC infra, DI,   │ │
                         │   │ WorkContext, StoreContext, auth)    │ │
                         │   └─────────────────────────────────────┘ │
                         │   ┌─────────────────────────────────────┐ │
                         │   │ Nop.Services  (business logic:      │ │  Domain / services
                         │   │ Orders, Catalog, Customers, Payments│ │
                         │   │ Shipping, Tax, Discounts, ...)      │ │
                         │   └─────────────────────────────────────┘ │
                         │   ┌──────────────────┐ ┌────────────────┐ │
                         │   │ Nop.Core (domain │ │ Nop.Data       │ │  Core + data
                         │   │ entities, engine)│ │ (LinqToDB +    │ │
                         │   │                  │ │ FluentMigrator)│ │
                         │   └──────────────────┘ └────────────────┘ │
                         │   ┌─────────────────────────────────────┐ │
                         │   │ Plugins/* loaded at startup via     │ │  Extensibility
                         │   │ INopStartup + ITypeFinder discovery │ │
                         │   └─────────────────────────────────────┘ │
                         │   In-process TaskScheduler (TaskThread)   │  Background work
                         └────────────────────┬──────────────────────┘
                                              │
                            ┌─────────────────┴───────────────────┐
                            │  Relational DB (MS SQL / PG / MySQL) │
                            └──────────────────────────────────────┘
              Optional: Redis / SQL-Server distributed cache (DistributedCacheType)
```

- **Runtime processes** — **FACT**: A **single** web process (`Program.cs:44-50`, `ENTRYPOINT ["dotnet", "Nop.Web.dll"]` in `Dockerfile`). Scheduled/background tasks run **inside** that process via an in-memory `TaskScheduler` that spins up `TaskThread` instances (`src/Libraries/Nop.Services/ScheduleTasks/TaskScheduler.cs:16-49`). **INFERENCE**: there is **no separate worker/scheduler process** — background jobs share the web app's lifetime and container.
- **Data stores** — **FACT**: One relational DB, provider chosen at install (`DataProviderType` in `src/Libraries/Nop.Data/DataProviderType.cs`; providers in `DataProviders/`).
- **Queues** — **UNKNOWN → likely none**: No message-broker dependency found (no RabbitMQ/Kafka/Azure Service Bus package references surfaced in §6 search). Email is queued **in the database** (`QueuedEmail` model + `IQueuedEmailModelFactory`, admin factory in `NopStartup.cs:69`) and drained by a scheduled task — **INFERENCE**. Inspect `src/Libraries/Nop.Services/Messages` to confirm.

---

## 4. Repository map

**FACT** — Top-level (`/` and `/src`):

| Path | Responsibility (evidence) |
|---|---|
| `src/NopCommerce.sln` | Solution aggregating all projects (`src/`). |
| `src/Libraries/Nop.Core` | Domain entities, the composition **engine**, infrastructure. Key: `Infrastructure/NopEngine.cs`, `Infrastructure/EngineContext.cs`, `Infrastructure/ITypeFinder.cs`, `BaseEntity.cs`, `IWorkContext.cs`, `IStoreContext.cs`, `Domain/*`. |
| `src/Libraries/Nop.Data` | Persistence. **LinqToDB**-based repositories + **FluentMigrator** migrations. Key: `IRepository.cs`, `EntityRepository.cs`, `DataProviders/*`, `Migrations/` (45 migration files), `DataSettingsManager.cs`. |
| `src/Libraries/Nop.Services` | Business logic. **92 `I*Service` interfaces** across ~40 domain folders (Orders, Catalog, Customers, Payments, Shipping, Tax, Discounts, Messages, Media, Security, Gdpr, ScheduleTasks, ...). |
| `src/Presentation/Nop.Web.Framework` | MVC/web infra shared by app + plugins: DI wiring (`Infrastructure/NopStartup.cs`, 336 lines), request pipeline (`Infrastructure/Extensions/ApplicationBuilderExtensions.cs`, `ServiceCollectionExtensions.cs`), `WebWorkContext`, `WebStoreContext`, auth. |
| `src/Presentation/Nop.Web` | The **runnable app**. Public `Controllers/`, `Areas/Admin/` (admin), `Factories/` (view-model builders), `Views/`, `Themes/`, `Program.cs`, `Infrastructure/NopStartup.cs`. |
| `src/Plugins` | **32 plugin projects** (payments, shipping, tax, widgets, external auth, misc). Each ships an `INopStartup` implementation (e.g. `Nop.Plugin.Payments.PayPalCommerce/Infrastructure/NopStartup.cs`). |
| `src/Tests/Nop.Tests` | Single test project (NUnit). Runs against SQLite. |
| `src/Build`, `deploy.cmd`, `.deployment` | Build/deploy tooling. **Tentative** — not deeply inspected. |
| `upgradescripts/` | SQL upgrade scripts between versions. **INFERENCE** from name. |
| `Dockerfile`, `*docker-compose.yml`, `.dockerignore` | Container build + local orchestration per DB flavor. |
| `.github/workflows/dotnet.yml` | CI (build + test on `windows-latest`). |

**Major-module public interfaces / composition mechanism** — **FACT**:
- **Engine + service location**: `EngineContext.Current` exposes `Resolve<T>()` used pervasively at startup (`ApplicationBuilderExtensions.cs:54,65`; `NopEngine.cs`). This is a **service-locator** pattern layered over the DI container.
- **Type discovery**: `ITypeFinder`/`WebAppTypeFinder` scan assemblies to auto-find `INopStartup`, `IStartupTask`, `IConfig`, `IOrderedMapperProfile`, `IExternalAuthenticationRegistrar` (`NopEngine.cs:39-68`, `ServiceCollectionExtensions.cs:56-75,288-289`). This is how **plugins self-register** without central edits.
- **Service registration is largely explicit**, not convention-based: `Nop.Web.Framework/Infrastructure/NopStartup.cs` manually `AddScoped`s the bulk of services (lines ~77-336), and `Nop.Web/Infrastructure/NopStartup.cs` registers all admin/public **factories** (~90 registrations, `NopStartup.cs:22-115`). **INFERENCE**: adding a service means editing these startup files (or shipping a plugin `INopStartup`).

---

## 5. Critical flows

### Flow A — Place an order (checkout → persistence → side-effects)
- **Trigger/entrypoint** — **FACT**: `OrderProcessingService.PlaceOrderAsync(ProcessPaymentRequest)` (`src/Libraries/Nop.Services/Orders/OrderProcessingService.cs:1594`). Called from checkout controllers/factories (public checkout). **Tentative** on exact caller — inspect `Controllers/CheckoutController`.
- **Steps (with code references)** — **FACT**:
  1. Validate request + order GUID (`:1596-1599`).
  2. Prepare & validate cart, checkout attributes, addresses, prices (`PreparePlaceOrderDetailsAsync`, `:1602`; validation at `:489-502`).
  3. Process payment via the selected provider (`_paymentService.ProcessPaymentAsync` / recurring variants, `:1610-1614`, `:1414-1424`).
  4. On success: `SaveOrderDetailsAsync` (`:1616`), move cart items → order items (`:1621`, event `ShoppingCartItemMovedToOrderItemEvent` `:1343`), record discount + gift-card usage (`:1624-1627`).
  5. Notifications & order notes (`:1634`), reset checkout data / clear coupons (`:1637-1638`), activity log (`:1639-1641`).
  6. Publish `OrderPlacedEvent` (`:1644`); re-check status; if paid → `ProcessOrderPaidAsync` (raises `OrderPaidEvent` `:1112`).
- **Side effects** — **FACT**: DB writes (order, order items, usage history), payment provider call (external), customer activity log row, domain events (`OrderPlacedEvent`, `OrderPaidEvent`, `OrderStatusChangedEvent` `:1025`) that plugins/consumers can handle, notification emails queued.
- **Failure modes** — **FACT**:
  - All work is wrapped in `try/catch`; exceptions are logged via `_logger.ErrorAsync` and converted into `PlaceOrderResult` errors (`:1661-1676`) — **failures are swallowed into a result object, not thrown to the caller**. **INFERENCE**: a partial failure after `SaveOrderDetailsAsync` but before later steps could leave side-effects half-applied; there is no explicit DB transaction visible around the whole `placeOrder` body (**Tentative** — verify by inspecting `SaveOrderDetailsAsync` and whether an ambient transaction/unit-of-work exists).
  - Optional pessimistic locking: when `_orderSettings.PlaceOrderWithLock` is set, the order is placed under a distributed lock keyed by customer id (`:1679-1683+`) — **INFERENCE**: this guards against double-submit/duplicate orders, and is **off by default** unless configured.
- **Where errors are costly** — **INFERENCE**: payment capture vs. order persistence consistency; duplicate-order risk when locking is disabled; event handlers with side-effects that run after payment.

### Flow B — Inbound HTTP request pipeline
- **Trigger/entrypoint** — **FACT**: `app.ConfigureRequestPipeline()` (`Program.cs:47`) delegates to `EngineContext.Current.ConfigureRequestPipeline(application)` (`ApplicationBuilderExtensions.cs:52-55`), which invokes each `INopStartup.Configure` in `Order` sequence. **INFERENCE**: middleware order is therefore determined by plugin/startup `Order` properties, not a single linear file.
- **Steps** — **FACT (partial)**: exception handling (`UseNopExceptionHandler`, `ApplicationBuilderExtensions.cs:73+`), localization, static files, forwarded headers, WebOptimizer/WebMarkupMin minification (imports at `ApplicationBuilderExtensions.cs:37-38`). **Tentative** on full ordering — read the rest of `ApplicationBuilderExtensions.cs` and each `INopStartup.Configure`.
- **Side effects / failure modes** — **INFERENCE**: unhandled exceptions routed to `UseNopExceptionHandler`, which resolves `AppSettings` from the engine (`ApplicationBuilderExtensions.cs:75`); a mis-registered service at this point fails the whole request.

### Flow C — Scheduled/background tasks
- **Trigger/entrypoint** — **FACT**: `TaskScheduler.InitializeAsync()` short-circuits if the DB is not installed (`TaskScheduler.cs:47-49`), otherwise starts in-process `TaskThread`s. Tasks implement `IScheduleTask` and are run by `ScheduleTaskRunner` (`src/Libraries/Nop.Services/ScheduleTasks/`).
- **Side effects / failure modes** — **INFERENCE**: because tasks run in the web process, a heavy or failing task can affect request-serving capacity; scaling the web tier scales the scheduler too unless coordinated (**Tentative** — check whether tasks guard against concurrent execution across web-farm nodes).

---

## 6. Dependencies

- **Internal module dependencies** — **FACT/INFERENCE**: `Nop.Core` (no upward deps) ← `Nop.Data` ← `Nop.Services` ← `Nop.Web.Framework` ← `Nop.Web`; `Plugins/*` depend on Services/Framework. Layering is enforced by project references (`.csproj`) — **Tentative** on strictness (not every reference edge was read).
- **External services** — **FACT**:
  - **Relational DB**: MS SQL / PostgreSQL / MySQL (`Nop.Data/DataProviders/`).
  - **Distributed cache (optional)**: In-memory, **SQL Server**, or **Redis** selected via `DistributedCacheType` (`Nop.Web.Framework/Infrastructure/NopStartup.cs:100-119`; `RedisConnectionWrapper`, `RedisCacheManager`).
  - **Third-party via plugins**: PayPal, UPS, Twilio (SMS), Avalara (tax), Brevo/Omnisend (email/marketing), Facebook/Google auth & analytics, Azure Blob, Cloudflare Images, Dynamics 365 (each under `src/Plugins/`).
- **Build/ORM tooling** — **FACT**: **LinqToDB** (repository layer, `IRepository.cs` references LinqToDB provider types), **FluentMigrator** (`Nop.Data/DataProviders/Fluentmigrator`, `Migrations/`), **AutoMapper** (`NopEngine.cs:58-79`), **Autofac** (optional container, `Program.cs:29`), **iTextSharp** (PDF, imported in `ApplicationBuilderExtensions.cs:4`), **WebOptimizer/WebMarkupMin** (asset optimization), **Gulp** (front-end build). No EF Core evidence in the data layer (**INFERENCE** — LinqToDB is the ORM, not EF).
- **Not found (UNKNOWN / likely absent)**: message broker, dedicated search engine in core (note: `Nop.Plugin.Search.Lucene` exists as an **optional plugin**), OpenTelemetry/Serilog/AppInsights (see §8).

---

## 7. Configuration & environments

- **Where config lives** — **FACT**:
  - App settings JSON loaded at startup from `NopConfigurationDefaults.AppSettingsFilePath` (`Program.cs:14`) plus a per-environment overlay `AppSettingsEnvironmentFilePath` formatted with `EnvironmentName` (`Program.cs:16-18`). **INFERENCE**: these resolve under `App_Data/` (consistent with `dataSettings.json` living there).
  - **DB connection + provider**: `~/App_Data/dataSettings.json` (`NopDataSettingsDefaults.cs:16`; legacy `App_Data/Settings.txt` at `:11`). Written by the installer, read by `DataSettingsManager`.
  - Config objects implement `IConfig` and are bound at startup via `ITypeFinder` discovery (`ServiceCollectionExtensions.cs:74-75`) — **INFERENCE**: plugins can contribute their own config sections.
- **Environment variables** — **FACT**: `builder.Configuration.AddEnvironmentVariables()` (`Program.cs:20`), so any setting can be overridden via env vars. `Dockerfile` sets `ASPNETCORE_URLS`, `DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=false`. **UNKNOWN**: no `.env` file or documented variable catalog found — inspect the `AppSettings`/`IConfig` classes for the full list.
- **Secrets handling** — **FACT (observation)**: `docker-compose.yml:12` hardcodes `SA_PASSWORD: "nopCommerce_db_password"` (dev only). DB credentials live in `App_Data/dataSettings.json` (plaintext JSON on disk). Data-protection keys are persisted under `App_Data/DataProtectionKeys` (`Dockerfile` chmod list; `AddNopDataProtection`, `ServiceCollectionExtensions.cs:242`). **INFERENCE**: no external secrets manager (Vault/Key Vault) is wired in core; secrets are file-based. **Risk: Configuration risk** (see §10).
- **Local vs prod differences** — **FACT/INFERENCE**: environment-specific `appSettings.{Environment}.json` overlay (`Program.cs:16-18`); cache backend (Memory vs Redis/SQL) is the main scale-out switch; `.deployment`/`deploy.cmd` suggest Azure App Service deployment (**Tentative**).

---

## 8. Observability

- **Logging** — **FACT**: Custom **database-backed** logger. `DefaultLogger : ILogger` writes to `IRepository<Log>` (`src/Libraries/Nop.Services/Logging/DefaultLogger.cs:13-30`). Errors in the order flow call `_logger.ErrorAsync` (`OrderProcessingService.cs:1663,1674`). There is also a `CustomerActivityService` for user-facing audit trail (`Logging/CustomerActivityService.cs`).
  - **FACT**: **No Serilog / NLog / Elmah / Application Insights / OpenTelemetry** package references found (grep over all `*.csproj` returned nothing). Structured logging to external sinks appears absent in core.
- **Metrics** — **UNKNOWN → likely none**: no Prometheus/metrics endpoint or OpenTelemetry found. Inspect plugins (`Nop.Plugin.Misc.PowerBI`) for reporting, but that is analytics, not app metrics.
- **Tracing** — **UNKNOWN → likely none**: no distributed-tracing library found.
- **Alerts** — **UNKNOWN**: none found in repo. **INFERENCE**: operational alerting would be external (host/APM), not in-code.
- **Net observability assessment (INFERENCE)**: Observability is **DB-log-centric**. Log growth is managed by a `ClearLogTask` (`Logging/ClearLogTask.cs`). For production diagnosis you rely on the admin log grid + host-level tooling. **Risk: Lack of tests/observability tooling** for high-scale ops.

---

## 9. Testing & quality gates

- **Types of tests present** — **FACT**: A single **NUnit** project (`Nop.Tests`, `NUnit 4.5.1` + `NUnit3TestAdapter 6.2.0` + `Moq 4.20.72`, per `Nop.Tests.csproj`). Subfolders mirror layers: `Nop.Core.Tests`, `Nop.Data.Tests`, `Nop.Services.Tests`, `Nop.Web.Tests`. **125 `*Tests.cs` files** found.
  - **INFERENCE**: These are primarily **unit/integration** tests executed against an **in-memory SQLite** provider (`SqLiteNopDataProvider.cs`, `TestDataProviderManager.cs`, `BaseNopTest.cs`) — i.e., service logic is exercised against a real (SQLite) schema rather than mocked repositories in many cases.
  - **UNKNOWN**: No browser/e2e (Selenium/Playwright) test project found. No load tests found.
- **How they run in CI** — **FACT**: `.github/workflows/dotnet.yml` on `windows-latest`, triggered on push/PR to `develop`: `dotnet restore src` → `dotnet build -c Release src` → `dotnet test -c Release src` (`:23-30`). **INFERENCE**: build failure or any failing test fails the PR gate. No coverage threshold or linting gate is present in the workflow.
- **Coverage gaps / risks** — **INFERENCE**:
  - No e2e coverage of the actual HTTP pipeline / checkout UI.
  - Plugins have **no visible test projects** (search under `src/Plugins` found no `*Tests.csproj`; **Tentative**), so integrations (PayPal, UPS, tax) are likely untested in CI.
  - `OrderProcessingService` (3561 lines) is high-risk and hard to fully cover (see §10).

---

## 10. Migration hotspots / tech debt (evidence-based)

### Hotspot 1 — `OrderProcessingService` god class
- **Why it matters**: 3561 lines in one file (`src/Libraries/Nop.Services/Orders/OrderProcessingService.cs`), orchestrating validation, pricing, payment, persistence, notifications, discounts, gift cards, recurring orders, and event publishing (`:1594-1683`, plus refund/status methods elsewhere). Mixed concerns + broad ctor (many injected deps, `:47-170`).
- **Refactoring seam (Strangler-friendly)**: Extract cohesive collaborators behind interfaces — `IOrderPricingCalculator`, `IOrderPersister` (wraps `SaveOrderDetails*`), `IOrderNotificationCoordinator`, `ICheckoutValidator`. `PlaceOrderAsync` already delegates to well-named private helpers (`PreparePlaceOrderDetailsAsync`, `SaveOrderDetailsAsync`, `MoveShoppingCartItemsToOrderItemsAsync`), which are natural extraction points.
- **Candidate facade**: keep `IOrderProcessingService.PlaceOrderAsync` stable as the facade; move internals behind the new interfaces.
- **Test strategy**: characterization tests around `PlaceOrderAsync` (success + payment-failure + exception paths) using the SQLite test provider before extracting; then unit-test each extracted collaborator.
- **Risk type**: Mixed concerns | Hidden side-effects | Coupling.
- **Migration risk level**: **High** (money path; failures are swallowed into result objects, `:1661-1676`).
- **Confidence**: **High**.

### Hotspot 2 — Service-locator via `EngineContext.Current.Resolve<T>()`
- **Why it matters**: Runtime resolution through a static engine (`EngineContext.cs`, used in `ApplicationBuilderExtensions.cs:54,65`, `NopEngine.cs`) hides dependencies and complicates testing/analysis. ≥2 references → **FACT**.
- **Seam**: prefer constructor injection at call sites; confine `EngineContext` use to true composition-root code.
- **Risk type**: Coupling | Hidden side-effects.
- **Migration risk level**: **Medium** (wide but mechanical).
- **Confidence**: **High**.

### Hotspot 3 — Manual, centralized DI registration
- **Why it matters**: Hundreds of explicit `AddScoped` calls in `Nop.Web.Framework/Infrastructure/NopStartup.cs` (`:77-336`) and `Nop.Web/Infrastructure/NopStartup.cs` (`:22-115`). Adding/removing a service means editing large hand-maintained lists; easy to drift.
- **Seam**: convention-based registration (scan `I*Service`/`*Service` pairs via existing `ITypeFinder`) for the uniform cases, leaving explicit registration only for the switch-based ones (cache managers, `:100-129`).
- **Risk type**: Configuration risk | Coupling.
- **Migration risk level**: **Medium**.
- **Confidence**: **High**.

### Hotspot 4 — In-process background scheduler
- **Why it matters**: `TaskScheduler` runs jobs on threads inside the web process (`TaskScheduler.cs:16-49`); no process isolation. In a web farm this needs cross-node coordination to avoid duplicate runs. **Tentative** on current coordination mechanism.
- **Seam**: extract a hosted `BackgroundService` (or external worker) with leader-election/locking.
- **Risk type**: Hidden side-effects | Coupling.
- **Migration risk level**: **Medium/High**.
- **Confidence**: **Medium** (coordination behavior not fully read).

### Hotspot 5 — File-based plaintext secrets
- **Why it matters**: DB credentials in `App_Data/dataSettings.json` and a hardcoded dev DB password in `docker-compose.yml:12`.
- **Seam**: source connection strings/secrets from env vars (already supported, `Program.cs:20`) or a secrets manager; keep files for local only.
- **Risk type**: Configuration risk.
- **Migration risk level**: **Low/Medium**.
- **Confidence**: **High** (for the observation).

### Hotspot 6 — Observability gap
- **Why it matters**: DB-only logging, no metrics/tracing/structured sinks (§8). Hard to diagnose the order money-path under load.
- **Seam**: add OpenTelemetry (traces + metrics) and a structured-log sink alongside the existing `ILogger`.
- **Risk type**: Lack of tests/observability.
- **Migration risk level**: **Low** (additive).
- **Confidence**: **High**.

**Candidate boundaries for extraction (module/package level)** — **INFERENCE**: `Nop.Services/Orders`, `Nop.Services/Payments`, `Nop.Services/Catalog`, `Nop.Services/Messages` are the most self-contained domains and the best first candidates for facade-guarded module boundaries; payments already has a plugin-contract boundary (`IPaymentService` + payment plugins), making it the cleanest Strangler target.

---

## 11. Open questions

- **UNKNOWN**: Is there a DB transaction / unit-of-work around the full `PlaceOrderAsync` body? → inspect `SaveOrderDetailsAsync` and `EntityRepository.cs` / LinqToDB `DataConnection` usage.
- **UNKNOWN**: Full middleware order and every `INopStartup.Configure` contribution. → read the rest of `ApplicationBuilderExtensions.cs` + all `Infrastructure/NopStartup.cs` `Configure` methods and their `Order` values.
- **UNKNOWN**: Complete environment-variable / `AppSettings` catalog. → enumerate `IConfig` implementers and `AppSettings` members.
- **UNKNOWN**: How the installer bootstraps the schema and seeds data. → `Nop.Services/Installation` + `Nop.Web/Infrastructure/Installation`.
- **UNKNOWN**: Web-farm coordination for scheduled tasks (duplicate-run prevention). → `ScheduleTaskRunner.cs`, `TaskScheduler.cs`, distributed-lock usage.
- **UNKNOWN**: Authn/authz depth beyond cookie schemes (roles/permissions/ACL). → `Nop.Services/Security` (`IPermissionService`?), `AddNopAuthentication` (`ServiceCollectionExtensions.cs:255-288`).
- **Tentative**: Whether any plugins have automated tests. → confirm absence of `*Tests.csproj` under `src/Plugins`.

---

## 12. First 90 minutes checklist

1. **Confirm the runtime** (5 min): read `global.json`, `Dockerfile`, `.github/workflows/dotnet.yml` — the app is **.NET 10** despite `README.md` saying .NET 9.
2. **Get it building** (15 min): `dotnet restore src && dotnet build -c Release src`. Fix SDK version if `global.json` roll-forward fails.
3. **Run the test suite** (10 min): `dotnet test src`. Confirms the SQLite-backed suite runs with no external DB. Skim a failing/slow area if any.
4. **Trace the entrypoint** (10 min): `Program.cs` → `ConfigureApplicationServices` (`ServiceCollectionExtensions.cs:101`) → `ConfigureRequestPipeline` (`ApplicationBuilderExtensions.cs:52`). Understand the `EngineContext` + `ITypeFinder` + `INopStartup` composition model.
5. **Understand DI reality** (10 min): open both `Infrastructure/NopStartup.cs` files (Framework + Web) to see where services and factories are registered; note the `UseAutofac` switch in `Program.cs:26`.
6. **Read the money path** (15 min): `OrderProcessingService.PlaceOrderAsync` (`:1594-1683`) end to end; note the swallow-into-result error handling and the optional lock.
7. **Learn the data layer** (10 min): `IRepository.cs`, `EntityRepository.cs`, `DataProviders/`, and `Migrations/` — remember it's **LinqToDB + FluentMigrator**, not EF Core.
8. **Spin it up (optional)** (15 min): `docker compose up` (uses MS SQL 2019), complete the web installer, watch `App_Data/dataSettings.json` get created.
9. **Bookmark the hotspots** (5 min): `OrderProcessingService` (god class), `EngineContext` (service locator), the two `NopStartup.cs` registration lists, and the DB-only logging in `DefaultLogger.cs`.
```
