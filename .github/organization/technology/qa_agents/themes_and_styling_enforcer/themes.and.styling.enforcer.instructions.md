---
name: themes and styling enforcer instructions
description: Domain-specific checks for ThemesAndStylingEnforcer. Violations to look for when measuring UI/theming implementation against theme.styling.guidance.instructions.md. References qa.enforcement.pattern.instructions.md for retry loop rules.
---

# Themes and Styling Enforcer Instructions

## Guidance Source
Verify all changed files against every checklist item in `theme.styling.guidance.instructions.md`.

## Key Violation Categories

### Theme token violations
- Hardcoded colour value (hex, RGB, named colour) instead of a theme colour token
- Hardcoded text style (fontSize, fontWeight, etc.) instead of a text theme style
- Hardcoded spacing value (padding, margin) instead of a spacing token or constant

### Component violations
- Custom widget reimplementing a pattern that already exists in the design system component library
- Design system component used with overridden styling that breaks the intended token hierarchy

### Font violations
- Font family defined inline instead of using the theme's text theme
- Font weight or size overriding what the text theme specifies without justification

### Dark mode / theme violations
- Colour or style that is not responsive to the current ThemeData (breaks dark mode support)
- `Theme.of(context)` accessed outside of a `build()` method

## Enforcement Loop
Follow the retry loop rules in `qa.enforcement.pattern.instructions.md`.

## Checklist
- [ ] All changed files read
- [ ] Every item in `theme.styling.guidance.instructions.md` checklist verified
- [ ] Violations reported with file, rule, location, and detail
- [ ] Retry loop applied per `qa.enforcement.pattern.instructions.md`
