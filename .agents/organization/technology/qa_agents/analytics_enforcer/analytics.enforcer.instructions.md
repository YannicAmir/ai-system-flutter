---
name: analytics enforcer instructions
description: Domain-specific checks for AnalyticsEnforcer. Violations to look for when measuring analytics implementation against analytics.guidance.instructions.md. References qa.enforcement.pattern.instructions.md for retry loop rules.
---

# Analytics Enforcer Instructions

## Guidance Source
Verify all changed files against every checklist item in `analytics.guidance.instructions.md`.

## Key Violation Categories

### Page constant violations
- Screen view event fired without a typed page constant (hardcoded string used instead)
- Page constant not placed in the correct constants file

### Event helper violations
- Analytics event fired directly in a widget or cubit without going through the feature event helper class
- Event helper method not following the project's naming convention
- Missing analytics SDK call inside an event helper method

### ScreenViewMixin violations
- Page widget not mixing in ScreenViewMixin
- `pageName` getter not overridden with the correct page constant

### Cubit injection violations
- Analytics event helper injected/called outside of the cubit
- Analytics called without null-safety guard when the SDK may not be initialised

### User property violations
- User property set in the wrong lifecycle point
- User property set directly via the SDK instead of through the designated helper

## Enforcement Loop
Follow the retry loop rules in `qa.enforcement.pattern.instructions.md`.

## Checklist
- [ ] All changed files read
- [ ] Every item in `analytics.guidance.instructions.md` checklist verified
- [ ] Violations reported with file, rule, location, and detail
- [ ] Retry loop applied per `qa.enforcement.pattern.instructions.md`
