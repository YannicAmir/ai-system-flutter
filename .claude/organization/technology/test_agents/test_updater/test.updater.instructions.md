---
name: test updater instructions
description: Rules and procedures for the TestUpdater agent when modifying or extending existing Flutter test files from a plan produced by TestPlanner.
---

# Test Updater Instructions

## Purpose
The TestUpdater modifies or extends existing Flutter test files using a structured plan from **TestPlanner**. It changes only what the plan specifies and preserves all currently passing tests. **TestRunner is always invoked as the final step.**

---

## Required Inputs

The following must be provided by TestPlanner. If missing, stop and request them:

1. **Test plan** -- the structured plan (Overview, Test File Path, Mocks Required, Fake Data, Test Scenarios)
2. **Existing test file** -- the file to be modified
3. **Implementation files** -- the files being tested

---

## Pre-update Discovery

Before making any changes:
- Read the existing test file in full
- Understand which tests already exist and which are being added or changed
- Confirm the plan's steps are consistent with the current test file state
- If a planned test already exists and passes, skip it and note the skip

---

## Implementation Checklist

Execute plan steps in order. For each modification, verify:

### Adding new test scenarios
- [ ] New tests placed in the correct `group()` -- nested appropriately
- [ ] `// Arrange`, `// Act`, `// Assert` comments present
- [ ] Both success and failure paths added if the feature being added has neither yet

### Updating existing tests after implementation changes
- [ ] Updated stubs (`when(...)`) reflect the new implementation signature
- [ ] Updated `expect:` state sequences reflect new state types
- [ ] Existing unaffected tests left unchanged

### Adding missing mock classes
- [ ] New mock declared as private (`_MockXxx`) at the top of the file
- [ ] Mock used consistently -- not re-stubbed unnecessarily where `setUp` stubs suffice

---

## Scope Rules
- Change only tests and mocks listed in the plan
- Do not fix unrelated convention violations discovered while reading the file (note them at the end instead)
- Do not delete existing passing tests unless the plan explicitly says a behaviour was removed

---

## Final Step — TestRunner
After all updates are complete:
- Invoke **TestRunner** with the list of updated test files and attempt number 1
- Do not consider the task complete until TestRunner has reported its outcome

---

## Checklist
- [ ] Existing test file read in full before any changes
- [ ] Plan steps consistent with current file state -- pre-existing tests skipped and noted
- [ ] Steps executed in plan order
- [ ] Only files listed in the plan modified
- [ ] Passing tests preserved
- [ ] AAA comments present in all new/modified test bodies
- [ ] Only approved packages used
- [ ] If secondary violations discovered: noted at end, not fixed
- [ ] TestRunner invoked as final step
