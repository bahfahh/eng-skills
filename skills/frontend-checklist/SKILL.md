---
name: frontend-checklist
description: "Frontend development quality checklist serving two modes. (1) Coding / plan mode — trigger before writing frontend code or when a coding agent is planning a frontend task: read to build awareness and add missing UX/UI concerns to the todolist, covering interaction states, write-path patterns, API error handling, auth/session safety, and responsive design that AI agents commonly overlook. (2) Code review mode — trigger when reviewing completed frontend code: audit each concern area against checklist-full.md. Always trigger for frontend-only tasks in plan mode, before implementing any UI feature, or when doing frontend-specific code review."
---

<!-- Frontend development quality checklist. Two modes: (1) coding/plan — add missing items to todolist; (2) code review — audit against checklist-full.md. -->

## Mode

**Coding / Plan mode** — planning or about to implement a frontend feature:
- Read the reference file for your task type (table below)
- Identify items not yet in your plan or todolist
- Add them as tasks — prevent UX/UI gaps from being missed during implementation
- Focus especially on `references/uiux.md` for interaction patterns AI agents commonly omit

**Code review mode** — reviewing completed frontend code:
- Load `references/checklist-full.md`
- Go through each phase and check whether the implementation meets the standard
- Flag any gaps as review findings

---

## Core Guardrails

These apply to all frontend work — check them before writing any code:

- All write paths (create/update/delete) need interaction feedback: loading, success, and error states
- Don't manage loading with `useState` directly — use the project's standardized async action hook
- Mobile must work: touch targets ≥ 44px, keyboard doesn't cover inputs, can scroll normally
- No silent failures, no infinite spinners, no duplicate submissions on repeated clicks
- Never expose raw technical errors to users (no token strings, DB error codes, stack traces)

---

## Checklist Selection

Read the relevant reference file based on the task at hand. Use it as an awareness guide — work through it as you develop, not just at the end.

| Task | Reference to load |
|------|-------------------|
| Full UI feature development / PR review | `references/checklist-full.md` |
| Create / Update / Delete (write path) | `references/write-path.md` |
| Auth / Session / Middleware changes | `references/auth-session.md` |
| File upload / Storage | `references/storage-upload.md` |
| UI component / API errors / Accessibility / Responsive | `references/uiux.md` |

When in doubt about scope, default to `references/checklist-full.md`.

---

## Quick Reference

| Standard | Value |
|----------|-------|
| Immediate feedback | within 100ms |
| Touch target minimum | 44×44px |
| Button height (typical) | 48–56px (follow project token) |
| Loading timeout before fallback | 10–15s |
| Text/background color contrast | ≥ 4.5:1 |
| Large data threshold (consider search/filter) | > 30 items |
| Micro-interaction duration | 150–250ms |
| Larger entrance animation | 400–800ms |
