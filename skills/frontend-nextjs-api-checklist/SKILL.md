---
name: frontend-nextjs-api-checklist
description: "Next.js API Route Handler development checklist covering Route Handler structure, Service Layer separation, error handling, React Query/SWR integration, security (auth, multi-tenant, input validation), and acceptance testing. Use whenever building, modifying, or reviewing any Next.js Route Handler or API endpoint — including: creating new API routes, implementing auth/permission checks in route handlers, designing error response formats, integrating React Query mutations or queries with Next.js routes, reviewing API security (tenant isolation, input validation), or debugging unexpected API status codes. Also trigger for Next.js fullstack tasks where the user writes both the frontend fetch logic and the backend route handler. Do NOT use for general React component questions, non-Next.js backends (e.g. Express, C#, Fastify), or pure frontend data fetching without a route handler involved."
---

## Mode

**Coding / Plan mode** — planning or implementing a Next.js API route:
- Load `references/checklist-full.md` for relevant phases
- Identify items not yet in your plan or todolist
- Add them as tasks — prevent common API pitfalls from being missed

**Code review mode** — reviewing completed Next.js route handler code:
- Load `references/checklist-full.md`
- Go through each phase and check whether the implementation meets the standard
- Flag any gaps as review findings

**Single route mode** — building or modifying one specific endpoint:
- Load `references/route-handler.md` for a focused step-by-step workflow
- Load `references/error-handling.md` for status code and response format reference

---

## Core Guardrails

Apply to all Next.js route handler work — check before writing any code:

- Auth verification must be server-side; never trust user ID or role from request body
- Retrieve tenant/org ID from server-side context only (session/claims/profile) — never from request body
- Business logic belongs in Service Layer — not in the Route Handler directly
- Never expose technical errors (SQL detail, RLS codes, stack trace) in the response body
- Use unified response envelope: `{ data: ... }` for success, `{ error: '...' }` for errors
- POST success returns 201, not 200
- Use `logger` — not `console.log` — for error logging

---

## Checklist Selection

| Task | Reference |
|------|-----------|
| Full API route development / PR review | `references/checklist-full.md` |
| Route Handler structure + auth | `references/checklist-full.md` Phase 1–2 |
| Service Layer design | `references/checklist-full.md` Phase 2 |
| Error handling + status codes | `references/error-handling.md` |
| React Query / SWR integration | `references/checklist-full.md` Phase 4 |
| Security audit (auth, tenant, input) | `references/checklist-full.md` Phase 5 |
| Acceptance testing (curl / unit / integration) | `references/checklist-full.md` Phase 6 |
| Single endpoint step-by-step | `references/route-handler.md` |

When in doubt, load `references/checklist-full.md`.

---

## Quick Reference

| Pattern | Standard |
|---------|----------|
| Auth check | `getUser()` server-side — first thing in handler |
| Unauthenticated | Return 401 immediately |
| Permission denied | Return 403 |
| Tenant ID source | Server-side profile/session/claims — never request body |
| Business logic | Service Layer (`src/services/` or `lib/api/`) |
| POST success | 201 |
| Validation failure | 422 + `{ error, fields }` |
| Unexpected error | 500 + generic message, log internally |
| Response format | `{ data }` / `{ error }` — no raw DB output |
| Logging | `logger.error(...)` — not `console.log` |
