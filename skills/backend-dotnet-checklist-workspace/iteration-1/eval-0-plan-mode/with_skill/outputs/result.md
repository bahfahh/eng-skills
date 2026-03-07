# Background Email Notification Service - Implementation Checklist

## Your Current Plan Review
✅ **Good foundation:** IHostedService, EF Core queries, SMTP sending, status updates
⚠️ **Missing critical items identified below**

---

## Phase 1: Service Architecture & DI

### Background Service Implementation
- [ ] **Use `BackgroundService` base class** (not raw `IHostedService`) for cleaner cancellation handling
- [ ] **Inject `IServiceProvider`** (not scoped services directly) — background service is singleton, DbContext is scoped
- [ ] **Create scope per work cycle** using `IServiceProvider.CreateScope()` to get fresh DbContext
- [ ] **Use `PeriodicTimer`** instead of `Task.Delay` for consistent intervals
- [ ] **Respect `CancellationToken`** in all async operations — pass `stoppingToken` through entire chain

### Exception Handling
- [ ] **Wrap work cycle in try-catch** — don't let exceptions kill the hosted service
- [ ] **Log exceptions but continue running** — service should be resilient to transient failures
- [ ] **Separate transient vs permanent failures** — retry transient, skip permanent

---

## Phase 2: EF Core & Data Layer

### DbContext Configuration
- [ ] **Configure `QueryTrackingBehavior.NoTracking` by default** in DbContext constructor
- [ ] **Use `AsTracking()` explicitly** when updating email status (or call `Update()` method)
- [ ] **Never inject DbContext directly** into the background service — use scoped resolution

### Query Patterns
- [ ] **Use `AsNoTracking()`** for read-only pending email queries
- [ ] **Batch status updates** — don't call `SaveChangesAsync` inside loop
- [ ] **Consider `ExecuteUpdateAsync`** for bulk status updates (EF Core 7+)
- [ ] **Add database index** on email status and created date columns for query performance
- [ ] **Use `CancellationToken`** in all EF Core operations

### Transaction Handling
- [ ] **Use explicit transaction** for multi-step operations (query → send → update status)
- [ ] **Handle transaction rollback** if email sending fails after status update
- [ ] **Consider idempotency** — what happens if service restarts mid-processing?

---

## Phase 3: SMTP & External Dependencies

### SMTP Configuration
- [ ] **Use `IOptions<SmtpSettings>`** for SMTP configuration — not raw `IConfiguration`
- [ ] **Store credentials securely** — use user secrets (dev) or key vault (prod)
- [ ] **Configure connection pooling** or reuse `SmtpClient` instances appropriately
- [ ] **Set reasonable timeouts** for SMTP operations

### Resilience Patterns
- [ ] **Implement retry logic** with exponential backoff for transient SMTP failures
- [ ] **Circuit breaker pattern** if SMTP server becomes unavailable
- [ ] **Dead letter queue** for emails that fail permanently after retries
- [ ] **Rate limiting** — don't overwhelm SMTP server with concurrent sends

---

## Phase 4: Monitoring & Observability

### Logging
- [ ] **Structured logging** with email ID, recipient, attempt count
- [ ] **Log processing metrics** — emails processed per cycle, success/failure rates
- [ ] **Different log levels** — Debug for processing details, Warning for retries, Error for failures

### Health Checks
- [ ] **Add health check** for the background service status
- [ ] **SMTP connectivity check** in health endpoint
- [ ] **Database connectivity check** for email queue access

### Metrics
- [ ] **Track processing latency** — time from email creation to sending
- [ ] **Monitor queue depth** — pending emails count
- [ ] **Success/failure rates** for alerting

---

## Phase 5: Configuration & Security

### Email Queue Design
- [ ] **Add retry count column** to track failed attempts
- [ ] **Add next retry timestamp** to implement exponential backoff
- [ ] **Add priority field** for urgent vs normal emails
- [ ] **Soft delete pattern** — mark as deleted instead of hard delete for audit

### Security Considerations
- [ ] **Sanitize email content** — prevent injection attacks
- [ ] **Validate recipient addresses** before sending
- [ ] **Rate limiting per recipient** — prevent spam accusations
- [ ] **Audit trail** — log all email sending attempts

---

## Phase 6: Testing Strategy

### Unit Tests
- [ ] **Mock `IServiceProvider`** and scoped services in background service tests
- [ ] **Test exception handling** — verify service continues after failures
- [ ] **Test cancellation** — verify clean shutdown on `CancellationToken`

### Integration Tests
- [ ] **Use Testcontainers** for database integration tests
- [ ] **Mock SMTP server** (or use test SMTP service like Papertrail)
- [ ] **Test full email processing cycle** — queue → process → verify status

---

## Phase 7: Production Readiness

### Performance
- [ ] **Batch processing** — process multiple emails per cycle, not one-by-one
- [ ] **Configurable batch size** — balance throughput vs memory usage
- [ ] **Parallel processing** — use `Parallel.ForEachAsync` for independent email sends
- [ ] **Connection pooling** for database and SMTP connections

### Deployment
- [ ] **Graceful shutdown** — finish current batch before stopping
- [ ] **Zero-downtime deployment** — handle multiple instances processing same queue
- [ ] **Database migration strategy** for email queue schema changes

### Monitoring
- [ ] **Application Insights** or similar APM for distributed tracing
- [ ] **Custom metrics** exported to monitoring system
- [ ] **Alerting rules** for queue backup, high failure rates, service down

---

## Critical Anti-Patterns to Avoid

❌ **Don't inject `DbContext` directly** into background service (lifetime mismatch)
❌ **Don't use `.Result` or `.Wait()`** on async SMTP operations (deadlock risk)
❌ **Don't call `SaveChangesAsync` inside loop** (performance killer)
❌ **Don't let exceptions kill the service** (wrap in try-catch)
❌ **Don't ignore `CancellationToken`** (prevents graceful shutdown)
❌ **Don't hardcode SMTP settings** (use `IOptions<T>`)

---

## Implementation Priority

1. **Core service structure** with proper DI scoping
2. **Basic email processing loop** with exception handling
3. **EF Core query optimization** and transaction handling
4. **SMTP resilience** and retry logic
5. **Monitoring and health checks**
6. **Performance optimization** and batch processing