# Accessibility Fixes

## Images

- Add descriptive `alt` text to all informative `<img>` elements
- Add `alt=""` to decorative images (not `alt` missing)
- Add `aria-label` or `aria-labelledby` to `<svg>` icons used as buttons

## Color contrast

- Check Lighthouse's contrast ratio findings
- Increase foreground/background contrast to meet WCAG AA (4.5:1 for normal text, 3:1 for large text)
- Never fix by removing the color: fix the specific hex values

## Keyboard navigation

- Ensure all interactive elements are reachable via Tab
- Add `tabindex="0"` to custom interactive elements (divs/spans acting as buttons)
- Never use `tabindex="-1"` on visible interactive elements unless intentional
- Ensure focus indicators are visible (don't suppress `:focus` outline without a replacement)

## ARIA

- Add missing `aria-label` to icon buttons and inputs without visible labels
- Ensure `role` attributes are valid and match the element's purpose
- Add `aria-expanded`, `aria-controls` to disclosure widgets (accordions, menus)
- Ensure `<form>` inputs have associated `<label>` elements

## Landmarks and headings

- Wrap main content in `<main>`
- Ensure heading hierarchy is sequential (h1 → h2 → h3, no skipping levels)
- Add `<nav>` around navigation lists

## Documents

- Add `lang` attribute to `<html>` element: `<html lang="en">`
- Ensure `<title>` is present and descriptive on every page
