# Error Handling Patterns

Complete error handling strategies for bugfix workflow edge cases.

## Overview

This document covers error scenarios that can occur during bugfix workflow and how to handle them systematically.

---

## Missing Information

### Scenario

Bug report lacks critical details for root cause analysis.

### Symptoms

- No reproduction steps
- Missing stack traces
- Unclear expected behavior
- Unknown environment details

### Handling Strategy

**Ask specific questions** rather than proceeding with incomplete information:

```
Required information checklist:
□ Exact steps to reproduce
□ Stack trace or error message
□ Expected vs actual behavior
□ Environment (OS, runtime version, dependencies)
□ Data state when bug occurs
□ Frequency (always, sometimes, rarely)
```

**Question templates:**

- "What are the exact steps to reproduce this bug?"
- "Can you provide the stack trace or error message?"
- "What did you expect to happen vs what actually happened?"
- "What environment are you running in (OS, Node version, etc.)?"
- "What data was in the system when the bug occurred?"
- "Does this happen every time or intermittently?"

### When to Proceed

Only proceed to Phase 1 (bugfix.md) when you have:
- Minimal reproduction steps
- Clear symptom description
- At least one concrete example of bug occurring

### When to Defer

If user cannot provide reproduction steps:
- Document what is known
- Mark as "needs reproduction"
- Wait for more information
- Do NOT create bugfix.md yet

---

## Multiple Root Causes

### Scenario

Bug has multiple contributing factors, not a single root cause.

### Symptoms

- Multiple code paths lead to same symptom
- Bug requires combination of conditions
- Fixing one cause doesn't resolve bug completely

### Handling Strategy

**Define compound bug condition:**

```
C(X) = C1(X) AND C2(X) AND ... AND Cn(X)
```

**Example:**

```
Bug: User data corruption
C(X) = (concurrent_writes == true) AND (validation_disabled == true)

Not just one cause, but combination of:
- C1: Concurrent writes without locking
- C2: Validation checks disabled
```

### Phase 1: bugfix.md Structure

```markdown
## Bug Condition C(X)

C(X) = C1(X) AND C2(X)

Where:
- C1(user, request) = (user.role == 'admin') causes permission bypass
- C2(request) = (request.validation == false) causes data corruption

Bug manifests when BOTH conditions are true simultaneously.
```

### Phase 2: design.md Strategy

Address each condition:

```markdown
## Fix Strategy

Fix C1: Add permission check even for admin role
Fix C2: Always enable validation regardless of request flag

Both fixes required to prevent bug.
```

### Phase 3: tasks.md Structure

```markdown
## Task 1: Write Exploration Tests

- [ ] 1.1 Test for C1 (admin permission bypass)
- [ ] 1.2 Test for C2 (validation disabled)
- [ ] 1.3 Test for C1 AND C2 (full bug condition)

## Task 2: Implement Fixes

- [ ] 2.1 Fix C1 (permission check)
- [ ] 2.2 Fix C2 (validation enforcement)
```

### Separate vs Combined Tests

**Option 1: Separate tests for each condition**
- Test C1 independently
- Test C2 independently
- Test C1 AND C2 together

**Option 2: Single combined test**
- Only test C1 AND C2 together
- Simpler but less diagnostic

Recommend Option 1 for complex bugs.

---

## Fix Introduces Regression

### Scenario

Fix resolves original bug but breaks existing functionality.

### Symptoms

- Task 3 regression tests fail
- New errors appear in unrelated code
- Performance degrades significantly
- Different bug manifests

### Handling Strategy

**Step 1: Document the regression**

```markdown
## Regression Detected

Original bug: quantity=0 crash
Fix applied: Added guard `if (quantity <= 0) return error`
Regression: Negative quantities now rejected (breaking change)
Impact: Existing code expects negative quantities for refunds
```

**Step 2: Revise fix strategy in design.md**

```markdown
## Fix Strategy (Revised)

Original approach: Reject quantity <= 0
Problem: Breaks refund functionality

Revised approach: 
- Reject quantity == 0 (division by zero)
- Allow quantity < 0 (refunds)
- Add separate validation for refund flow
```

**Step 3: Update tasks.md**

```markdown
## Task 2: Implement Core Fix (Revised)

- [ ] 2.1 Add guard for quantity == 0 only
- [ ] 2.2 Preserve negative quantity handling
- [ ] 2.3 Add refund-specific validation
- [ ] 2.4 Update related code paths

## Task 3: Validate Fix (Updated)

- [ ] 3.1 Run exploration test (MUST PASS)
- [ ] 3.2 Run regression suite (MUST PASS)
- [ ] 3.3 Test refund flow specifically
- [ ] 3.4 Performance check
```

**Step 4: Re-run full test suite**

After revising fix:
- Re-run exploration test
- Re-run all regression tests
- Add new regression test for the issue found

### Prevention

**In Phase 2 (design.md), always include:**

```markdown
## Preservation Checking

Functionality that must not break:
- Refund processing (uses negative quantities)
- Bulk order discounts (uses quantity multipliers)
- Inventory adjustments (uses quantity deltas)

Regression risk areas:
- Any code path using quantity parameter
- Related calculations (price, tax, shipping)
```

---

## Cannot Reproduce Bug

### Scenario

Bug cannot be reproduced in development environment.

### Symptoms

- Reproduction steps don't trigger bug
- Bug only occurs in production
- Bug is intermittent or timing-dependent
- Bug requires specific data state

### Handling Strategy

**Step 1: Document reproduction attempts**

```markdown
## Reproduction Attempts

Attempt 1: Followed steps 1-5, no crash observed
Attempt 2: Tried with production data snapshot, no crash
Attempt 3: Tested on staging environment, no crash

Hypothesis: Bug may be environment-specific or timing-dependent
```

