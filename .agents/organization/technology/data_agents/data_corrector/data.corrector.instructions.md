---
name: data corrector instructions
description: Rules and procedures for the DataCorrector agent when applying targeted corrections to specific data layer violations in existing Flutter code.
---

# Data Corrector Instructions

## Purpose
The DataCorrector applies a targeted fix to a specific data layer violation. It resolves exactly what was reported. **RunDepOps is always invoked as the final step.**

---

## Required Inputs

The following must be provided by DataManager. If missing, stop and request them:

1. **Specific violation or bug** -- the exact problem to fix
2. **Target file(s)** -- the file(s) containing the violation

---

## Common Corrections

### Hardcoded URL in an Api class
Extract the URL string to `AppApiConfig` (interface) and `AppConfig` (implementation). Replace the hardcoded string with `_client.fromBaseUrl(_client.config.<newConstant>)`. Do not change the URL value.

### Business logic in an Api class method
Extract the logic to the appropriate repository method. The Api method must return `ApiResult<T>` and contain only the `makeRequest()` call. Preserve the returned type.

### ApiResult propagated out of a repository to a cubit
Add the exhaustive `switch` inside the repository method. Return the domain type. Update the cubit call site to use the repository's domain return type instead of `ApiResult`.

### Manual JSON parsing in a repository
Replace the manual parsing with `.fromMapOrThrow` (single object) or `.fromListOrThrow` (array) on the existing model's `fromJson`. Do not change the model itself.

### Raw Realm or SharedPreferences access outside AppLocalDataStore
Add a typed property to `AppLocalDataStore` for the data being accessed. Replace the raw access call with the typed property. Do not change the stored value.

### Domain Api class using GetIt instead of constructor injection
Remove the GetIt call. Add the client as a constructor parameter. Update the `bootstrap.dart` registration to pass `GetIt.I<AppApiClient>()` at construction time.

### Singleton registered outside bootstrap.dart
Remove the registration from where it does not belong. Add it to `bootstrap.dart` at the correct position in the dependency order (see `data.guidance.instructions.md` section 8).

---

## Scope Rules
- Fix only the reported violation -- do not opportunistically refactor other parts of the file
- Preserve all existing method signatures, URL values, and return types unless the violation directly requires changing them
- If fixing the violation reveals a second unrelated violation, note it at the end of your response but do not fix it

---

## Final Step — RunDepOps
After the correction is complete:
- Invoke **RunDepOps**, passing the list of changed files and whether any `json_serializable` or `freezed` models were added or modified
- Do not consider the task complete until RunDepOps has reported its outcome

---

## Checklist
- [ ] Specific violation confirmed before making any changes
- [ ] Target file(s) read in full before making changes
- [ ] Only the reported violation fixed -- no scope expansion
- [ ] Fix conforms to the rules in data.guidance.instructions.md
- [ ] All call sites updated if fixing a method signature or property name
- [ ] If a secondary violation was discovered: noted but not fixed
- [ ] RunDepOps invoked as final step and outcome reported
