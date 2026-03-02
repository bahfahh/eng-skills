---
name: security-audit-orchestrator
description: All-in-one security audit orchestrator for a repository. Use when asked to do a security audit/review, secrets scan, dependency or supply-chain risk review, CI/CD security review, auth/API security assessment, or pre-production security check. Intelligently decides whether to use 0–3 specialized sub-agents and always merges into a single security report. Does not modify code.
---

# Security Audit Orchestrator

Deliver exactly one unified security audit report. Hide orchestration details from the user.

Hard rules:
- Do not modify code.
- Do not run destructive commands or exploit/attack anything.
- Do not print secrets. If found, redact values and focus on rotation/revocation steps.

References (load only when needed):
- Always read: `references/routing.md`, `references/output-format.md`
- Read only for selected agents: `references/agent-secrets.md`, `references/agent-supplychain.md`, `references/agent-appsec.md`

## Inputs

- Default: audit the whole repository.
- If the user provides a scope (paths/modules/keywords): limit inventory + checks to that scope.

## Workflow

### Step 1 — Inventory (fast, signal-based)

Build a short inventory summary (bullets) with:
- Tech: languages/frameworks/runtime/package managers
- Entry points: API routes/controllers, auth/middleware, DB access layers
- Supply chain: package manifests + lockfiles, GitHub Actions/workflows, Docker/IaC
- Secrets risk: `.env*`, config files, key/token patterns, CI secret usage
- AI/LLM signals: OpenAI/Anthropic/LangChain/LlamaIndex/vector DB/agent frameworks

Use `references/routing.md` to compute:
- `aspects_detected`
- `risk_level` (low/medium/high)
- `workload` (low/medium/high)

### Step 2 — Decide orchestration (0–3 agents)

Use `references/routing.md` to decide:
- `agent_count` in {0,1,2,3}
- which agents to run (Secrets / SupplyChain / AppSec)

If `agent_count = 0`, perform a minimal audit yourself using the same focus areas and output schema, then produce the unified report.

### Step 3 — Assemble shared context (once)

Prepare a compact shared context for all selected agents:
- the inventory summary + computed scores
- the exact scope rules (what was included/excluded)
- a short list of “hot files” to read (only the most relevant files per aspect)

Keep context small; prefer pointing agents at file paths over pasting large files.

### Step 4 — Run selected agents in parallel

Spawn selected agents in parallel. For each agent:
- Include the shared context.
- Tell the agent to follow its reference file and output YAML findings only.

### Step 5 — Merge into one report

Use `references/output-format.md` to:
- dedupe and merge findings
- compute a single overall verdict
- output one unified report with a prioritized remediation plan

## Output

Return a single report (no separate agent transcripts) with:
- activated agents (or none) + routing reasons
- findings by severity (with `file:line` evidence when possible)
- immediate actions (if critical)
- remediation plan (ordered)
- limits + confidence

