<!-- File upload checklist. Load when implementing or modifying upload/storage features. -->

## File Upload Checklist

Use when implementing file upload or modifying storage access logic.

**Guardrails**
- File names must use UUID/ULID — never use the original filename from the user
- Don't skip concurrency testing
- Complete each step before moving to the next

---

### Step 1: Confirm uniqueness strategy

- [ ] File names generated with UUID or ULID (not original filename)
- [ ] Database has a unique index on the stored filename / path column

### Step 2: Implement

### Step 3: Concurrency testing (do not skip)

- [ ] Rapidly click upload 20 times in quick succession → no filename collisions
- [ ] Upload multiple files simultaneously → all succeed

### Step 4: Acceptance

- [ ] Filename uniqueness confirmed
- [ ] Concurrency test passes
- [ ] Uploaded files are immediately readable after upload completes

**Verify**
- Run the project's typecheck command
- Manual test: upload → immediately read back the file
