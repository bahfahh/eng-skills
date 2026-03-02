---
name: ai-native-design
description: Language-agnostic AI-native architecture and refactor planning. Use when code feels hard to change, modules are getting tangled, you need to design module boundaries or reduce coupling, or you want a structured refactor plan with layered testing and migration steps.
---

# AI-Native Design (Principles Skill)

## Goal
Produce AI-friendly design/refactor plans that minimize:
- reasoning load (shallow call graph, fewer indirections)
- change blast radius (change-locality)
- hidden coupling (explicit contracts, explicit invariants)

Default output: a concise, prioritized suggestion list (not a long memo).

## Stack-agnostic vocabulary (map to your tech)
This skill is language/framework neutral. Translate these terms to your stack:
- **Presentation**: entrypoints/triggers (anything that receives an external request or starts work).
- **Application/Facade (use-cases)**: orchestration (multi-step recipe, dependency coordination, error mapping).
- **Domain**: business rules, invariants, policies, decisions (pure-ish logic).
- **Infra/Adapters**: I/O and integration details (DB, network, filesystem, SDKs, queues).

## Operating principles
- Prefer **few, high-level entrypoints** over many tiny helpers.
- Prefer **semantic names** over generic abstractions (avoid "Manager/Helper/Util" without domain meaning).
- Prefer **explicit contracts** (types, Result/error codes, invariants) over implicit behavior.
- Prefer **policy/version** for changing rules instead of scattering conditional logic across layers.
- Keep **Domain logic pure** when possible; keep I/O in adapters; keep orchestration in a facade.

## Workflow (principle-driven)
1) Frame the change
- State goal, constraints, and non-goals.
- List invariants that must always hold (limits, safety rules, data isolation, etc.).

2) Map coupling (fast, not perfect)
- Identify call sites, duplicated recipes, cross-layer logic, and shared state.
- If needed, include **0–1 Mermaid** diagram for the coupling map (keep it conceptual; label nodes using your system vocabulary).

3) Identify hotspots
- Duplication at 2+ call sites
- Multi-step recipes (e.g., upload -> update -> invalidate, retry, offline queue)
- High churn areas (frequent requirement changes)
- Places enforcing invariants/limits/concurrency (often belong in deep modules)

4) Choose boundaries (responsibility split)
- **Domain**: rules, invariants, decisions (pure-ish)
- **Application/Facade**: use-case orchestration (multi-step flows, retries, error mapping)
- **Infra/Adapters**: DB/storage/network/UI toolkit specifics

5) Choose encapsulation technique
- **Wrapper**: normalize call shape, defaults, error mapping; reduce repetition.
- **Facade**: provide a single high-level use-case API; hide multi-step recipes.
- **Deep Module**: keep interface small; move complexity inside; protect invariants.

6) Design the contract
- Define 1–2 stable entrypoints.
- Prefer Result/error codes for expected failures; keep exceptions for truly unexpected faults.
- Make dependencies explicit (inputs/outputs), and keep the call graph shallow.

7) Add policy/version for changing rules
- Represent changing rules as **data** (policy object) + **stable identifiers** (policyId/version).
- Centralize selection of policy/version in one place (do not scatter conditionals across layers).
- Persist policy/template version with data when backward compatibility matters.

8) Wrap up: propose tests + migration plan
These are output deliverables, not a new analysis step. Produce alongside the boundary/contract decisions above.
- Domain: property/invariant tests.
- Application: integration tests for orchestration (BDD naming is fine here).
- E2E: minimal critical-path smoke; assert UX signals and "no infinite loading".
- Migration: wrapper-first/strangler; move call sites gradually; keep compatibility.

## Default response format
Return a concise plan using these sections:
- Findings (coupling/hotspots)
- Proposed boundaries (Domain/Application/Infra)
- Contracts (entrypoints, Result/errors, key types)
- Policy/version (what changes, how versions are selected/persisted)
- Tests (domain/integration/e2e)
- Migration steps (ordered, low risk)
- Risks / non-goals

## Avoid common misunderstandings
- Do not treat these layers as folders you must create. They are responsibilities; map them onto your architecture.
- Do not put presentation-specific details into Domain rules.
- Do not make Domain depend on I/O clients. Push I/O into adapters and call Domain from Application.
- Do not over-abstract for \"reuse\". Prefer reducing coupling and improving change-locality.

## Optional references
- For copy-paste output templates: `references/templates.md`
- For Mermaid diagram templates: `references/mermaid.md`

## Note on agents/openai.yaml
This file configures OpenAI GPT Actions integration (`allow_implicit_invocation: false`). It has no effect in Claude Code — safe to ignore unless deploying to a GPT Actions environment.
