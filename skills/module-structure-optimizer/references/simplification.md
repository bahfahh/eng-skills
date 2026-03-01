# Simplification (Not "Shared")

Goal: remove complexity that has no semantic value.
This is not a "create shared utilities" exercise.

## What to Remove or Simplify

- dead code (unused functions, unreachable branches, commented-out blocks)
- meaningless abstractions (interface-per-class with single impl, wrappers that add no behavior)
- duplicated logic that is accidental (copy/paste drift), not a stable shared domain rule
- overly compact code that reduces clarity (dense one-liners, nested ternaries)
- confusing naming that hides domain rules

## What NOT to Do

- do not extract "shared" helpers unless a stable domain rule is named and reused
- do not introduce new abstraction layers for hypothetical flexibility
- do not refactor unrelated areas outside the selected scope

## Output Requirements

For each simplification:
- location
- why it is meaningless/duplicative
- what to replace/remove
- what evidence protects behavior (tests, facade boundary, existing invariants)

