---
name: data enforcer instructions
description: Domain-specific checks for DataEnforcer. Violations to look for when measuring data layer implementation against data.guidance.instructions.md. References qa.enforcement.pattern.instructions.md for retry loop rules.
---

# Data Enforcer Instructions

## Guidance Source
Verify all changed files against every checklist item in `data.guidance.instructions.md`.

## Key Violation Categories

### URL and endpoint violations
- Hardcoded URL strings in Api class methods (must use ApiClient path constants)
- Base URL defined outside the correct configuration class

### ApiClient hierarchy violations
- Api class not extending the correct ApiClient subclass
- Request method not using the ApiClient helper methods
- Response not parsed using the correct fromJson helper pattern

### Model violations
- json_serializable model missing required annotations
- Manual JSON parsing instead of generated fromJson/toJson
- Model placed in the wrong directory

### Storage violations
- Raw storage access (SharedPreferences, Hive, etc.) outside AppLocalDataStore
- Storage key defined as a magic string instead of a typed constant

### Bootstrap violations
- Dependency registered in the wrong order in bootstrap
- Missing registration for a new class that is injected elsewhere

### Logic in data layer
- Business logic placed inside an Api class method or a json model

## Enforcement Loop
Follow the retry loop rules in `qa.enforcement.pattern.instructions.md`.

## Checklist
- [ ] All changed files read
- [ ] Every item in `data.guidance.instructions.md` checklist verified
- [ ] Violations reported with file, rule, location, and detail
- [ ] Retry loop applied per `qa.enforcement.pattern.instructions.md`
