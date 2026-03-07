# .NET Concurrency Patterns

> Source: Aaronontheweb/dotnet-skills (csharp-concurrency-patterns)

## Philosophy

**Start simple, escalate only when needed.** Most concurrency problems are solved with `async/await`. Avoid shared mutable state — design it away with immutability or message passing.

**Escalation path:**
```
async/await → Parallel.ForEachAsync → Channel<T> → Akka.NET Streams/Actors
```

Only escalate when you have a concrete need.

---

## Decision Tree

| Need | Tool |
|------|------|
| Wait for I/O (HTTP, DB, file) | `async/await` |
| Parallel CPU-bound work | `Parallel.ForEachAsync` |
| Producer/consumer work queue | `Channel<T>` |
| Fire multiple independent async ops | `Task.WhenAll` |
| Periodic background work | `PeriodicTimer` |
| UI event debounce/throttle | Reactive Extensions |
| Server-side batching with backpressure | Akka.NET Streams |
| Stateful entity management | Akka.NET Actors |

---

## Level 1: async/await (Default)

```csharp
// Sequential I/O
public async Task<Order> GetOrderAsync(string id, CancellationToken ct)
{
    var order = await _db.GetAsync(id, ct);
    var customer = await _customerService.GetAsync(order.CustomerId, ct);
    return order with { Customer = customer };
}

// Parallel independent operations
public async Task<Dashboard> LoadDashboardAsync(string userId, CancellationToken ct)
{
    var ordersTask = _orderService.GetRecentAsync(userId, ct);
    var statsTask  = _statsService.GetAsync(userId, ct);
    await Task.WhenAll(ordersTask, statsTask);
    return new Dashboard(await ordersTask, await statsTask);
}
```

---

## Level 2: Parallel.ForEachAsync (CPU-Bound)

```csharp
await Parallel.ForEachAsync(orders,
    new ParallelOptions { MaxDegreeOfParallelism = Environment.ProcessorCount, CancellationToken = ct },
    async (order, token) => await ProcessOrderAsync(order, token));
```

**Not for:** pure I/O, order-sensitive work, when you need backpressure.

---

## Level 3: Channel\<T\> (Producer/Consumer)

```csharp
public class OrderProcessor
{
    private readonly Channel<Order> _channel = Channel.CreateBounded<Order>(
        new BoundedChannelOptions(100) { FullMode = BoundedChannelFullMode.Wait });

    // Producer
    public async Task EnqueueAsync(Order order, CancellationToken ct)
        => await _channel.Writer.WriteAsync(order, ct);

    // Consumer (run as background task)
    public async Task ProcessAsync(CancellationToken ct)
    {
        await foreach (var order in _channel.Reader.ReadAllAsync(ct))
            await ProcessOrderAsync(order, ct);
    }
}
```

---

## Anti-Patterns

```csharp
// ❌ Blocking on async — deadlock in ASP.NET
var result = GetDataAsync().Result;
var result = GetDataAsync().GetAwaiter().GetResult();

// ✅ Async all the way
var result = await GetDataAsync();

// ❌ Race condition — shared mutable list
var results = new List<Result>();
await Parallel.ForEachAsync(items, async (item, ct) =>
    results.Add(await ProcessAsync(item, ct))); // Race!

// ✅ Thread-safe collection
var results = new ConcurrentBag<Result>();

// ❌ Manual thread creation
new Thread(() => ProcessOrders()).Start();

// ✅ Task.Run or proper abstractions
_ = Task.Run(() => ProcessOrdersAsync(ct), ct);

// ❌ Lock for business logic
private readonly object _lock = new();
lock (_lock) { _orders[id] = order; }

// ✅ Channel<T> or actor to serialize access
```

---

## IHostedService / BackgroundService Pattern

```csharp
public class EmailSenderService : BackgroundService
{
    private readonly IServiceProvider _sp;
    private readonly ILogger<EmailSenderService> _logger;

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        using var timer = new PeriodicTimer(TimeSpan.FromMinutes(1));
        while (await timer.WaitForNextTickAsync(stoppingToken))
        {
            try
            {
                // Always create scope — DbContext is scoped, service is singleton
                using var scope = _sp.CreateScope();
                var processor = scope.ServiceProvider.GetRequiredService<IEmailProcessor>();
                await processor.ProcessPendingAsync(stoppingToken);
            }
            catch (Exception ex) when (ex is not OperationCanceledException)
            {
                _logger.LogError(ex, "Email processing failed");
                // Don't rethrow — keep the service running
            }
        }
    }
}
```

Key rules for `BackgroundService`:
- Always use `IServiceScopeFactory` / `IServiceProvider` to resolve scoped services
- Never inject `DbContext` directly (it's scoped, service is singleton)
- Catch and log exceptions — don't let them kill the hosted service
- Respect `stoppingToken` in all async calls
