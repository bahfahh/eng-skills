---
name: frontend-nextjs-checklist
description: "Next.js development quality checklist for App Router, auth/session safety, data fetching, bundle optimization, and version upgrades. Use whenever the task involves Next.js — including: planning or implementing any Next.js feature (App Router pages, middleware, Server/Client Components, server actions); debugging Next.js-specific issues (session hang, params await errors, router.push redirect loops, hydration mismatches, tab restore loading); reviewing Next.js code for auth patterns, data fetching strategy, or component boundaries; upgrading Next.js versions (14→15 breaking changes, async params/cookies/headers). Do NOT use for general React questions without Next.js, backend API reviews, Vue/Nuxt, or pure frontend UX/accessibility concerns unrelated to Next.js."
---

## Mode

**Coding / Plan mode** — planning or about to implement a Next.js feature:
- Read `references/checklist-full.md` for your relevant phases
- Identify items not yet in your plan or todolist
- Add them as tasks — prevent common Next.js pitfalls from being missed
- For performance phases (4–6), load the `react-best-practices` skill if available

**Code review mode** — reviewing completed Next.js code:
- Load `references/checklist-full.md`
- Go through each phase and check whether the implementation meets the standard
- Flag any gaps as review findings

**Upgrade mode** — upgrading Next.js version:
- Load `references/upgrade.md`
- Follow each step sequentially — do not skip smoke tests

---

## Core Guardrails

These apply to all Next.js work — check them before writing any code:

- Default to Server Components; only add `'use client'` when interactivity or browser APIs are required
- Never use session-based auth initialization that can hang in React 19+ — use user-based retrieval (e.g., `getUser()` not `getSession()`)
- After login/logout, use hard navigation (e.g., `window.location.href`) not `router.push()` — ensures fresh session and state reset
- Refresh auth state on tab focus and visibility change — prevents stale session after background tab
- Never use barrel imports (`index.ts` re-exports) — import directly from source files to avoid bundle bloat
- Run typecheck + lint + build before every PR — no silent type errors

---

## Checklist Selection

| Task | Reference |
|------|-----------|
| Full Next.js feature development / PR review | `references/checklist-full.md` |
| App Router / routing / middleware | `references/checklist-full.md` Phase 1 |
| Auth / Session implementation | `references/checklist-full.md` Phase 2 |
| Data fetching strategy | `references/checklist-full.md` Phase 3 |
| Bundle / code splitting optimization | `references/checklist-full.md` Phase 4 |
| Rendering / re-render / hydration | `references/checklist-full.md` Phases 5–6 |
| Component structure / hooks | `references/checklist-full.md` Phase 7 |
| TypeScript / build checks | `references/checklist-full.md` Phase 8 |
| Version upgrade | `references/upgrade.md` |

When in doubt about scope, default to `references/checklist-full.md`.

---

## Quick Reference

| Pattern | Standard |
|---------|----------|
| Component default | Server Component (no `'use client'`) |
| Auth initialization | `getUser()` — not `getSession()` |
| Post-login redirect | `window.location.href` — not `router.push()` |
| Background tab auth | Listen to `focus` + `visibilitychange` |
| Imports | Direct path — not barrel `index.ts` |
| Heavy components | `next/dynamic` with loading state |
| Server data dedup | `React.cache()` |
| Parallel fetching | `Promise.all()` — avoid waterfall |
| Performance rules | Load `react-best-practices` skill |
