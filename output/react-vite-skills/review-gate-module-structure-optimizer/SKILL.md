---
name: review-gate-module-structure-optimizer
description: Deep module + simplification + efficiency/architecture optimizer for post-feature refactors.Use when a large feature is already implemented and you want to (1) identify deep-module/facade candidates and encapsulate stable behavior with integration/contract tests, (2) simplify/remove redundant or meaningless code and unnecessary abstraction without changing behavior, and (3) detect architecture/performance risks (e.g., N+1 queries, sequential awaits, broad DB reads).
---


# Module Structure Optimizer

This skill produces a **refactor/encapsulation plan** (not a merge gate).
Default behavior: **do not modify repo code**. Output a Markdown report with actionable steps + tests-to-add.

## Inputs (Choose Based on User Request)

- `whole-project` mode: analyze the codebase (optionally scoped to paths/modules/keywords).
- `change-focused` mode: analyze a diff/commit range to narrow the scan, but do not treat the diff as the only source of truth for "ready to encapsulate".

## Reference Loading

Always load on startup:
- `references/workflow.md` — the orchestration entry point; it will tell you when to load other references
- `references/output-format.md` — needed for every output

Load on-demand as directed by workflow.md:
- `references/ground-truth.md` — at Step 2 (building context)
- `references/deep-module.md` — when running the Deep Module lens
- `references/simplification.md` — when running the Simplification lens
- `references/efficiency-arch.md` — when running the Efficiency/Arch lens

Load stack-specific references based on touched/target file extensions:

| Extensions | Load |
|---|---|
| `.ts`, `.tsx`, `.js`, `.jsx` | `references/stacks/ts-frontend.md` |
| `.cs`, `.csproj` | `references/stacks/csharp-dotnet.md` |
| Other languages | No stack reference; apply the three lenses using general patterns |

If mixed languages are present, load both stack references.

## Workflow (High Level)

1. Decide scope (whole-project vs change-focused).
2. Build project-level ground truth context: requirements/spec + tests + usage/call graph + (optional) diff.
3. Generate module candidates from usage patterns (fixed sequences repeated across call sites, interface-depth leaks, missing facade tests).
4. For each candidate, produce:
   - deep-module encapsulation plan (facade boundary + interface language)
   - tests-to-add plan (integration/contract/e2e to pin behavior)
   - simplifications (remove dead/meaningless code, reduce unnecessary abstraction)
   - efficiency/architecture risks + how to validate
5. Output using the Markdown template in `references/output-format.md` (Markdown-first; optional small YAML `summary` only).

Deep Module requirement:
- Always include a "Deep Module Pass" section (even if you find no good candidates).

## Hybrid Multi-Agent Rule

Default: single-pass report (Deep Module + Simplification + Efficiency/Arch sections).
Escalate to multi-agent review (split into 3 reviewers, then synthesize) when any is true:
- mixed languages (TS + C#)
- data-access heavy areas are touched
- suspected N+1 / broad queries / sequential awaits across independent calls
- changes span many modules or large surface area

Note: most production codebases will meet at least one condition above — multi-agent is the expected path, not the exception. It takes longer but produces more reliable results across all three lenses.
