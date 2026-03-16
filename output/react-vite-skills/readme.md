# React Vite Frontend Skills Pack

This folder contains all skills needed for a **Vite + React** frontend project.
Copy the entire folder contents into your project's `.kiro/skills/` directory.

---

## Skills Usage by Development Phase

Use the skills in order as you progress through the development lifecycle.

### Phase 1 — Planning
**When**: Starting a new feature or before writing any code.

| Skill | How to trigger |
|-------|---------------|
| `plan-mode-pro` | "Help me plan [feature]" / "Let's plan this task" |

Produces: `requirements.md`, `plan.md`, `todolist.md`, `acceptance.md`, `tests.md`

---

### Phase 2 — Architecture Design
**When**: Designing module boundaries, choosing patterns, or planning a refactor.

| Skill | How to trigger |
|-------|---------------|
| `architecture-design-patterns` | "Which design pattern should I use for..." / "Help me choose a pattern" |
| `architecture-ai-native-design` | "Help me design the module structure" / "Review my architecture" |

Use **both** when starting a non-trivial feature from scratch.

---

### Phase 3 — Before Writing Code
**When**: About to implement a feature. Run these to add missing concerns to the todolist.

| Skill | How to trigger |
|-------|---------------|
| `frontend-vite-react-checklist` | "I'm about to build [Vite/React feature]" / triggered automatically for React tasks |
| `frontend-checklist` | "Review my frontend plan" / triggered for any UI task |

> Load `frontend-vite-react-checklist` first — it covers Vite+React specific traps (routing, TanStack Query, env vars, alias config).
> Load `frontend-checklist` second — it covers universal UX concerns (loading states, write paths, mobile, accessibility).

---

### Phase 4 — Code Review
**When**: A feature is complete, or reviewing a PR.

| Skill | How to trigger |
|-------|---------------|
| `codereview-unified-pro` | "Review this PR" / "Review my code" / "Code review" |
| `frontend-vite-react-checklist` (review mode) | "Review my Vite React code" / triggered alongside codereview |
| `frontend-checklist` (review mode) | "Check my frontend UX" / triggered for UI-heavy PRs |

> `codereview-unified-pro` is the main entry point — it automatically decides which sub-reviews to run in parallel.

---

### Phase 5 — Deep Audits
**When**: Feature is shipped. Run periodically or before a major release.

| Skill | How to trigger | What it does |
|-------|---------------|--------------|
| `review-gate-security-audit` | "Run a security audit" | Secrets, supply chain, AppSec — produces a single report |
| `review-gate-performance-audit` | "Run a performance audit" | Frontend perf, load testing, test automation — single report |
| `review-gate-module-structure-optimizer` | "Optimize module structure" / "Find N+1 and bad abstractions" | Post-feature cleanup: redundant abstractions, N+1, sequential awaits |
| `review-gate-deep-module-refactor-advisor` | "Analyze coupling" / "Find tightly coupled modules" | Shallow modules, high-coupling analysis — refactor plan report |

> These are **read-only audit skills** — they produce reports and do NOT modify code.

---

### Phase 6 — Bug Fixing
**When**: A bug has a ticket, spans multiple services, is hard to reproduce, or the root cause is unknown.

| Skill | How to trigger |
|-------|---------------|
| `bugfix-systematic` | "Fix this bug systematically" / "Investigate this regression" / "Debug this race condition" |

Three-phase flow: `bugfix.md` → `design.md` → `tasks.md`

---

## Quick Reference — Trigger Cheatsheet

| Situation | Skill to use |
|-----------|-------------|
| Starting a new task | `plan-mode-pro` |
| Designing structure or picking a pattern | `architecture-design-patterns` + `architecture-ai-native-design` |
| About to code a React/Vite feature | `frontend-vite-react-checklist` (coding mode) |
| About to code any UI feature | `frontend-checklist` (coding mode) |
| PR is ready for review | `codereview-unified-pro` |
| Pre-release security check | `review-gate-security-audit` |
| Pre-release performance check | `review-gate-performance-audit` |
| Post-feature code cleanup | `review-gate-module-structure-optimizer` |
| Something is broken and unclear why | `bugfix-systematic` |

---

## Official Skills (install separately)

These are **not included** in this folder — install from Vercel official source:

| Official Skill Name | Install as |
|--------------------|-----------|
| `vercel-react-best-practices` | `react-official-best-practices` |
| `web-design-guidelines` | `web-official-design-guidelines` |

Use `react-official-best-practices` during development (57 React performance rules).
Use `web-official-design-guidelines` during code review for UI/UX design standards.
