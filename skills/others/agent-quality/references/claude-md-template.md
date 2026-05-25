# CLAUDE.md Quality Tools Template

Add the relevant subsections to your project `CLAUDE.md` based on which tools are installed:

```markdown
## Quality Tools

### react-doctor (if installed)
Run `npx react-doctor@latest` after making React changes to check the health score.
Target: score ≥ 75. Use `--diff origin/main` on PRs to scan only changed files.

### react-scan (if installed)
Open the app in the browser: the toolbar shows re-render highlights in real time.
Only optimize components that react-scan confirms are actually re-rendering unnecessarily.

### react-grab (if installed)
Hover over any UI element and press ⌘C / Ctrl+C to copy its component name, file path, and HTML.
Paste the output here when referencing a specific UI element: don't describe it vaguely.

### expect (if installed)
Run `/expect` after any UI change to verify in a real browser.
Use `/expect -m "<area>"` to focus on a specific flow.

### agentation (if installed)
For UI feedback, use Agentation to annotate elements and paste the export here.
Annotations include CSS selectors and file paths: use them directly.
```
