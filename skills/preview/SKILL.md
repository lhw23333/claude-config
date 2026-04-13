---
name: preview
description: Deploy current branch to preview environment (Vercel/Netlify/Cloudflare Pages) and post the URL back to the PR. Use after /validate passes.
---

# /preview — Preview Deployment

Deploy the current branch to a preview environment and return a shareable URL, optionally commenting back on the GitHub PR.

## When to use

- After `/validate` passes
- Before requesting code review
- When sharing work-in-progress with stakeholders

## Detect platform

Check in order:
1. `vercel.json` or `.vercel/` → Vercel
2. `netlify.toml` or `.netlify/` → Netlify
3. `wrangler.toml` → Cloudflare Pages/Workers
4. `render.yaml` → Render
5. Fallback: static build + upload instructions only

## Steps

### Step 1: Preflight

- Ensure working tree is clean (`git status`) or all changes are committed
- Ensure current branch is pushed to origin
- Note the current commit SHA

### Step 2: Deploy

**Vercel:**
```bash
npx vercel --yes --prebuilt=false
```
Capture the preview URL from stdout.

**Netlify:**
```bash
npx netlify deploy --build --alias=$(git rev-parse --abbrev-ref HEAD | tr '/' '-')
```
Capture the preview URL.

**Cloudflare Pages:**
```bash
npx wrangler pages deploy dist --branch=$(git rev-parse --abbrev-ref HEAD)
```

### Step 3: Smoke test

Use Playwright via Chrome MCP to:
1. Visit the preview URL
2. Check page loads (200, title present)
3. Take a screenshot for the PR comment
4. Optionally run accessibility check via `a11y-auditor` agent

### Step 4: Comment on PR (if GitHub MCP available)

```markdown
🚀 **Preview deployed**

- URL: <preview-url>
- Commit: <sha>
- Built from: <branch>

**Smoke test**: ✓ loads, ✓ title, ✓ a11y
**Screenshot**: <inline image>
```

Post via GitHub MCP `mcp__github__create_comment` on the current PR.

### Step 5: Output summary

```
Preview deployed:
  🌐 URL:    <preview-url>
  📌 Commit: <sha>
  🌿 Branch: <branch>
  💬 PR comment: posted / skipped (no PR)
```

## Rules

- Never deploy to production. This is preview-only.
- Never leak secrets — rely on platform env var configuration, don't bake credentials into the build.
- If the platform is unknown, ask the user instead of guessing.
- If the working tree is dirty, refuse to deploy and prompt to commit first.
