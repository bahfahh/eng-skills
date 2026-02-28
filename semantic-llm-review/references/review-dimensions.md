# Review Dimensions — The 4 Semantic Thinking Frameworks

These are not checklists. They are ways of thinking about code that surface problems no rule can catch.

---

## D1 — Intent Alignment（意圖對齊）

**Question:** Does this code actually do what it's supposed to do?

This is the most valuable dimension because it catches "works but wrong" bugs — code that passes all tests, compiles fine, and does the wrong thing.

**How to evaluate:**

1. Read the requirement or design intent for this feature
2. Read the code
3. Ask: "If I were a user, would this code produce the result the spec describes?"

**What to look for:**

- **Partial implementation:** Spec says "filter by date range AND status." Code only filters by date range.
- **Inverted logic:** Spec says "teachers can only edit their own observations." Code checks `created_by !== user.id` (inverted condition).
- **Missing edge in spec:** Spec says "send weekly report." Code sends on Monday. But what if there are no observations that week? Spec doesn't say — this is a gap to flag.
- **Drift from design:** Design says "all data access through API layer." New code calls database directly from a component.

**Example finding:**
```yaml
dimension: intent_alignment
description: "Spec says 'teachers can view all observations in their organization.'
  This query filters by class_id, which means teachers only see their own class.
  Cross-class visibility is missing."
grounding: "requirements.md section 2.3: 'Teachers have cross-class observation access'"
```

**When no spec exists:** Compare against test expectations, function naming, and PR description. Flag lower confidence.

---

## D2 — Boundary Reasoning（邊界條件推理）

**Question:** What happens at the edges that nobody explicitly coded for?

Most bugs live at boundaries. The happy path works; the edge cases don't.

**How to evaluate:**

Don't just read the code — actively imagine inputs that the author probably didn't think about.

**Boundary categories to check:**

| Category | Questions |
|----------|-----------|
| Null / Empty | What if the input is null, undefined, empty string, empty array? |
| Concurrent | What if two requests hit this at the same time? |
| Timing | What if this runs at midnight? At year boundary? During DST change? |
| Overflow | What if the array has 10,000 items? What if the string is 1MB? |
| Auth edge | What if the user's session expires mid-operation? |
| Partial failure | What if step 2 of 3 fails? Is step 1 rolled back? |
| Type coercion | What if a number comes as a string from the API? |

**How to prioritize:** Don't check all categories for every function. Focus on the ones relevant to the function's domain:
- Database operations → concurrent, partial failure
- Date calculations → timing, timezone
- User input → null/empty, overflow, type coercion
- Auth flows → auth edge, timing

**Example finding:**
```yaml
dimension: boundary_reasoning
description: "getWeekRange(date) doesn't handle timezone. If server is UTC
  and user is UTC+8, a Monday 00:30 local time is still Sunday in UTC.
  Week calculation returns wrong week."
grounding: "No explicit timezone handling in function. Date constructor uses
  server timezone by default."
```

---

## D3 — Hidden Assumptions（隱性假設）

**Question:** What does this code assume that isn't stated in its interface?

Hidden assumptions are the #1 source of bugs when code is modified by someone who didn't write it (including AI agents).

**How to evaluate:**

For each function, ask: "What must be true for this to work, that isn't enforced by the type system or validated at runtime?"

**Common hidden assumptions:**

| Assumption | Example |
|-----------|---------|
| Call order | "init() must be called before process()" |
| Data shape | "The array is always sorted" or "ID is always UUID format" |
| External state | "The database connection is alive" or "env var is set" |
| Timing | "This runs after auth middleware" or "This runs within a transaction" |
| Uniqueness | "File names are unique" or "User IDs don't collide" |
| Availability | "The LINE API is reachable" or "Storage bucket exists" |

**How to surface them:**

Look for:
- Functions that don't validate their inputs but would break on unexpected values
- Code that accesses properties without null checks on objects that could be null
- Middleware ordering assumptions (auth before business logic)
- Database query assumptions (RLS is enabled, indexes exist)

**Example finding:**
```yaml
dimension: hidden_assumption
description: "uploadPhoto() assumes file.type is always a valid MIME type.
  If a browser sends an empty string (which Safari does for some file types),
  the storage path becomes 'photos//.jpg' with a double slash."
grounding: "No validation on file.type before constructing storage path at line 34."
```

---

## D4 — Cross-layer Consistency（跨層一致性）

**Question:** Is this change consistent with the layers above and below it?

Code doesn't exist in isolation. A change in the API layer might be inconsistent with the hook that calls it, the component that displays the result, or the database schema it queries.

**How to evaluate:**

For each changed function, trace one layer up (who calls it) and one layer down (what it calls):

```
Component → Hook → API Layer → Database
    ↑                              ↓
    └── Are all layers consistent? ─┘
```

**What to look for:**

| Inconsistency | Example |
|--------------|---------|
| Interface mismatch | API returns `{data, error}` but hook expects `data` directly |
| Missing propagation | API adds a new required field but hook doesn't pass it |
| Type drift | Database column is `text` but TypeScript type says `number` |
| Permission gap | API checks org_id but database RLS doesn't (or vice versa) |
| Error handling gap | API throws on error but caller doesn't catch |
| Cache staleness | Data is updated in DB but query cache isn't invalidated |

**Example finding:**
```yaml
dimension: cross_layer_inconsistency
description: "API function addStudent() now returns {student, inviteCode} but
  the hook useStudents.ts line 45 still destructures as {data} and passes
  data directly to the component. The inviteCode is silently dropped."
grounding: "Hook at hooks/useStudents.ts:45 was not updated to match new API return shape."
```

---

## Dimension Interaction

Some combinations are especially revealing:

**D1 + D3 = Spec-assumption gap:**
The spec says X, the code assumes Y. Neither is wrong alone, but together they create a bug that only appears in production.

**D2 + D4 = Boundary propagation:**
A boundary condition in one layer (null from DB) propagates through layers that don't handle it. The bug appears in the UI, but the cause is in the data layer.

**D3 + D4 = Hidden contract:**
Layer A assumes Layer B always returns sorted data. Layer B doesn't guarantee this. Works today because the database happens to return sorted results. Breaks when an index changes.

---

## Prioritization

Not all dimensions are equally important for every change:

| Change Type | Primary Dimensions |
|------------|-------------------|
| New feature | D1 (intent alignment), D4 (cross-layer) |
| Bug fix | D2 (boundary), D3 (hidden assumptions) |
| Refactor | D4 (cross-layer), D3 (hidden assumptions) |
| Auth/security change | D2 (boundary), D3 (hidden assumptions), D4 (cross-layer) |
| Database migration | D4 (cross-layer), D1 (intent alignment) |
