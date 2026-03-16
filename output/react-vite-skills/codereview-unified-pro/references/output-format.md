# Output Format Reference

## Full Schema

```yaml
# One block per reviewed unit (function, class, or module)
unit: "<FunctionName | ClassName | file path>"

# From static analysis tools — null if not available
metrics_input:
  cyclomatic_complexity: <integer | null>
  cognitive_complexity: <integer | null>
  lines_of_code: <integer | null>
  flagged_by_tools:
    - "<tool>: <warning message>"   # e.g. "ESLint: complexity(14)"
    # empty list if no tool warnings

# 6-axis judgment
axes:
  intent_cohesion: clear | mixed | fragmented
  semantic_stability: high | medium | low
  ai_rewrite_safety: safe | risky | unsafe
  test_derivability: easy | partial | hard
  domain_visibility: explicit | implicit | hidden
  change_blast_radius: contained | wide | unknown

# Final verdict
verdict: accept | accept_with_risk | refactor_recommended | refactor_required

# Concise bullets — human reads this + verdict only
reasoning:
  - "<specific observation, reference file:line if possible>"
  - "<why this matters for maintainability or safety>"

# Null if verdict is accept
suggested_action: "<one concrete next step>" | null

# Only include if escalating to human decision
human_decision_needed: true | false
human_decision_reason: "<trade-off description>" | null
```

---

## Verdict Decision Rules

```
ALL axes pass (clear/high/safe/easy/explicit/contained)
  → accept

1–2 axes are medium/partial/implicit/wide
  → accept_with_risk
  → document which axes and why

Any axis is low/risky/hidden/wide
  → refactor_recommended
  → explain what to change

R3 = unsafe  OR  R6 = unknown
  → refactor_required  (regardless of other axes)
  → these two indicate active maintenance danger
```

---

## Annotated Examples

### Example 1: Accept — Complex but safe

```yaml
unit: "getObservationPermissions"
metrics_input:
  cyclomatic_complexity: 14
  flagged_by_tools:
    - "ESLint: complexity(14) exceeds threshold of 10"

axes:
  intent_cohesion: clear          # "returns permission set for a user+observation pair"
  semantic_stability: high        # pure function, all inputs explicit
  ai_rewrite_safety: safe         # named constants, clear conditions
  test_derivability: easy         # each permission has a clear derivable test
  domain_visibility: explicit     # canTeacherEdit(), canPrincipalEdit() named
  change_blast_radius: contained  # callers depend on interface, not internals

verdict: accept

reasoning:
  - "CC=14 is flagged by ESLint but complexity is justified: 4 roles × 3 permission types"
  - "All domain rules are named functions — changing a rule doesn't require hunting conditions"
  - "Pure function with stable interface; callers are isolated from internal changes"

suggested_action: null
```

### Example 2: Refactor Required — Hidden domain + unknown blast radius

```yaml
unit: "processWeeklyReport"
metrics_input:
  cyclomatic_complexity: 22
  lines_of_code: 187
  flagged_by_tools:
    - "SonarQube: Cognitive Complexity of 31 exceeds 15"
    - "ESLint: max-lines-per-function(187) exceeds 60"

axes:
  intent_cohesion: fragmented     # fetches data + generates PDF + sends LINE + updates DB
  semantic_stability: medium      # depends on org config injected via closure
  ai_rewrite_safety: unsafe       # magic flags, domain rules buried in conditions
  test_derivability: hard         # only testable via full integration
  domain_visibility: hidden       # "if status === 3 && !flags[2]" — no named rules
  change_blast_radius: unknown    # called from scheduler + manual trigger + webhook

verdict: refactor_required

reasoning:
  - "R3 unsafe: `if (report.type === 2 && !org.flags[5])` — no named constant or function, rewrite would guess wrong"
  - "R6 unknown: function called from 3 different entry points with different assumptions about pre-conditions"
  - "R1 fragmented: function does I/O, domain decision, PDF generation, and LINE send — 4 separate concerns"

suggested_action: "Extract domain rules into named functions first (isEligibleForAutoSend, shouldSkipThisWeek). Then split into: fetchReportData → generateReportPdf → sendReportToParent → markReportSent."

human_decision_needed: false
```

### Example 3: Accept with Risk — One weak axis

```yaml
unit: "calculateDomainScore"
metrics_input:
  cyclomatic_complexity: 8
  flagged_by_tools: []

axes:
  intent_cohesion: clear
  semantic_stability: high
  ai_rewrite_safety: safe
  test_derivability: easy
  domain_visibility: implicit     # scoring weights are hardcoded numbers, not named
  change_blast_radius: contained

verdict: accept_with_risk

reasoning:
  - "R5 implicit: weights [0.3, 0.25, 0.2, 0.15, 0.1] are magic numbers — unclear if these are configurable or fixed domain rules"
  - "All other axes pass; function is otherwise well-structured"

suggested_action: "Extract weights to named constants (EMOTIONAL_DOMAIN_WEIGHT = 0.3) or a config object. Low priority — acceptable to defer."

human_decision_needed: false
```

---

## Multi-unit Review Summary

When reviewing a full PR with multiple units, add a summary block at the end:

```yaml
pr_summary:
  units_reviewed: <count>
  verdicts:
    accept: <count>
    accept_with_risk: <count>
    refactor_recommended: <count>
    refactor_required: <count>

  blocking: <true if any refactor_required>
  
  top_risks:
    - "<most important finding>"
    - "<second most important>"

  human_decisions_needed: <count>
```
