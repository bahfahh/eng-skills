# Phase Document Templates

Complete templates for bugfix.md, design.md, and tasks.md with examples.

## Overview

This document provides structured templates for each phase document, showing required sections and examples.

---

## Phase 1: bugfix.md Template

```markdown
# Bugfix: [Component] - [Symptom]

## Bug Description

### Observable Symptoms
[Describe what the user sees/experiences]
- Error message: [exact error text]
- Stack trace: [if available]
- Visual symptoms: [if UI bug]

### Steps to Reproduce
1. [First step]
2. [Second step]
3. [Third step]
4. [Observe bug]

### Expected vs Actual Behavior
- Expected: [what should happen]
- Actual: [what actually happens]

### Impact Assessment
- Severity: [Critical/High/Medium/Low]
- Affected users: [percentage or count]
- Workarounds: [if any exist]
- Business impact: [revenue, reputation, etc.]

## Root Cause Analysis

### Code Location
- File: [path/to/file.ts]
- Function: [functionName]
- Line: [line number or range]

### Why It Fails
[Explain the logic error or assumption violation]

### Violated Assumptions
[What assumptions were made that don't hold]

### Related Code Paths
[Other code that might be affected]
- [path/to/related1.ts] - [why it's related]
- [path/to/related2.ts] - [why it's related]

## Bug Condition C(X)

### Formal Definition
C(X) = [condition] causes [symptom]

Where X = [describe the variables]

### Triggering Conditions
[Describe input/state that makes C(X) true]
- Input condition 1: [description]
- State condition 2: [description]

### Boundary Cases
[Edge cases where C(X) becomes true]
- [boundary case 1]
- [boundary case 2]

### Counterexamples
[Specific values where C(X) is true]
- X = [value 1] → [symptom occurs]
- X = [value 2] → [symptom occurs]

## Reproduction Context

### Environment
- OS: [operating system]
- Runtime: [Node 18.x, Python 3.11, etc.]
- Dependencies: [key dependency versions]
- Configuration: [relevant config values]

### Data State
[Required data state to trigger bug]
- Database: [specific records or state]
- Cache: [cache state if relevant]
- Session: [session state if relevant]

### Timing/Concurrency
[If timing-dependent]
- Concurrency level: [number of threads/requests]
- Timing window: [specific timing requirements]
- Load conditions: [if load-dependent]
```

### Example: bugfix.md

```markdown
# Bugfix: Checkout - Zero Quantity Crash

## Bug Description

### Observable Symptoms
Application crashes with "Division by zero" error during checkout.
- Error message: `Error: Division by zero in calculateUnitPrice`
- Stack trace:
  ```
  at calculateUnitPrice (checkout.ts:45)
  at processOrder (checkout.ts:120)
  at POST /api/orders (routes.ts:89)
  ```

### Steps to Reproduce
1. Add item to cart
2. Set quantity to 0
3. Click "Checkout"
4. Application crashes

### Expected vs Actual Behavior
- Expected: Show error message "Quantity must be at least 1"
- Actual: Application crashes with unhandled exception

### Impact Assessment
- Severity: High
- Affected users: ~2% of checkout attempts
- Workarounds: Users must refresh and re-add items
- Business impact: Lost sales, poor user experience

## Root Cause Analysis

### Code Location
- File: src/checkout/calculateUnitPrice.ts
- Function: calculateUnitPrice
- Line: 45

```typescript
function calculateUnitPrice(totalPrice: number, quantity: number): number {
  return totalPrice / quantity; // ← Division by zero when quantity=0
}
```

### Why It Fails
Function assumes quantity is always > 0, but cart allows quantity=0.
No validation before division operation.

### Violated Assumptions
- Assumption: Quantity is always positive
- Reality: Cart UI allows setting quantity to 0

### Related Code Paths
- src/cart/updateQuantity.ts - Allows setting quantity to 0
- src/checkout/validateOrder.ts - Should validate but doesn't check quantity

## Bug Condition C(X)

### Formal Definition
C(quantity) = (quantity == 0) causes division by zero crash

Where quantity = number of items in order line

### Triggering Conditions
- Input: quantity parameter equals 0
- State: No validation before calculateUnitPrice call

### Boundary Cases
- quantity = 0 → crash
- quantity = -1 → also crashes (negative)
- quantity = null → also crashes (null)
- quantity = undefined → also crashes (undefined)

### Counterexamples
- quantity = 0 → Division by zero error
- quantity = -5 → Division by negative (wrong result)
- quantity = null → TypeError

## Reproduction Context

### Environment
- OS: Any
- Runtime: Node 18.x
- Dependencies: Express 4.18.2
- Configuration: Default

### Data State
- Cart must have at least one item
- Item quantity set to 0
- No other specific state required

### Timing/Concurrency
Not timing-dependent. Occurs consistently when quantity=0.
```

