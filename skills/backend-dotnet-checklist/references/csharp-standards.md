# C# Coding Standards

> Source: Aaronontheweb/dotnet-skills (modern-csharp-coding-standards)

## Core Principles

1. **Immutability by Default** — Use `record` types and `init`-only properties
2. **Type Safety** — Enable nullable reference types; use value objects
3. **Modern Pattern Matching** — Use `switch` expressions and property patterns
4. **Async Everywhere** — All I/O async with `CancellationToken`
5. **Composition Over Inheritance** — No abstract base classes; use interfaces + composition
6. **Value Objects as Structs** — Use `readonly record struct` for value objects

---

## Records & Value Objects

```csharp
// DTOs and domain entities → record class
public record CustomerDto(string Id, string Name, string Email);

// Value objects → readonly record struct
public readonly record struct OrderId(Guid Value)
{
    public static OrderId New() => new(Guid.NewGuid());
}
public readonly record struct Money(decimal Amount, string Currency);
```

- Use `record class` for entities, aggregates, DTOs
- Use `readonly record struct` for value objects (OrderId, Money, Email)
- Never use implicit conversion operators on value objects

---

## Pattern Matching

```csharp
public decimal CalculateDiscount(Order order) => order switch
{
    { Total: > 1000m } => order.Total * 0.15m,
    { Total: > 500m }  => order.Total * 0.10m,
    _ => 0m
};

// Null guard (C# 11+)
ArgumentNullException.ThrowIfNull(order);
```

---

## Async/Await Rules

```csharp
// Always accept CancellationToken with = default
public async Task<Order> GetOrderAsync(string id, CancellationToken ct = default)
{
    return await _repository.GetAsync(id, ct);
}

// ValueTask for hot paths that are often synchronous
public ValueTask<Order?> GetCachedOrderAsync(string id, CancellationToken ct)
{
    if (_cache.TryGetValue(id, out var order))
        return ValueTask.FromResult<Order?>(order);
    return GetFromDatabaseAsync(id, ct);
}

// Parallel independent operations
var (orders, stats) = await (
    _orderService.GetAsync(userId, ct),
    _statsService.GetAsync(userId, ct)
).WhenAll();
```

**Never:** `.Result`, `.Wait()`, `.GetAwaiter().GetResult()` — deadlock risk in ASP.NET context.

---

## Error Handling

- Use `Result<T, TError>` for expected business errors (not found, validation failure)
- Use exceptions only for unexpected/system errors
- Never throw `Exception` directly — use domain-specific exception types or Result

```csharp
public readonly record struct OrderError(string Code, string Message);
// Return Result<Order, OrderError> instead of throwing for business errors
```

---

## DO / DON'T

| DO | DON'T |
|----|-------|
| `record` for DTOs and entities | Mutable classes when records work |
| `readonly record struct` for value objects | Classes for value objects |
| Pattern matching with `switch` | Deep `if/else` chains |
| `CancellationToken` in all async methods | Forget cancellation |
| `Result<T>` for expected errors | Throw exceptions for business rules |
| Composition via interfaces | Abstract base class hierarchies |
| Explicit mapping extension methods | AutoMapper / Mapster |
