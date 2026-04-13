---
name: perf
description: Full performance audit — Lighthouse, bundle size, Web Vitals, and runtime profiling. Use before release or when performance regresses.
---

# /perf — Performance Audit Pipeline

Run a comprehensive performance audit combining static (bundle size) and runtime (Lighthouse, Web Vitals) analysis.

## When to use

- Before a release
- When users report slowness
- After dependency upgrades
- As part of the 7-stage workflow's "triple validation" step

## Pipeline

### Step 1: Bundle analysis

Invoke the `bundle-analyzer` subagent. Expect output: total bundle size, top-N largest dependencies, tree-shaking issues.

### Step 2: Lighthouse run

If project has a dev server:

```bash
npm run dev &
DEV_PID=$!
sleep 3
npx lighthouse http://localhost:3000 \
  --output=json \
  --output-path=./lighthouse-report.json \
  --chrome-flags="--headless" \
  --only-categories=performance,accessibility,best-practices,seo
kill $DEV_PID
```

Extract scores:
- Performance
- LCP (Largest Contentful Paint)
- FCP (First Contentful Paint)
- TBT (Total Blocking Time)
- CLS (Cumulative Layout Shift)
- TTI (Time to Interactive)

### Step 3: Web Vitals targets

Compare against targets:
| Metric | Good | Needs Improvement | Poor |
|--------|------|-------------------|------|
| LCP | < 2.5s | 2.5-4.0s | > 4.0s |
| FID / INP | < 100ms | 100-300ms | > 300ms |
| CLS | < 0.1 | 0.1-0.25 | > 0.25 |

### Step 4: Runtime profiling (optional)

If chrome-devtools MCP is available and an interactive flow was flagged:
- Record a performance trace
- Identify long tasks (> 50ms)
- Report main-thread blocking

### Step 5: Cross-check against budget

If `performance-budget.json` exists in repo, compare actual vs. budget and flag regressions.

### Step 6: Output

```
## Performance Audit Report

### Lighthouse scores
- Performance: XX / 100
- Accessibility: XX / 100
- Best Practices: XX / 100
- SEO: XX / 100

### Web Vitals
| Metric | Value | Status |
|--------|-------|--------|
| LCP | X.Xs | ✓/⚠/✗ |
| INP | XXms | ✓/⚠/✗ |
| CLS | 0.XX | ✓/⚠/✗ |

### Bundle
Total: XXX KB gzip (target: XXX KB) — delta: +/-X%

### Top opportunities
1. [high] Replace moment with dayjs (-70 KB, LCP -0.3s)
2. [med]  Code-split admin route (-120 KB initial)
3. [low]  Preload hero image

### Regressions vs. last run
- LCP +0.2s (was 1.8s, now 2.0s) — investigate commit abc1234

**Verdict**: PASS / FAIL
```

## Rules

- Do not auto-fix. Report only.
- Always kill the dev server after Lighthouse run.
- If no dev server exists, report that runtime metrics are unavailable and proceed with bundle-only audit.
