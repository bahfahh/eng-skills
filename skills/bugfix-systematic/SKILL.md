---
name: bugfix-systematic
description: Systematic bug investigation and fix workflow for complex, non-obvious bugs — especially when someone has opened a ticket, filed an issue, or needs to formally investigate before coding. Use when the root cause is unclear and spans multiple services/layers, or when the bug is intermittent, hard to reproduce, or involves regressions, data corruption, race conditions, integration failures, or security vulnerabilities. Applies formal bug condition C(X) methodology, property-based exploration tests, and three-phase investigation (bugfix.md → design.md → tasks.md). Trigger whenever someone says "bug ticket", "filed a bug", "opened an issue", "investigating a bug", or describes a complex failure they can't immediately explain. Do NOT use for simple one-liner fixes where the cause is already obvious, new feature requests, refactoring, vague performance complaints without a specific bug, or code quality improvements.
---

# Bugfix Workflow

Systematic bug fixing using formal bug condition methodology with property-based testing validation.

## Overview

Transform bug reports into systematic fixes through three phases:
1. **Bug Investigation** → bugfix.md (bug description, root cause, formal condition C(X))
2. **Fix Design** → design.md (fix strategy, preservation checking, test plan)
3. **Implementation** → tasks.md (exploration test, fix, validation)

## Core Concept: Bug Condition C(X)

Define formal condition C(X) where:
- C(X) = true → bug manifests
- C(X) = false → correct behavior

Format: `C(X) = [condition] causes [symptom]`

Example: `C(quantity) = (quantity == 0)` causes division by zero crash

Fix goal: make C(X) never true.

## Three-Phase Workflow

### Phase 1: Bug Investigation → bugfix.md

Before creating any file, detect the project's document convention by checking for existing patterns:
- `.kiro/specs/` → use `.kiro/specs/{bug-name}/`
- `docs/` or `doc/` → use `docs/{bug-name}/`
- `docs/tickets/`, `docs/tasks/`, `doc/issues/` → follow that structure
- No convention found → ask the user where to place the files, or default to `docs/{bug-name}/`

Create `bugfix.md` under the resolved path with four required sections:

**1. Bug Description**
- Observable symptoms with stack trace
- Minimal reproduction steps
- Expected vs actual behavior
- Impact: severity, affected users, workarounds

**2. Root Cause Analysis**
- Exact code location
- Why it fails under specific conditions
- Violated assumptions
- Related affected code paths

**3. Bug Condition C(X)**
- Formal definition: `C(X) = [condition] causes [symptom]`
- Input/state variables triggering bug
- Boundary cases
- Specific counterexamples where C(X) holds

**4. Reproduction Context**
- Environment: OS, runtime, dependencies
- Required data state
- Timing/concurrency factors

GATE: Present bugfix.md to user, wait for approval before Phase 2.

### Phase 2: Fix Design → design.md

Create `design.md` in the same directory resolved in Phase 1 with four required sections:

**1. Fix Strategy**
- How to make C(X) never true
- Files/functions/logic to change
- Alternatives considered and rejected
- Complexity: simple guard vs architectural

**2. Preservation Checking**
- Functionality that must not break
- Regression risk areas
- Performance impact
- Backward compatibility

**3. Test Strategy**
- Exploration test: property-based test checking C(X) on unfixed code (MUST FAIL)
- Fix validation: same test MUST PASS after fix
- Regression tests
- Edge case coverage

**4. Implementation Approach**
- Code patterns
- Error handling
- Logging/monitoring
- Rollback plan

GATE: Present design.md to user, wait for approval before Phase 3.

### Phase 3: Implementation Tasks → tasks.md

Create `tasks.md` in the same directory resolved in Phase 1:

```markdown
## Task 1: Write Bug Condition Exploration Property Test
- [ ] 1.1 Set up property-based testing framework
- [ ] 1.2 Define property checking NOT C(X)
- [ ] 1.3 Run on unfixed code (MUST FAIL)
- [ ] 1.4 Document counterexamples

## Task 2: Implement Core Fix
- [ ] 2.1 Apply fix strategy
- [ ] 2.2 Add guards preventing C(X)
- [ ] 2.3 Error handling for edges
- [ ] 2.4 Update related paths

## Task 3: Validate Fix
- [ ] 3.1 Run exploration test (MUST PASS)
- [ ] 3.2 Regression suite
- [ ] 3.3 Manual reproduction test
- [ ] 3.4 Performance check

## Task 4: Documentation
- [ ] 4.1 Code comments
- [ ] 4.2 Logging for C(X) near-misses
- [ ] 4.3 User docs if needed
```

