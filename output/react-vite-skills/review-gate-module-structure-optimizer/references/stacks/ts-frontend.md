# Stack Rules: TypeScript / React (Frontend)

Preserve behavior. Optimize for clarity and maintainability.

1) Prefer explicit code over clever one-liners.
2) Avoid nested ternaries; use if/else or switch.
3) Avoid redundant state (do not store derived values).
4) Separate concerns: fetching/auth/state vs presentational UI.
5) Use consistent naming aligned with domain language.
6) Ensure error paths have user-visible handling in UI flows.

When simplifying:
- remove dead code and unused branches
- remove redundant abstractions that add indirection without value
- keep interfaces stable; do not change public contracts unless explicitly requested

