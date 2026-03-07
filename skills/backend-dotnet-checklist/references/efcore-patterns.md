# EF Core Patterns

> Source: Aaronontheweb/dotnet-skills (efcore-patterns)

## Core Principles

1. **NoTracking by Default** — Configure DbContext to disable tracking globally; opt-in for writes
2. **Never Edit Migrations Manually** — Always use CLI commands
3. **Dedicated Migration Service** — Separate migration execution from app startup
4. **ExecutionStrategy for Retries** — Handle transient DB failures
5. **Bulk Operations** — Use `ExecuteUpdateAsync`/`ExecuteDeleteAsync` for batch writes

---

## NoTracking by Default

```csharp
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options)
    {
        // Disable tracking globally — opt-in with .AsTracking() for writes
        ChangeTracker.QueryTrackingBehavior = QueryTrackingBehavior.NoTracking;
    }
}
```

### Read vs Write Pattern

```csharp
// ✅ Read — no tracking needed
var orders = await _db.Orders
    .Where(o => o.Status == OrderStatus.Pending)
    .ToListAsync(ct);

// ✅ Write — explicitly mark modified
var order = await _db.Orders.FirstOrDefaultAsync(o => o.Id == id, ct);
order.Status = OrderStatus.Shipped;
_db.Orders.Update(order);   // Required when NoTracking is default
await _db.SaveChangesAsync(ct);

// ✅ Write alternative — use AsTracking() on the query
var order = await _db.Orders.AsTracking().FirstOrDefaultAsync(o => o.Id == id, ct);
order.Status = OrderStatus.Shipped;
await _db.SaveChangesAsync(ct);
```

**Silent failure trap:** Without `Update()` or `AsTracking()`, `SaveChanges` does nothing when NoTracking is default.

---

## Bulk Operations (EF Core 7+)

```csharp
// ❌ Slow — loads all entities into memory
var expired = await _db.Orders.Where(o => o.ExpiresAt < now).ToListAsync();
foreach (var o in expired) { o.Status = OrderStatus.Expired; }
await _db.SaveChangesAsync();

// ✅ Fast — single SQL UPDATE
await _db.Orders
    .Where(o => o.ExpiresAt < now)
    .ExecuteUpdateAsync(s => s
        .SetProperty(o => o.Status, OrderStatus.Expired)
        .SetProperty(o => o.UpdatedAt, now));

// ✅ Fast — single SQL DELETE
await _db.Orders
    .Where(o => o.Status == OrderStatus.Cancelled && o.CreatedAt < cutoff)
    .ExecuteDeleteAsync();
```

---

## Query Splitting (Cartesian Explosion Prevention)

```csharp
// Enable globally to prevent cartesian explosion with multiple Includes
services.AddDbContext<AppDbContext>(options =>
    options.UseNpgsql(conn, o =>
        o.UseQuerySplittingBehavior(QuerySplittingBehavior.SplitQuery)));

// Override per-query when single query is better
var orders = await _db.Orders
    .Include(o => o.Items)
    .AsSingleQuery()
    .ToListAsync(ct);
```

---

## DbContext Lifetime

```csharp
// ASP.NET Core — Scoped (default, correct)
builder.Services.AddDbContext<AppDbContext>(options => options.UseNpgsql(conn));

// Background services — create scope per unit of work
public class MyBackgroundService : BackgroundService
{
    private readonly IServiceProvider _sp;
    protected override async Task ExecuteAsync(CancellationToken ct)
    {
        using var scope = _sp.CreateScope();
        var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();
        // use db...
    }
}

// Long-lived objects (actors, singletons) — use factory
builder.Services.AddDbContextFactory<AppDbContext>(options => options.UseNpgsql(conn));
// Then: await using var db = await _dbFactory.CreateDbContextAsync();
```

---

## Migrations

```bash
# Create
dotnet ef migrations add AddUserEmailIndex \
    --project src/MyApp.Infrastructure \
    --startup-project src/MyApp.Api

# Remove last (if not applied)
dotnet ef migrations remove ...

# Generate idempotent SQL for production
dotnet ef migrations script --idempotent ...
```

- Never manually edit migration files (except custom SQL in `Up()`/`Down()`)
- Never delete migration files directly — use `migrations remove`
- Use `--idempotent` script for production deployments, not `database update`

---

## Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| `SaveChanges` does nothing | Add `_db.Update(entity)` or use `AsTracking()` |
| N+1 queries | Use `Include()` / `ThenInclude()` or `Contains()` batch query |
| `SaveChanges` inside loop | Batch outside loop, or use `ExecuteUpdateAsync` |
| Querying inside loop | Single `Where(o => ids.Contains(o.Id))` query |
| Lazy loading in prod | Disable; use explicit `Include()` |
| Cartesian explosion | Enable `SplitQuery` globally |
