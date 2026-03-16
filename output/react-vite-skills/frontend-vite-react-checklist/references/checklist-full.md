# Vite + React Full Checklist

## Table of Contents
1. [Vite Config](#1-vite-config)
2. [Environment Variables](#2-environment-variables)
3. [Routing (React Router v6+)](#3-routing-react-router-v6)
4. [State Management](#4-state-management)
5. [Data Fetching (TanStack Query)](#5-data-fetching-tanstack-query)
6. [Component Patterns](#6-component-patterns)
7. [Performance & Code Splitting](#7-performance--code-splitting)
8. [Build & Production](#8-build--production)
9. [TypeScript Integration](#9-typescript-integration)
10. [Error Handling](#10-error-handling)

---

## 1. Vite Config

- [ ] Path aliases defined in `vite.config.ts` (e.g., `@/` → `./src/`)
- [ ] Same aliases mirrored in `tsconfig.json` under `paths` — no drift
- [ ] Dev proxy configured for API calls to avoid CORS in local dev (`server.proxy`)
- [ ] `server.host` set if running in Docker/WSL/remote environment
- [ ] `build.sourcemap` set appropriately (`false` for prod, `true` for staging)
- [ ] `optimizeDeps.include` populated for slow cold-start deps (e.g., large UI libs)

## 2. Environment Variables

- [ ] All client-side env vars start with `VITE_`
- [ ] Secret keys (API keys, DB URLs) are NOT prefixed `VITE_` — they stay server-only
- [ ] `import.meta.env.VITE_*` used in code (not `process.env`)
- [ ] `.env.example` committed with all required keys (no values)
- [ ] `.env`, `.env.local` in `.gitignore`
- [ ] Type declarations in `src/vite-env.d.ts` or `env.d.ts` for custom vars

## 3. Routing (React Router v6+)

- [ ] Route definitions centralized (single `routes.tsx` or router file)
- [ ] All page-level components lazy loaded: `const Page = React.lazy(() => import('./Page'))`
- [ ] `<Suspense fallback={<Spinner />}>` wraps lazy routes
- [ ] 404 catch-all route defined (`path="*"`)
- [ ] Protected routes: auth check happens server-side or via loader, not inside component render
- [ ] Navigation after mutation uses `useNavigate` (not `window.location.href` unless full reload needed)
- [ ] Scroll restoration handled (React Router `<ScrollRestoration />` or manual)
- [ ] URL state used for shareable filters/pagination (not local state)

## 4. State Management

- [ ] Server state (API data) managed by TanStack Query — not `useState` + `useEffect`
- [ ] Client-only global state (UI state, theme, user preferences) in Zustand or Jotai
- [ ] No prop drilling beyond 2 levels — use context or store
- [ ] Form state managed by React Hook Form or Formik — not manual `useState` per field
- [ ] Derived state computed inline or with `useMemo` — not duplicated in separate state

## 5. Data Fetching (TanStack Query)

- [ ] `queryKey` is deterministic and includes all variables the query depends on
- [ ] `enabled` flag used to prevent queries running before prerequisites are ready
- [ ] Loading state displayed while `isLoading` or `isPending`
- [ ] Error state handled: `isError` + user-friendly message (no raw error objects shown)
- [ ] Mutations use `useMutation` with `onSuccess` invalidating related queries
- [ ] Optimistic updates only implemented where rollback is fully handled in `onError`
- [ ] `staleTime` / `gcTime` configured per query type — not all left at default 0
- [ ] Pagination uses `useInfiniteQuery` or cursor-based approach — not loading all pages into state

## 6. Component Patterns

- [ ] Components are pure (same props → same output) — side effects only in hooks
- [ ] `key` prop on list items is stable and unique (not array index for reorderable lists)
- [ ] `useCallback` / `useMemo` used only where profiling shows benefit — not by default
- [ ] Custom hooks extract reusable stateful logic (not copy-pasted `useEffect` blocks)
- [ ] No direct DOM manipulation (`document.querySelector`) — use refs
- [ ] Controlled vs uncontrolled inputs consistent within a form (never mixed)

## 7. Performance & Code Splitting

- [ ] Every route-level page uses `React.lazy`
- [ ] Heavy third-party libs (charts, editors, maps) loaded with `React.lazy` or dynamic import
- [ ] No barrel imports (`export * from './components'`) in hot paths
- [ ] Images: `loading="lazy"` on below-fold images, correct `width`/`height` to prevent CLS
- [ ] `React.StrictMode` enabled in dev entry point
- [ ] No re-renders from objects/arrays created inline in JSX (use `useMemo` or stable refs)

## 8. Build & Production

- [ ] `vite build` runs without warnings
- [ ] Bundle analyzer run (`rollup-plugin-visualizer` or `vite-bundle-analyzer`) — no surprise large chunks
- [ ] Each chunk < 500 KB (warn threshold)
- [ ] `build.rollupOptions.output.manualChunks` configured for vendor splitting if needed
- [ ] Asset filenames include content hash (`[name]-[hash].[ext]`) for cache busting
- [ ] `build.minify` is `esbuild` or `terser` (not disabled)
- [ ] Console logs stripped in production (`drop_console: true` in terser or esbuild options)

## 9. TypeScript Integration

- [ ] `strict: true` in `tsconfig.json`
- [ ] No `any` without explicit comment justification
- [ ] API response types defined (not inferred from runtime data)
- [ ] Path alias types synced: `compilerOptions.paths` matches `vite.config.ts resolve.alias`
- [ ] `tsc --noEmit` passes before every PR

## 10. Error Handling

- [ ] React Error Boundary wraps route subtrees — crashes don't blank the entire app
- [ ] `QueryErrorResetBoundary` used with TanStack Query error boundaries
- [ ] Network errors shown as user-friendly messages (no raw `Error: Network request failed`)
- [ ] 401/403 from API triggers redirect to login / permission denied page (not silent fail)
- [ ] Unhandled promise rejections caught globally (`window.onunhandledrejection`)
