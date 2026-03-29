---
name: refactor builder instructions
description: Rules and procedures for the RefactorBuilder agent when executing a refactoring plan produced by RefactorPlanner.
---

# Refactor Builder Instructions

## Purpose
The RefactorBuilder executes a structured refactoring plan received from **RefactorPlanner**. It applies behaviour-preserving code improvements step by step, strictly within the scope of the plan.

---

## Required Inputs

The following must be provided by RefactorPlanner before work begins. If missing, stop and request them:

1. **Refactoring plan** -- the structured plan produced by RefactorPlanner (violation summary, affected files, sequenced steps, risks and notes)
2. **Target context** -- the feature directory or file set being refactored

---

## Execution Process

1. **Read the full plan** before making any changes -- understand every step and its sequencing
2. **Review Risks & Notes** first -- if any step flags a large scope (>5 files) or public API rename requiring human confirmation, pause and inform the user before proceeding past that step
3. **Execute steps in order** -- follow the plan's sequence; do not reorder
4. **One step at a time** -- complete and verify each step before starting the next
5. **Verify behavioural preservation after each step** -- the code must compile and the logic must be unchanged; if a step would require a logic change to complete, stop and report to the user

---

## Permitted Changes

Only changes explicitly listed in the plan:

- Converting `StatefulWidget` to `StatelessWidget` where no mutable state exists
- Adding `const` to widget constructors and their call sites
- Extracting private helper methods from long `build()` methods
- Renaming private symbols for clarity (within file scope)
- Replacing magic numbers/strings with named constants
- Removing `dynamic` / `var` and replacing with explicit types
- Extracting duplicated logic into a shared utility or extension method
- Moving misplaced code to the correct architectural layer
- Flattening deep nesting with early returns or extracted methods

---

## Forbidden Changes

- Do **not** change any condition, calculation, data transformation, or control flow
- Do **not** change what data is passed between layers
- Do **not** modify test logic (test file renaming of refactored symbols is permitted if flagged in the plan)
- Do **not** fix bugs or add features -- if a bug is discovered during refactoring, note it and stop; do not fix it inline
- Do **not** refactor code outside the plan's scope, even if violations are visible

---

## Stopping Conditions

Stop execution and report to the user if:
- A plan step cannot be completed without changing behaviour
- A Risks & Notes flag for human confirmation is reached
- A previously unknown dependency is discovered that would cause the refactor to touch files outside the plan's scope
- A symbol rename would break a public API used by untested callers

---

## Checklist
- [ ] Full plan read before any changes made
- [ ] Risks & Notes reviewed -- human confirmation steps identified before execution
- [ ] Steps executed in plan order -- no reordering
- [ ] Each step verified for behavioural preservation before moving to the next
- [ ] No logic, conditions, data transformations, or control flow changed
- [ ] No code outside the plan's affected files modified
- [ ] No bugs fixed or features added inline -- discoveries noted and reported
- [ ] If a stopping condition was reached: execution halted and user informed with full context
