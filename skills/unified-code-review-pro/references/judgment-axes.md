# Judgment Axes — Detailed Reference

The 6 axes replace traditional "is this too long/complex?" with "is this safe to maintain and evolve?"

---

## R1 — Intent Cohesion（意圖一致性）

**Core question:** Can you describe what this function/module does in one sentence, without using "and", "also", "while", or "at the same time"?

**Why it matters:** Functions that do multiple things are harder to test, harder to name, and harder to change safely. This is a more useful signal than SRP because it's evaluable from reading the code, not from abstract principles.

**Scoring:**

| Score | Meaning | Signal |
|-------|---------|--------|
| `clear` | One sentence, no conjunctions needed | Safe |
| `mixed` | Needs "and" once — two related concerns | Review |
| `fragmented` | Multiple unrelated concerns, or "it depends on the mode" | Refactor |

**Fail patterns:**
```typescript
// FRAGMENTED: does I/O + domain decision + formatting
async function processAndSendReport(reportId: string) {
  const data = await db.query(...)      // I/O
  if (data.status === 'draft') return   // domain decision
  const pdf = await generatePdf(data)  // transformation
  await sendToLine(pdf)                 // I/O again
  await db.update(reportId, 'sent')    // state mutation
}
```

**Module depth fail pattern [D-1]:** Caller must compose multiple functions to complete one business operation — the intent is split across the call site.
```typescript
// FRAGMENTED at call site: business intent "get user config" is assembled by caller
const path = getConfigFilePath(userId)
const raw = readFile(path)
const config = mergeDefaults(parse(raw))
```

**Pass pattern:**
```typescript
// CLEAR: one job — orchestrate the send flow
async function sendApprovedReport(reportId: string) {
  const report = await reportRepository.getApproved(reportId)
  const pdf = await pdfService.generate(report)
  await lineService.send(report.parentLineId, pdf)
  await reportRepository.markSent(reportId)
}
// Each called function has its own clear intent

// CLEAR: one call completes the business operation
const config = getUserConfig(userId)
```

---

## R2 — Semantic Stability（語意穩定度）

**Core question:** Does this function's behavior depend on anything that isn't visible in its signature?

**Why it matters:** Functions with hidden dependencies (global state, env vars, call order, time) are unpredictable. High complexity + stable semantics = acceptable. Low complexity + unstable semantics = dangerous.

**Scoring:**

| Score | Meaning |
|-------|---------|
| `high` | Output determined entirely by inputs; no hidden state |
| `medium` | Depends on injected dependencies (acceptable if explicit) |
| `low` | Depends on global state, call order, time, or env without declaration |

**Fail patterns:**
```typescript
// LOW: behavior changes based on undeclared external state
function getWeekLabel() {
  const now = new Date()           // hidden time dependency
  const org = currentOrg           // hidden global state
  return formatWeek(now, org.locale)
}

// LOW: call-order dependency
function initAndProcess() {
  if (!this.initialized) throw Error('call init() first')
  // ...
}
```

**Pass pattern:**
```typescript
// HIGH: all dependencies explicit
function getWeekLabel(date: Date, locale: string): string {
  return formatWeek(date, locale)
}
```

---

## R3 — AI-Rewrite Safety（AI 可重寫性）

**Core question:** Could another AI agent rewrite this function correctly, given only the current code and its tests — without access to git history, Slack threads, or the original author?

**Why it matters:** In AI-assisted development, code gets rewritten frequently. Code that requires tribal knowledge to rewrite safely is a long-term liability. This axis is forward-looking.

**Scoring:**

| Score | Meaning |
|-------|---------|
| `safe` | Intent is clear from naming + structure; tests cover behavior |
| `risky` | Some magic numbers/flags; behavior partially inferrable |
| `unsafe` | Domain rules buried in conditions; rewrite would likely break behavior |

**Fail patterns:**
```typescript
// UNSAFE: magic numbers, no named constants, domain rule invisible
if (score >= 3 && mode !== 2 && !flags[7]) {
  applyDiscount(0.15)
}

// UNSAFE: side effects scattered, no way to know what "clean" state is
function updateStudent(id: string, data: any) {
  db.update('students', id, data)
  cache.invalidate(`student:${id}`)
  if (data.classId) cache.invalidate(`class:${data.classId}`)
  if (data.status === 'inactive') notifyTeachers(id)  // surprise!
}
```

**Shallow module fail pattern [D-2][D-3]:** Same input object is passed to multiple utils in fixed order at the call site — the composition rule exists only in the caller's head, not in any named function.
```typescript
// RISKY: if call order changes or one util is swapped, AI has no way to know
createMessage({
  dateRange: getDateRangeLabel(content),   // always called together
  highlights: getHighlights(content),      // always called together
  counts: getDomainCounts(content),        // always called together
})
```

**Interface depth fail pattern [D-4]:** Interface exposes internal steps as parameters — AI must reconstruct the composition logic to call correctly.
```typescript
// UNSAFE: caller must know 7 internal details to make one business call
sendReport(reportId, lineToken, studentName, className, teacherName, pdfUrl, flexParams)
```

**Pass pattern:**
```typescript
// SAFE: named constants, explicit domain rule, no surprises
const MINIMUM_DOMAIN_SCORE = 3
const STANDARD_DISCOUNT_RATE = 0.15

if (isEligibleForDiscount(score, mode, flags)) {
  applyDiscount(STANDARD_DISCOUNT_RATE)
}

function isEligibleForDiscount(score: number, mode: Mode, flags: Flags): boolean {
  return score >= MINIMUM_DOMAIN_SCORE
    && mode !== Mode.EXEMPT
    && !flags.discountAlreadyApplied
}

// SAFE: interface hides composition — one business identifier is enough
sendReport(reportId)

// SAFE: facade encapsulates fixed-sequence utils
createMessage(prepareMessageParams(content))
```

