# Deep Module Review (Encapsulation)

Goal: increase long-term stability by encapsulating stable behavior behind clear facades and pinning behavior with integration/contract tests.

## Candidate Signals

Flag a candidate when you observe:
- repeated fixed sequences across call sites (caller composes a business operation from many utilities)
- shallow modules: many helpers, but callers still know the "recipe"
- interface depth: a single business operation requires many parameters that leak internal steps
- wrong test target: utilities have tests, but the facade/business boundary does not

## Encapsulation Output Requirements

For each candidate, provide:
- a proposed `facade` name in domain language
- boundary definition: inputs/outputs using domain terms (avoid storage/infra details)
- a "what moves behind the facade" list
- a "what stays outside" list

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

