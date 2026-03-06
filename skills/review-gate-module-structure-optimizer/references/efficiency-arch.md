# Efficiency and Architecture Risks

Goal: find real scalability/latency/resource risks and propose how to validate.
Prefer issues with clear user impact.

## Checks

### N+1 and Broad Reads
- DB/API calls inside loops
- repeated queries for related entities
- loading whole tables/collections when only a subset is needed

### Missed Concurrency
- sequential awaits with no dependency (suggest Promise.all / Task.WhenAll)
- independent network calls performed serially

### Repeated Computation
- recomputing same derived values in a single request/flow
- expensive formatting/parsing repeated in loops

### Resource Risks
- unbounded arrays/lists
- missing dispose/close (streams, connections)
- event listeners not removed (frontend)

### Wrong-Layer IO
- domain logic reaching into DB/HTTP directly without a stable boundary
- caching/invalidation mismatches after writes

## Output Requirements

For each risk:
- location + pattern
- why it matters (impact)
- concrete mitigation (batching, indexing, caching, boundary move)
- how to validate (specific tests/metrics/logs to add)