---

## R4 — Test Derivability（測試可推導性）

**Core question:** Can you derive the test cases by reading the function — without running it or asking the author?

**Why it matters:** If tests can only be written by someone who already knows the behavior, the function is not self-documenting. This is different from "does it have tests" — it's about whether the *right* tests are derivable.

**Scoring:**

| Score | Meaning |
|-------|---------|
| `easy` | Happy path + boundary conditions + error paths all visible from code |
| `partial` | Happy path clear; some edge cases require domain knowledge |
| `hard` | Tests can only be written by running the code or asking the author |

**Fail patterns:**
- Tests only compare output snapshots (no behavioral assertions)
- Behavior only testable via full integration (can't isolate)
- Edge cases only discoverable by running in production

**Test coupling fail pattern [T-3]:** Tests assert on internal call paths rather than behavior — they break on refactor even if behavior is unchanged.
```typescript
// HARD: test is coupled to implementation, not behavior
expect(getHighlights).toHaveBeenCalledWith(content)
expect(getDomainCounts).toHaveBeenCalledWith(content)
```

**Wrong test target fail pattern [T-1]:** Utils have thorough tests but the facade that composes them has none — business behavior is untested at the meaningful boundary.
```
HARD: tests/utils/report.test.ts covers every util individually
      but tests/api/reports.test.ts (facade) does not exist
```

**Pass pattern:**
```typescript
// EASY: from reading this, you can derive:
// - test: valid org_id returns observations
// - test: wrong org_id returns empty (multi-tenant isolation)
// - test: inactive student excluded
// - test: date range boundary (start inclusive, end inclusive)
async function getObservations(orgId: string, filters: ObservationFilters) {
  return db.from('observations')
    .eq('organization_id', orgId)
    .eq('is_active', true)
    .gte('created_at', filters.startDate)
    .lte('created_at', filters.endDate)
}

// EASY: test targets behavior, not implementation path
expect(result.highlights).toEqual(['本週觀察重點...'])
expect(result.domainCounts.emotion).toBe(3)
```

---

## R5 — Domain Visibility（Domain 規則可見性）

**Core question:** Are the business rules named, findable, and separated from infrastructure code?

**Why it matters:** Domain complexity is unavoidable — kindergarten systems have real rules about enrollment, reporting periods, permissions. The problem isn't complexity, it's when those rules are invisible inside `if` chains.

**Scoring:**

| Score | Meaning |
|-------|---------|
| `explicit` | Business rules have names; decision logic is extractable |
| `implicit` | Rules exist but are embedded in conditions without names |
| `hidden` | Rules only discoverable by reading all branches carefully |

**Fail patterns:**
```typescript
// HIDDEN: what does this rule mean? Why these specific conditions?
if (user.role === 'teacher' && obs.created_by === user.id
    && obs.status !== 'archived' && !org.locked) {
  allowEdit()
}
```

**Naming fail pattern [N-1][N-2]:** Function name describes technical steps instead of business concept — reader must parse the body to understand intent.
```typescript
// IMPLICIT: name is a step description, not a business concept
parseAndMergeConfig(raw)
getAndFormatAndMergeReport(id)

// IMPLICIT: interface leaks storage layer details instead of business language
createObservation(supabase, orgId, { observation_date, created_by, organization_id })
```

**Pass pattern:**
```typescript
// EXPLICIT: rule has a name, intent is clear
if (canTeacherEditObservation(user, obs, org)) {
  allowEdit()
}

function canTeacherEditObservation(user: User, obs: Observation, org: Org): boolean {
  return user.role === 'teacher'
    && obs.created_by === user.id      // teachers own their records
    && obs.status !== 'archived'       // archived = immutable
    && !org.locked                     // org lock prevents all edits
}

// EXPLICIT: name reflects business concept, interface uses business language
getUserConfig(userId)
createObservation(orgId, { date, teacherId, note, studentIds })
```

---

## R6 — Change Blast Radius（變更爆炸半徑）

**Core question:** If one condition or constant in this function changes, what else breaks?

**Why it matters:** This replaces "is the function too big?" with a more useful question. A large function with a contained blast radius is safer than a small function that's called from 40 places with implicit assumptions.

**Scoring:**

| Score | Meaning |
|-------|---------|
| `contained` | Change affects only this function and its direct callers |
| `wide` | Change ripples to multiple modules; callers have implicit assumptions |
| `unknown` | Cannot determine impact without running the full system |

**Fail patterns:**
- Shared utility functions with no abstraction boundary
- Functions that mutate shared state
- Functions whose behavior callers depend on implicitly (no interface contract)

**Pass pattern:**
```typescript
// CONTAINED: clear interface, callers depend on the contract not the implementation
interface ObservationPermissions {
  canEdit: boolean
  canDelete: boolean
  canView: boolean
}

function getObservationPermissions(user: User, obs: Observation): ObservationPermissions {
  // Callers depend on the interface, not the internal logic
  // Changing the internal rules doesn't break callers
}
```

---

## Axis interaction patterns

Some combinations are especially important:

**R3 unsafe + R6 unknown = highest risk**
Code that can't be safely rewritten AND has unpredictable blast radius. Refactor required regardless of other axes.

**R1 clear + R2 high + R5 explicit = acceptable complexity**
Even if CC is 20+, if intent is clear, semantics are stable, and domain rules are named — the complexity is justified.

**R4 hard + R5 hidden = testing debt**
The code works today but accumulates risk with every change. Recommend adding named domain functions before adding more features.
