<!-- UI/UX focused checklists. Load when working on components, API error handling, accessibility, responsive design, or large data display. -->

## Table of Contents

1. [API Status Code → UI Mapping](#workflow-01-api-status-code--ui-mapping)
2. [Component State Completeness](#workflow-02-component-state-completeness)
3. [Accessibility](#workflow-03-accessibility)
4. [Responsive Design](#workflow-04-responsive-design)
5. [Large Data UI Strategy](#workflow-05-large-data-ui-strategy)

---

## Workflow 01: API Status Code → UI Mapping

Goal: Prevent "button does nothing" and "error message is unreadable".

### When to apply
- Any UI that calls an API (create/update/delete, upload, login)
- When API response behavior changes (status codes, error body format)

### Status code mapping

| Code | Situation | UI behavior | User message | User's next step |
|------|-----------|-------------|--------------|-----------------|
| 401 | Not logged in / session expired | Redirect to login | "Session expired, please log in again" | Re-login |
| 403 | Insufficient permissions | Toast | "Insufficient permissions for this action" | Go back |
| 409 | Concurrency conflict | Dialog | "Data was updated elsewhere, please refresh and try again" | Refresh |
| 422 | Validation error | Inline error | "Please check your input and try again" | Fix and retry |
| 429 | Rate limited | Toast + delay | "Too many requests, please wait a moment" | Retry later |
| 5xx | Server error | Toast + retry | "Service temporarily unavailable, please try again" | Retry |

### Implementation notes
- Never display raw technical errors (token strings, PGRST codes, stack traces)
- 422 → inline field error; 401/403 → redirect or page-level state change
- Every pending state must have a timeout + fallback

---

## Workflow 02: Component State Completeness

Goal: Every interactive component has the full set of states.

### Required states

| State | Visual | Behavior | When |
|-------|--------|----------|------|
| Default | Normal styles | Interactive | Initial load |
| Hover | Color / shadow change | Interactive | Mouse over |
| Active | Press effect | Interactive | Click moment |
| Disabled | Grayed / reduced opacity | Not interactive | Condition unmet |
| Loading | Spinner + text change | Not interactive | API in flight |
| Error | Red border / text | Interactive | Validation failure |
| Success | Green / checkmark (brief) | Auto-recovers | Action success |

### Acceptance

- [ ] All states have visual feedback
- [ ] Button is disabled while loading
- [ ] Error state shows a clear message
- [ ] Success state auto-recovers or navigates away

---

## Workflow 03: Accessibility

Goal: All users can complete the main flows.

| Item | Requirement | How to verify |
|------|-------------|---------------|
| Keyboard navigation | Tab reaches all interactive elements | Navigate with keyboard only |
| Focus indicator | Clearly visible when focused | Watch while tabbing |
| ARIA labels | Icon-only buttons have `aria-label` | Inspect HTML |
| Color contrast | Text/background ≥ 4.5:1 | Browser DevTools |
| Touch targets | Buttons/links ≥ 44×44px | Test on a real device |

### Acceptance

- [ ] Main flows completable with keyboard only
- [ ] All icon-only buttons have `aria-label`
- [ ] Color contrast passes
- [ ] Touch targets sufficient on mobile

---

## Workflow 04: Responsive Design

Goal: Usable on mobile without special effort from the user.

| Item | Requirement | How to verify |
|------|-------------|---------------|
| Touch targets | ≥ 44×44px | Click on a real device |
| Keyboard overlay | Doesn't obscure the focused input | Type in a text field on mobile |
| Scrolling | Scrolls normally | Scroll on a real device |
| Navigation | Back button works | Navigate on a real device |
| Breakpoints | Correct layout at key widths | Chrome DevTools device mode |

### Acceptance

- [ ] Usable at 320px viewport width
- [ ] Keyboard does not block the focused input
- [ ] All buttons are tappable
- [ ] Back button works correctly

---

## Workflow 05: Large Data UI Strategy

Goal: Avoid unusable vertical lists when item count grows large.

### When to apply
- Selectable items or list views with > 30 entries

### Strategy selection

| Strategy | Use when | UI pattern |
|----------|----------|------------|
| Search-first | User knows what they're looking for | Search box + result list |
| Browse-first | User is exploring options | Category tabs + pagination or virtualization |
| Frequent-first | Clear high-frequency items exist | Pinned frequent items + search |

### Acceptance

- [ ] Users have a way to quickly locate items (search, filter, or frequent pins)
- [ ] No scroll lag or jank with large datasets
- [ ] Mobile usable with large data
