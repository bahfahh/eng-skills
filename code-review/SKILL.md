---
name: code-review
description: Unified code review orchestrator. Use whenever reviewing a PR, diff, or code change.Automatically selects review depth based on what changed — from lightweight correctness,checks to full architecture and efficiency analysis. Covers correctness, security, UX,architecture quality, and efficiency. Use this skill for any code review request,pull request review, or merge request review.
---

# Unified Code Review

Orchestrator that reads the diff, classifies what changed, launches 1-3 specialized review agents in parallel, then synthesizes a unified report.

DO NOT modify the code. Only provide review findings.

---

## Step 1: Read and Classify the Diff

Run `git diff` (or `git diff HEAD` if staged changes exist) to get the full diff.

Classify each changed file into one or more categories:

| Category         | Signals                                                                        |
| ---------------- | ------------------------------------------------------------------------------ |
| `ui`             | components/, pages/, hooks/use*, *.tsx with JSX, CSS/SCSS, layouts             |
| `business-logic` | services/, domain/, lib/, handlers, controllers, utils with domain terms       |
| `data-access`    | queries/, repositories/, \*.sql, ORM calls (.from/.query/.include), migrations |
| `auth-security`  | auth, middleware, session, permission, role, tenant, RLS keywords              |
| `shared-util`    | utils/, shared/, lib/, common/, helpers/                                       |
| `config-infra`   | CI/CD, docker, config files, env, package.json, .csproj                        |
| `test`           | _.test._, _.spec._, **tests**/                                                 |

Determine structural signals:

| Signal                | Condition                                                                  |
| --------------------- | -------------------------------------------------------------------------- |
| `new-module`          | New file that defines a class, service, or module                          |
| `architecture-change` | New interfaces, types, abstractions, DI registrations, new patterns        |
| `large-refactor`      | >100 lines changed in `business-logic` files                               |
| `perf-sensitive`      | Loops, data pipelines, middleware, render functions, DB queries, API calls |

Record:

- `complexity`: low (renames/docs/small fixes) | medium (logic/features) | high (arch/core)
- `diff_size`: small (<50 lines) / medium (50-200) / large (>200)
- `touches_auth`: true if any `auth-security` tagged file
- `has_ui`: true if any `ui` tagged file
- `has_dotnet`: true if any .cs or .csproj file
- `has_frontend`: true if any `ui` tagged file

---

## Step 2: Assemble Shared Context

Before launching agents, assemble context ONCE. Read `references/context-assembly.md` for the full strategy.

Minimum context to gather:

1. The full diff
2. Full content of changed files (not just diff hunks)
3. Related tests for changed files (if they exist)
4. Acceptance criteria for this feature area (Given/When/Then, error cases, edge cases)
5. Requirements or specs that mention this feature area (search docs/ by keywords from diff)
6. Callers and callees of changed functions (one layer up and down)

Token budget: keep total under ~25K tokens. Place requirements first, known issues last.

Pass this assembled context to ALL agents launched in Step 3. Agents do NOT re-gather context.

---

## Step 3: Select and Launch Agents

### Agent A: Semantic Review — ALWAYS launch

Read and follow `references/semantic-review.md`.

Pass flags:

- `is_lightweight = true` if `complexity = low`
- `include_ux_gate = true` if `has_ui` AND `complexity != low`
- `include_critic = true` if `complexity = high` OR (`touches_auth` AND `complexity != low`)

### Agent B: Optimizer Review — CONDITIONAL

Launch when `complexity != low` AND ANY of:

- Signal `new-module` detected
- Signal `architecture-change` detected
- Signal `large-refactor` detected
- Any file tagged `shared-util`
- New or changed interface/type/contract files

Pass flags:

- `include_dotnet = true` if `has_dotnet`

Read and follow `references/optimizer-review.md`.

### Agent C: Efficiency Review — CONDITIONAL

Launch when ANY of:

- Any file tagged `data-access`
- Signal `perf-sensitive` detected
- Diff contains SQL/ORM keywords, forEach/map/reduce chains, fetch/axios/HttpClient calls
- New dependency added (package.json or .csproj changed with new package)

Pass flags:

- `include_frontend = true` if `has_frontend`

Read and follow `references/efficiency-review.md`.

### Launch Rules

- Always launch Agent A.
- Launch Agent B and/or Agent C only if their conditions are met.
- Launch all selected agents in PARALLEL using the Agent tool in the same turn.
- Each agent receives: the assembled context + the diff + their flags.

---

## Step 4: Synthesize Final Report

After all agents return, produce a unified report.

### Report Structure

```
## Code Review Summary

**Files reviewed:** N
**Agents activated:** [list with reasons]
**Overall verdict:** accept | accept_with_risk | refactor_recommended | refactor_required

### Blocking Issues (must fix before merge)
[critical findings from all agents]

### Warnings (should fix)
[warning findings from all agents]

### Suggestions (nice to have)
[info findings from all agents]

### Architecture Verdict
[Only if Agent B ran — include YAML verdict block]

### UX Gate
[Only if Agent A ran with UX flag — include UX gate answers]
```

### Deduplication

- If two agents flag the same file:line, keep the finding with higher severity and more specific grounding.
- If findings conflict, include both and mark "needs human judgment."

### Blocking Rule

If Agent B produced `refactor_required`, the overall verdict MUST be `refactor_required` and the summary must say: "BLOCKING: Architecture review requires refactor before merge."

---

## Step 5: Escalate if Needed

Flag for human decision when:

- Any verdict is `refactor_required` but refactor cost is high (large surface area)
- R6 blast radius is `unknown` on a shared utility
- Agents disagree on severity for the same issue
- Code is intentionally complex due to domain constraints (regulatory, compliance)

Present the trade-off clearly and let the human decide.

---

## Reference Files

| File                                    | Purpose                                          | Loaded by              |
| --------------------------------------- | ------------------------------------------------ | ---------------------- |
| `references/semantic-review.md`         | Agent A prompt — correctness, security, UX       | Orchestrator → Agent A |
| `references/optimizer-review.md`        | Agent B prompt — 6-axis architecture review      | Orchestrator → Agent B |
| `references/efficiency-review.md`       | Agent C prompt — waste, duplication, performance | Orchestrator → Agent C |
| `references/context-assembly.md`        | How to select and structure context              | Orchestrator Step 2    |
| `references/judgment-axes.md`           | 6-axis detailed criteria and fail patterns       | Agent B                |
| `references/output-format.md`           | YAML verdict schema with examples                | Agent B                |
| `references/ux-guideline.md`            | UX review checklist and blockers                 | Agent A (conditional)  |
| `references/refactor-guide-frontend.md` | Frontend clarity and simplification rules        | Agent C (conditional)  |
| `references/refactor-guide-dotnet.md`   | C# over-engineering detection rules              | Agent B (conditional)  |
