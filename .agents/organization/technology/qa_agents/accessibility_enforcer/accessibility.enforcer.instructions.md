---
name: accessibility enforcer instructions
description: Domain-specific checks for AccessibilityEnforcer. Violations to look for when measuring accessibility implementation against flutter.accessibility.guidance.instructions.md. References qa.enforcement.pattern.instructions.md for retry loop rules.
---

# Accessibility Enforcer Instructions

## Guidance Source
Verify all changed files against every checklist item in `flutter.accessibility.guidance.instructions.md`.

## Key Violation Categories

### Semantics violations
- Interactive element missing a Semantics label
- Semantics label not using the `eaa`-prefixed l10n string (using hardcoded string instead)
- Decorative image not excluded from the semantic tree (`excludeFromSemantics: true` missing)
- `semanticsLabel` on an Image widget using visible text instead of an accessibility-specific string

### Touch target violations
- Tappable widget smaller than 48×48 dp without an explicit minimum touch target wrapper

### Focus management violations
- Modal or bottom sheet not managing focus (focus not moved to first interactive element)
- Focus not restored after modal dismissed
- `FocusNode` created but not disposed

### Screen reader violations
- Custom interactive widget not announcing its state (e.g., toggle not announcing checked/unchecked)
- Button wrapped in a GestureDetector without Semantics (not reachable by TalkBack/VoiceOver)
- Live region not used for dynamically updated content that users need to be informed of

## Enforcement Loop
Follow the retry loop rules in `qa.enforcement.pattern.instructions.md`.

## Checklist
- [ ] All changed files read
- [ ] Every item in `flutter.accessibility.guidance.instructions.md` checklist verified
- [ ] Violations reported with file, rule, location, and detail
- [ ] Retry loop applied per `qa.enforcement.pattern.instructions.md`
