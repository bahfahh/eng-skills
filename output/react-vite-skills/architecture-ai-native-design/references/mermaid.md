# Mermaid templates (optional)

Use at most 0–1 diagram unless the user explicitly asks for more.

## Purpose (avoid misreading)
These diagrams are **conceptual coupling/flow sketches**, not framework-specific blueprints.
- Rename nodes to match your system vocabulary.
- Use diagrams to check **responsibility boundaries** and **dependency direction**.
- Do not diagram every helper function; only show the stable concepts/entrypoints.

## Vocabulary mapping
See "Stack-agnostic vocabulary" in SKILL.md for the canonical definitions. Rename diagram nodes to match your system's actual vocabulary.

## Coupling map (dependency graph)
```mermaid
flowchart TD
  P[Presentation] --> APP[Application/Facade]
  APP --> DOM[Domain]
  APP --> INF[Infra Adapters]
  INF --> EXT[(External systems)]
```

## Use-case orchestration (sequence)
```mermaid
sequenceDiagram
  participant P as "Presentation"
  participant APP as "Application/Facade"
  participant DOM as "Domain"
  participant INF as "Infra Adapter"
  P->>APP: doUseCase(input)
  APP->>INF: load state (I/O)
  INF-->>APP: state
  APP->>DOM: decide(state, input, policy)
  DOM-->>APP: next state OR error code
  APP->>INF: persist with guard (I/O)
  INF-->>APP: ok/conflict
  APP-->>P: Result (ok/error code)
```

## Policy/version selection
```mermaid
flowchart LR
  C[Call site] --> S[Select policy/version in one place]
  S --> V1[policy v1]
  S --> V2[policy v2]
  V1 --> DOM[Domain rules]
  V2 --> DOM
```

## Notes
- Keep diagrams semantic: show responsibilities and dependencies, not every helper function.
- In `sequenceDiagram`, participants should be active entities (UI/service/domain/adapter), not passive data objects.
- Quote any participant alias or label that contains spaces or non-ASCII characters.
