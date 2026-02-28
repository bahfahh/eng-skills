---
name: ai-first-code-review
description: Perform AI-first semantic code review using the 6-axis judgment framework (Intent Cohesion, Semantic Stability, AI-Rewrite Safety, Test Derivability, Domain Visibility, Change Blast Radius). Use this skill whenever reviewing a PR, diff, or code change — especially when traditional metrics (cyclomatic complexity, LOC) have already flagged a file, or when the change touches business logic, auth, multi-tenant isolation, or complex domain rules. This skill treats metrics as risk radar and AI as the primary semantic judge.
---

# AI-First Code Review

This skill performs semantic code review using 6 judgment axes. Traditional metrics (Cyclomatic Complexity, LOC) only tell you *where* to look — this skill tells you *what to think* about what you find there.

The core idea: a 500-line function with clear intent and stable semantics is safer than a 30-line function with hidden domain rules and unpredictable blast radius.

## When to use this skill

- A PR diff is available for review
- Static analysis tools (ESLint, Roslyn, SonarQube) have flagged files with high complexity
- The change touches: business logic, auth flows, multi-tenant data access, domain rules, or shared utilities
- You want a verdict beyond "complexity is too high"

## Workflow

### Step 1: Collect Metrics Input (if available)

Before semantic analysis, gather available tool output to use as risk signals — not verdicts.

**TypeScript/JavaScript projects:**
```bash
# Get complexity warnings from ESLint
npx eslint --rule '{"complexity": ["warn", 10]}' <changed-files> --format json

# Or if SonarCloud is configured, fetch the metrics API
```

**C# projects:**
```bash
# Build with Roslyn analyzers to get SARIF output
dotnet build /p:RunAnalyzersDuringBuild=true /p:ErrorLog=analysis.sarif

# Or use dotnet-sonarscanner results
```

Extract from tool output:
- Which functions/methods have CC > 10 or Cognitive Complexity > 15
- Which files have LOC > 120 in changed sections
- Any security hotspots flagged

These become the **priority targets** for semantic review — not automatic failures.

### Step 2: Apply the 6-Axis Judgment

For each flagged function/module (or the entire diff if no tools available), evaluate all 6 axes. Read `references/judgment-axes.md` for detailed criteria and examples.

**Quick reference:**

| Axis | Core Question |
|------|--------------|
| R1 Intent Cohesion | Can you describe this in one sentence without "and"? |
| R2 Semantic Stability | Does behavior depend on hidden state or call order? |
| R3 AI-Rewrite Safety | Could another AI rewrite this correctly without history? |
| R4 Test Derivability | Can test cases be derived from reading the code alone? |
| R5 Domain Visibility | Are business rules named and findable? |
| R6 Change Blast Radius | If one condition changes, what breaks? |

Evaluate each axis independently. A function can score well on R1 but poorly on R5 — that's useful information.

### Step 3: Determine Verdict

Apply the acceptance rule:

**Accept as-is** if ALL of these hold:
- R1: `clear`
- R2: `high`
- R3: `safe`
- R4: `easy`
- R5: `explicit`
- R6: `contained`

**Accept with risk** if most axes pass but 1–2 are `medium`/`partial`/`implicit` — document the risk.

**Refactor recommended** if any axis is `low`/`risky`/`hidden`/`wide` — explain what specifically needs to change.

**Refactor required** if R3 is `unsafe` OR R6 is `unknown` — these are the two axes that indicate the code is actively dangerous to maintain.

### Step 4: Output Structured Verdict

Always produce the structured YAML output. See `references/output-format.md` for the full schema and examples.

Minimum required output per reviewed unit:

```yaml
unit: "FunctionName / ClassName / module path"
metrics_input:
  cyclomatic_complexity: <number or null>
  flagged_by_tools: [list of tool warnings, or empty]

axes:
  intent_cohesion: clear | mixed | fragmented
  semantic_stability: high | medium | low
  ai_rewrite_safety: safe | risky | unsafe
  test_derivability: easy | partial | hard
  domain_visibility: explicit | implicit | hidden
  change_blast_radius: contained | wide | unknown

verdict: accept | accept_with_risk | refactor_recommended | refactor_required

reasoning:
  - <bullet: specific observation with file/line reference>
  - <bullet: why this matters>

suggested_action: <one concrete next step, or null if accept>
```

Keep reasoning bullets concise — the human reviewer reads verdict + reasoning only.

### Step 5: Escalate to Human if needed

Flag for human decision when:
- Verdict is `refactor_required` but the refactor cost is high (large surface area)
- R6 blast radius is `unknown` and the change is in a shared utility
- The code is intentionally complex due to a domain constraint (e.g., regulatory logic)

In these cases, present the trade-off clearly: "This function has hidden domain rules (R5: hidden) but refactoring requires touching 8 call sites. Recommend human decision on whether to accept the risk or invest in refactor."

---

## Key principles

**Complexity is not the enemy — invisible complexity is.**
A complex function that is well-named, well-tested, and has a contained blast radius is safer than a simple function with hidden state and unclear intent.

**Metrics trigger review, they don't determine outcome.**
CC > 10 means "look here carefully." It does not mean "this must be refactored." The 6-axis judgment determines the outcome.

**R3 (AI-Rewrite Safety) is the forward-looking axis.**
In an AI-assisted development world, code that cannot be safely rewritten by an AI without full history is a long-term liability — even if it works today.

---

## Reference files

- `references/judgment-axes.md` — Detailed criteria, fail patterns, and examples for each of the 6 axes
- `references/output-format.md` — Full output schema with annotated examples
