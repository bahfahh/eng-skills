# Ground Truth and Stability

This skill is used after a feature is "done". The risk is refactoring/encapsulating incomplete or unstable behavior.

## Ground Truth Sources (Strong → Weak)

1) requirements/specs/design intent (project-level, not just a PR description)
2) executable tests (integration/contract/e2e > unit)
3) usage/call graph and real entry points (how the system is actually invoked)
4) code conventions and naming in the codebase
5) PR/commit messages (helpful, but not ground truth)

## Stability Gate: "Ready to Encapsulate"

A module/feature is **ready** only if:
- requirements/spec cover the primary behavior and key error/edge cases, AND
- there is test coverage that pins behavior at a meaningful boundary (integration/contract/e2e preferred)

If either is missing:
- still list the encapsulation candidate,
- mark as `not_ready`,
- and list `tests_to_add` / `spec_gaps` as prerequisites.

## What to Do When Specs Are Missing

Be explicit:
- "No requirements/spec found for this area; grounding relies on existing tests + usage patterns."

Then:
- avoid recommending irreversible encapsulation that would freeze unknown behavior
- recommend first adding integration tests around current behavior (black-box at the feature boundary)

## Usage-Driven Candidate Discovery

Even in whole-project mode, do not "scan everything". Prefer:
- top entry points (routes/controllers/handlers)
- repeated fixed sequences across multiple call sites
- thin utils that require callers to compose business operations
- interfaces that expose internal steps as parameters