---

## Phase 2: design.md Template

```markdown
# Fix Design: [Component] - [Symptom]

## Fix Strategy

### Primary Approach
[How to make C(X) never true]

### Code Changes Required
[Specific files and changes]
- File: [path/to/file]
  - Change: [description]
  - Reason: [why this change]

### Alternative Approaches Considered
[Other approaches and why rejected]
1. [Alternative 1]
   - Pros: [advantages]
   - Cons: [disadvantages]
   - Rejected because: [reason]

### Complexity Assessment
[Simple guard vs architectural change]
- Complexity: [Low/Medium/High]
- Risk: [Low/Medium/High]
- Effort: [hours/days estimate]

## Preservation Checking

### Functionality That Must Not Break
[Existing features to preserve]
- [Feature 1]: [why it's important]
- [Feature 2]: [why it's important]

### Regression Risk Areas
[Code that might break]
- [Area 1]: [why it's at risk]
- [Area 2]: [why it's at risk]

### Performance Impact
[Expected performance changes]
- [Impact description]
- Mitigation: [if needed]

### Backward Compatibility
[API/interface changes]
- Breaking changes: [Yes/No]
- Migration required: [Yes/No]
- Deprecation plan: [if applicable]

## Test Strategy

### Exploration Test
[Property-based test for C(X)]
- Framework: [fast-check/Hypothesis/jqwik/FsCheck]
- Property: [formal property definition]
- Expected: MUST FAIL on unfixed code
- Counterexamples: [what to expect]

### Fix Validation Test
[Same test after fix]
- Expected: MUST PASS on fixed code
- Runs: [number of test iterations]

### Regression Tests
[Tests to ensure no breakage]
- [Test 1]: [what it checks]
- [Test 2]: [what it checks]

### Edge Case Coverage
[Specific edge cases to test]
- [Edge case 1]: [expected behavior]
- [Edge case 2]: [expected behavior]

## Implementation Approach

### Code Patterns
[Specific patterns to use]
```[language]
[code example]
```

### Error Handling
[How to handle errors]
- [Error type 1]: [handling strategy]
- [Error type 2]: [handling strategy]

### Logging/Monitoring
[What to log]
- Log level: [INFO/WARN/ERROR]
- Message: [log message format]
- Metrics: [if applicable]

### Rollback Plan
[How to rollback if needed]
1. [Rollback step 1]
2. [Rollback step 2]
```

### Example: design.md

