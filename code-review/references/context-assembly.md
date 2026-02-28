# Context Assembly — How to Build the Semantic Ground Truth

The single most important step in semantic review. Bad context = bad review, regardless of how smart the LLM is.

---

## Principle: Select, Don't Dump

Do not feed the entire codebase into context. Select based on what the diff touches.

Important: technical plans (e.g. `plan.md`) are not semantic ground truth. Use them only to locate intended change areas, not to validate correctness.

**Selection algorithm:**

```
1. Parse the diff → list of changed files and functions
2. For each changed file:
   a. Find the FULL file content (not just the diff — LLM needs surrounding context)
   b. Find acceptance criteria (Given/When/Then, error/edge cases) for this feature area
   c. Find requirements/specs that define intent behind this feature area
   d. Find tests for this file or function
   e. Find callers (who calls this function?) and callees (what does it call?)
3. Assemble in priority order within token budget
```

---

## Context Structure Template

Organize context in this order (most important first and last, due to LLM attention patterns):

```markdown
## 1. Acceptance Context (FIRST — highest priority)
[Relevant sections from acceptance criteria: Given/When/Then, error cases, edge cases]

## 2. Requirements Context
[Relevant sections from requirements/specs]
[Only the sections related to the changed code]

## 3. PR Diff
[The actual changes being reviewed]

## 4. Full File Context
[Complete content of changed files — not just the diff]

## 5. Related Code
[Callers: functions that call the changed code]
[Callees: functions called by the changed code]
[Similar patterns: other files that follow the same convention]

## 6. Test Context
[Existing tests for the changed code]
[What behavior is currently expected]

## 7. Known Issues (LAST — second highest priority position)
[Lessons learned, known pitfalls for this area]
[Previous bugs in similar code]
```

Why this order: LLMs have strongest attention at the beginning and end of context (the "Lost in the Middle" problem). Acceptance goes first because it defines the BDD ground truth. Known issues go last because they're the most actionable warnings.

---

## Finding Acceptance/Requirements for a Diff

Most projects don't have a clean mapping from code to requirements. Use these heuristics:

**By file path:**
```
lib/api/observations.ts  → search specs for "observation"
lib/api/auth.ts          → search specs for "authentication", "login", "session"
middleware.ts            → search specs for "routing", "authorization", "tenant"
```

**By function name:**
```
createObservation()      → search for "create observation", "new observation"
getWeeklyReport()        → search for "weekly report", "report generation"
```

**By domain keywords in the diff:**
```
diff contains "organization_id" → search for "multi-tenant", "isolation"
diff contains "role === 'teacher'" → search for "teacher permissions"
```

If no acceptance/requirements are found, state it: "No acceptance/requirements doc found for this area. Review grounded in: [existing tests / codebase conventions / PR description / general knowledge]."

---

## Token Budget Guidelines

| Context Type | Typical Size | Priority |
|-------------|-------------|----------|
| Acceptance excerpt | 1–3K tokens | Must include (if available) |
| Requirements excerpt | 1–3K tokens | Must include (if available) |
| PR diff | 2–8K tokens | Must include |
| Full changed files | 3–10K tokens | Should include |
| Callers/callees | 2–5K tokens | Include if available |
| Tests | 2–5K tokens | Include if available |
| Known issues | 0.5–1K tokens | Include if available |
| **Total target** | **~15–25K tokens** | |

If total exceeds 30K tokens:
1. Trim "Related Code" to only direct callers
2. Trim "Full File Context" to only the changed functions + 20 lines above/below
3. Never trim Acceptance (if present), Requirements (if present), or PR Diff

---

## When No Documentation Exists

Many projects have no specs, no design docs, no ADRs. The semantic source becomes:

1. **Acceptance criteria** — if present anywhere (even in PR description), treat as the strongest
2. **Tests** — the strongest executable signal of expected behavior
2. **Function signatures + types** — the interface contract
3. **Naming conventions** — what the author intended
4. **Commit history** — why the code was written this way (if accessible)
5. **Comments** — inline documentation (often outdated, verify against code)

In this case, the review shifts from "does this match the spec?" to "is this internally consistent and does it handle edge cases?" — a weaker but still valuable form of semantic review.

---

## Context Assembly for Different Project Types

**Projects with acceptance + requirements/specs:**
```
Strongest review possible.
Context = acceptance + requirements/specs + diff + related code + tests
Findings grounded in: "Per acceptance/requirement X, this should..."
```

**Projects with tests but no acceptance/requirements:**
```
Good review.
Context = tests + diff + related code
Findings grounded in: "Existing tests expect X, but this change..."
```

**Projects with neither specs nor tests:**
```
Limited review — be honest about confidence.
Context = diff + full files + similar patterns in codebase
Findings grounded in: "Based on codebase conventions..." or "General best practice..."
Mark confidence: low
```
