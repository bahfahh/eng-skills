---
description: Migration 開發流程。當建立或修改 database migration 時參考。
---

# Migration Workflow

> Check `.kiro/steering/supabase_migration.md` first if it exists — follow that instead and use this as a supplement.

**Guardrails**
- Never modify an already-pushed migration
- One migration = one concern
- Always validate on local before pushing

---

## Steps

Track these as TODOs and complete one by one.

### 1. Create the migration file

```bash
supabase migration new <name>
```

- Use snake_case to describe the action (e.g. `add_report_template_column`)
- Timestamp is auto-generated; if creating manually use `YYYYMMDDHHMMSS` (14 digits), at least 1 minute apart from others on the same day

### 2. Write the SQL

Start the file with:
```sql
set check_function_bodies = off;
```

Requirements:
- `DROP ... IF EXISTS` before recreating policies or functions
- `IF NOT EXISTS` for table creation
- `CREATE OR REPLACE` for functions
- Idempotent — safe to run multiple times
- Backward compatible — existing data unaffected
- Comments explaining intent and impact

If the migration involves RLS changes → also follow `rls-workflow.md`.

### 3. Local validation

```bash
supabase db reset
```

Confirm:
- Migration runs without errors
- Existing data is unaffected
- Re-running the migration causes no errors (idempotency check)

### 4. Handoff to user

AI stops here. User executes:

```bash
supabase db push
supabase gen types typescript
pnpm tsc --noEmit   # or project equivalent
```

---

## Acceptance Criteria

- [ ] Migration is idempotent
- [ ] Local `supabase db reset` passes without errors
- [ ] Types regenerated and TypeScript compiles
- [ ] Existing data unaffected
