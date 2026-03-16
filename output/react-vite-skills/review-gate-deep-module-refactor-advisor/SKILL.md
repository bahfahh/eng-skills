---
name: review-gate-deep-module-refactor-advisor
description: Audit a mature codebase for shallow modules, high coupling, and call-site “recipes”, then propose deep-module (facade) encapsulation options with a strict requirements+tests gate to avoid over-engineering. Use during refactors after features are mostly stable. Output an analysis report with relationship evidence (graph or recipe), readiness (ready/not_ready), and 2 options per candidate (minimal wrapper vs proper deep module). Supports TS/JS and .NET/C# by reading the repo (no scripts). Do not modify code by default.
---

# Deep Module Refactor Advisor

Goal: produce a **refactor advisory report** (no code changes) that helps a developer decide:
- Where deep modules (facades) will reduce coupling/cognitive load.
- Where encapsulation is **NOT** appropriate yet (unstable requirements or missing tests).

## Inputs

- Default: scan the whole repository.
- If the user provides scope (paths/modules/keywords): analyze only that scope.

## Hard Guardrails (Avoid Over-Engineering)

Only recommend a **Proper Deep Module** if BOTH are true:
1) Relevant behavior is grounded in `requirements.md` / specs (or equivalent requirement docs).
2) There are meaningful tests that pin behavior at a boundary (integration/contract/e2e preferred).

If either is missing:
- Mark the candidate `not_ready`.
- Recommend what to add first: `spec_gaps` and/or `tests_to_add`.
- Do **not** propose large encapsulation/refactors that would “freeze” unclear behavior.

## What to Look For (Signals)

Prioritize areas that show one or more of these:
- **Call-site recipe**: many call sites repeat the same ordered sequence to achieve one business intent.
- **Shallow modules**: lots of small helpers, but callers still need to know the “how”.
- **Interface depth**: one business action requires many technical parameters (leaky facade).
- **Wrong test target**: utils have unit tests but the real boundary/flow has no black-box tests.

## Workflow

1) Find requirement sources
   - Prefer: `requirements.md`, `specs/**/requirements.md`, `docs/**` requirement/spec files.
   - If none exist for the scope, say so explicitly and treat encapsulation as `not_ready` by default.

2) Map code to business flows (so humans understand “why”)
   - From the requirement sources, extract the top-level business flows/use-cases in plain words.
   - For each hotspot you pick later, attempt a best-effort mapping:
     - `business_flow`: a short, human-readable name for the flow
     - `requirement_trace`: links to the most relevant requirement sections/headings (or file+heading)
   - If you cannot map confidently, set:
     - `business_flow: unknown`
     - `requirement_trace: missing`
     - and list what keyword/path you tried (briefly).

3) Identify Top 5 hotspots (or fewer if scope is small)
   - Use the signals above to rank candidates.
   - Pick modules/services/flows (not single tiny helpers) as the primary units.

4) For each hotspot, assemble relationship evidence (“why”)
   - Provide **either**:
     - a small Mermaid dependency/call graph, **or**
     - a “Recipe Today” list + representative call sites (2–5 examples).

5) Decide readiness
   - `ready`: requirements + boundary tests exist.
   - `not_ready`: missing requirements and/or missing boundary tests.

6) Output 2 options per candidate
   - **Option A — Minimal Wrapper**: low-cost wrapper that collapses the call-site recipe into 1 entrypoint.
   - **Option B — Proper Deep Module**: a stable facade with domain-language I/O and hidden complexity.

## Output (Markdown)

Use this template:

```md
# Deep Module Refactor Advisory

## Executive Summary
- scope: <whole-repo | paths/keywords>
- languages: <detected>
- requirement_sources: <found paths or “none”>
- business_flows_detected: <1–10 short bullets, from requirements (if any)>

## Top 5 Hotspots
| # | Candidate | Business flow | Why it’s a hotspot (1 sentence) | Readiness | Next step |
|---:|---|---|---|---|---|

## Candidate Details

### 1) <Candidate Name> (<paths>)
- Status: ready | not_ready
- Why (plain words): <what hurts today>
- Business flow: <short business flow name | unknown>
- Requirement trace (if any):
  - `<path/to/requirements.md#Heading>` (or section name)
  - `<path/to/spec.md#Heading>`
- Grounding note: <if missing, say “missing” and what you tried>

#### Relationship evidence (pick one)
```mermaid
graph LR
  A["Entry Point"] --> B["Current Helpers / Services"]
  B --> C["Data Access / External"]
```
OR
- Recipe Today: A → B → C → D
- Representative call sites:
  - `path/to/file.ts:FunctionName`
  - `path/to/file.cs:MethodName`

#### Option A — Minimal Wrapper
- Proposed entrypoint (name in domain language): `<...>`
- What moves inside: <bullets>
- What stays outside: <bullets>
- Tests to add (if missing): <1–3 black-box tests at boundary>

#### Option B — Proper Deep Module
- Facade interface (domain language I/O): `<input> -> <output>`
- What moves inside / stays outside: <bullets>
- God-service guardrail: <if multiple responsibilities, keep facade thin and split internals>
- Tests to add/adjust: <list>

If `Status = not_ready`, replace the above with:
- Why not ready: <missing spec sections and/or missing boundary tests — be specific>
- Spec gaps: <list requirement sections that are unclear or absent>
- Tests to add first: <1–3 black-box integration tests that must exist before encapsulating>

## Do Not Encapsulate Yet (if any)
List areas where behavior is still changing or requirements/tests are missing.

## Requirement Coverage (Optional but recommended if requirements exist)
| Candidate | Business flow | Requirement trace | Coverage confidence |
|---|---|---|---|
```
