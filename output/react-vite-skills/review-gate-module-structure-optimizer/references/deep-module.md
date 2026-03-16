# Deep Module Review (Encapsulation)

Goal: increase long-term stability by encapsulating stable behavior behind clear facades and pinning behavior with integration/contract tests.

## Problem to Avoid

This skill can accidentally degrade into "debug/perf lint" (N+1, Promise.all, etc.) and produce shallow encapsulation advice.
To prevent that, a Deep Module pass MUST be executed explicitly and reported, even when no good candidates are found.

## Candidate Signals

Flag a candidate when you observe:
- repeated fixed sequences across call sites (caller composes a business operation from many utilities)
- shallow modules: many helpers, but callers still know the "recipe"
- interface depth: a single business operation requires many parameters that leak internal steps
- wrong test target: utilities have tests, but the facade/business boundary does not

## Required Deliverables (Always)

For the selected scope, you MUST output a "Deep Module Pass" section that includes:
- the top 3 business operations/flows in scope (from requirements/spec + usage)
- for each flow: the current call-site "recipe" (a short ordered list of key functions/modules involved)
- which D/T/N signals are present (use names, not just vibes):
  - D-1 module depth: business intent assembled in caller
  - D-2/D-3 shallow module: repeated fixed sequences / tool-chain composition
  - D-4 interface depth: facade leaks internal steps as parameters
  - T-1 wrong test target: utils tested but facade boundary not pinned
  - T-3 test coupling: tests assert internal call paths
  - N-1/N-2/N-3 naming/interface not in domain language

Then:
- propose 0..N encapsulation candidates
- if 0 candidates: explain why (what signals were checked and did not appear)

## Encapsulation Output Requirements

For each candidate, provide:
- a proposed `facade` name in domain language
- boundary definition: inputs/outputs using domain terms (avoid storage/infra details)
- a "what moves behind the facade" list
- a "what stays outside" list

Also include a "God Service guardrail":
- a facade is a boundary, not a single mega-file or a grab-bag service
- if the plan would move multiple unrelated responsibilities inside one service, split internal components
  (e.g., Composer/Sender/Scheduler/Storage) and keep the facade as a thin entry point

## Test Strategy (Encapsulation Gate)

If ready-to-encapsulate gate passes:
- propose integration/contract tests that pin the facade behavior

If not ready:
- propose the minimal integration tests needed to pin current behavior before encapsulating

Avoid tests that assert internal call paths (mock coupling).
Prefer black-box tests at the meaningful boundary.

## Anti-Patterns

- facade name describes technical steps instead of business intent
- interface requires callers to pass internal details ("leaky facade")
- encapsulation proposed without any plan to pin behavior with tests
- proposing a single "WeeklyXService" that absorbs CRUD + scheduling + sending + storage without an internal split
