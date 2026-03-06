# Stack Rules: C# / .NET

Preserve behavior. Default stance: simpler is correct.

## Over-Engineering Patterns to Challenge

- `IRepository<T>` wrappers over `DbContext` without added value
- CQRS/MediatR for simple CRUD
- interface-per-class with single implementation (no real polymorphism)
- factory/builder where variation does not exist
- strategy pattern where strategy never changes
- AutoMapper for trivial mapping

## Clarity and Safety

- reduce nesting with guard clauses
- keep domain rules named and explicit
- avoid changing async/sync shape
- preserve logging/tracing behavior
- preserve error-handling boundaries

## Output Expectation

When flagging over-engineering, state:
- location
- specific cost (mental overhead, boilerplate, test complexity)
- minimal alternative (remove layer, inline mapping, use DbContext directly)

