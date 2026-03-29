---
name: biz logic enforcer instructions
description: Domain-specific checks for BizLogicEnforcer. Violations to look for when measuring business logic implementation against biz.logic.guidance.instructions.md. References qa.enforcement.pattern.instructions.md for retry loop rules.
---

# BizLogic Enforcer Instructions

## Guidance Source
Verify all changed files against every checklist item in `biz.logic.guidance.instructions.md`.

## Key Violation Categories

### Layer violations
- Business logic (conditions, calculations, orchestration) placed inside a data layer class (Api class, repository) or presentation layer (widget, cubit)
- Api class methods containing logic beyond calling ApiClient and returning ApiResult
- Cubit directly calling an Api class instead of a repository

### Return type violations
- Repository methods not returning the project's expected result type pattern
- Raw exceptions thrown from a repository instead of being mapped to domain error types
- ApiResult propagated to a cubit instead of being unwrapped in the repository

### Structure violations
- Repository method placed in the wrong class or file
- Specialist repository not created when one is needed
- Helper utility with side effects (should be pure functions)
- Missing constructor injection -- dependency obtained via GetIt inside a method body

### Naming and placement violations
- File not placed in the correct layer directory
- Class or method name not following project conventions

## Enforcement Loop
Follow the retry loop rules in `qa.enforcement.pattern.instructions.md`.

## Checklist
- [ ] All changed files read
- [ ] Every item in `biz.logic.guidance.instructions.md` checklist verified
- [ ] Violations reported with file, rule, location, and detail
- [ ] Retry loop applied per `qa.enforcement.pattern.instructions.md`
