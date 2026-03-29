---
name: test builder instructions
description: Rules and procedures for the TestBuilder agent when writing new Flutter test files from a plan produced by TestPlanner.
---

# Test Builder Instructions

## Purpose
The TestBuilder writes new Flutter test files from scratch based on a structured plan from **TestPlanner**. It follows the plan step by step, ensuring every test conforms to project conventions. **TestRunner is always invoked as the final step.**

---

## Required Inputs

The following must be provided by TestPlanner. If missing, stop and request them:

1. **Test plan** -- the structured plan (Overview, Test File Path, Mocks Required, Fake Data, Test Scenarios)
2. **Implementation files** -- the files being tested

---

## Implementation Checklist

For every new test file written, verify:

### File placement
- [ ] Test file placed at the path mirroring `lib/` under `test/`
- [ ] File named `<subject>_test.dart`

### Structure
- [ ] Top-level `group()` by class name
- [ ] Nested `group()` by method or scenario where applicable
- [ ] Mock classes declared as private (`_MockXxx`) at the top of the file
- [ ] Fake data constants declared at the top of the file, `const` where possible, locally scoped

### Every `test()` or `blocTest` body
- [ ] `// Arrange`, `// Act`, `// Assert` comments present (or `// Assert` alone for one-liners)
- [ ] Both success and failure paths present for every feature under test

### Repository tests
- [ ] Plain `test()` calls
- [ ] SUT assigned to `sut` variable, constructed via constructor injection
- [ ] `when(...)` stubs on Api/repository mocks set in Arrange section

### Cubit tests
- [ ] `blocTest<CubitType, StateType>` used
- [ ] `build:` constructs cubit with injected mock repos
- [ ] `expect:` lists full emitted state sequence
- [ ] `verify:` used when mock interaction count matters

### Widget tests
- [ ] `testWidgets` with `TestHelpers.pumpFakeApp()` used
- [ ] `whenListen(mockCubit, stream, initialState: ...)` used -- no direct state emission
- [ ] Feature-scoped cubits passed via `BlocProvider.value`
- [ ] `await tester.pump()` or `pumpAndSettle()` after interactions
- [ ] `GetIt.I.reset()` in `tearDown` if GetIt was used

---

## Approved Packages Only
Use only the packages listed in `test.guidance.instructions.md` section 1. Do not introduce any additional test dependencies.

---

## Final Step — TestRunner
After all test files are written:
- Invoke **TestRunner** with the list of newly written test files and attempt number 1
- Do not consider the task complete until TestRunner has reported its outcome

---

## Checklist
- [ ] Plan steps executed in order
- [ ] All test files placed at correct mirror paths under `test/`
- [ ] AAA comments present in every test body
- [ ] Both success and failure cases written for every feature
- [ ] Mock classes private, fake data local and `const`
- [ ] Only approved packages used
- [ ] TestRunner invoked as final step
