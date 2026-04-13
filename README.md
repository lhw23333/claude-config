# Claude Code Global Config — Web/TS Spec-Driven

Personal global configuration for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI, optimized for Web/TypeScript development with **executable spec-driven workflow**.

## Philosophy

> Natural-language specs do not constrain LLMs. Executable specs do.

This config enforces a three-tier specification system — business (Gherkin), interface (Zod), behavioral (tests) — so every feature produces artifacts the compiler, runtime, and CI can verify. Spec drift is caught automatically instead of discovered in production.

## Structure

```
.
├── CLAUDE.md                      # Global workflow (loaded every session)
├── settings.json                  # Permissions, hooks, MCP servers
├── settings.local.json            # Machine-specific overrides
├── .commitlintrc.json             # Conventional Commits lint
├── agents/
│   ├── spec-writer.md             # ★ Three-tier spec generation
│   ├── feature-validator.md       # Full-chain feature validation
│   ├── test-runner.md             # Auto-detect & run tests
│   ├── type-coverage.md           # ★ Type coverage + any-leakage audit
│   ├── bundle-analyzer.md         # ★ Bundle size + tree-shaking
│   ├── a11y-auditor.md            # ★ WCAG 2.1 AA audit (axe-core)
│   ├── visual-regression.md       # ★ Playwright screenshot diff
│   ├── dead-code-hunter.md        # ★ knip + ts-prune cleanup
│   ├── deployment-engineer.md     # CI/CD, GitOps, Docker/K8s
│   └── github-analyzer.md         # OSS project analysis
└── skills/
    ├── spec/SKILL.md              # ★ /spec — executable spec generator
    ├── contract/SKILL.md          # ★ /contract — schema → tRPC/OpenAPI
    ├── e2e/SKILL.md               # ★ /e2e — spec → Playwright tests
    ├── perf/SKILL.md              # ★ /perf — Lighthouse + bundle audit
    ├── preview/SKILL.md           # ★ /preview — Vercel/Netlify deploy
    └── validate/SKILL.md          # /validate — lint→type→test→review
```

★ = new in this revision.

## Seven-Layer Toolchain

| Layer | Tool | Purpose |
|-------|------|---------|
| 0 | **Spec** | Business (Gherkin) + Interface (Zod) + Behavioral (tests) — executable specs |
| 1 | MCP | Playwright (browser/E2E), GitHub (PR/Issue), chrome-devtools (profiling) |
| 2 | Plugin | Superpowers (brainstorming, TDD, debugging, code-review, worktrees) |
| 3 | Agents | spec-writer, type-coverage, bundle-analyzer, a11y-auditor, visual-regression, dead-code-hunter, feature-validator, test-runner |
| 4 | Skills | /spec, /contract, /e2e, /perf, /preview, /validate, /simplify, /commit |
| 5 | Hooks | PostToolUse biome fix, pre-commit lint + type-coverage, commitlint, notifications |
| 6 | Bash | biome, tsx/bun, type-coverage, knip, ts-prune, syncpack, fast-check, changeset |

## Development Workflow (7 stages)

```
brainstorm → spec → write-plan → contract → TDD → validate+perf+a11y → commit+preview
```

1. `/brainstorm` — Socratic requirement refinement
2. **`/spec`** — produce business + Zod + test skeletons
3. `/write-plan` — decompose into precise tasks referencing spec ID
4. **`/contract`** — expand schemas into tRPC/OpenAPI/GraphQL contracts
5. **TDD** (RED-GREEN-REFACTOR) — implement against spec tests
6. Triple validation: `/validate` + `/perf` + `a11y-auditor`
7. `/commit` (hooks run lint + type-coverage) → `/preview`

## Spec Programming — Five Iron Rules

1. **Zod-first** — all cross-boundary data must have a Zod schema; types via `z.infer`, never hand-written
2. **Spec ID traceability** — every commit references `spec:NNN`; deprecated specs get `@deprecated`, never deleted
3. **Property-Based Testing** — pure functions and state machines must have `fast-check` property tests
4. **Spec drift detection** — periodic scans verify schemas still match specs
5. **Design by Contract** — `tiny-invariant` / `ts-assert` for pre/post conditions

## Installation

```bash
# Copy config files to ~/.claude/
cp CLAUDE.md settings.local.json ~/.claude/
cp -r agents/ skills/ ~/.claude/

# Copy settings.json and fill in your GitHub PAT
cp settings.json ~/.claude/settings.json
# Edit ~/.claude/settings.json: replace <YOUR_GITHUB_PAT_HERE>

# Copy commitlint config to home directory
cp .commitlintrc.json ~/

# Install global toolchain
npm install -g @commitlint/cli @commitlint/config-conventional \
  @biomejs/biome type-coverage knip ts-prune syncpack

# Per-project dev deps (install once you adopt the workflow)
# npm i -D zod vitest fast-check @playwright/test @axe-core/playwright
# npm i -D tiny-invariant @asteasolutions/zod-to-openapi

# Superpowers plugin (if not auto-loaded)
# /plugin marketplace add obra/superpowers-marketplace
# /plugin install superpowers@superpowers-marketplace

# (Optional) Windows Toast notifications
# powershell -Command "Install-Module -Name BurntToast -Force"
```

## Security

- `settings.json` contains a **placeholder** for the GitHub PAT — replace after copying
- Never commit real tokens to version control
- `settings.local.json` is for machine-specific permission overrides
- Hooks never execute arbitrary remote code; all commands are pinned to known tools

## What changed in this revision

- **Layer 0 (Spec)** added as first-class citizen
- **6 new agents**: spec-writer, type-coverage, bundle-analyzer, a11y-auditor, visual-regression, dead-code-hunter
- **5 new skills**: /spec, /contract, /e2e, /perf, /preview
- **Hook upgrade**: PostToolUse biome auto-fix on `.ts/.tsx/.js/.jsx/.json`, PreToolUse type-coverage threshold
- **MCP swap**: removed SSH (rarely used in Web/TS), added chrome-devtools (profiling)
- **Toolchain**: biome replaces ESLint+Prettier (10× faster), fast-check for property testing
- **Workflow**: 7-stage pipeline with spec as gate before planning
