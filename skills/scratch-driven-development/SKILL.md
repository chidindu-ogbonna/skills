---
name: scratch-driven-development
description: Development method where a scratch script observes current behavior before a change, the change is implemented, then a probe script validates the new output matches expectations. Use when changing or migrating existing behavior that must be verified end-to-end — API field renames, refactors, integration updates, or any change where "before vs after" needs to be confirmed against a live system.
---

# Scratch-Driven Development

Write a scratch script that captures current behavior → implement the change → validate the new output with a probe.

## Workflow

### 0. Bootstrap (first time only)

Ensure `scratch/` directories are gitignored globally so probe scripts never accidentally land in a commit. It is recommended to place the `scratch/` folder close to the code you are modifying (e.g., inside the specific package, service, or feature module).

```bash
grep -qxF '**/scratch/' .gitignore || echo '**/scratch/' >> .gitignore
```

### 1. Observe current behavior (before script)

Create a `scratch/<topic>/` directory near the code you are changing, and add `probe-before.ts` (or a descriptive name). The script should:

- Call the real code path against a live system or dev environment
- Print the output clearly — what fields come back, what values, what errors
- Be runnable standalone: `npx ts-node path/to/scratch/<topic>/probe-before.ts`

Run it and save the output mentally (or in a comment) as the **baseline**.

### 2. Implement the change

Make the code change. Touch only what is required — types first if types drive the implementation.

### 3. Validate with a probe (after script)

Create `path/to/scratch/<topic>/probe-after.ts`. The probe should:

- Exercise the same code paths as the before script
- Assert the output matches the expected new behavior
- Print `✅ / ❌` per assertion and exit `1` on any failure
- Clean up any side effects (created records, test data)

Run it:

```bash
npx ts-node path/to/scratch/<topic>/probe-after.ts
```

All assertions must pass before the change is considered done.

> Keep both scripts — they serve as permanent regression references.

---

## Scratch script conventions

- Load env: `import "dotenv/config";` at the top
- Use `@/` imports to reach src code
- Use real dev credentials (not mocks) — the point is live validation
- Use `process.exit(0/1)` so the script has a clear success/failure signal
- Name codes/records with a timestamp (`Date.now()`) to avoid collisions and make cleanup easy

---

## Example structure

```
services/main/src/pricing/
  index.ts
  scratch/
    discount-migration/
      probe-before.ts   ← captures customerSelection behavior
      probe-after.ts    ← validates context field works identically
```

---

## When to use one combined script instead of two

If the change is purely additive (new field, no behavioral difference to observe), a single `probe.ts` that exercises the new behavior is enough. Use separate before/after scripts when the existing behavior is what you're replacing and you need a documented baseline.
