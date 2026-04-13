---
name: spec
description: Create an executable three-tier specification (business Gherkin + Zod schema + test skeleton) for a new feature. Use BEFORE writing implementation plan. Enforces Zod-first spec-driven development.
---

# /spec — Executable Spec Generator

Produces machine-verifiable specifications across three tiers in strict order. This is step 2 of the 7-stage workflow (after `/brainstorm`, before `/write-plan`).

## When to use

- After `/brainstorm` has produced a design decision record
- Before writing any implementation code
- When adopting spec-driven development on an existing feature

## Core principle

**Natural language specs do not constrain LLMs. Executable specs do.**

Every spec must produce artifacts the compiler, runtime, or CI can verify.

## Steps

### Step 1: Delegate to spec-writer agent

Invoke the `spec-writer` subagent with the brainstorm output as context. The agent will enforce the three-tier order:

1. **Business spec** — `specs/NNN-*.spec.md` (Gherkin, acceptance criteria, non-functional requirements)
2. **Interface spec** — `src/schemas/<feature>.schema.ts` (Zod schemas with `@spec NNN` tags)
3. **Behavioral spec** — `src/__tests__/NNN-*.spec.ts` (Vitest + fast-check skeletons, all `it.todo`)

### Step 2: Verify artifacts compile

```bash
npx tsc --noEmit
```

Must pass. If not, return to spec-writer with the error.

### Step 3: Register spec ID

- Note the new NNN spec ID.
- Tell the user to reference it from subsequent `/write-plan`, commits, and PR.

### Step 4: Output summary

```
Spec NNN created:
  📋 specs/NNN-<name>.spec.md
  📐 src/schemas/<name>.schema.ts
  🧪 src/__tests__/NNN-<name>.spec.ts

Next: /write-plan → /contract → TDD
```

## Rules

- Never write implementation code in this step. That is the TDD phase.
- Never delete a spec — deprecate it via `status: deprecated` frontmatter.
- Every spec gets a unique NNN id. Never reuse.
- Cross-boundary data MUST have a Zod schema. No exceptions.

## Tips

- For feature changes: update existing spec and bump `updated:` date; do NOT create a new spec unless requirements genuinely differ.
- For refactors: spec should already exist; if not, write one first (reverse-engineering the current behavior).
- Keep Gherkin scenarios focused on observable behavior, not implementation details.
