<!-- Complete 10-phase frontend development checklist. Load when doing a full UI feature or PR review. -->

## Table of Contents

1. [Design System](#phase-1-design-system)
2. [Interaction Behavior](#phase-2-interaction-behavior)
3. [API Status Code Mapping](#phase-3-api-status-code-mapping)
4. [Component State Completeness](#phase-4-component-state-completeness)
5. [Accessibility](#phase-5-accessibility)
6. [Responsive Design](#phase-6-responsive-design)
7. [Large Data UI](#phase-7-large-data-ui)
8. [Development Workflow](#phase-8-development-workflow)
9. [Information Presentation](#phase-9-information-presentation)
10. [Final Acceptance](#phase-10-final-acceptance)

---

## Phase 1: Design System

### 0. Calibrate first: find the project's real design source

- [ ] Locate the project's design token / theme source (colors, fonts, border-radius, shadows, spacing, breakpoints)
- [ ] Prioritize existing tokens before adding new ones — avoid scattered hardcoded values
- Common locations: `app/globals.css`, `tailwind.config.{js,ts}`, `@theme` block (Tailwind v4), `tokens.css`, `theme.ts`, `src/constants/colors.ts`, design system docs

### 1. Color system

- [ ] Semantic color roles defined: Primary / Secondary / Accent / Neutral / Success / Warning / Destructive
- [ ] Primary → main CTA; Secondary → secondary actions; Destructive → delete/disable
- [ ] Focus/hover/disabled states follow consistent rules (same token set or algorithm)
- [ ] Category/tag colors match project constants, not arbitrary picks
- [ ] Colors defined only in token/theme files; UI uses CSS variables or token references — no hardcoded hex in components

### 2. Typography and spacing

- [ ] Typography scale and line-height follow consistent rules (headings tighter, body looser — e.g., 1.2–1.3 vs 1.45–1.6)
- [ ] Spacing uses a fixed scale (e.g., 4px/8px grid) — avoid one-off values
- [ ] Border-radius uses tokens (`--radius-*` or Tailwind `rounded-*`) — consistent across components
- [ ] Interactive elements (Button/Input) sized per token, meeting touch targets (≥ 44×44px)

### 3. Fonts

- [ ] Using the project's specified fonts (Body/Heading if distinct), consistent weight and letter-spacing
- [ ] Font fallback stack complete (watch for CJK/mixed-script layouts)
- [ ] Monospace font only for code/log/technical blocks — not general UI text

### 4. Component styles

- [ ] Primary Button: height/radius/shadow/color from tokens (e.g., 48–56px height)
- [ ] Outline/Secondary Button: border and hover rules consistent; disabled state doesn't look clickable
- [ ] Input: background/border/focus ring from tokens; error/success states clearly distinct
- [ ] Card: shadow and padding from scale; information hierarchy clear (title / subtitle / action area)
- [ ] Tag/Chip: semantic color + sufficient contrast; avoid dense stacking that hurts readability

### 5. Animation

- [ ] Duration and easing consistent (150–250ms for micro-interactions; 400–800ms for larger entrances)
- [ ] Easing: prefer ease-out / standard easing
- [ ] Supports `prefers-reduced-motion`
- [ ] Animation used only for hero sections, card entrances, and primary CTA — not decorative everywhere

---

## Phase 2: Interaction Behavior

### 6. Button interaction standards

- [ ] **Pending state**: disabled + spinner or text change
- [ ] **Double-click prevention**: pending state blocks re-trigger
- [ ] **Success feedback**: toast or UI state update (at least one required)
- [ ] **Error feedback**: understandable message — not a technical error
- [ ] **Copy**: specific action verb ("Save", "Add", "Delete") — not "OK" or "Confirm"

### 7. Confirmation dialogs

- [ ] Delete / disable / overwrite / batch operations require a dialog
- [ ] Dialog clearly states the impact and scope
- [ ] Primary button copy is specific ("Delete Student" not "Confirm")
- [ ] Default focus does not land on the destructive button
- [ ] ESC and clicking the backdrop cancel the action

### 8. Toast notifications

- [ ] Success operations show a toast (saved, submitted, copied)
- [ ] Lightweight errors use toast (temporary network failure)
- [ ] Form validation errors use inline error — not toast
- [ ] Errors requiring a decision use dialog — not just toast
- [ ] Avoid stacking too many toasts simultaneously

### 9. Loading indicators

- [ ] Page loading: route-level loading UI (e.g., `loading.tsx` in Next.js) or skeleton
- [ ] Inline loading: affects only the local area — does not block the whole page
- [ ] No infinite spinners: must have timeout + fallback
- [ ] After 10–15 seconds: show actionable fallback (retry / refresh / return to login)
- [ ] Offline state shows an offline indicator

### 10. Navigation button feedback

- [ ] Active state switches within 100ms of click (use a `clickedHref` state or equivalent)
- [ ] Does not wait for page data to load before updating nav state
- [ ] Page loading indicator appears in the content area — not on the nav button
- [ ] Navigation and data loading are decoupled

---

## Phase 3: API Status Code Mapping

### 11. Status code → UI behavior

| Code | Situation | UI | User message | Next step |
|------|-----------|----|--------------|-----------|
| 401 | Not logged in / session expired | Redirect to login | "Session expired, please log in again" | Re-login |
| 403 | Insufficient permissions | Toast | "Insufficient permissions for this action" | Go back |
| 409 | Concurrency conflict | Dialog | "Data was updated elsewhere, please refresh and try again" | Refresh |
| 422 | Validation error | Inline error | "Please check your input and try again" | Fix and retry |
| 429 | Rate limited | Toast + delay | "Too many requests, please wait a moment" | Retry later |
| 5xx | Server error | Toast + retry | "Service temporarily unavailable, please try again" | Retry |

- Never display raw technical errors (token strings, RLS codes, DB error codes)
- 422 → inline error; 401/403 → redirect or page state change
- Any pending state must have a timeout + fallback

---

## Phase 4: Component State Completeness

### 12. Required states for every interactive component

| State | Visual | Behavior | When |
|-------|--------|----------|------|
| Default | Normal styles | Interactive | Initial |
| Hover | Color / shadow change | Interactive | Mouse over |
| Active | Press effect | Interactive | Click moment |
| Disabled | Grayed / reduced opacity | Not interactive | Condition unmet |
| Loading | Spinner + text change | Not interactive | API in flight |
| Error | Red border / text | Interactive | Validation fail |
| Success | Green / checkmark (brief) | Auto-recovers | Action success |

---

## Phase 5: Accessibility

### 13. Keyboard navigation

- [ ] Tab reaches all interactive elements
- [ ] Focus indicator is clearly visible
- [ ] Main user flows are completable using keyboard only

### 14. ARIA labels

- [ ] All icon-only buttons have `aria-label`
- [ ] All form fields have an associated `<label>`

### 15. Color contrast

- [ ] Text-to-background contrast ≥ 4.5:1
- [ ] Verified with browser DevTools or an accessibility checker

---

## Phase 6: Responsive Design

### 16. Touch targets

- [ ] All buttons and links ≥ 44×44px
- [ ] Primary actions meet the project's token-defined height (typically 48–56px)
- [ ] Physically tested on a mobile device

### 17. Mobile layout

- [ ] Usable at 320px viewport width
- [ ] Edge padding uses spacing scale (e.g., 16px)
- [ ] Card padding uses spacing scale (e.g., 16–24px)
- [ ] Keyboard does not obscure the focused input

### 18. Scroll and navigation

- [ ] Page scrolls normally
- [ ] Back button works correctly
- [ ] Tested at relevant breakpoints (e.g., 320px / 768px / 1024px — follow project config)

---

## Phase 7: Large Data UI

### 19. Lists with > 30 items

- [ ] Avoid a plain vertical list with no navigation aid
- [ ] Provide fast locating: search, filter, or pinned frequent items
- [ ] Choose strategy based on user behavior:
  - **Search-first** (user knows what they want): search box + result list
  - **Browse-first** (user is exploring): category + pagination or virtualization
  - **Frequent-first** (clear high-frequency items): pin top + search
- [ ] No jank or scroll lag with large datasets

---

## Phase 8: Development Workflow

### 20. Write path (Create / Update / Delete)

- [ ] Using the project's standard async action pattern (e.g., standardized async hook + async button component)
- [ ] Loading state displays correctly
- [ ] Double-click prevention works
- [ ] Success / error feedback appears correctly
- [ ] Works on mobile

### 21. File upload

- See `storage-upload.md` for the focused checklist

### 22. Auth / Session

- See `auth-session.md` for the focused checklist

---

## Phase 9: Information Presentation

### 23. Copy and understandability

- [ ] UI text is written from the user's perspective
- [ ] Technical details go only to logs — not the UI
- [ ] Technical errors are translated: "Insufficient permissions", "Session expired", "Network connection lost"

---

## Phase 10: Final Acceptance

### 24. Completeness checks

- [ ] Project typecheck passes (e.g., `pnpm tsc --noEmit`)
- [ ] Project tests pass (e.g., `pnpm test`)
- [ ] Project lint / format passes (e.g., `pnpm lint`)
- [ ] Manual testing of all interaction flows
- [ ] Full test on a mobile device

### 25. PR review checklist

- [ ] What happens while an action is pending? Can it be double-clicked?
- [ ] How does the user know when the action succeeded?
- [ ] What does the user see when it fails? What's their next step?
- [ ] Can the UI get stuck in an infinite loading state? What's the fallback?
- [ ] On mobile: can everything be tapped, scrolled, and typed into without the keyboard blocking anything?
