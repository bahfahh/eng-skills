---
name: design-patterns
description: Guide for selecting appropriate design patterns during software development. Use whenever the user asks how to structure code, which approach is better, how to clean up messy if-else chains, how to make code more extensible, or how to integrate two different APIs/systems. Also trigger for explicit pattern questions, architectural decisions, new class/module design, refactoring, or any "what's the best way to implement this?" question. Prioritizes AI-friendly patterns and steers away from over-engineering.
---

# Design Patterns Selection Guide

## Core Principles

1. Only introduce a pattern when problem complexity justifies it — simple problems get simple solutions
2. Prefer patterns AI can generate correctly and maintain easily
3. Avoid AI anti-patterns (Singleton, Visitor) unless explicitly necessary
4. Align with SOLID principles

## Quick Reference

| Pattern | AI Rating | Use When |
|---------|-----------|----------|
| Strategy | ★★★★★ | Switching algorithms/behaviors at runtime |
| Factory Method | ★★★★★ | Creating objects by condition without modifying core logic |
| Adapter | ★★★★★ | Integrating incompatible interfaces/APIs |
| Builder | ★★★★☆ | Constructing objects with many parameters |
| Facade | ★★★★☆ | Simplifying complex subsystem access |
| Command | ★★★★☆ | Task queues, undo/redo, audit trails |
| Chain of Responsibility | ★★★★☆ | Multi-step pipelines, middleware chains |
| Proxy | ★★★☆☆ | Caching, access control, lazy loading |
| State | ★★★☆☆ | Finite state machines (persist state to DB) |
| Decorator | ★★☆☆☆ | Dynamic behavior extension (limit to 2-3 layers) |
| Observer | ★★☆☆☆ | Event-driven systems (implicit control flow is hard to debug) |
| Template Method | ★★☆☆☆ | Algorithm skeletons (deep inheritance causes override errors) |
| Singleton | ★☆☆☆☆ | **Avoid** — use Dependency Injection instead |
| Visitor | ★☆☆☆☆ | **Avoid** — only for compilers/AST processing |

## Decision Flow

```
Requirement Analysis
  │
  ├─ Create different objects by condition? → Factory Method
  ├─ Construct complex object with many params? → Builder
  ├─ Integrate incompatible interfaces? → Adapter
  ├─ Simplify complex subsystem access? → Facade
  ├─ Switch algorithms at runtime? → Strategy
  ├─ Encapsulate operations as executable objects? → Command
  ├─ Multi-step processing chain? → Chain of Responsibility
  ├─ Caching / access control / lazy loading? → Proxy
  ├─ Object has distinct lifecycle states with clear transitions? → State (★★★☆☆ — persist to DB)
  ├─ Add optional behaviors to an object without subclassing? → Decorator (★★☆☆☆ — max 2-3 layers)
  ├─ Algorithm skeleton with variable steps across subclasses? → Template Method (★★☆☆☆ — max 2 levels)
  ├─ Decouple event producers from consumers in event-driven arch? → Observer (★★☆☆☆ — AI caution)
  └─ None of the above? → No pattern needed, keep it simple
```

## Anti-Pattern Warnings

- **Singleton**: Leads to hidden dependencies, global state drift, untestable code. AI tends to misuse it for quick data-passing fixes. Replace with DI container.
- **Visitor**: Dual coupling — adding an Element requires modifying all Visitors. AI frequently forgets to implement new node handling. Only use for compiler/AST domains.
- **Decorator**: When nesting exceeds 2-3 layers, AI cannot correctly reason about execution order. Consider AOP (annotation-based) as alternative.
- **Observer**: Implicit control flow makes it nearly impossible for AI to trace event propagation during debugging.

## Detailed Pattern Guidance

For per-pattern scenarios, AI advantages, and implementation guidance, see [references/patterns-detail.md](references/patterns-detail.md).
