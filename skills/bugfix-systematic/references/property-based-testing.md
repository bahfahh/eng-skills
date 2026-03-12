# Property-Based Testing Integration

How to transform bug conditions into executable property-based tests.

## Overview

Property-based testing (PBT) provides formal verification that bug condition C(X) no longer holds after the fix. This document covers framework selection, property definition, and integration patterns.

---

## Bug Condition to Property Transformation

### Transformation Rule

Given bug condition: `C(X) = [condition] causes [symptom]`

Property to test: `for all X, NOT (C(X))`

### Examples

**Example 1: Division by Zero**
- Bug condition: `C(quantity) = (quantity == 0)` causes crash
- Property: `for all quantity, system handles quantity without crashing`
- Test: Generate random quantities including 0, verify no crash

**Example 2: Null Pointer**
- Bug condition: `C(user) = (user == null)` causes NullPointerException
- Property: `for all user (including null), system returns valid response`
- Test: Generate users including null, verify no exception

**Example 3: Array Bounds**
- Bug condition: `C(index) = (index >= array.length)` causes IndexOutOfBounds
- Property: `for all index, system handles index safely`
- Test: Generate indices including out-of-bounds, verify safe handling

**Example 4: Concurrent Access**
- Bug condition: `C(threads) = (threads > 1 accessing shared state)` causes race condition
- Property: `for all thread counts, final state is consistent`
- Test: Generate concurrent operations, verify state consistency

---

## Framework Selection by Language

### JavaScript / TypeScript: fast-check

**Installation:**
```bash
npm install --save-dev fast-check
```

**Basic property test:**
```typescript
import fc from 'fast-check';

test('system handles all quantities without crashing', () => {
  fc.assert(
    fc.property(fc.integer(), (quantity) => {
      const result = processOrder(quantity);
      return result !== undefined; // No crash
    })
  );
});
```

**With counterexample shrinking:**
```typescript
fc.assert(
  fc.property(fc.integer(), (quantity) => {
    const result = processOrder(quantity);
    return result.success || result.error !== 'crash';
  }),
  { numRuns: 1000 } // Run 1000 test cases
);
```

### Python: Hypothesis

**Installation:**
```bash
pip install hypothesis
```

**Basic property test:**
```python
from hypothesis import given
from hypothesis.strategies import integers

@given(integers())
def test_handles_all_quantities(quantity):
    result = process_order(quantity)
    assert result is not None  # No crash
```

**With custom strategies:**
```python
from hypothesis import given, strategies as st

@given(st.one_of(st.integers(), st.none()))
def test_handles_null_users(user_id):
    result = get_user(user_id)
    assert 'error' in result or 'data' in result
```

### Java: jqwik

**Dependency:**
```xml
<dependency>
    <groupId>net.jqwik</groupId>
    <artifactId>jqwik</artifactId>
    <version>1.7.4</version>
    <scope>test</scope>
</dependency>
```

**Basic property test:**
```java
@Property
void handlesAllQuantities(@ForAll int quantity) {
    Result result = processOrder(quantity);
    assertNotNull(result);
}
```

### C#: FsCheck

**Installation:**
```bash
dotnet add package FsCheck
```

**Basic property test:**
```csharp
[Property]
public Property HandlesAllQuantities() {
    return Prop.ForAll<int>(quantity => {
        var result = ProcessOrder(quantity);
        return result != null;
    });
}
```

---

## Property Definition Patterns

### Pattern 1: No Exception Property

Verify operation completes without throwing:

```typescript
fc.property(fc.anything(), (input) => {
  try {
    processInput(input);
    return true;
  } catch (e) {
    return false; // Property fails if exception thrown
  }
});
```

### Pattern 2: Valid Output Property

Verify output meets specification:

```typescript
fc.property(fc.integer(), (quantity) => {
  const result = calculateTotal(quantity);
  return typeof result === 'number' && result >= 0;
});
```

### Pattern 3: Invariant Property

Verify system invariant holds:

```typescript
fc.property(fc.array(fc.integer()), (operations) => {
  const state = applyOperations(operations);
  return state.balance >= 0; // Invariant: balance never negative
});
```

### Pattern 4: Idempotence Property

Verify operation is idempotent:

```typescript
fc.property(fc.string(), (input) => {
  const result1 = normalize(input);
  const result2 = normalize(result1);
  return result1 === result2;
});
```

### Pattern 5: Commutativity Property

Verify order doesn't matter:

```typescript
fc.property(fc.integer(), fc.integer(), (a, b) => {
  return add(a, b) === add(b, a);
});
```

---

## Exploration Test Structure

### Phase 1: Before Fix (Test Must Fail)

```typescript
describe('Bug Condition Exploration', () => {
  test('MUST FAIL: quantity=0 causes crash', () => {
    // This test should FAIL before fix is applied
    fc.assert(
      fc.property(fc.integer(), (quantity) => {
        const result = processOrder(quantity);
        return result.success || result.error !== 'crash';
      })
    );
  });
});
```

