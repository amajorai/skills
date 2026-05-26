---
name: bundle-analysis
description: Analyze JavaScript bundle size, identify bloat, and implement fixes: code splitting, lazy loading, tree shaking, and dependency swaps. Use when page load is slow, Lighthouse performance score is low, or before launching a web app.
argument-hint: <framework: next | vite | remix | other>
---

# Bundle Analysis

You are analyzing and optimizing the JavaScript bundle. Work through each phase in order.

**Framework:** {{args}}


## Phase 1: Measure Baseline

Generate a bundle analysis report:

```bash
command -v bun >/dev/null 2>&1 && PM=bun || (command -v pnpm >/dev/null 2>&1 && PM=pnpm || PM=npm)
```

**Next.js**:
```bash
# First install and wire up the analyzer (prerequisite — ANALYZE=true is a no-op without it):
$PM add -d @next/bundle-analyzer
# Wrap the config in next.config.(js|mjs|ts):
#   const withBundleAnalyzer = require('@next/bundle-analyzer')({ enabled: process.env.ANALYZE === 'true' })
#   module.exports = withBundleAnalyzer(nextConfig)
# Then build with the env var set:
ANALYZE=true $PM run build
```

**Vite**:
```bash
$PM add -d rollup-plugin-visualizer
# Add to vite.config: visualizer({ open: true, gzipSize: true })
$PM run build
```

**Other**:
```bash
$PM add -d webpack-bundle-analyzer  # or source-map-explorer
```

Record baseline metrics:
- Total JS size (gzipped)
- Largest chunks by size
- First Load JS (if Next.js)
- Lighthouse Performance score


## Phase 2: Identify Top Offenders

Spawn **2 parallel subagents**:

| Subagent | Focus |
|----------|-------|
| 1 | Read the bundle analyzer output: which modules are largest? Which appear in multiple chunks when they shouldn't? |
| 2 | `package.json` dependencies: find packages with cheaper alternatives (e.g., `moment` → `date-fns`, `lodash` → native, `axios` → `fetch`) |

Produce a ranked list of optimizations by estimated savings.


## Phase 3: Code Splitting

For routes/pages that are not on the critical path:

1. **Dynamic imports**: lazy-load components loaded below the fold or on interaction:
   ```typescript
   const HeavyChart = lazy(() => import('./HeavyChart'))
   ```
2. **Route-based splitting**: verify each route is its own chunk (Next.js does this automatically; Vite needs route lazy imports)
3. **Vendor chunk splitting**: ensure large dependencies (React, charting libs) are split into their own chunks for long-term caching
4. **Conditional imports**: only import heavy polyfills or libs when the browser actually needs them


## Phase 4: Dependency Swaps

For each identified heavy dependency with a lighter alternative:

1. Confirm the replacement covers all current usage patterns
2. Install replacement, remove old package
3. Update all import sites
4. Run tests: confirm nothing breaks
5. Measure the size delta

Common swaps:
- `moment` / `dayjs` → `date-fns` (tree-shakeable) or `Temporal` (native in Node 26+ and most modern browsers as of 2026, but Safari still lacks support — needs the `@js-temporal/polyfill` for full browser coverage)
- `lodash` → native array/object methods or `lodash-es` with tree shaking
- `axios` → native `fetch`
- `uuid` → `crypto.randomUUID()` (native)
- Full `@mui/material` → pick only needed components


## Phase 5: Tree Shaking

Verify tree shaking is working:

1. Check that all imports from large packages use named imports: `import { specific } from 'lib'` not `import lib from 'lib'`
2. Confirm `"sideEffects": false` in the package.json of any internal packages
3. For barrel files (`index.ts` that re-exports everything), consider direct imports instead
4. Verify `browserslist` target is set: shipping modern JS to modern browsers saves size


## Phase 6: Image & Font Optimization (bonus)

While in the performance mindset:

- [ ] Images use next-gen formats (WebP, AVIF)
- [ ] Images have explicit `width` and `height` to prevent layout shift
- [ ] Fonts use `font-display: swap` and are subsetted to used characters
- [ ] No unused CSS (check with PurgeCSS or built-in framework tooling)


## Phase 7: Measure After

Re-run the bundle analyzer and Lighthouse:

- [ ] Total JS size reduced
- [ ] First Load JS improved (if Next.js)
- [ ] Lighthouse Performance score improved
- [ ] No functionality regressions (run test suite)

Present a before/after comparison table.


## Completion Report

- Baseline metrics (size, Lighthouse score)
- Optimizations applied (list with estimated savings each)
- Final metrics (size, Lighthouse score)
- Deferred optimizations (with reason)
