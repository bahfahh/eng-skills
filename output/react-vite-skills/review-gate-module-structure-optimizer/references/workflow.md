# Workflow (Orchestrator)

Goal: produce an actionable **encapsulation + refactor plan** for a completed feature area.

## Step 0: Choose Input Mode

- whole-project: scan the codebase (optionally scoped).
- change-focused: user provides a diff/commit range to narrow the scan.

Always state in the output:
- `input_mode`
- `scope` (paths/modules/keywords, if any)
- `languages_detected`

## Step 1: Detect Languages and Load Stack References

Use extension mapping from `../SKILL.md` to load:
- TS/React rules when `.ts/.tsx/.js/.jsx` present
- C#/.NET rules when `.cs/.csproj` present

If mixed, load both.

## Step 2: Build Ground Truth Context (Project-Level)

Follow `ground-truth.md`.

Minimum:
- requirements/spec relevant to the feature area (if available)
- current tests that cover the feature area (integration/contract/e2e preferred)
- usage/call graph (one layer up/down for key entry points and facades)
- optional: diff/commit range if provided

## Step 3: Run 3 Lenses

1) Deep Module lens (`deep-module.md`)
- identify facade candidates and interface depth issues
- decide whether the feature is stable enough to encapsulate now
- propose encapsulation steps + integration tests to pin behavior
- MUST output a "Deep Module Pass" even if no candidates

2) Simplification lens (`simplification.md`)
- remove meaningless duplication/abstraction
- prioritize clarity and domain language

3) Efficiency/Architecture lens (`efficiency-arch.md`)
- detect N+1, missed concurrency, broad reads, repeated computation, wrong-layer IO
- propose how to validate (tests/metrics), not just opinions

Output constraint:
- Produce a 1-page "Executive Summary" + "Action Plan (Table)" first.
- Put heavy details under "Module Details" and prefer collapsible sections.
- Write recommendations and decisions in plain language so non-specialists can understand the "why" and the "next step".

## Step 4: Decide Single vs Multi-Agent

See the Hybrid Multi-Agent Rule in `../SKILL.md` for escalation conditions.

If multi-agent:
- Reviewer A: Deep Module (reads `deep-module.md`)
- Reviewer B: Simplification (reads `simplification.md`)
- Reviewer C: Efficiency/Architecture (reads `efficiency-arch.md`)
- All reviewers receive the same assembled context.
- Final output must be de-duplicated and synthesized using the Markdown template in `output-format.md`.