```markdown
# Fix Design: Checkout - Zero Quantity Crash

## Fix Strategy

### Primary Approach
Add validation before division to ensure quantity > 0.
Return error result instead of crashing.

### Code Changes Required
- File: src/checkout/calculateUnitPrice.ts
  - Change: Add guard clause at function start
  - Reason: Prevent division by zero

- File: src/checkout/validateOrder.ts
  - Change: Add quantity validation
  - Reason: Catch invalid quantities early

- File: src/cart/updateQuantity.ts
  - Change: Prevent setting quantity to 0
  - Reason: Prevent invalid state at source

### Alternative Approaches Considered
1. Only fix calculateUnitPrice
   - Pros: Minimal change
   - Cons: Doesn't prevent invalid state
   - Rejected because: Allows invalid cart state

2. Remove quantity=0 option from UI
   - Pros: Prevents user error
   - Cons: Users need quantity=0 for "save for later"
   - Rejected because: Breaks existing feature

3. Treat quantity=0 as "remove item"
   - Pros: Intuitive behavior
   - Cons: Changes existing behavior
   - Rejected because: Breaking change

### Complexity Assessment
- Complexity: Low (simple validation)
- Risk: Low (defensive programming)
- Effort: 2-3 hours

## Preservation Checking

### Functionality That Must Not Break
- Cart "save for later": Uses quantity=0 to mark items
- Bulk operations: May temporarily set quantity=0
- Order history: May have historical orders with quantity=0

### Regression Risk Areas
- Cart operations: All quantity updates
- Checkout flow: All price calculations
- Order processing: All order validation

### Performance Impact
Negligible. Adding simple validation check.

### Backward Compatibility
- Breaking changes: No
- Migration required: No
- API changes: Error response added for invalid quantity

## Test Strategy

### Exploration Test
- Framework: fast-check (JavaScript)
- Property: `for all quantity, calculateUnitPrice does not crash`
- Expected: MUST FAIL on unfixed code (finds quantity=0)
- Counterexamples: quantity=0, quantity=-1, quantity=null

```typescript
fc.property(fc.integer(), (quantity) => {
  const result = calculateUnitPrice(100, quantity);
  return result.success || result.error !== 'crash';
});
```

### Fix Validation Test
- Expected: MUST PASS on fixed code
- Runs: 1000 iterations

### Regression Tests
- Test cart operations with various quantities
- Test checkout flow with edge cases
- Test order history retrieval

### Edge Case Coverage
- quantity = 0 → return error
- quantity < 0 → return error
- quantity = null → return error
- quantity = undefined → return error
- quantity = 1 → normal operation
- quantity = 1000000 → normal operation

## Implementation Approach

### Code Patterns
```typescript
function calculateUnitPrice(
  totalPrice: number, 
  quantity: number
): Result<number> {
  // Guard clause
  if (quantity == null || quantity <= 0) {
    return {
      success: false,
      error: 'Quantity must be a positive number'
    };
  }
  
  return {
    success: true,
    value: totalPrice / quantity
  };
}
```

### Error Handling
- Invalid quantity: Return error result, don't throw
- Null/undefined: Treat as invalid, return error
- Negative: Treat as invalid, return error

### Logging/Monitoring
- Log level: WARN
- Message: `Invalid quantity in calculateUnitPrice: ${quantity}`
- Metrics: Counter for invalid quantity attempts

### Rollback Plan
1. Revert calculateUnitPrice.ts changes
2. Revert validateOrder.ts changes
3. Keep updateQuantity.ts changes (defensive)
4. Monitor for crashes
```

---

## Phase 3: tasks.md Template

```markdown
# Implementation Tasks: [Component] - [Symptom]

## Task 1: Write Bug Condition Exploration Property Test
- [ ] 1.1 Set up property-based testing framework
- [ ] 1.2 Define property checking NOT C(X)
- [ ] 1.3 Run test on unfixed code (MUST FAIL)
- [ ] 1.4 Document counterexamples found

## Task 2: Implement Core Fix
- [ ] 2.1 Apply fix strategy from design.md
- [ ] 2.2 Add guard conditions to prevent C(X)
- [ ] 2.3 Add error handling for edge cases
- [ ] 2.4 Update related code paths

## Task 3: Validate Fix
- [ ] 3.1 Run exploration test (MUST PASS now)
- [ ] 3.2 Run regression test suite
- [ ] 3.3 Manual testing of reproduction steps
- [ ] 3.4 Performance validation

## Task 4: Documentation and Monitoring
- [ ] 4.1 Update code comments explaining fix
- [ ] 4.2 Add logging for C(X) near-misses
- [ ] 4.3 Update user-facing documentation if needed
- [ ] 4.4 Add monitoring/alerting if needed
```

### Example: tasks.md

