# .NET Development Checklist

## Table of Contents

- [Phase 1 — API Layer](#phase-1--api-layer)
- [Phase 2 — Service Layer](#phase-2--service-layer)
- [Phase 3 — Data Layer](#phase-3--data-layer)
- [Phase 4 — Configuration & Middleware](#phase-4--configuration--middleware)
- [Phase 5 — Testing](#phase-5--testing)
- [Phase 6 — Build & CI](#phase-6--build--ci)

## Phase 1 — API Layer

### Endpoints & Routing
- [ ] Minimal API or controller — choose one pattern per project, don't mix
- [ ] Route naming follows REST conventions (`/api/v1/resources/{id}`)
- [ ] HTTP verbs match semantics: GET (read), POST (create), PUT/PATCH (update), DELETE
- [ ] Versioning strategy defined (`/v1/`, header, or query param) before first release

### Validation
- [ ] Input validated before reaching service layer (FluentValidation or DataAnnotations)
- [ ] Validation errors return `400 Bad Request` with structured `ProblemDetails`
- [ ] No business logic in validators — validators check shape, not rules

### Error Handling
- [ ] Global exception handler middleware registered first in pipeline
- [ ] All exceptions map to `ProblemDetails` (RFC 7807) — no raw exception messages to client
- [ ] Domain errors (not found, conflict) return correct HTTP status codes
- [ ] Sensitive error details (stack traces, connection strings) never exposed in responses

### Response
- [ ] Consistent response envelope or direct resource — pick one, don't mix
- [ ] Pagination for list endpoints (`page`/`pageSize` or cursor-based)
- [ ] `ETag` / `Last-Modified` for cacheable resources

---

## Phase 2 — Service Layer

### Dependency Injection
- [ ] Services registered with correct lifetime: transient (stateless, cheap), scoped (per-request), singleton (thread-safe, expensive to create)
- [ ] No scoped service injected into singleton — causes captive dependency bug
- [ ] `IServiceScopeFactory` used when singleton needs scoped service
- [ ] Interfaces defined for all services that will be unit tested or swapped

### Async Patterns
- [ ] All I/O operations are async — no sync-over-async (`.Result`, `.Wait()`, `.GetAwaiter().GetResult()`)
- [ ] `CancellationToken` accepted in every async public method and passed to all downstream I/O
- [ ] `Task.WhenAll` used for independent parallel operations — not sequential awaits
- [ ] `ConfigureAwait(false)` used in library code (not required in ASP.NET app code)
- [ ] No `async void` except event handlers — use `async Task` everywhere else

### Business Logic
- [ ] Business rules live in service layer, not controllers or repositories
- [ ] Domain exceptions thrown for business rule violations (not generic `Exception`)
- [ ] No direct `DbContext` access in controllers — always through service/repository

---

## Phase 3 — Data Layer

### EF Core
- [ ] `DbContext` registered as scoped — never singleton or transient
- [ ] `AsNoTracking()` on all read-only queries
- [ ] `SaveChangesAsync` called once per unit of work — not inside loops
- [ ] Explicit transactions used for multi-step writes (`BeginTransactionAsync`)
- [ ] No lazy loading in production — use explicit `Include()` / `ThenInclude()`
- [ ] N+1 queries identified and eliminated — check generated SQL in development

### Migrations
- [ ] Migration names are descriptive (`AddUserEmailIndex`, not `Migration1`)
- [ ] Each migration is idempotent — safe to run multiple times
- [ ] `MigrationBuilder.Sql` used for data migrations, not C# loops
- [ ] Migrations reviewed for destructive operations (column drops, renames) before production
- [ ] `dotnet ef migrations script` used to generate SQL for production — not `database update` in prod

### Connection & Performance
- [ ] Connection string in configuration — never hardcoded
- [ ] Connection pooling configured (default is fine for most cases)
- [ ] Indexes added for all foreign keys and frequently filtered columns
- [ ] Query performance verified with `LogTo` or SQL profiler in development

---

## Phase 4 — Configuration & Middleware

### Configuration
- [ ] All settings in `appsettings.json` / environment variables — no hardcoded values
- [ ] Secrets in `dotnet user-secrets` (dev) or secret manager (prod) — never in source control
- [ ] `IOptions<T>` used to inject typed config into services — not raw `IConfiguration`
- [ ] `IOptionsSnapshot<T>` used when config needs to reload without restart
- [ ] Config validated at startup with `ValidateDataAnnotations()` or `ValidateOnStart()`

### Middleware Pipeline Order
```
ExceptionHandler → HSTS → HTTPS Redirect → Static Files →
Routing → CORS → Auth → Authorization → Endpoints
```
- [ ] Exception handler is first
- [ ] Authentication before Authorization
- [ ] CORS before Auth
- [ ] `UseRouting` before `UseAuthentication` / `UseAuthorization`

### Health Checks
- [ ] `/health` endpoint registered with `AddHealthChecks()`
- [ ] DB health check included (`AddDbContextCheck<T>`)
- [ ] Liveness vs readiness probes separated if deploying to Kubernetes

---

## Phase 5 — Testing

### Unit Tests
- [ ] Services unit tested with mocked dependencies (Moq or NSubstitute)
- [ ] No `DbContext` in unit tests — use repository interface mocks
- [ ] Async tests use `async Task` — not `async void`
- [ ] Test names follow `MethodName_Scenario_ExpectedResult` convention

### Integration Tests
- [ ] `WebApplicationFactory<T>` used for API integration tests
- [ ] Real database used for integration tests (Testcontainers or SQLite in-memory)
- [ ] Each test cleans up its own data — no shared state between tests
- [ ] Auth bypassed or mocked in integration tests with test auth handler

### Coverage
- [ ] Happy path covered for all public service methods
- [ ] Error paths covered: not found, validation failure, domain exception
- [ ] No tests for EF Core internals — test behavior, not implementation

---

## Phase 6 — Build & CI

### Code Quality
- [ ] `<Nullable>enable</Nullable>` in all `.csproj` files
- [ ] `<TreatWarningsAsErrors>true</TreatWarningsAsErrors>` in CI builds
- [ ] Roslyn analyzers enabled (`Microsoft.CodeAnalysis.NetAnalyzers`)
- [ ] No suppressed warnings without documented reason

### CI Pipeline
- [ ] `dotnet build` → `dotnet test` → `dotnet publish` in CI
- [ ] Tests run with `--no-build` after build step
- [ ] Code coverage reported (Coverlet + ReportGenerator)
- [ ] `dotnet format --verify-no-changes` in CI to enforce formatting
