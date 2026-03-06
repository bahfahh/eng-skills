---
description: RLS policy 變更流程。當新增 table、修改 RLS policy、修改函式/trigger 時參考。
---

# RLS Workflow

> Check `.kiro/steering/database/rls-rules.md` first if it exists — follow that instead and use this as a supplement.

**Guardrails**
- Never modify an already-pushed migration
- Complete each step before moving to the next
- Never test RLS with mocks — use real accounts on local

---

## Steps

Track these as TODOs and complete one by one.

### 1. Write the Policy SQL directly

Write executable SQL, not prose descriptions.

```sql
-- ❌ Don't do this: "role A can read their own rows"
-- ✅ Do this:
CREATE POLICY "role_a_own_rows" ON your_table
FOR SELECT TO authenticated
USING (created_by = auth.uid());
```

If the project uses multi-tenancy, add tenant isolation to every policy:
```sql
-- Example: tenant isolation via organization_id
USING (
  organization_id = get_current_org_id()   -- example helper function
  AND created_by = auth.uid()
)
```
*(Replace `organization_id` and `get_current_org_id()` with your project's actual column and function names)*

Helper functions must have:
```sql
SECURITY DEFINER
SET search_path = public
```

### 2. Fill the role permission matrix

For each table being modified, fill out:

| Role | SELECT | INSERT | UPDATE | DELETE |
|------|--------|--------|--------|--------|
| (role A) | USING condition | WITH CHECK condition | USING + WITH CHECK | USING condition |
| (role B) | ... | ... | ... | ... |

*(Example roles: `teacher`, `principal`, `system_owner` — replace with your project's actual roles)*

Rules:
- `USING` — filters rows for read operations
- `WITH CHECK` — validates rows for write operations
- Every role must have an explicit policy or be explicitly denied
- Super-admin / system owner role needs a separate policy if cross-tenant access is required

### 3. Local dry-run

Apply the migration locally:
```bash
supabase db reset
```

Test with real accounts (not mocks):
- Test each role's SELECT / INSERT / UPDATE / DELETE
- Confirm tenant isolation works (if applicable)
- Confirm super-admin cross-tenant access works (if applicable)

### 4. Handoff to user

AI stops here. User executes and validates:

```bash
supabase db push
```

Then test each role manually:
- [ ] SELECT: can read what they should, cannot read what they shouldn't
- [ ] INSERT: can create what they should, blocked otherwise
- [ ] UPDATE: can modify what they should, blocked otherwise
- [ ] DELETE: can delete what they should, blocked otherwise
- [ ] Tenant isolation: role A cannot access role B's data

---

## Acceptance Criteria

- [ ] Policy SQL written (not just described)
- [ ] Role permission matrix filled
- [ ] Local dry-run passes with real accounts
- [ ] `search_path` set on all functions and triggers
