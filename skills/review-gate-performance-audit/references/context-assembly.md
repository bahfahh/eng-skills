# Context assembly (共享上下文組裝)

Assemble context once, then pass it to all selected agents. Keep it compact and path-oriented.

## Required inputs to capture (from user)

- `symptoms`: what is slow / what regressed / when it happens
- `where`: frontend | api | database | build | memory | caching/cdn | monitoring | load testing
- `environment`: local | dev | staging | prod
- `SLO/SLA`: target latency/throughput/error budget (if available)
- `constraints`: deadline, risk tolerance, whether edits are allowed

If any of these are unknown, record them under `limits` and propose the minimal next measurement to unblock.

## Baseline metrics (minimal set)

Collect only what the user has already provided or what exists in the repo (logs, CI output, monitoring dashboards). Do not invent numbers.

Claude cannot execute the system. Metrics come from one of three sources:
1. **User-provided** — the user pastes numbers, screenshots, or log excerpts in the conversation.
2. **Repo-embedded** — CI timing logs, bundle-analyzer output, lighthouse reports, or recorded benchmarks committed to the repo.
3. **Inferred (mark as unknown)** — if no source exists, record `"unknown"` and output a `measurement_command` so the user can obtain it.

Metrics to collect:
- Latency: p50/p95/p99 (ms) for critical endpoints/pages
- Throughput: RPS/TPS and concurrency
- Errors: rate %, timeouts
- Resource: CPU %, RSS/heap MB, GC pauses (if available)
- Frontend: LCP/CLS/INP (if applicable), bundle sizes (KB)
- Build: clean build time (s), incremental build time (s)

## Repo hot files selection (paths only)

Pick 5–20 hot files per detected aspect:

- Frontend: entry points, largest routes, rendering hotspots, bundler config, image/font handling
- API: routing/middleware, serialization, outbound calls, rate limiting, caching middleware
- DB: query layer/repositories, schema/migrations, index definitions, N+1 call sites
- Build: build configs, CI workflow build steps, caching config, bundler plugins
- Memory: long-lived caches, event listeners, background jobs, streaming/buffer usage
- Caching/CDN: cache headers config, CDN config, invalidation hooks, Redis client setup
- Monitoring: tracing init, metrics exporters, logging config, web vitals/RUM scripts
- Load testing: existing k6/locust scripts, infra autoscaling, capacity docs

Prefer listing file paths + a one-line “why hot” note over pasting file contents.

## Output package (what agents receive)

Provide to each agent:
- scope rules (included/excluded)
- symptoms + environment + constraints
- baseline metrics (or measurement gaps + plan)
- hot files list
- routed aspects + selected playbooks