```markdown
# Implementation Tasks: Checkout - Zero Quantity Crash

## Task 1: Write Bug Condition Exploration Property Test
- [ ] 1.1 Install fast-check: `npm install --save-dev fast-check`
- [ ] 1.2 Create test file: `src/checkout/__tests__/calculateUnitPrice.property.test.ts`
- [ ] 1.3 Define property: `for all quantity, calculateUnitPrice does not crash`
- [ ] 1.4 Run test on unfixed code (expect FAIL with counterexample quantity=0)
- [ ] 1.5 Document counterexamples in test output

## Task 2: Implement Core Fix
- [ ] 2.1 Update calculateUnitPrice.ts
  - [ ] 2.1.1 Add guard clause for quantity <= 0
  - [ ] 2.1.2 Add guard clause for null/undefined
  - [ ] 2.1.3 Return error result instead of throwing
  - [ ] 2.1.4 Update return type to Result<number>
- [ ] 2.2 Update validateOrder.ts
  - [ ] 2.2.1 Add quantity validation check
  - [ ] 2.2.2 Return validation error for invalid quantity
- [ ] 2.3 Update updateQuantity.ts
  - [ ] 2.3.1 Add validation before setting quantity
  - [ ] 2.3.2 Prevent quantity <= 0 (except for "save for later" feature)
- [ ] 2.4 Update error handling in checkout flow
  - [ ] 2.4.1 Handle error result from calculateUnitPrice
  - [ ] 2.4.2 Display user-friendly error message

## Task 3: Validate Fix
- [ ] 3.1 Run exploration test (expect PASS with 1000 iterations)
- [ ] 3.2 Run regression test suite
  - [ ] 3.2.1 Cart operations tests
  - [ ] 3.2.2 Checkout flow tests
  - [ ] 3.2.3 Order processing tests
- [ ] 3.3 Manual testing
  - [ ] 3.3.1 Try to set quantity to 0
  - [ ] 3.3.2 Try to checkout with quantity 0
  - [ ] 3.3.3 Verify error message displays
  - [ ] 3.3.4 Test "save for later" still works
- [ ] 3.4 Performance check
  - [ ] 3.4.1 Measure checkout time before/after
  - [ ] 3.4.2 Verify no performance regression

## Task 4: Documentation and Monitoring
- [ ] 4.1 Add code comments
  - [ ] 4.1.1 Document why guard clause is needed
  - [ ] 4.1.2 Document edge cases handled
- [ ] 4.2 Add logging
  - [ ] 4.2.1 Log WARN when invalid quantity detected
  - [ ] 4.2.2 Include quantity value in log
- [ ] 4.3 Update user documentation
  - [ ] 4.3.1 Update checkout guide with quantity requirements
  - [ ] 4.3.2 Document "save for later" behavior
- [ ] 4.4 Add monitoring
  - [ ] 4.4.1 Add counter metric for invalid quantity attempts
  - [ ] 4.4.2 Set up alert if rate exceeds threshold
```

---

## Template Selection Guide

### Simple Bug (1-2 files, clear fix)

Use abbreviated templates:
- bugfix.md: Focus on Bug Description and Bug Condition
- design.md: Brief Fix Strategy and Test Strategy
- tasks.md: 3 tasks (test, fix, validate)

### Complex Bug (multi-component, architectural)

Use full templates:
- bugfix.md: All sections with detailed analysis
- design.md: All sections with alternatives considered
- tasks.md: 4+ tasks with sub-tasks

### Non-Deterministic Bug

Emphasize in templates:
- bugfix.md: Detailed Reproduction Context
- design.md: Stress testing in Test Strategy
- tasks.md: Multiple test runs in Task 3

---

## Anti-Patterns

**Do NOT:**
- Skip sections in templates
- Use vague descriptions ("fix the bug")
- Omit counterexamples
- Forget to document alternatives considered
- Skip regression testing tasks

**DO:**
- Fill all template sections
- Be specific and concrete
- Document all counterexamples found
- Explain why alternatives were rejected
- Include comprehensive validation tasks
