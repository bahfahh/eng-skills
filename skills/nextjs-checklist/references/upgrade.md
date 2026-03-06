# Next.js Version Upgrade Workflow

Track these steps as TODOs and complete them one by one. Do not advance to the next step until the current one is complete.

## Guardrails

- Never upgrade multiple major versions at once — upgrade one major version at a time
- Do not skip smoke tests — they catch breaking changes that type checks miss
- Run full smoke test before committing the upgrade

---

## Step 1: Review Release Notes

- Read the official Next.js release notes for the target version
- Identify all breaking changes
- List the affected areas (e.g., `params`/`searchParams` async changes, middleware API, etc.)

## Step 2: Upgrade Dependencies

- Update Next.js to the target version (e.g., `pnpm update next@latest`)
- Update related packages in sync (e.g., `@next/*`, `eslint-config-next`)

## Step 3: Fix Breaking Changes

- Address each item from Step 1 systematically
- Common example — Next.js 15: `params` and `searchParams` changed to Promise → add `await` to all usages

## Step 4: Smoke Test (do not skip)

- [ ] Build succeeds with no errors (e.g., `pnpm build`)
- [ ] All dynamic route pages load correctly
- [ ] Middleware functions correctly
- [ ] API routes respond correctly

## Step 5: Acceptance

- [ ] All breaking changes from Step 1 addressed
- [ ] All smoke tests pass
- [ ] Typecheck passes (e.g., `pnpm tsc --noEmit`)
- [ ] Lint passes (e.g., `pnpm lint`)

---

## Reference

- Development checklist: `checklist-full.md`
- Performance rules: load the `react-best-practices` skill
