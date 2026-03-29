---
name: analytics builder instructions
description: Rules and procedures for the AnalyticsBuilder agent when implementing analytics from scratch for Flutter features that currently have no analytics code.
---

# Analytics Builder Instructions

## Purpose
The AnalyticsBuilder implements analytics for Flutter features that currently have no analytics code. It is called by the **Analytics Planner** agent with a structured implementation plan and the target feature directory.

---

## Required Inputs

The following must be provided by AnalyticsPlanner before work begins. If missing, stop and request them:

1. **Implementation plan** -- the structured plan produced by AnalyticsPlanner
2. **Target feature directory** -- the `@feature` directory or path

---

## Strict Scope of Changes

### Permitted Actions
- Create `lib/features/<feature>/helpers/<feature>_event.dart` with all page constants and the feature event helper class
- Add `ScreenViewMixin` to routable page `State` classes and implement `updateScreenViewEvent`
- Add `logAdditionalEvent` calls where the plan specifies additional page-arrival events
- Add `AnalyticsRepository` constructor injection to cubits using the `?? GetIt.I()` fallback pattern
- Add `_analyticsRepository.logEvent(...)` calls inside cubit methods as specified in the plan
- Add `MyFeatureEvent.logUIEvent(...)` calls inside widget `onTap` / callback handlers as specified

### Forbidden Actions
- Do **not** modify any UI layout, styling, or widget hierarchy
- Do **not** modify state management logic, business logic, or data handling
- Do **not** add or modify user properties -- these belong in `UserCubit` only
- Do **not** call platform SDKs (Firebase, Airship, AppDynamics) directly
- Do **not** hardcode event name strings -- always use `AnalyticsEvents` constants
- Do **not** hardcode page dimension strings -- always use the constants defined in the helpers file

---

## Execution Process

1. **Confirm inputs** -- verify the implementation plan and target feature directory are present
2. **Review the plan** -- read every step before writing any code
3. **Create the helpers file first** -- page constants and feature event helper must exist before they are referenced by other files
4. **Build in plan order** -- follow the plan's sequenced steps
5. **Verify analytics conventions** -- confirm every event name uses `AnalyticsEvents`, every page dimension uses a constant, and no SDK is called directly

---

## Checklist
- [ ] Implementation plan and target feature directory confirmed before starting
- [ ] `helpers/<feature>_event.dart` created with all page constants (`pageName`, `pageType`, `contentGroup`) and action detail strings
- [ ] Feature event helper class (`XxxEvent.logUIEvent`) created in the helpers file
- [ ] `ScreenViewMixin` applied to all routable pages specified in the plan with `updateScreenViewEvent` implemented
- [ ] `logAdditionalEvent` added where the plan specifies additional page-arrival events
- [ ] `AnalyticsRepository` injected into cubits via constructor with `?? GetIt.I()` fallback
- [ ] All cubit `logEvent` calls use `AnalyticsEvents` constants and page constant fields
- [ ] All widget `logUIEvent` calls use action detail string constants
- [ ] No platform SDK called directly
- [ ] No event name or page dimension string hardcoded at any call site
- [ ] No UI layout, state logic, or business logic modified