---
name: performance-audit-orchestrator
description: All-in-one performance audit orchestrator for a repository or system. Use when asked to analyze or optimize performance issues such as slow APIs, high latency (p95/p99), slow queries/N+1, memory leaks/OOM, large bundles, slow builds, caching/CDN strategy, Core Web Vitals (LCP/CLS/INP), load testing/capacity planning, or performance monitoring (APM/RUM/tracing). Intelligently decides whether to use 0–3 specialized sub-agents and always merges into a single performance report. Asks before making any code/config edits.
---

# Performance Audit Orchestrator

Deliver exactly one unified performance report. Hide orchestration details from the user.

Hard rules:
- Default to report-only. Ask before any code/config changes.
- Static analysis only — Claude reads code but cannot execute the system, run benchmarks, or measure live latency. All findings from code inspection are labeled `confidence: low` (inferred risk) unless the user supplies actual measured numbers.
- Measurement is the user's responsibility. For every finding, output a concrete measurement command or tool the user can run (e.g., `EXPLAIN ANALYZE`, `time npm run build`, `curl -w '%{time_total}'`, Lighthouse CLI). When the user returns with numbers, revise the affected findings to `confidence: high` and update the remediation priority accordingly.
- Do not invent metrics. If a baseline number is unknown, record it as `"unknown"` in the metrics block and explain how to obtain it.
- Avoid destructive commands. Prefer scope-limited analysis when the user provides a scope.
- Keep context small. Prefer pointing to file paths over pasting large files.

References (load only when needed):
- Always read: `references/routing.md`, `references/output-format.md`, `references/context-assembly.md`
- Read only for selected agents: `references/agent-performance-engineer.md`, `references/agent-frontend-performance.md`, `references/agent-load-testing.md`, `references/agent-test-automation.md`
- Playbooks (load only when routed):
  - `references/playbook-performance-audit.md`
  - `references/playbook-optimize-api-performance.md`
  - `references/playbook-optimize-database-performance.md`
  - `references/playbook-optimize-memory-usage.md`
  - `references/playbook-optimize-bundle-size.md`
  - `references/playbook-optimize-build.md`
  - `references/playbook-implement-caching-strategy.md`
  - `references/playbook-setup-cdn-optimization.md`
  - `references/playbook-add-performance-monitoring.md`
  - `references/playbook-system-behavior-simulator.md`

## Inputs

- Default: analyze the whole repository.
- If the user provides a scope (paths/modules/endpoints/keywords/environments): limit inventory + checks to that scope.
- If the user provides symptoms (e.g., “p95 latency is 2s”, “OOM in prod”): treat them as primary constraints and build the plan around them.
- If no repository is available (e.g., user describes symptoms verbally with no code context): derive aspects from the symptoms alone, set `baseline_status: unknown`, and front-load the report with an instrumentation plan. Label all findings `confidence: low` until measurements are available.

## Workflow

### Step 1 — Inventory & baseline (fast, signal-based)

Use `references/context-assembly.md` to assemble a compact inventory:
- Tech stack + runtime + package managers
- Entry points (frontend routes/pages; backend endpoints; background jobs)
- Data layer (DB/ORM/caches/queues)
- Build pipeline (bundlers, CI, build scripts)
- Existing performance monitoring (APM/RUM/logging/tracing)

Collect baseline metrics if available; otherwise produce an instrumentation plan (and record the missing data in `limits`).

### Step 2 — Decide routing (0–3 agents)

Use `references/routing.md` to compute:
- detected aspects
- `workload` (low/medium/high)
- `agent_count` (0–3)
- which agents to run
- which playbooks to load (only those needed for the routed aspects)

If `agent_count = 0`, perform the same checks yourself and still output using the unified report template.

### Step 3 — Assemble shared context (once)

Prepare a minimal shared context for all selected agents:
- scope rules (included/excluded)
- symptoms + constraints (env, SLOs, deadlines)
- baseline metrics (or explicit gaps + instrumentation plan)
- “hot files” list (paths only; no large pastes)
- the aspects detected + routing decision

### Step 4 — Run selected agents in parallel

For each selected agent:
- Provide the shared context.
- Tell the agent to follow its reference file.
- Require YAML-only output using the schema in `references/output-format.md`.

### Step 5 — Merge into one report

Use `references/output-format.md` to:
- merge/dedupe findings
- normalize metrics naming
- produce a single prioritized remediation plan

If the user wants fixes implemented:
- Ask permission.
- Propose a staged change list with a verification plan (benchmarks + regression guardrails) before editing anything.

## Output

Return a single unified report (no separate agent transcripts) with:
- Summary (scope, baseline, activated agents with routing reasons)
- Baseline Metrics (measured vs unknown)
- Findings (prioritized; grounded with `file:line` when possible)
- Remediation Plan (ordered; each step includes “how to verify”)
- Implementation mode (only if the user approves edits)
- Limits & Confidence