**Expected output:**
```
FAILED after 1 test
Counterexample: quantity = 0
Error: Division by zero
```

### Phase 2: After Fix (Test Must Pass)

Same test, but now passes:

```
PASSED after 1000 tests
No counterexamples found
```

---

## Custom Generators for Bug Conditions

### Generate Edge Cases

Focus generator on values that trigger C(X):

```typescript
// Generate quantities including edge cases
const quantityGen = fc.oneof(
  fc.constant(0),           // Bug trigger
  fc.constant(-1),          // Negative
  fc.integer(1, 1000),      // Normal range
  fc.constant(Number.MAX_SAFE_INTEGER) // Overflow
);

fc.property(quantityGen, (quantity) => {
  // Test with focused edge cases
});
```

### Generate Null/Undefined

```typescript
const userGen = fc.oneof(
  fc.constant(null),
  fc.constant(undefined),
  fc.record({ id: fc.integer(), name: fc.string() })
);
```

### Generate Concurrent Scenarios

```python
from hypothesis import given, strategies as st

@given(st.lists(st.integers(), min_size=2, max_size=10))
def test_concurrent_access(operations):
    # Simulate concurrent operations
    results = run_concurrent(operations)
    assert is_consistent(results)
```

---

## Integration with Test Frameworks

### Jest (JavaScript)

```typescript
// jest.config.js
module.exports = {
  testTimeout: 30000, // PBT needs more time
};

// test file
import fc from 'fast-check';

describe('Bugfix: quantity=0 crash', () => {
  test('exploration test (should fail before fix)', () => {
    fc.assert(
      fc.property(fc.integer(), (quantity) => {
        const result = processOrder(quantity);
        return result !== undefined;
      }),
      { numRuns: 1000 }
    );
  });
});
```

### pytest (Python)

```python
# conftest.py
from hypothesis import settings

settings.register_profile("ci", max_examples=1000)
settings.load_profile("ci")

# test file
from hypothesis import given
from hypothesis.strategies import integers

@given(integers())
def test_exploration(quantity):
    result = process_order(quantity)
    assert result is not None
```

### xUnit (C#)

```csharp
using FsCheck;
using FsCheck.Xunit;

public class BugfixTests {
    [Property(MaxTest = 1000)]
    public Property ExplorationTest() {
        return Prop.ForAll<int>(quantity => {
            var result = ProcessOrder(quantity);
            return result != null;
        });
    }
}
```

---

## Counterexample Documentation

### Capture Counterexamples

When test fails (expected in exploration phase):

```typescript
fc.assert(
  fc.property(fc.integer(), (quantity) => {
    try {
      const result = processOrder(quantity);
      return true;
    } catch (e) {
      console.log(`Counterexample found: quantity=${quantity}`);
      console.log(`Error: ${e.message}`);
      return false;
    }
  })
);
```

### Save to File

```typescript
const counterexamples = [];

fc.assert(
  fc.property(fc.integer(), (quantity) => {
    const result = processOrder(quantity);
    if (!result.success) {
      counterexamples.push({ quantity, error: result.error });
    }
    return result.success;
  })
);

fs.writeFileSync('counterexamples.json', JSON.stringify(counterexamples, null, 2));
```

---

## Advanced Patterns

### Stateful Testing

For bugs involving state transitions:

```typescript
fc.assert(
  fc.property(fc.array(fc.record({
    action: fc.constantFrom('add', 'remove', 'update'),
    value: fc.integer()
  })), (actions) => {
    const state = new StateMachine();
    for (const action of actions) {
      state.apply(action);
    }
    return state.isValid(); // Check invariants
  })
);
```

### Shrinking for Minimal Counterexamples

PBT frameworks automatically shrink counterexamples:

```
Initial failure: quantity = 847362
Shrinking...
Minimal counterexample: quantity = 0
```

This helps identify the exact boundary where C(X) becomes true.

---

## Performance Considerations

### Number of Test Runs

- Development: 100-1000 runs
- CI: 1000-10000 runs
- Critical bugs: 10000+ runs

```typescript
fc.assert(
  fc.property(/* ... */),
  { numRuns: process.env.CI ? 10000 : 1000 }
);
```

### Timeout Configuration

PBT takes longer than unit tests:

```typescript
// Jest
test('exploration test', () => {
  // ...
}, 30000); // 30 second timeout

// pytest
@settings(deadline=30000)
@given(integers())
def test_exploration(quantity):
    # ...
```

---

## Anti-Patterns

**Do NOT:**
- Write property tests that always pass (non-discriminating)
- Use too few test runs (< 100)
- Ignore counterexamples without investigation
- Mix property tests with example-based tests in same test case
- Forget to document counterexamples from exploration phase

**DO:**
- Focus generators on edge cases that trigger C(X)
- Run enough iterations to find rare bugs
- Document all counterexamples found
- Keep property tests separate from unit tests
- Use shrinking to find minimal counterexamples
