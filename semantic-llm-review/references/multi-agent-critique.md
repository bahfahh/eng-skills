# Multi-Agent Critique — Reviewer + Critic Pattern

A single LLM reviewing code has confirmation bias — it tends to agree that the code is correct. A second pass with a different mindset catches what the first pass missed.

Research shows this improves bug detection by ~11%. The cost is one additional LLM call.

---

## Why Single-Pass Review Is Insufficient

When an LLM reviews code, it:
1. Reads the code
2. Forms an initial understanding of what it does
3. Evaluates based on that understanding

The problem: step 2 anchors step 3. If the LLM's initial understanding is wrong (or incomplete), the evaluation is wrong too. This is the same confirmation bias humans have.

The fix: a second pass that explicitly challenges the first pass's conclusions.

---

## The Simplest Version (Same Agent, Two Passes)

No infrastructure needed. Just two sequential prompts.

**Pass 1 — Reviewer:**
```
Review this code change. Here is the context:
[requirements, diff, related code]

Evaluate using the 4 dimensions:
- Intent Alignment
- Boundary Reasoning
- Hidden Assumptions
- Cross-layer Consistency

Output your findings.
```

**Pass 2 — Critic:**
```
Here is a code review that was just performed:
[Pass 1 findings]

Here is the same code and context:
[requirements, diff, related code]

Your job is to challenge this review:
1. What did the reviewer miss?
2. Are any of the reviewer's findings wrong or overstated?
3. What boundary conditions or assumptions were not considered?

Be adversarial — assume the review is incomplete.
```

**Merge:** Combine findings from both passes. Remove duplicates. If the Critic contradicts the Reviewer, include both perspectives and mark as "needs human judgment."

---

## The Two-Agent Version

If your system supports parallel or sequential agent calls:

**Agent A (Reviewer):**
- Role: Thorough, systematic reviewer
- Input: Context + diff
- Output: Structured findings with dimensions and grounding
- Mindset: "Find everything that could be wrong"

**Agent B (Critic):**
- Role: Adversarial challenger
- Input: Context + diff + Agent A's findings
- Output: Missed issues + challenges to Agent A's findings
- Mindset: "What did Agent A miss? Where is Agent A wrong?"

**Key design decisions:**

| Decision | Recommendation | Why |
|----------|---------------|-----|
| Sequential or parallel? | Sequential (B sees A's output) | B needs A's findings to challenge them |
| Same model or different? | Same model is fine | The different prompt/role is what matters |
| Same context? | Yes, both get full context | B needs the same information to find gaps |

---

## Critic Prompt Design

The Critic prompt is the most important part. It must explicitly push against the Reviewer's conclusions.

**Effective Critic prompt structure:**

```markdown
You are reviewing a code review that was just performed. Your job is NOT to
agree with the reviewer. Your job is to find what they missed.

## The reviewer's findings:
[findings from Pass 1]

## The code and context:
[same context as Pass 1]

## Your tasks:

1. MISSED ISSUES: What problems exist in the code that the reviewer
   did not mention? Focus on:
   - Boundary conditions not considered
   - Hidden assumptions not surfaced
   - Cross-layer inconsistencies not checked
   - Intent mismatches not caught

2. FALSE POSITIVES: Are any of the reviewer's findings incorrect?
   Did the reviewer flag something that is actually fine? If so, explain why.

3. SEVERITY CALIBRATION: Are any findings over- or under-rated?
   A "critical" that's actually a "warning"? A "warning" that's actually "critical"?

Output your findings in the same structured format.
Mark each finding as:
  source: "critic_new" (missed by reviewer)
  source: "critic_challenge" (disagreeing with reviewer)
  source: "critic_recalibrate" (adjusting severity)
```

---

## Merging Findings

After both passes, merge:

```
1. All Reviewer findings that Critic did not challenge → keep as-is
2. Reviewer findings that Critic challenged → include both perspectives,
   mark as "disputed — needs human judgment"
3. New findings from Critic → add with source: "critic_new"
4. Severity recalibrations → use Critic's severity with note
5. Remove exact duplicates
```

---

## When to Use Multi-Agent vs Single-Pass

| Situation | Recommendation |
|-----------|---------------|
| Small diff (< 50 lines), low-risk area | Single pass is sufficient |
| Large diff or complex business logic | Use Critic pass |
| Auth, security, or multi-tenant changes | Always use Critic pass |
| Metrics flagged high complexity | Use Critic pass |
| No requirements docs available | Use Critic pass (higher uncertainty) |

The Critic pass roughly doubles the LLM cost. Use it where the risk justifies the cost.

---

## Common Critic Findings

In practice, Critics most often catch:

1. **Boundary conditions the Reviewer assumed were handled** — "Reviewer said error handling is fine, but line 45 only catches network errors, not auth errors"

2. **Cross-layer issues the Reviewer didn't trace** — "Reviewer checked the API function but didn't verify the hook passes the new parameter"

3. **Over-confident severity** — "Reviewer marked this critical, but the RLS policy at the database level already prevents this scenario"

4. **Under-confident severity** — "Reviewer marked this as info, but this hidden assumption (sorted array) will break when the index is removed"
