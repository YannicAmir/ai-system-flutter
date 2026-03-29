---
name: state management enforcer instructions
description: Domain-specific checks for StateManagementEnforcer. Violations to look for when measuring BLoC/Cubit implementation against flutter.bloc.best.practice.instructions.md. References qa.enforcement.pattern.instructions.md for retry loop rules.
---

# State Management Enforcer Instructions

## Guidance Source
Verify all changed files against every checklist item in `flutter.bloc.best.practice.instructions.md`.

## Key Violation Categories

### State class violations
- State class not sealed or not using the correct base class pattern
- Missing copyWith on a state class that has mutable fields
- State class holding mutable objects (lists, maps) without unmodifiable wrappers
- Multiple responsibility states (a single state class doing too many things)

### Cubit/Bloc violations
- Business logic or UI logic placed directly in a cubit (belongs in a repository or helper)
- Cubit directly calling an Api class instead of a repository
- Cubit holding local state in instance variables instead of in the state class
- Missing error state or loading state for async operations
- `emit()` called after the cubit is closed (missing `isClosed` guard)

### Event violations (Bloc only)
- Event class not sealed
- Event handler not awaited correctly

### Dependency violations
- Repository or service obtained via GetIt inside a cubit method (must be injected via constructor)

## Enforcement Loop
Follow the retry loop rules in `qa.enforcement.pattern.instructions.md`.

## Checklist
- [ ] All changed files read
- [ ] Every item in `flutter.bloc.best.practice.instructions.md` checklist verified
- [ ] Violations reported with file, rule, location, and detail
- [ ] Retry loop applied per `qa.enforcement.pattern.instructions.md`
