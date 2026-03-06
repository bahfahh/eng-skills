# Output Format (單一交付物)

All sub-agents output YAML using the schema below. The orchestrator merges everything into one report.

## Sub-agent YAML schema

```yaml
findings:
  - severity: critical | high | medium | low | info
    area: frontend | api | database | build | memory | caching | cdn | monitoring | load_testing
    location: "path/to/file:line" # if no line, use "path:(section description)"
    title: "short risk/perf statement"
    evidence: "repo evidence + observed metric (or explicit missing instrumentation note)"
    impact: "user impact + system impact"
    recommendation: "concrete fix steps; include measurement/verification plan"
    measurement_command: "exact command or tool to measure this finding, e.g. EXPLAIN ANALYZE <query>, time npm run build, lighthouse https://... --output json"
    confidence: high | medium | low  # low = inferred from static analysis; high = confirmed with measured numbers
metrics:
  baseline:
    - name: "p95_latency_ms" | "p99_latency_ms" | "bundle_kb" | "build_time_s" | "rss_mb" | "heap_used_mb"
      value: "number or 'unknown'"
      method: "how measured (command/tool/source)"
  target:
    - name: "p95_latency_ms" | "bundle_kb" | "build_time_s" | "rss_mb"
      value: "number or 'unknown'"
      method: "how to validate target"
notes:
  limits:
    - "what was not checked / missing tools / missing repro"
```

## Orchestrator merge rules

- Dedupe: same `location + title` → keep higher `severity`; merge the most specific `recommendation`.
- Conflicts: keep both recommendations and mark `needs human judgment`.
- Evidence: prefer repo-grounded evidence; if only hypothesis is possible, set `confidence: low` and add an explicit measurement plan.
- Never output separate agent transcripts; merge findings only.

## Unified report template

```md
# Performance Report

## Summary
- scope:
- baseline_status: measured | partial | unknown
- activated_agents: [none | perf_engineer | frontend_perf | load_testing | test_automation] (with routing reasons)
- top_issues:
  - ...

## Baseline Metrics
- measured:
  - ...
- missing (instrumentation needed):
  - ...

## Findings
### Critical
### High
### Medium
### Low / Info

## Remediation Plan (ordered)
1. ...

## Implementation Mode (only if approved)
- proposed changes:
- verification plan:
- rollback plan:

## Limits & Confidence
- ...
```