GATE: Present tasks.md to user, wait for approval before execution.

## CRITICAL: Exploration Test Execution Protocol

See `references/exploration-test-protocol.md` for complete handling of expected and unexpected test outcomes.

**Expected: Test FAILS on Unfixed Code**
- Test finds counterexamples where C(X) = true
- This is SUCCESS (proves bug exists)
- Document counterexamples
- Proceed to Task 2

**Unexpected: Test PASSES on Unfixed Code**
- STOP immediately
- This means bug doesn't exist, root cause wrong, or test doesn't check C(X)
- Follow user decision protocol in references
- Do NOT proceed to Task 2 until resolved

## File Structure

All three files live in the same directory. Detect the project convention before Phase 1 and keep it consistent throughout:

```
# Kiro projects
.kiro/specs/{bug-name}/
├── bugfix.md
├── design.md
└── tasks.md

# Doc-based projects
docs/{bug-name}/        ← or docs/tickets/, docs/issues/, doc/tasks/, etc.
├── bugfix.md
├── design.md
└── tasks.md
```

Bug name format: `{component}-{symptom}-fix` (kebab-case)

Examples: `checkout-zero-quantity-crash-fix`, `auth-session-timeout-fix`, `api-null-response-fix`

## Property-Based Testing Integration

Transform C(X) to executable property:

**Bug condition**: `C(quantity) = (quantity == 0)` causes crash
**Property**: `for all quantity, NOT (quantity == 0 AND crashes)`

Framework by language:
- JS/TS: fast-check
- Python: Hypothesis
- Java: jqwik
- C#: FsCheck

See `references/property-based-testing.md` for detailed integration patterns.

## Complexity Branching

**Simple** (1-2 files, clear fix):
- Brief design.md
- Focus on test + fix

**Complex** (multi-component, architectural):
- Detailed design.md
- Break into sub-tasks
- May need patterns/refactoring

**Non-deterministic** (race/timing):
- Emphasize reproduction context
- Stress testing in Phase 3
- Concurrency tests

## User Interaction Gates

After each phase:
1. Present document
2. Ask: "Does this look correct?"
3. Wait for approval: "looks good", "continue", "yes"
4. If changes requested: update, re-present, wait again
5. Only proceed after explicit approval

Execution mode ("execute tasks", "run all"):
- Sequential execution
- Update taskStatus: queued → in_progress → completed
- Handle unexpected pass per protocol
- Report progress per task

## Error Handling

See `references/error-handling.md` for complete error handling patterns:
- Missing information
- Multiple root causes
- Fix introduces regression
- Cannot reproduce bug

## Post-Completion

After all tasks pass:
1. Mark spec complete
2. Suggest regression test suite
3. Recommend monitoring for C(X) near-misses
4. Consider architectural changes for prevention

If bug reveals design flaw:
- Complete bugfix first (immediate)
- Create separate feature spec (long-term)
- Reference bugfix in feature requirements

## Orchestrator Instructions

When orchestrating bugfix workflow:
1. Never create documents yourself - delegate to subagent
2. Validate prerequisites before phase transitions
3. Check subagent for `unexpected_pass` status
4. Do NOT auto-proceed to Task 2 on unexpected pass
5. Sequential task execution only
6. Call taskStatus for each transition

## Success Checklist

- [ ] C(X) formally defined
- [ ] Exploration test failed on unfixed (or unexpected handled)
- [ ] Fix implemented per design
- [ ] Exploration test passes on fixed
- [ ] Regression tests pass
- [ ] User confirms resolution

## Anti-Patterns

Do NOT:
- Skip exploration test
- Proceed on unexpected pass without user decision
- Fix without root cause understanding
- Over-engineer simple fixes
- Ignore preservation checking
- Mix bugfix with feature work

## References

For detailed guidance, see:
- `references/exploration-test-protocol.md` - Complete unexpected pass handling
- `references/property-based-testing.md` - PBT integration patterns
- `references/error-handling.md` - Error scenarios and solutions
- `references/phase-templates.md` - Document structure templates
