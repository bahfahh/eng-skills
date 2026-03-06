---
name: db-supabase-checklist
description: Supabase database development checklist for planning and code review. Use when creating or modifying database migrations, designing RLS policies, adding tables or columns, modifying functions/triggers, or reviewing any Supabase DB changes before pushing. Trigger for both planning ("how should I structure this DB change?", "help me write this migration") and review ("review my migration", "check my RLS policy", "is this DB change safe?"). Covers migration naming rules, RLS policy design, multi-tenant isolation, data integrity, indexes, soft delete, TypeScript type sync, and AI/user responsibility boundaries.
---

# Supabase DB Checklist

## Modes

Detect mode from context:

- **coding / plan mode** — User is planning or writing DB changes. Read this checklist and add relevant items to the todolist before starting.
- **code review mode** — User has completed DB changes. Review the changes against this checklist and report issues.

---

## Steering Files (Check First)

Before applying this checklist, check if project-specific steering files exist:

- If `.kiro/steering/database/rls-rules.md` exists → follow it for RLS rules (use this checklist as supplement)
- If `.kiro/steering/supabase_migration.md` exists → follow it for migration rules (use this checklist as supplement)
- If neither exists → apply this checklist as-is

---

## Guardrails

These rules are non-negotiable regardless of mode:

- Never modify an already-pushed migration — create a new one instead
- Never execute `supabase db push` — only design the SQL files; the user pushes
- Never execute `supabase gen types typescript` or `supabase db reset` — user's responsibility
- Never hard-delete data that has foreign key references — use soft delete
- Never test RLS with mocks — test with real accounts on local

---

## Checklist

### Migration

- [ ] Filename timestamp is exactly 14 digits `YYYYMMDDHHMMSS` — **not just the date** (e.g. `20260206` is wrong; `20260206170000` is correct)
- [ ] Filename: snake_case description + `.sql` (e.g. `20260206170000_add_report_template_column.sql`)
- [ ] Starts with `set check_function_bodies = off;`
- [ ] Uses `DROP ... IF EXISTS` before recreating policies/functions
- [ ] Uses `IF NOT EXISTS` for table creation, `CREATE OR REPLACE` for functions
- [ ] One migration = one concern (atomic)
- [ ] Idempotent — safe to run multiple times
- [ ] Backward compatible — existing data unaffected
- [ ] Destructive changes follow the safe sequence: add new column → migrate data → switch app → (later) drop old column
- [ ] Comments explain intent and impact

See `references/migration-workflow.md` for step-by-step workflow.

### RLS Policy

- [ ] Every table has RLS enabled: `ALTER TABLE xxx ENABLE ROW LEVEL SECURITY;`
- [ ] Every table has `ALTER TABLE xxx FORCE ROW LEVEL SECURITY;`
- [ ] Policy SQL is written directly (not just described in prose)
- [ ] Role permission matrix is filled for each table:

  | Role | SELECT | INSERT | UPDATE | DELETE |
  |------|--------|--------|--------|--------|
  | (role A) | ? | ? | ? | ? |
  | (role B) | ? | ? | ? | ? |

  *(Example roles: `teacher`, `principal`, `system_owner` — replace with your project's actual roles)*

- [ ] `USING` condition: filters what rows can be read
- [ ] `WITH CHECK` condition: validates what rows can be written
- [ ] Helper functions exist for auth checks and role resolution
- [ ] Helper functions have `SECURITY DEFINER` and `search_path` set

See `references/rls-workflow.md` for step-by-step workflow.

### Multi-Tenant Isolation

*(Apply if the project uses multi-tenancy — e.g. `organization_id`, `tenant_id`, or similar)*

- [ ] Every table has a tenant identifier column
- [ ] Every policy filters by tenant identifier
- [ ] `USING` enforces tenant isolation
- [ ] `WITH CHECK` enforces tenant isolation on writes
- [ ] Super-admin / system owner role has a separate policy allowing cross-tenant access

### Data Integrity

- [ ] All foreign keys explicitly defined with correct `ON DELETE` behavior (`CASCADE` / `RESTRICT` / `SET NULL`)
- [ ] Tables with foreign key references use soft delete (e.g. `is_active = false`), not hard delete
- [ ] `ON DELETE CASCADE` scope reviewed — confirm no unintended data loss
- [ ] Columns requiring uniqueness have a unique index
- [ ] Concurrent write scenarios considered (e.g. rapid duplicate submissions)

### Indexes & Performance

- [ ] Tenant identifier column has an index (if multi-tenant)
- [ ] Foreign key columns have indexes
- [ ] Columns used in frequent filters/sorts have indexes
- [ ] No excessive indexes that would hurt write performance
- [ ] Complex queries use RPC functions where appropriate
- [ ] No N+1 query patterns introduced

### Functions & Triggers

- [ ] Functions have `SECURITY DEFINER` if privilege elevation is needed
- [ ] Functions and triggers have `search_path` set
- [ ] Functions have `STABLE` or `IMMUTABLE` label where applicable
- [ ] Triggers have correct timing (`BEFORE` / `AFTER`) and level (`FOR EACH ROW` / `STATEMENT`)
- [ ] No infinite trigger loops possible

### Timestamps

- [ ] `created_at` uses `DEFAULT now()`
- [ ] `updated_at` auto-updates via trigger or app layer
- [ ] All timestamp columns use `TIMESTAMPTZ`
- [ ] `created_at` is not manually modifiable

### Type Sync (User Executes)

- [ ] After push: `supabase gen types typescript`
- [ ] TypeScript compiles: `pnpm tsc --noEmit` (or project equivalent)
- [ ] New columns appear in generated types
- [ ] Nullable columns typed as `| null`

### Documentation

- [ ] Migration file has comments explaining why the change is made and its impact
- [ ] Rollback approach noted if the change is risky
- [ ] RLS policy document updated if applicable

---

## AI / User Responsibility Split

**AI designs (never executes):**
- SQL file content
- RLS policy SQL
- Index and constraint design
- Function and trigger logic
- Clear comments and rollback notes

**User executes and validates:**
- `supabase db push`
- `supabase gen types typescript`
- `pnpm tsc --noEmit`
- RLS testing with real accounts (each role: SELECT / INSERT / UPDATE / DELETE)
- Data integrity and performance testing
- Commit and PR

---

## Common Pitfalls

| Mistake | Correct Approach |
|---------|-----------------|
| Editing an already-pushed migration | Create a new migration with `CREATE OR REPLACE` |
| AI runs `supabase db push` | AI only creates the file; user pushes |
| Hard-deleting rows with FK references | `UPDATE table SET is_active = false WHERE id = ?` |
| Testing RLS with jest mocks | Test with real accounts on local Supabase |
| Function missing `search_path` | Always set `search_path` on functions and triggers |
| Manually editing generated type files | Re-run `supabase gen types typescript` |

---

## References

- Migration step-by-step: `references/migration-workflow.md`
- RLS step-by-step: `references/rls-workflow.md`
