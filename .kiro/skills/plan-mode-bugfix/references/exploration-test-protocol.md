# Exploration Test Execution Protocol

Complete handling of exploration test outcomes in bugfix workflow.

## Overview

Task 1 (exploration test) has unique success criteria: the test MUST FAIL on unfixed code to prove the bug exists. This document defines the complete protocol for handling both expected and unexpected outcomes.

---

## Expected Outcome: Test FAILS on Unfixed Code

### What This Means

- Test finds counterexamples where C(X) = true
- Bug condition is correctly defined
- Test properly checks for the bug
- This is the SUCCESS case

### Required Actions

1. **Document counterexamples** found by the test
2. **Save to test output** for reference
3. **Mark Task 1 as complete** (test successfully proved bug exists)
4. **Proceed to Task 2** (implement fix)

### Example

```
Bug condition: C(quantity) = (quantity == 0) causes crash
Test result: FAILED - found counterexample quantity=0
Action: Proceed to fix implementation
```

---

## Unexpected Outcome: Test PASSES on Unfixed Code

### What This Means

Test did NOT find counterexamples. This indicates:
1. Bug doesn't actually exist (code already fixed)
2. Root cause analysis is incorrect
3. Test doesn't properly check C(X)
4. Bug is non-deterministic or environment-specific

### CRITICAL: STOP Immediately

Do NOT proceed to Task 2. The workflow must pause for investigation.

### Required Actions Sequence

#### Step 1: Output Detailed Analysis

Provide analysis in chat explaining:
- Test passed when it should have failed
- What this likely means (which of the 4 scenarios above)
- Evidence from test output
- Recommendation for which option to choose

Example analysis:
```
The exploration test passed unexpectedly. This means the test did not find 
any cases where quantity=0 causes a crash. 

Looking at the test output, I see the system is now handling quantity=0 
gracefully by returning an error message instead of crashing.

This suggests either:
1. The bug was already fixed in a previous commit
2. The root cause analysis identified the wrong code path

I recommend "Re-investigate" to verify the root cause before proceeding.
```

#### Step 2: Call userInput Tool

Present decision to user with structured options:

```javascript
userInput({
  question: "The exploration test passed unexpectedly. How should we proceed?",
  reason: "general-question",
  options: [
    {
      title: "Continue anyway",
      description: "Proceed with remaining tasks despite unexpected pass",
      recommended: false  // or true based on your analysis
    },
    {
      title: "Re-investigate",
      description: "Stop and investigate other root causes",
      recommended: true   // or false based on your analysis
    }
  ]
})
```

#### Step 3: Mark Recommendation

Set `recommended: true` on the option you believe is correct based on your analysis.

#### Step 4: STOP Execution

Do NOT proceed to Task 2 or any subsequent tasks. Wait for user decision.

#### Step 5: Handle User Choice

**If user chooses "Continue anyway":**
- Document the unexpected pass in task notes
- Proceed to Task 2 (implement fix)
- Note that fix validation may be less reliable

**If user chooses "Re-investigate":**
- Return to Phase 1 (bugfix.md)
- Revise root cause analysis
- Redefine bug condition C(X)
- Update exploration test
- Re-run test on unfixed code

---

## Decision Tree

```
Run exploration test on unfixed code
│
├─ Test FAILS (finds counterexamples)
│  └─ SUCCESS → Document counterexamples → Proceed to Task 2
│
└─ Test PASSES (no counterexamples)
   └─ STOP → Analyze why → Present options → Wait for user
      │
      ├─ User: "Continue anyway"
      │  └─ Document unexpected pass → Proceed to Task 2
      │
      └─ User: "Re-investigate"
         └─ Return to Phase 1 → Revise C(X) → Update test → Re-run
```

---

## Common Scenarios

### Scenario 1: Bug Already Fixed

**Symptoms:**
- Test passes cleanly
- No errors or exceptions
- Code handles edge case correctly

**Analysis:**
- Bug was likely fixed in a previous commit
- Root cause analysis may still be valid
- Fix may already be in place

**Recommendation:** Re-investigate to confirm fix is already present

### Scenario 2: Wrong Root Cause

**Symptoms:**
- Test passes but bug still reproducible manually
- Test checks wrong code path
- Test doesn't trigger the actual bug condition

**Analysis:**
- Root cause analysis identified wrong location
- Bug condition C(X) is incorrectly defined
- Need to find actual root cause

**Recommendation:** Re-investigate to find correct root cause

### Scenario 3: Test Implementation Error

**Symptoms:**
- Test passes but should fail based on code inspection
- Test doesn't actually check C(X)
- Test has logic errors

**Analysis:**
- Test implementation is incorrect
- Need to fix test to properly check C(X)
- Root cause may still be correct

**Recommendation:** Re-investigate to fix test implementation

### Scenario 4: Non-Deterministic Bug

**Symptoms:**
- Test sometimes passes, sometimes fails
- Bug depends on timing, concurrency, or external state
- Difficult to reproduce consistently

**Analysis:**
- Bug is non-deterministic
- May need stress testing or different test approach
- Root cause may be correct but hard to trigger

**Recommendation:** Continue anyway, but add stress testing in Task 3

---

## Orchestrator-Specific Instructions

When orchestrating bugfix workflow and Task 1 completes:

1. **Check subagent response** for `unexpected_pass` status or indication
2. **If unexpected pass detected:**
   - Do NOT automatically proceed to Task 2
   - Do NOT mark Task 2 as in_progress
   - Wait for user decision from userInput
3. **If expected fail (success):**
   - Mark Task 1 as completed
   - Proceed to Task 2 normally

### Status Tracking

Use a status field in task execution:
- `exploration_test_failed` (expected) → proceed
- `exploration_test_passed` (unexpected) → stop and wait
- `user_decision_continue` → proceed to Task 2
- `user_decision_reinvestigate` → return to Phase 1

---

## Anti-Patterns

**Do NOT:**
- Proceed to Task 2 automatically when test passes
- Assume test is correct without verification
- Skip user decision when test passes unexpectedly
- Ignore unexpected pass and continue workflow
- Mark Task 1 as complete when test passes (it's a failure case)

**DO:**
- Stop immediately when test passes
- Analyze why test passed
- Present clear options to user
- Wait for explicit user decision
- Document the unexpected outcome
