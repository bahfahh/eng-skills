# Output Format (Markdown-First, Human-Scannable)

Primary output MUST be **Markdown**, optimized for humans to execute a refactor plan with low cognitive load.

Key design goals:
- 1-page summary with tables (what to do next, in what order, and why)
- predictable, short per-module "next actions"
- details available but not forced (use collapsible sections)
- recommendations must be understandable in plain language for non-specialists

Do NOT output "almost-YAML" with separators, duplicated keys, or indentation-sensitive blocks as the primary content.

Optional: include a small machine-readable `summary` YAML block at the top (max ~20 lines). Everything else is Markdown.

## Required Markdown Template

Use this structure:

```markdown
# Module Structure Optimizer Report

## Executive Summary
- input_mode: whole-project | change-focused
- scope: <null | paths/modules/keywords>
- languages_detected: [ts, tsx, dotnet]
- verdict: ok | optimize_recommended | optimize_required

### In plain words
Explain the situation and the decision in simple language:
- What is slowing us down or making changes risky?
- What is the top fix and what will improve for users/devs?
- What is NOT being changed (to keep behavior stable)?

### Top Wins (Do These First)
1. <one-liner win + outcome>
2. <...>
3. <...>

### Top Risks (Watch Outs)
1. <one-liner risk + what can break>
2. <...>
3. <...>

## Action Plan (Table)
| Priority | Recommendation | In plain words | Outcome | Prerequisites (Spec/Tests) | Effort | Risk | Key Files |
|---|---|---|---|---|---|---|---|
| P1 | ... | simple explanation for non-specialists | ... | ... | Low/Med/High | Low/Med/High | `path`, `path` |

## Module Candidate Index
| # | Module | Status | In plain words | Main Signal(s) | Next Action | Owner/Notes |
|---:|---|---|---|---|---|---|
| 1 | ... | ready/not_ready | why this matters + what we do next | D-1, T-1, n_plus_1 | Do X | ... |

## Deep Module Pass (Always)
Summarize the top 3 business flows as a table.

| Flow | In plain words | Recipe Today (Short) | Signals | Candidate | Ready? | Recommended Next Step |
|---|---|---|---|---|---|
| A | <simple explanation of what the flow does and why it is risky> | A→B→C | D-1, T-1 | WeeklyReportDispatcher | not_ready | Add integration test at boundary |

## Module Details (Short)
For each module, keep the default view short: "What/Why/Do next/Tests".
Put heavy details inside a collapsible block.

### 1) <Module Name> (<paths>)
- Status: ready | not_ready (confidence: high/medium/low)
- Why it matters: <one sentence>
- In plain words: <explain without jargon; focus on user/dev impact>
- Do next (max 3 steps):
  1. ...
  2. ...
  3. ...
- Tests to add (if not_ready): <1-3 bullets, name the boundary>
- Decision: <do/don't + why; keep behavior unchanged>

<details>
<summary>Details (Encapsulation / Simplification / Perf/Arch)</summary>

#### Encapsulation (Deep Module)
- Facade name: <DomainFacadeName>
- Boundary: inputs/outputs in domain language
- Move inside / keep outside
- God Service guardrail: name internal split if multiple responsibilities exist

#### Simplifications (max 5)
- `file:line` — what / how / evidence

#### Perf/Arch Risks (max 5)
- `file:line` — pattern / impact / mitigation / validate

</details>

## Open Questions (Blocking Decisions)
- <question> (why it blocks: <...>)
```

## Rules

- "Action Plan (Table)" must be complete even if you include deep details later.
- Each module must include "Do next" steps; do not force readers to parse long prose.
- Each P1/P2 item must include "In plain words" and "Decision" phrased for non-specialists.
- If `Status = not_ready`, list prerequisites as concrete tests/spec gaps.
- Concurrency/batching suggestions MUST call out behavior constraints (errors, idempotency, rate limits) to avoid semantic changes.
- Avoid suggestions that require behavior changes unless requirements/spec explicitly demand it.
