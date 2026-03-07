---
name: validate
description: Run the full validation pipeline on current changes. Combines lint, test, type-check, code review, and feature validation into one command. Use after completing a feature and before committing.
---

# /validate — Full Validation Pipeline

Run the complete validation pipeline for current changes. Execute each step sequentially, stop on critical failures.

## Pipeline Steps

### Step 1: Lint Check
Run the project's linter:
```bash
npm run lint
```
If lint script doesn't exist, skip this step.

### Step 2: Type Check
If TypeScript project (tsconfig.json exists):
```bash
npx tsc --noEmit
```

### Step 3: Unit Tests
Run the project's test suite:
- Check package.json for test commands
- Prefer: `npm run test:unit:all` → `npm test` → `npx vitest run`
- Only run if test scripts/configs exist

### Step 4: Code Review
Delegate to the Superpowers code review skill:
- Invoke `superpowers:requesting-code-review` via the Skill tool
- Reviews: code quality, architecture, testing, requirements alignment, production readiness
- Report issues by severity (Critical / Important / Minor)

### Step 5: Summary Report
Produce a final summary:

```
## Validation Report

| Step        | Status | Details          |
|------------|--------|------------------|
| Lint        | ✓/✗/⊘  | X errors         |
| TypeCheck   | ✓/✗/⊘  | X errors         |
| Tests       | ✓/✗/⊘  | X passed, X fail |
| Code Review | ✓/✗    | X issues found   |

✓ = passed, ✗ = failed, ⊘ = skipped

**Ready to commit**: YES / NO
**Blocking issues**: (list if any)
```

## Rules
- ⊘ Skip steps that don't apply (no lint config = skip lint)
- Stop pipeline early if Step 1-3 has critical failures
- Step 4 (review) runs even if earlier steps have warnings
- Do NOT auto-fix — only report. User decides what to fix.
