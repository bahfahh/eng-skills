# eng-skills

A collection of Claude Code skills for software engineering — covering planning, architecture, development checklists, code review, auditing, and bug fixing.

---

## Skills by Category

### `frontend-`

| Skill | Description |
|-------|-------------|
| `frontend-checklist` | Frontend quality checklist (2 modes): coding/plan mode adds missing UX concerns to todolist; review mode audits completed code. Covers interaction states, write-path patterns, auth/session safety, responsive design. |
| `next-checklist` | Next.js quality checklist (3 modes): coding/plan, review, and upgrade. Covers App Router, auth/session safety, data fetching, bundle optimization, Next.js 15 async API changes. |
| `next-api-checklist` | Next.js Route Handler / API checklist (2 modes): coding/plan and review. Covers Route Handler structure, Service Layer separation, error handling, React Query/SWR integration, auth/multi-tenant security. |

### `architecture-`

| Skill | Description |
|-------|-------------|
| `architecture-design-patterns` | Context-aware design pattern selector. Recommends AI-friendly patterns and steers away from over-engineering. |
| `architecture-ai-native-design` | Language-agnostic AI-native architecture principles for module boundaries, layered responsibilities, stable interfaces, and test strategy. |

### `codereview-`

| Skill | Description |
|-------|-------------|
| `codereview-unified-pro` | PR/diff review coordinator. Auto-selects review depth and runs parallel checks across correctness, security, architecture, and efficiency. |

### `review-gate-`

| Skill | Description |
|-------|-------------|
| `review-gate-security-audit` | All-in-one security audit orchestrator. Launches 0–3 sub-agents (secrets / supply-chain / appsec) and merges into a single report. Read-only. |
| `review-gate-performance-audit` | All-in-one performance audit orchestrator. Launches 0–4 sub-agents and merges into a single performance report. Read-only. |
| `review-gate-module-structure-optimizer` | Post-feature optimization: finds encapsulation candidates, removes redundant abstractions, detects N+1 and sequential-await risks. |
| `review-gate-deep-module-refactor-advisor` | Audits mature codebases for shallow modules and high coupling. Produces facade encapsulation recommendations without touching code. |

### `plan-mode-`

| Skill | Description |
|-------|-------------|
| `plan-mode-pro` | Structured planning workflow. Produces requirements.md, plan.md, todolist.md, acceptance.md, and tests.md in sequence. |

### `db-`

| Skill | Description |
|-------|-------------|
| `db-supabase-checklist` | Supabase DB checklist (2 modes): coding/plan mode adds DB concerns to todolist; review mode audits migrations, RLS policies, and data integrity. |

### `backend-`

| Skill | Description |
|-------|-------------|
| `backend-dotnet-checklist` | ASP.NET Core / C# checklist (2 modes): coding/plan mode enforces 8 Core Guardrails (DI scope, async/await, EF Core NoTracking, IOptions, etc.); review mode audits .NET code. Includes C# standards, EF Core patterns, and concurrency references. |

### `bugfix-`

| Skill | Description |
|-------|-------------|
| `bugfix-systematic` | Systematic bug investigation and fix workflow for complex, non-obvious bugs — especially when a ticket has been filed or the root cause spans multiple services. Uses formal bug condition C(X), property-based exploration tests, and a three-phase flow (bugfix.md → design.md → tasks.md). |

---

## Shared Skills (used across all stacks)

| Skill | Next.js | React | .NET |
|-------|:-------:|:-----:|:----:|
| `plan-mode-pro` | ✓ | ✓ | ✓ |
| `architecture-ai-native-design` | ✓ | ✓ | ✓ |
| `architecture-design-patterns` | ✓ | ✓ | ✓ |
| `codereview-unified-pro` | ✓ | ✓ | ✓ |
| `review-gate-security-audit` | ✓ | ✓ | ✓ |
| `review-gate-performance-audit` | ✓ | ✓ | ✓ |
| `review-gate-module-structure-optimizer` | ✓ | ✓ | ✓ |
| `review-gate-deep-module-refactor-advisor` | ✓ | ✓ | ✓ |
| `bugfix-systematic` | ✓ | ✓ | ✓ |
