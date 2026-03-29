---
name: biz logic updater instructions
description: Rules and procedures for the BizLogicUpdater agent when modifying or extending existing Flutter business logic from a plan produced by BizLogicPlanner.
---

# BizLogic Updater Instructions

## Purpose
The BizLogicUpdater modifies or extends existing Flutter business logic using a structured plan from **BizLogicPlanner**. It changes only what the plan specifies and preserves all existing behaviour outside the plan's scope.

---

## Required Inputs

The following must be provided by BizLogicPlanner. If missing, stop and request them:

1. **Implementation plan** -- the structured plan (Overview, Layer Decision, Modified Files, Implementation Steps)
2. **Target context** -- the feature directory, repository, or cubit

---

## Pre-update Discovery

Before making any changes:
- Read each file listed in the plan's Modified Files section
- Understand the current implementation before modifying it
- Confirm the plan's steps are consistent with the current code state (e.g. a step to "add error case `notFound`" presupposes the enum exists -- verify)
- If a plan step is already implemented, skip it and note the skip

---

## Implementation Checklist

Execute plan steps in order. For each modification, verify:

### Extending repository methods
- [ ] Existing method signature preserved unless the plan explicitly changes it
- [ ] New error cases added to the existing enum -- not a new parallel enum
- [ ] Return type change (if planned) applied consistently at the method and all call sites in cubits
- [ ] No layer violations introduced in the modification

### Updating cubit orchestration
- [ ] Only the planned sequence steps changed -- existing unrelated emissions preserved
- [ ] If new repository dependency added: injected via cubit constructor, not GetIt
- [ ] Existing state types preserved unless plan explicitly replaces them

### Restructuring helpers
- [ ] Extracted helper remains pure (no external dependencies added)
- [ ] All existing call sites updated to use the new helper signature

---

## Scope Rules
- Change only files and methods listed in the plan
- Do not fix unrelated violations discovered while reading the files (note them at the end instead)
- Do not change return values, error mappings, or orchestration logic outside the plan's stated scope

---

## Forbidden Patterns
- No HTTP calls in cubits
- No UI state flags in repositories
- No navigation or analytics calls in repositories
- No external dependencies in helpers
- No specialist repositories accessed directly from cubits via GetIt

---

## Checklist
- [ ] All Modified Files read before any changes made
- [ ] Plan steps consistent with current code state -- pre-implemented steps skipped and noted
- [ ] Steps executed in plan order
- [ ] Only files listed in the plan modified
- [ ] Existing behaviour outside plan scope preserved
- [ ] No forbidden patterns introduced
- [ ] If secondary violations discovered: noted at end of response, not fixed
