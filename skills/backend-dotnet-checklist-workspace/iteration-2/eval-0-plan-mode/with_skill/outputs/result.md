# Background Email Notification Service - Implementation Checklist

## Core Guardrails (Critical - Check First)

- [ ] **Never use `.Result`, `.Wait()`, or `.GetAwaiter().GetResult()`** — Use `async/await` all the way through SMTP operations
- [ ] **DbContext registered as scoped** — Never inject DbContext directly into IHostedService (singleton)
- [ ] **Always accept `CancellationToken`** in async methods and pass to EF Core queries and SMTP operations
- [ ] **Use `IOptions<T>` for SMTP configuration** — Never inject `IConfiguration` directly into services
- [ ] **Use `AsNoTracking()` for email queries** — Read-only operations don't need change tracking
- [ ] **Call `SaveChanges` once per batch** — Not inside the email sending loop
- [ ] **Enable `<Nullable>enable</Nullable>`** — Treat warnings as errors

## Background Service Implementation

### IHostedService Pattern
- [ ] **Use `IServiceProvider.CreateScope()`** — Resolve scoped services (DbContext, email service) within ExecuteAsync
- [ ] **Use `PeriodicTimer`** instead of `Timer` or `Task.Delay` for periodic execution
- [ ] **Catch and log exceptions** — Don't let exceptions kill the hosted service
- [ ] **Respect `CancellationToken`** in all async operations (DB queries, SMTP calls)
- [ ] **Dispose scope properly** — Use `using var scope = _sp.CreateScope()`

### Email Processing Logic
- [ ] **Batch email queries** — Load pending emails in batches (e.g., 50-100) to avoid memory issues
- [ ] **Use `ExecuteUpdateAsync` for status updates** — Bulk update email status without loading entities
- [ ] **Implement retry logic** — Handle transient SMTP failures with exponential backoff
- [ ] **Add circuit breaker** — Stop processing if SMTP server is consistently failing
- [ ] **Track processing metrics** — Log success/failure rates, processing time

## Data Layer (EF Core)

### Email Entity Design
- [ ] **Add proper indexes** — Index on `Status`, `CreatedAt`, `ScheduledAt` for efficient queries
- [ ] **Use enum for email status** — `Pending`, `Sent`, `Failed`, `Retrying`
- [ ] **Add retry count field** — Track failed attempts to prevent infinite retries
- [ ] **Add `LastAttemptAt` timestamp** — For retry delay calculations
- [ ] **Store serialized email data** — JSON column for recipients, subject, body, attachments

### Query Patterns
- [ ] **Use `AsNoTracking()` for read queries** — Email lookup doesn't need change tracking
- [ ] **Batch status updates** — Use `ExecuteUpdateAsync` instead of loading entities
- [ ] **Implement proper pagination** — Process emails in batches to avoid memory issues
- [ ] **Add query timeout** — Set reasonable timeout for email queries

## SMTP Configuration & Error Handling

### Configuration
- [ ] **Use `IOptions<SmtpSettings>`** — Typed configuration for SMTP settings
- [ ] **Store credentials securely** — Use user secrets (dev) or key vault (prod)
- [ ] **Configure connection pooling** — Reuse SMTP connections when possible
- [ ] **Set reasonable timeouts** — SMTP send timeout, connection timeout

### Error Handling & Resilience
- [ ] **Implement exponential backoff** — Retry failed emails with increasing delays
- [ ] **Set maximum retry limit** — Prevent infinite retry loops (e.g., 3-5 attempts)
- [ ] **Handle different failure types** — Permanent failures (invalid email) vs transient (network)
- [ ] **Dead letter queue** — Move permanently failed emails to separate table/queue
- [ ] **Rate limiting** — Respect SMTP server rate limits to avoid being blocked

## Monitoring & Observability

### Logging
- [ ] **Structured logging** — Use structured logs with email ID, batch size, processing time
- [ ] **Log SMTP errors** — Capture SMTP error codes and messages for debugging
- [ ] **Performance metrics** — Track emails processed per minute, success rate
- [ ] **Health check endpoint** — Include email service status in health checks

### Health Checks
- [ ] **SMTP connectivity check** — Verify SMTP server is reachable
- [ ] **Database connectivity** — Check if email queries are working
- [ ] **Queue depth monitoring** — Alert if pending email count grows too large
- [ ] **Processing lag alerts** — Alert if emails are not being processed timely

## Security Considerations

### Email Content Security
- [ ] **Sanitize email content** — Prevent HTML injection in email bodies
- [ ] **Validate email addresses** — Use proper email validation before sending
- [ ] **Rate limiting per recipient** — Prevent spam by limiting emails per recipient
- [ ] **Audit trail** — Log who triggered each email and when

### SMTP Security
- [ ] **Use TLS/SSL** — Encrypt SMTP connections
- [ ] **Secure credential storage** — Never hardcode SMTP passwords
- [ ] **IP whitelisting** — Restrict SMTP access if possible
- [ ] **Monitor for abuse** — Track unusual sending patterns

## Testing Strategy

### Unit Tests
- [ ] **Mock SMTP client** — Test email processing logic without actual sending
- [ ] **Test retry logic** — Verify exponential backoff and retry limits
- [ ] **Test batch processing** — Verify correct handling of email batches
- [ ] **Test cancellation** — Verify proper handling of CancellationToken

### Integration Tests
- [ ] **Use test SMTP server** — MailHog or similar for integration testing
- [ ] **Test database operations** — Verify email status updates work correctly
- [ ] **Test configuration** — Verify IOptions binding works correctly
- [ ] **Test hosted service lifecycle** — Start/stop behavior

## Performance Optimization

### Database Performance
- [ ] **Optimize email queries** — Use covering indexes for common query patterns
- [ ] **Implement connection pooling** — Configure appropriate pool size
- [ ] **Use read replicas** — If available, read pending emails from replica
- [ ] **Archive old emails** — Move sent/failed emails to archive tables

### SMTP Performance
- [ ] **Connection reuse** — Keep SMTP connections alive between sends
- [ ] **Parallel processing** — Send multiple emails concurrently (with rate limiting)
- [ ] **Batch operations** — Group similar emails when possible
- [ ] **Optimize email size** — Compress attachments, optimize HTML content