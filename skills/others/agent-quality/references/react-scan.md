# react-scan Reference

## What it catches

Unnecessary re-renders caused by inline functions, inline objects, and missing memoization. Highlights offending components directly in the browser with a visual overlay: no code changes needed to start seeing problems.

## Install & run

```bash
npx -y react-scan@latest init
```

The init command auto-detects your framework (Next.js, Vite, Remix, etc.) and adds the necessary script or import.

**Manual install:**
```bash
bun add -d react-scan
```

Then add to your root layout (Next.js example):
```tsx
import { scan } from 'react-scan'
if (typeof window !== 'undefined') scan({ enabled: true })
```

## Use

Open the app in the browser: a floating toolbar appears in the corner. Components that re-render unnecessarily are highlighted in real time. Click a highlighted component to see why it re-rendered.

**Common fixes react-scan surfaces:**
- Inline functions as props: `onClick={() => ...}` → `useCallback`
- Inline objects as props: `style={{ color: 'red' }}` → extract to a constant or `useMemo`
- Missing `React.memo` on expensive pure components

## Add to CLAUDE.md

```markdown
## Performance
react-scan is installed. Run the dev server and open the app in the browser to see re-render highlights.
Before optimizing a component, confirm it's actually highlighted by react-scan first.
```

## Verify

- [ ] Dev server starts without errors after init
- [ ] Toolbar appears in the browser
- [ ] Triggering a state change highlights the re-rendering components
- [ ] Fixing an inline prop removes the highlight
