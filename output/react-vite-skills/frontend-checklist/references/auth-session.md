<!-- Auth/Session checklist. Load when modifying auth, session, or middleware logic. -->

## Auth / Session Checklist

Use when making changes to authentication, session handling, or route protection middleware.

**Guardrails**
- Don't touch the session storage layer unless you need to
- Don't move to the next step before completing the current one
- Don't skip E2E verification

---

### Step 1: Verify session source consistency

- Where does the browser store the session? (e.g., localStorage, cookie — varies by framework)
- What does the middleware/server read? (e.g., cookie, Authorization header)
- Are the two in sync?
- If not → stop and resolve the mismatch before proceeding

### Step 2: Implement

- Keep changes scoped to auth-related modules
- Don't widen the change surface unnecessarily

### Step 3: E2E verification (do not skip)

Test all three scenarios manually:

- [ ] Open a new tab → navigate to a protected route → no infinite loading
- [ ] Switch to background tab and back → auth state is correct
- [ ] Session expiry → fallback works (retry or redirect to login)

### Step 4: Acceptance

- [ ] Session source is consistent between browser and server/middleware
- [ ] Infinite loading has a fallback
- [ ] Background tab restore works correctly
- [ ] New tab navigation works correctly

**Verify**
- Run the project's typecheck command
- Manually test all three scenarios from Step 3
