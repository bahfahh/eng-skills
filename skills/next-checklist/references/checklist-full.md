# Next.js Full Development Checklist

## Table of Contents
1. [Phase 1: App Router & Routing](#phase-1-app-router--routing)
2. [Phase 2: Auth & Session](#phase-2-auth--session)
3. [Phase 3: Data Fetching](#phase-3-data-fetching)
4. [Phase 4: Bundle Optimization](#phase-4-bundle-optimization)
5. [Phase 5: Rendering Optimization](#phase-5-rendering-optimization)
6. [Phase 6: JavaScript Performance](#phase-6-javascript-performance)
7. [Phase 7: Component & Hook Structure](#phase-7-component--hook-structure)
8. [Phase 8: Type & Build Checks](#phase-8-type--build-checks)
9. [Phase 9: Final Acceptance](#phase-9-final-acceptance)

---

## Phase 1: App Router & Routing

### Dynamic Routes
- [ ] Dynamic routes use `[param]` naming convention
- [ ] Route params handled as Promise where required (Next.js 15+: `params` is async — must `await params`)
- [ ] Search params (`searchParams`) handled correctly (Next.js 15+: also async — must `await searchParams`)
- [ ] `cookies()` and `headers()` calls use `await` (Next.js 15+: these are now async)
- [ ] Route segments configured correctly (`layout` / `page` / `loading` / `error`)

### Middleware
- [ ] Middleware intercepts the correct route patterns
- [ ] Auth check logic is correct
- [ ] Redirect logic is correct
- [ ] Tenant/scope isolation handled if applicable

### Route Protection
- [ ] Protected routes have auth checks
- [ ] Unauthenticated users redirect to login
- [ ] Role/permission checks applied where needed

---

## Phase 2: Auth & Session

### Auth Client Setup
- [ ] Browser-side auth client created with SSR-compatible method
- [ ] Server-side auth client created with SSR-compatible method
- [ ] Middleware auth client correctly configured
- [ ] Session storage is consistent between client and server (e.g., cookie sync)

### Session Initialization
- [ ] Using user-based retrieval (e.g., `getUser()`) — not session-based (e.g., `getSession()` can hang in React 19+)
- [ ] Initial load has a loading fallback with timeout (no infinite spinner)
- [ ] Opening a new tab does not cause infinite loading

### Background Tab Restore
- [ ] Auth state refreshes on `window.focus`
- [ ] Auth state refreshes on `document.visibilitychange`
- [ ] Relevant state (e.g., org/tenant ID) re-syncs after tab restore
- [ ] No infinite loading after returning from background tab

### Login / Logout Flow
- [ ] Login success redirects correctly
- [ ] Post-login navigation uses hard redirect (e.g., `window.location.href`) — not `router.push()` which may not reset server state
- [ ] Logout clears local state and cache
- [ ] Logout redirects to login page

### Session Expiry Handling
- [ ] 401 errors are caught
- [ ] Expired session shows user-friendly re-login prompt
- [ ] Auto-redirect to login on session expiry
- [ ] No infinite retry loop on auth failure

---

## Phase 3: Data Fetching

### Server-Side Fetching
- [ ] `React.cache()` used for request deduplication
- [ ] `Promise.all()` used for independent parallel fetches
- [ ] No waterfall (sequential awaits for unrelated data)
- [ ] Suspense boundaries used for streaming content
- [ ] `fetch()` cache behavior is explicitly set where needed — Next.js 15+ defaults to `no-store` (no caching); add `{ cache: 'force-cache' }` or `next: { revalidate: N }` if caching is intended

### Client-Side Fetching
- [ ] Server state managed with React Query, SWR, or equivalent
- [ ] No duplicate requests
- [ ] `staleTime` and `cacheTime` configured appropriately

### Error Handling
- [ ] Network errors caught
- [ ] 401/403 errors handled distinctly
- [ ] User-facing error messages are understandable (no raw API errors)
- [ ] Retry mechanism available

### Data Serialization
- [ ] Only necessary data passed to client
- [ ] No functions or non-serializable objects passed across server/client boundary
- [ ] Sensitive data not exposed to client

---

## Phase 4: Bundle Optimization

> For detailed rules, load the `react-best-practices` skill.

### Import Strategy
- [ ] No barrel imports — import directly from source file
- [ ] Unused imports removed

### Code Splitting
- [ ] Heavy components use `next/dynamic`
- [ ] Modals, drawers, and offscreen UI use dynamic imports
- [ ] Third-party libraries (analytics, logging) loaded dynamically
- [ ] `loading` and `error` states set on dynamic imports

### Preloading
- [ ] Resources preloaded on hover/focus where appropriate
- [ ] Critical resources use `<link rel="preload">`
- [ ] Likely-needed resources use `<link rel="prefetch">`

---

## Phase 5: Rendering Optimization

> For detailed rules, load the `react-best-practices` skill.

### Re-render Prevention
- [ ] `React.memo()` applied to components that re-render unnecessarily
- [ ] `useCallback()` used for stable callback references
- [ ] `useMemo()` used for expensive calculations
- [ ] Derived state not stored in `useState`

### Conditional Rendering
- [ ] Ternary operator used (not `&&` for falsy edge cases)
- [ ] No new object/array literals created inline during render
- [ ] Static JSX extracted outside component body where possible

### Animation & Layout
- [ ] Animations applied to wrapper `div`, not directly to SVG elements
- [ ] Long lists use `content-visibility` for rendering optimization
- [ ] `will-change` used only where the performance benefit is clear

### Hydration
- [ ] No hydration mismatch between server and client render
- [ ] Client-only data handled with inline scripts or `useEffect` — not inline conditionals that differ server/client
- [ ] No flash of unstyled or incorrect content on hydration

---

## Phase 6: JavaScript Performance

> For detailed rules, load the `react-best-practices` skill.

- [ ] Batch DOM/CSS mutations where possible
- [ ] `Set` or `Map` used for O(1) lookups instead of array `find`/`filter`
- [ ] Object property access cached in local variables in hot paths
- [ ] Multiple `filter`/`map` chains merged into a single loop where practical
- [ ] Array length checked before expensive comparisons in loops
- [ ] RegExp instances hoisted outside loops
- [ ] `min`/`max` calculated via loop, not `sort`

---

## Phase 7: Component & Hook Structure

### Server vs Client Components
- [ ] Default to Server Components — no `'use client'` unless needed
- [ ] `'use client'` only added when component uses hooks, event handlers, or browser APIs
- [ ] No hooks called inside Server Components

### Custom Hooks
- [ ] All hooks placed in `hooks/` directory
- [ ] Hook names start with `use`
- [ ] Hooks have clear TypeScript type definitions
- [ ] `useActionState` used instead of deprecated `useFormState` (React 19 / Next.js 15+)

### Component Design
- [ ] Named exports used (not default exports)
- [ ] Props have explicit TypeScript types
- [ ] Components are small and single-purpose
- [ ] No deeply nested JSX that can be broken into sub-components

---

## Phase 8: Type & Build Checks

- [ ] Typecheck passes with no errors (e.g., `tsc --noEmit`)
- [ ] All variables and return types are typed
- [ ] No `any` — use `unknown` where type is genuinely unknown
- [ ] Lint passes with no errors
- [ ] No unused variables or imports
- [ ] Build succeeds with no errors or warnings
- [ ] Build output size is reasonable

---

## Phase 9: Final Acceptance

### Functional Completeness
- [ ] All features work as intended
- [ ] All routes are accessible
- [ ] Auth flow works end-to-end
- [ ] Data fetching works correctly

### Performance
- [ ] First contentful paint is acceptable
- [ ] Interaction latency is acceptable
- [ ] Bundle size is reasonable
- [ ] No performance regressions vs previous state

### Security
- [ ] Sensitive data not leaked to client
- [ ] API routes have auth checks
- [ ] No XSS vulnerabilities (unsanitized user content rendered as HTML)
- [ ] No CSRF vulnerabilities on state-changing routes
