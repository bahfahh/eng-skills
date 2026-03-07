---
name: backend-dotnet-checklist
description: .NET / C# backend development quality checklist covering async patterns, DI lifetime, EF Core, middleware ordering, configuration, and API design. Use whenever the task involves .NET or C# backend work — including: planning or implementing ASP.NET Core features (minimal API, Web API controllers, middleware, background services, IHostedService, EF Core migrations); debugging .NET-specific issues (deadlocks from sync-over-async, DbContext lifetime errors, DI scope mismatches, missing CancellationToken, IOptions not resolving, N+1 queries); reviewing .NET / C# code for async correctness, EF Core tracking behavior, DI configuration, or middleware pipeline order; designing .NET API endpoints, service layers, or repository patterns. Also use when the user says "backend", "API", "dotnet", "C# service", "ASP.NET", "Web API", or "EF Core". Do NOT use for Blazor frontend UI, general C# syntax questions unrelated to backend, or non-.NET stacks.
---

## Mode

**Coding / Plan mode** — planning or about to implement a .NET feature:
- First check all 8 Core Guardrails below against the current plan — flag any that are missing
- Then read `references/checklist-full.md` for the relevant phases only (not all phases)
- Add at most 10–15 items to the todolist — prioritize .NET-specific pitfalls that AI commonly misses over general best practices
- For C# language patterns (records, value objects, pattern matching), read `references/csharp-standards.md`
- For EF Core work (queries, migrations, tracking), read `references/efcore-patterns.md`
- For concurrency/async/background services, read `references/concurrency-patterns.md`

**Code review mode** — reviewing completed .NET code:
- Read `references/checklist-full.md`
- For C# code style issues, read `references/csharp-standards.md`
- For EF Core issues (N+1, tracking, migrations), read `references/efcore-patterns.md`
- For async/concurrency issues, read `references/concurrency-patterns.md`
- Go through each phase and flag gaps as review findings

---

## Core Guardrails

These apply to all .NET work — check them before writing any code:

- Never use `.Result`, `.Wait()`, or `.GetAwaiter().GetResult()` on async methods — causes deadlocks in ASP.NET context; propagate `async/await` all the way
- Never register `DbContext` as singleton — always scoped; never inject scoped services into singleton services
- Always accept `CancellationToken` in async methods and pass it through to EF Core queries and HTTP calls
- Use `IOptions<T>` (or `IOptionsSnapshot<T>`) to inject configuration into services — never inject `IConfiguration` directly
- Register exception handling middleware first in the pipeline — before routing, auth, and all other middleware
- Never call `SaveChanges` inside a loop — batch writes outside the loop
- Use `AsNoTracking()` for read-only EF Core queries
- Enable `<Nullable>enable</Nullable>` and treat warnings as errors in production code

---

## Checklist Selection

| Task | Reference |
|------|-----------|
| Full .NET feature development / PR review | `references/checklist-full.md` |
| API layer (endpoints, routing, validation, error handling) | `references/checklist-full.md` Phase 1 |
| Service layer (DI, async patterns, business logic) | `references/checklist-full.md` Phase 2 |
| Data layer (EF Core, migrations, transactions) | `references/checklist-full.md` Phase 3 |
| Configuration & middleware | `references/checklist-full.md` Phase 4 |
| Testing (unit, integration, testcontainers) | `references/checklist-full.md` Phase 5 |
| Build & CI (nullable, analyzers, health checks) | `references/checklist-full.md` Phase 6 |

---

## Quick Reference

| Pattern | Standard |
|---------|----------|
| Async default | `async/await` all the way — no `.Result` / `.Wait()` |
| DbContext lifetime | Scoped — never singleton |
| DI scope rule | Never inject scoped into singleton |
| Cancellation | Accept `CancellationToken`, pass to all I/O |
| Configuration | `IOptions<T>` — not raw `IConfiguration` in services |
| Read-only queries | `AsNoTracking()` |
| Batch writes | `SaveChanges` once outside loop |
| Exception middleware | Register first in pipeline |
| Nullable | `<Nullable>enable</Nullable>` in all projects |
| C# standards | Read `references/csharp-standards.md` |
| EF Core patterns | Read `references/efcore-patterns.md` |
| Concurrency / async | Read `references/concurrency-patterns.md` |
