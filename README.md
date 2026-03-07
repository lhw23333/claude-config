# Claude Code Global Config

Personal global configuration for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI.

## Structure

```
.
├── CLAUDE.md                    # Global workflow instructions (loaded in every session)
├── settings.json                # Global settings (permissions, hooks, MCP servers)
├── settings.local.json          # Local permission overrides (not shared across machines)
├── .commitlintrc.json           # Conventional Commits lint config (place in ~/)
├── agents/
│   ├── feature-validator.md     # Full-chain feature validation
│   └── test-runner.md           # Auto-detect & run test suites
└── skills/
    └── validate/
        └── SKILL.md             # /validate - lint → typecheck → test → review pipeline
```

## Six-Layer Toolchain

| Layer | Tool | Purpose |
|-------|------|---------|
| 1 | MCP | Playwright (browser), GitHub (API) |
| 2 | Plugin | Superpowers (brainstorming, TDD, debugging, code-review, git-worktrees) |
| 3 | Agents | Explore, Plan, feature-validator, test-runner |
| 4 | Skills | /validate, /simplify, /commit, /claude-api |
| 5 | Hooks | Pre-commit lint, post-commit commitlint, desktop notification |
| 6 | Bash CLI | git, npm, commitlint |

## Installation

```bash
# Copy config files to ~/.claude/
cp CLAUDE.md settings.local.json ~/.claude/
cp -r agents/ skills/ ~/.claude/

# Copy settings.json and fill in your GitHub PAT
cp settings.json ~/.claude/settings.json
# Edit ~/.claude/settings.json: replace <YOUR_GITHUB_PAT_HERE> with your token

# Copy commitlint config to home directory
cp .commitlintrc.json ~/

# Install global dependencies
npm install -g @commitlint/cli @commitlint/config-conventional

# Install Superpowers plugin
# /plugin marketplace add obra/superpowers-marketplace
# /plugin install superpowers@superpowers-marketplace

# (Optional) Install BurntToast for Windows toast notifications
# powershell -Command "Install-Module -Name BurntToast -Force"
```

## Security

- `settings.json` contains a **placeholder** for the GitHub PAT — replace it after copying
- Never commit real tokens to version control
- `settings.local.json` is for machine-specific permission overrides
