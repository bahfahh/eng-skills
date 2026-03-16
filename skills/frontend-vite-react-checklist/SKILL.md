---
name: frontend-vite-react-checklist
description: "Vite + React frontend development quality checklist serving two modes. (1) Coding / plan mode — trigger before writing Vite+React code or when planning a React UI task: adds Vite-specific, React Router v6+, TanStack Query, and state management concerns to the todolist that AI agents commonly miss. (2) Code review mode — trigger when reviewing completed Vite+React code: audit Vite config, routing patterns, data fetching strategy, bundle optimization, and env variable safety. Always trigger for Vite+React projects — use alongside frontend-checklist for comprehensive UX coverage."
---

<!-- Vite + React specific checklist. Two modes: (1) coding/plan — add missing items to todolist; (2) code review — audit against references. -->

## Mode

**Coding / Plan mode** — planning or about to implement a Vite+React feature:
- Read the reference file for your task type (table below)
- Identify items not yet in your plan or todolist
- Add them as tasks before writing code
- Always check Core Guardrails first — these block the most common Vite+React mistakes

**Code review mode** — reviewing completed Vite+React code:
- Load `references/checklist-full.md`
- Audit each section: Vite config, routing, state/data, build, env safety
- Flag gaps as review findings

---

## Core Guardrails

Check before writing any Vite+React code:

- Never use `useEffect` to fetch server data — use TanStack Query (`useQuery` / `useMutation`)
- Never store server-fetched data in `useState` alongside query state — single source of truth
- Route-level code splitting with `React.lazy` + `Suspense` for every page component
- All env vars that reach the client must be prefixed `VITE_` — never expose secrets
- Path aliases must be defined in both `vite.config.ts` AND `tsconfig.json` — drift causes silent build errors
- No barrel imports (`index.ts` re-exports) — import directly from source to keep tree-shaking effective
- `React.StrictMode` must be on in dev — double-invokes effects to catch impurity early

---

## Checklist Selection

| Task | Reference to load |
|------|-------------------|
| Full feature / PR review | `references/checklist-full.md` |
| Routing / navigation / protected routes | `references/routing-state.md` |
| Data fetching / server state / mutations | `references/routing-state.md` |
| Vite config / aliases / proxy / HMR issues | `references/vite-build.md` |
| Build optimization / chunk splitting / production | `references/vite-build.md` |

When in doubt, load `references/checklist-full.md`.

---

## Quick Reference

| Standard | Value |
|----------|-------|
| Route lazy load | Every page-level component |
| Query stale time (typical) | 30s–5min depending on data freshness needs |
| Mutation optimistic update | Only when rollback is safe to implement |
| Env var for client | Must start with `VITE_` |
| Alias definition | Both `vite.config.ts` + `tsconfig.json` |
| Bundle chunk threshold (warn) | > 500 KB per chunk |
| HMR not working | Check `server.host` + `fs.allow` in vite config |
