---
name: semantic-llm-review
description: Perform semantic code review that goes beyond rule-checking and linting. Use this skill when reviewing code changes (PR, diff, commit) to detect intent mismatches, hidden assumptions, boundary condition gaps, and cross-layer inconsistencies that no static rule can catch. The core method is Context Engineering — assembling requirements, design docs, and codebase patterns as the semantic ground truth, then reasoning about whether the code actually does what it's supposed to do. Use this whenever a code change touches business logic, data access, auth, or any behavior that has a "should" behind it.
---

# Semantic LLM Review

Rule-based review checks "does this follow the rules?" Semantic review asks "does this code do what it's supposed to do, and what could go wrong that nobody wrote a rule for yet?"

The key insight: **semantics don't come from the prompt — they come from the context you assemble.** A clever prompt with no context produces generic advice. A simple prompt with rich context produces project-specific, actionable findings.

## When to use

- Reviewing any code change that involves behavior (not just formatting or renaming)
- After static tools (linters, SAST) have passed — this catches what they can't
- When the change touches logic that has requirements or design intent behind it
- When you suspect "it works but is it correct?"

## Core Concept: Context Engineering

The quality of semantic review is determined by the context, not the prompt.

```
Context quality hierarchy (strongest → weakest):

1. Requirements docs     → what the code SHOULD do
2. Design docs           → why the architecture is this way
3. Existing codebase     → what patterns this project uses
4. Related tests         → what behavior is expected
5. PR diff               → what changed
6. General LLM knowledge → generic programming wisdom (weakest, most hallucination-prone)
```

A semantic review grounded in requirements catches "this code doesn't match the spec." A review grounded only in general knowledge catches "this code looks weird" — much less useful.

Read `references/context-assembly.md` for how to gather and structure context for any project.

---

## Workflow

### Step 1: Identify the Semantic Source

Before reviewing any code, answer: **"What is the ground truth for what this code should do?"**

Find the strongest available source:

| Source | Where to look | Strength |
|--------|--------------|----------|
| Requirements / specs | `docs/`, `specs/`, issue tracker, PRD | Strongest — defines intent |
| Design docs | Architecture docs, ADRs, design.md | Strong — defines structure |
| Existing tests | Test files for the changed code | Good — defines expected behavior |
| Similar code | Other files following the same pattern | Moderate — defines convention |
| Commit message / PR description | The change itself | Weak — author's claim, not verified |

If no requirements exist, the semantic source becomes the existing tests + codebase patterns. State this explicitly in the review output — "No requirements doc found; review grounded in existing test expectations and codebase conventions."

### Step 2: Assemble Context

Gather context relevant to the diff. Do NOT dump everything — select based on what the diff touches.

Read `references/context-assembly.md` for the selection strategy.

**Context budget:** Keep total context under ~30K tokens to avoid Context Rot (LLM ignoring middle content). Place the most important context at the beginning and end.

### Step 3: Apply the 4 Review Dimensions

For each meaningful unit in the diff, evaluate these 4 dimensions. Read `references/review-dimensions.md` for detailed criteria.

| Dimension | Core Question |
|-----------|--------------|
| Intent Alignment | Does this code actually implement what the requirements/design say? |
| Boundary Reasoning | What happens at the edges — null, empty, concurrent, timeout, overflow? |
| Hidden Assumptions | What does this code assume that isn't stated in its interface? |
| Cross-layer Consistency | Is this change consistent with the layers above and below it? |

These are not checklists — they are thinking frameworks. The goal is to reason about the code, not to tick boxes.

### Step 4: Apply Critic Pass (Optional but Recommended)

After the initial review, do a second pass with a different mindset. Read `references/multi-agent-critique.md` for the method.

The simplest version: after writing your findings, ask yourself — "What did I miss? What would a hostile reviewer say about my review?"

### Step 5: Output Findings

Produce structured findings. Each finding must have:

```yaml
findings:
  - severity: critical | warning | info
    dimension: intent_alignment | boundary_reasoning | hidden_assumption | cross_layer_inconsistency
    location: "file:line (or function name)"
    description: "What the problem is — specific, not vague"
    grounding: "Why this is a problem — reference to requirement, design doc, or test expectation"
    suggestion: "What to do about it"

confidence: high | medium | low
confidence_reason: "What context was available vs missing"

semantic_source: "requirements.md section X | existing tests | codebase convention | general knowledge"
```

**Critical rules for findings:**
- Every finding must reference its grounding (requirement, test, or convention)
- If grounding is "general knowledge" only, mark confidence as `low`
- If you're unsure, say so — "Possible issue, needs human verification" is better than a false positive
- No vague findings like "consider refactoring" — be specific about what and why

---

## What This Skill Does NOT Do

- Does not replace linters or static analysis (those should run first)
- Does not check formatting, naming conventions, or style (rule-based tools do this)
- Does not guarantee finding all bugs (no tool does — 42-48% detection rate is state of art)
- Does not make merge/reject decisions (it provides findings; humans decide)

---

## Anti-patterns to Avoid

**Generic advice without grounding:**
```
❌ "Consider adding error handling"
✅ "Line 45: db.query() has no error handling. Per design.md section 3.2,
    all Supabase calls must handle {data, error} return. If error is non-null
    here, the function returns undefined, which caller processReport() doesn't check."
```

**Hallucinated requirements:**
```
❌ "This violates the requirement that all functions must be under 50 lines"
✅ "No explicit requirement found for function length. However, this 120-line
    function has 3 distinct concerns (R1 fragmented) which makes boundary
    reasoning difficult."
```

**Reviewing what didn't change:**
```
❌ Reviewing the entire file when only 5 lines changed
✅ Reviewing the changed lines + their immediate context (callers, callees)
```

---

## Tool Integration

Semantic review is most powerful when combined with observability and testing tools. See `../tool-integration.md` for:

- **Contract Testing (Pact)** — Verify API behavior matches spec
- **API Testing (Bruno/Karate)** — Validate endpoints before review
- **Mutation Testing (Stryker)** — Verify test quality
- **Property Testing (fast-check)** — Explore boundary conditions
- **Observability (Langfuse/PostHog)** — Trace agent reasoning and production behavior

These tools provide the empirical ground truth that semantic review reasons about.

---

## Reference Files

- `references/context-assembly.md` — How to select and structure context for any project
- `references/review-dimensions.md` — The 4 review dimensions with examples
- `references/multi-agent-critique.md` — How to do the Reviewer + Critic pattern
- `../tool-integration.md` — Tool stack for observability, testing, and verification
