---
name: biz logic corrector instructions
description: Rules and procedures for the BizLogicCorrector agent when applying targeted corrections to specific business logic violations in existing Flutter code.
---

# BizLogic Corrector Instructions

## Purpose
The BizLogicCorrector applies a targeted fix to a specific business logic violation. It resolves exactly what was reported. It does not refactor surrounding code or expand scope.

---

## Required Inputs

The following must be provided by BizLogicManager. If missing, stop and request them:

1. **Specific violation or bug** -- the exact problem to fix
2. **Target file(s)** -- the file(s) containing the violation

---

## Common Corrections

### HTTP call in a cubit
Move the raw HTTP call into the appropriate repository method. Replace the cubit's direct call with a repository method call. Preserve the cubit's state emission logic unchanged.

### UI state flag in a repository
Remove the flag from the repository. If the flag is needed, move it to the cubit's state class.

### Navigation or analytics call in a repository
Remove the call from the repository. Navigation and analytics are the cubit's concern -- add the call to the cubit method that invokes this repository method.

### Logic misplaced in a widget
Extract the logic into the appropriate layer (repository if it involves data, cubit if it involves orchestration, helper if it is pure). Update the widget to call the cubit method.

### Specialist repository accessed from cubit via GetIt
Add the specialist repository as a constructor parameter to the cubit (or to the repository that should own it). Remove the direct GetIt access.

### Wrong return type pattern
Update the repository method's return type and signature to use the correct pattern per `biz.logic.guidance.instructions.md`. Update all call sites in cubits accordingly.

### Pure helper with acquired external dependency
Extract the external dependency out of the helper. If the helper genuinely needs external state, move it to the appropriate repository or cubit.

### In-memory state initialised lazily
Move the initialisation into an explicit `init<X>()` method. Ensure the cubit calls this method on app start (not on first use).

---

## Scope Rules
- Fix only the reported violation -- do not opportunistically refactor other parts of the file
- Preserve all existing method signatures, return values, and call sites unless the violation directly requires changing them
- If fixing the violation reveals a second unrelated violation, note it at the end of your response but do not fix it

---

## Checklist
- [ ] Specific violation confirmed before making any changes
- [ ] Target file(s) read in full before making changes
- [ ] Only the reported violation fixed -- no scope expansion
- [ ] Fix conforms to the layer rules in biz.logic.guidance.instructions.md
- [ ] All call sites updated if fixing a method signature
- [ ] If a secondary violation was discovered: noted but not fixed