**Step 2: Gather more context**

Ask user:
- "Does this happen in all environments or just production?"
- "Is there anything unique about the production environment?"
- "Can you provide production logs around the time of the bug?"
- "Is there a specific time of day or load level when this occurs?"
- "Can you provide a database snapshot from when bug occurred?"

**Step 3: Consider environment factors**

```markdown
## Environment Factors to Investigate

- Load/concurrency (production has higher traffic)
- Data volume (production has more records)
- Configuration differences (environment variables)
- Dependency versions (production may have different versions)
- Timing/race conditions (production has different timing)
- External services (production uses real APIs)
```

**Step 4: Decision point**

**Option A: Defer until reproducible**
```markdown
## Status: Deferred

Cannot reproduce bug in development.
Waiting for:
- Production logs
- Database snapshot
- Environment configuration details

Will resume when reproduction is possible.
```

**Option B: Proceed with hypothesis**
```markdown
## Status: Proceeding with Hypothesis

Cannot reproduce directly, but evidence suggests:
- Root cause: Race condition in concurrent writes
- Bug condition: C(threads, timing) = (threads > 1) AND (timing overlap)

Will implement fix based on code analysis and production logs.
Note: Validation will be limited without reproduction.
```

### When to Defer

Defer if:
- No clear hypothesis about root cause
- No evidence from logs or monitoring
- User cannot provide more context
- Bug is critical and needs careful analysis

### When to Proceed

Proceed if:
- Strong hypothesis from code analysis
- Clear evidence from production logs
- Similar bugs have been fixed before
- Fix is low-risk (e.g., adding null check)

---

## Non-Deterministic Bugs

### Scenario

Bug occurs intermittently, not consistently.

### Types

1. **Race conditions**: Timing-dependent
2. **Concurrency bugs**: Thread interaction
3. **External dependencies**: API failures, network issues
4. **Resource exhaustion**: Memory, connections, file handles
5. **Floating point**: Precision errors

### Handling Strategy

**Phase 1: bugfix.md - Emphasize reproduction context**

```markdown
## Bug Description

Symptom: Data corruption in user records
Frequency: ~5% of requests
Pattern: Only under high load (>100 concurrent users)

## Reproduction Context

CRITICAL: Bug is non-deterministic

Environment:
- Production only (not reproducible in dev)
- High concurrency required (>100 threads)
- Specific timing window (~10ms)

Reproduction approach:
- Stress test with 200 concurrent requests
- Monitor for data inconsistencies
- May require multiple runs to trigger
```

**Phase 2: design.md - Stress testing strategy**

```markdown
## Test Strategy

Exploration test: Stress test with concurrency
- Run 1000 concurrent operations
- Check for data inconsistencies
- Repeat 10 times to catch intermittent failures

Fix validation:
- Same stress test must pass consistently
- Run 100 times to ensure fix is reliable
```

**Phase 3: tasks.md - Add stress testing**

```markdown
## Task 1: Write Exploration Test

- [ ] 1.1 Set up concurrency test framework
- [ ] 1.2 Create stress test (1000 concurrent ops)
- [ ] 1.3 Run 10 times, expect failures
- [ ] 1.4 Document failure rate and patterns

## Task 3: Validate Fix

- [ ] 3.1 Run stress test 100 times (MUST PASS all)
- [ ] 3.2 Monitor for race conditions
- [ ] 3.3 Verify under production-like load
```

### Concurrency Testing Patterns

**JavaScript:**
```typescript
test('concurrent access stress test', async () => {
  const operations = Array(1000).fill(null).map((_, i) => 
    updateUser(userId, { value: i })
  );
  
  await Promise.all(operations);
  
  const finalState = await getUser(userId);
  expect(finalState.isConsistent()).toBe(true);
});
```

**Python:**
```python
def test_concurrent_stress():
    with ThreadPoolExecutor(max_workers=100) as executor:
        futures = [executor.submit(update_user, user_id, i) 
                   for i in range(1000)]
        results = [f.result() for f in futures]
    
    final_state = get_user(user_id)
    assert final_state.is_consistent()
```

---

## Incomplete Fix

### Scenario

Fix resolves bug in some cases but not all.

### Symptoms

- Exploration test passes
- Some edge cases still fail
- Bug reappears under different conditions

### Handling Strategy

**Step 1: Identify missed cases**

```markdown
## Fix Validation Results

Exploration test: PASSED
Edge case 1 (quantity=0): PASSED
Edge case 2 (quantity=-1): FAILED ← Missed case
Edge case 3 (quantity=null): FAILED ← Missed case

Conclusion: Fix is incomplete
```

**Step 2: Revise bug condition**

```markdown
## Bug Condition (Revised)

Original: C(quantity) = (quantity == 0)
Revised: C(quantity) = (quantity <= 0 OR quantity == null)

Fix must handle all three cases.
```

**Step 3: Update fix and re-test**

```markdown
## Task 2: Implement Core Fix (Revised)

- [ ] 2.1 Add guard for quantity == 0
- [ ] 2.2 Add guard for quantity < 0
- [ ] 2.3 Add guard for quantity == null
- [ ] 2.4 Add guard for quantity == undefined
```

---

## Anti-Patterns

**Do NOT:**
- Proceed with incomplete information
- Ignore multiple root causes
- Skip regression testing after fix
- Give up on non-reproducible bugs without investigation
- Assume fix is complete without thorough validation
- Mix multiple bug fixes in single bugfix spec

**DO:**
- Ask specific questions when information is missing
- Define compound conditions for multiple root causes
- Always run regression tests after fixing
- Document reproduction attempts for non-reproducible bugs
- Test edge cases thoroughly
- Create separate bugfix specs for unrelated bugs
