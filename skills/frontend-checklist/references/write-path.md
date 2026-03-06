<!-- Write path checklist for create/update/delete operations. -->

## Write Path Checklist

Use when implementing any feature that creates, updates, or deletes data.

**Guardrails**
- Don't manage loading state with raw `useState` — use the project's standardized async action hook
- Don't skip checking for existing hooks/components before implementing

---

### Step 1: Check for a standardized async action hook

- Does the project have a standardized hook for handling async actions? (e.g., `useAsyncAction` or equivalent)
- If not → create it first, then continue
- Purpose: centralizes loading, error, and success state management so every write path behaves consistently

### Step 2: Check for a standardized async button component

- Does the project have a button component that handles pending/disabled/error states? (e.g., `AsyncButton` or equivalent)
- If not → create it first, then continue
- Purpose: prevents duplicate implementations of loading spinners and double-click guards

### Step 3: Implement the feature

- Use the project's async action hook for the async operation
- Use the project's async button component for the trigger UI

### Step 4: Acceptance

- [ ] Loading state displays while the action is in flight
- [ ] Button is disabled while loading (prevents double submission)
- [ ] Success feedback appears (toast or visible state change)
- [ ] Error feedback appears with a human-readable message
- [ ] Works correctly on mobile

**Verify**
- Run the project's typecheck command
- Manual test: click, observe loading → success → toast; try double-clicking during loading
