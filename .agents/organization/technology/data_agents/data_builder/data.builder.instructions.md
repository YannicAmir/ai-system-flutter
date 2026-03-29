---
name: data builder instructions
description: Rules and procedures for the DataBuilder agent when implementing new Flutter data layer code from a plan produced by DataPlanner.
---

# Data Builder Instructions

## Purpose
The DataBuilder creates new Flutter data layer code from scratch based on a structured plan from **DataPlanner**. It follows the plan step by step, ensuring every piece of data layer code conforms to the project conventions. **RunDepOps is always invoked as the final step.**

---

## Required Inputs

The following must be provided by DataPlanner. If missing, stop and request them:

1. **Implementation plan** -- the structured plan (Overview, New Files, Implementation Steps, build_runner decision)
2. **Target context** -- the domain area or package

---

## Implementation Checklist

Execute the plan in step order. For each new piece of data layer code, verify:

### URL constants
- [ ] URL string added to `AppApiConfig` interface
- [ ] URL value added to `AppConfig` implementation
- [ ] Not hardcoded directly in the Api class method

### Api class methods
- [ ] Method calls `_client.makeRequest()` -- not `http.Client` directly
- [ ] URL built via `_client.fromBaseUrl(_client.config.<urlConstant>)`
- [ ] `fromJson` uses `.fromMapOrThrow` (single object), `.fromListOrThrow` (array), or `(_) {}` (void)
- [ ] Returns `ApiResult<T>` -- zero business logic inside the method
- [ ] `AppApiClient` injected via constructor -- not GetIt

### New domain Api class
- [ ] Created under `packages/<api_package>/lib/src/apis/`
- [ ] Constructor accepts `AppApiClient` only
- [ ] Registered as GetIt singleton in `bootstrap.dart` after `AppApiClient`
- [ ] Exported from the package barrel file

### json_serializable model
- [ ] `fromJson(Map<String, dynamic>)` constructor present
- [ ] `@JsonSerializable()` annotation applied
- [ ] File added to the correct models directory
- [ ] `part '<filename>.g.dart'` directive included

### AppLocalDataStore property
- [ ] Uses Realm for structured/persistent data; SharedPreferences for simple key-value
- [ ] Typed getter and setter (no raw key strings outside the store class)
- [ ] Realm writes go through `writeToRealm`

### bootstrap.dart registration
- [ ] Singleton registered at the correct position in the dependency order (see `data.guidance.instructions.md` section 8)
- [ ] No singleton registered from features, cubits, or anywhere other than bootstrap

---

## Forbidden Patterns
- No `http.Client` calls outside `ApiClient.makeRequest()`
- No hardcoded endpoint URLs in Api class methods
- No business logic inside Api class methods
- No `ApiResult` propagated out of a repository
- No manual JSON parsing in repositories (use `.fromMapOrThrow` / `.fromListOrThrow`)
- No raw Realm or SharedPreferences access outside `AppLocalDataStore`
- No GetIt usage inside Api package classes (package has no app dependency)

---

## Final Step — RunDepOps
After all implementation steps are complete:
- Invoke **RunDepOps**, passing the list of changed files and whether any `json_serializable` or `freezed` models were added or modified
- Do not consider the task complete until RunDepOps has reported its outcome

---

## Checklist
- [ ] Plan steps executed in order
- [ ] URL constants added to `AppApiConfig` + `AppConfig` -- not hardcoded
- [ ] Api methods return `ApiResult<T>` only -- no logic inside
- [ ] `fromJson` helpers used -- no manual parsing
- [ ] New singletons registered in bootstrap at the correct position
- [ ] No forbidden patterns introduced
- [ ] RunDepOps invoked as final step and outcome reported
