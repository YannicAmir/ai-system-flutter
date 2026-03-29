---
name: biz logic builder instructions
description: Rules and procedures for the BizLogicBuilder agent when implementing new Flutter business logic from a plan produced by BizLogicPlanner.
---

# BizLogic Builder Instructions

## Purpose
The BizLogicBuilder creates new Flutter business logic from scratch based on a structured plan from **BizLogicPlanner**. It follows the plan step by step, ensuring every piece of logic is placed in the correct layer and conforms to project conventions.

---

## Required Inputs

The following must be provided by BizLogicPlanner. If missing, stop and request them:

1. **Implementation plan** -- the structured plan (Overview, Layer Decision, New Files, Implementation Steps)
2. **Target context** -- the feature directory, repository, or cubit

---

## Implementation Checklist

Execute the plan in step order. For every new piece of logic, verify:

### Repository methods
- [ ] Method placed in the correct repository file under `lib/repositories/<domain>/`
- [ ] Return type uses the correct pattern per `biz.logic.guidance.instructions.md` (tuple, nullable, named tuple)
- [ ] Error mapping uses a domain-specific enum with a `_mapError` helper -- not raw status codes in the cubit
- [ ] No UI state flags, navigation calls, or analytics calls in the repository
- [ ] If composing multiple data sources: all sources called within the repository, not split across cubit

### Cubit orchestration
- [ ] Cubit sequences repository calls -- does not re-implement repository logic
- [ ] State emissions driven by repository outcomes -- not raw API responses
- [ ] New repository dependencies injected via cubit constructor
- [ ] No HTTP calls, JSON parsing, or retry logic in the cubit

### Helper utilities
- [ ] Helper is a pure function or extension with no external dependencies
- [ ] Placed in `lib/helpers/` (global) or `lib/features/<feature>/helpers/` (feature-scoped)
- [ ] No mutable state, no side effects

### Specialist repositories
- [ ] Created under `lib/repositories/<domain>/`
- [ ] Injected into consuming repositories via constructor -- not accessed from cubits via GetIt
- [ ] Owns exactly one subsystem concern (e.g. token management, config values)

### In-memory state
- [ ] Private field with eager initialisation method (not lazy on first use)
- [ ] Initialisation called on app start in the appropriate cubit

---

## Forbidden Patterns
- No HTTP calls in cubits
- No UI state flags in repositories
- No navigation or analytics calls in repositories
- No external dependencies in helpers
- No specialist repositories accessed directly from cubits via GetIt

---

## Checklist
- [ ] Plan steps executed in order
- [ ] Each piece of logic placed in the correct layer
- [ ] Return type pattern matches the plan's decision
- [ ] Error enums and mapping implemented if specified
- [ ] No forbidden patterns introduced
