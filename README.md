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
│   ├── code-reviewer.md         # Independent code review (fork context)
│   ├── feature-validator.md     # Full-chain feature validation
│   └── test-runner.md           # Auto-detect & run test suites
└── skills/
    └── validate/
        └── SKILL.md             # /validate - lint → typecheck → test → review pipeline
```

## Five-Layer Toolchain

| Layer | Tool | Purpose |
|-------|------|---------|
| 1 | MCP | Playwright (browser), GitHub (API) |
| 2 | Agents | Explore, Plan, code-reviewer, feature-validator, test-runner |
| 3 | Skills | /validate, /simplify, /commit, /claude-api |
| 4 | Hooks | Pre-commit lint, post-commit commitlint, desktop notification |
| 5 | Bash CLI | git, npm, commitlint |

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

# (Optional) Install BurntToast for Windows toast notifications
# powershell -Command "Install-Module -Name BurntToast -Force"
```

## Security

- `settings.json` contains a **placeholder** for the GitHub PAT — replace it after copying
- Never commit real tokens to version control
- `settings.local.json` is for machine-specific permission overrides
