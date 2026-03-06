# Routing (智能派工規則)

Use these rules to decide 0–3 sub-agents. Always produce a single merged report.

## 1) Detect aspects

Detect aspects from the user prompt and repo signals (prefer scope-limited search).

Aspects (use these fixed ids):

### `frontend_perf`
Signals (any):
- `next.config.js`, `vite.config.*`, `webpack.config.*`, `rollup.config.*`
- `app/`, `pages/`, `src/components/`, `components/`, `routes/` with JSX/TSX
- Web Vitals keywords: `LCP`, `CLS`, `INP`, `FID`, `TTFB`, `Lighthouse`
- bundle keywords: `chunk`, `splitChunks`, `dynamic import`, `analyze`, `bundle-analyzer`
- images/fonts: `next/image`, `Image`, `font-display`, `preload`, `prefetch`

### `api_backend_perf`
Signals (any):
- API routes/controllers/handlers: `api/`, `routes/`, `controllers/`, `handlers/`
- middleware chains, validation/serialization, compression
- keywords: `latency`, `p95`, `p99`, `throughput`, `RPS`, `timeout`, `retry`
- outbound calls: `fetch`, `axios`, `HttpClient`, `grpc`, `GraphQL`

### `database_perf`
Signals (any):
- SQL/ORM usage: `SELECT`, `JOIN`, `EXPLAIN`, `prisma`, `typeorm`, `sequelize`, `knex`, `mongoose`
- slow query / index / migration keywords: `index`, `migration`, `N+1`, `query plan`, `connection pool`

### `build_bundle_perf`
Signals (any):
- build scripts: `npm run build`, `pnpm build`, `yarn build`, `turbo`, `nx`, `webpack`, `vite`
- CI build steps: `.github/workflows/*` mentioning build/cache/artifacts
- bundle tooling: `terser`, `esbuild`, `swc`, `babel`, `tsc --build`

### `memory_perf`
Signals (any):
- `OOM`, `out of memory`, heap, GC, `--max-old-space-size`, leak keywords
- long-lived caches, global maps, event listener patterns

### `caching_cdn`
Signals (any):
- `Cache-Control`, `ETag`, `Last-Modified`, `stale-while-revalidate`
- Redis/Memcached/CDN keywords: `redis`, `memcached`, `cloudflare`, `fastly`, `cloudfront`

### `monitoring_observability`
Signals (any):
- APM/RUM/tracing keywords: `sentry`, `datadog`, `newrelic`, `opentelemetry`, `prometheus`, `grafana`
- web vitals libraries: `web-vitals`, `next/script` RUM snippets

### `load_testing_capacity`
Signals (any):
- load tools: `k6`, `jmeter`, `locust`, `artillery`, `gatling`
- capacity planning keywords: `capacity`, `stress`, `soak`, `baseline`, `breaking point`
- system modeling keywords: `queueing`, `Little's Law`, `simulation`

## 2) Score workload

### `workload`
- `high` if: multi-service/multi-language repo OR the user asks for “full audit + plan + validation/load test”
- `medium` if: single stack but spans at least two major aspects (e.g., API+DB, frontend+bundle, caching+CDN)
- `low` if: very small repo or narrow scope with no strong signals

## 3) Decide agent count (0–3)

Let:
- `surface_count` = number of detected aspects among {frontend_perf, api_backend_perf, database_perf, build_bundle_perf, memory_perf, caching_cdn, monitoring_observability, load_testing_capacity}
- `primary_count` = number of detected aspects among {frontend_perf, api_backend_perf, database_perf, memory_perf}

Decision:
- `0 agents` if `workload = low` AND `surface_count <= 1`
- `1 agent` if `surface_count = 1` AND `workload != high`
- `2 agents` if `surface_count = 2` OR `workload = high`
- `3 agents` if `surface_count >= 3` OR (`load_testing_capacity` detected AND `primary_count >= 1`)

Cap at 3 agents.

## 4) Choose which agents

Agents (fixed ids used in reports and evals):
- `perf_engineer`: always include if any of {api_backend_perf, database_perf, memory_perf, caching_cdn, monitoring_observability} detected
- `frontend_perf`: include if `frontend_perf` detected
- `load_testing`: include if `load_testing_capacity` detected
- `test_automation`: include only if the user explicitly asks for performance regression tests/CI automation

If you must drop one due to the 3-agent cap:
1) keep `perf_engineer`
2) keep `frontend_perf` if frontend aspects exist
3) keep `load_testing` if load testing/capacity is explicitly requested
If `test_automation` would exceed the cap, do not run it; instead include a test plan in the unified report.

## 5) Map aspects to playbooks (load selectively)

- `frontend_perf` → `playbook-optimize-bundle-size.md`
- `api_backend_perf` → `playbook-optimize-api-performance.md`
- `database_perf` → `playbook-optimize-database-performance.md`
- `build_bundle_perf` → `playbook-optimize-build.md` and/or `playbook-optimize-bundle-size.md`
- `memory_perf` → `playbook-optimize-memory-usage.md`
- `caching_cdn` → `playbook-implement-caching-strategy.md` and/or `playbook-setup-cdn-optimization.md`
- `monitoring_observability` → `playbook-add-performance-monitoring.md`
- `load_testing_capacity` → `playbook-system-behavior-simulator.md` (and include load-test plan in report)
