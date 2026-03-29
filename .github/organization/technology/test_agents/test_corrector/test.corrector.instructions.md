---
name: test corrector instructions
description: Rules and procedures for the TestCorrector agent when fixing failing Flutter tests or test convention violations.
---

# Test Corrector Instructions

## Purpose
The TestCorrector fixes failing tests or convention violations -- whether called directly by TestManager or by TestRunner during the retry loop. After applying fixes, it always invokes **TestRunner** to verify the outcome.

---

## Required Inputs

The following must be provided by the caller. If missing, stop and request them:

1. **Failing test output** (when called by TestRunner) or **specific violation description** (when called by TestManager)
2. **Target test file(s)** -- the files containing the failures or violations
3. **Attempt number** -- provided by TestRunner in the retry loop; defaults to 1 if called directly by TestManager

---

## Common Fixes

### Test fails due to implementation change
Read the implementation file and the test file. Update stubs (`when(...)`), update expected state sequences in `blocTest`, or update widget assertions to match the new implementation. Do not change the implementation.

### Missing `// Arrange`, `// Act`, `// Assert` comments
Add the comments in the correct positions within each affected test body.

### Missing failure / error test case
Add a test case covering the failure path. Follow the pattern appropriate for the test type (repository `test()`, cubit `blocTest`, widget `testWidgets`).

### Mock class not declared as private
Rename the mock class to add the `_` prefix. Update all references in the file.

### GetIt used to inject the SUT
Replace the `GetIt.I<X>()` call with constructor injection. Add `GetIt.I.reset()` to `tearDown` if GetIt is still used for any other dependencies.

### `whenListen` missing `initialState` in widget test
Add the `initialState:` parameter to `whenListen(mockCubit, stream, initialState: ...)`.

### Wrong package used (e.g. mockito instead of mocktail)
Replace with the `mocktail` equivalent. Update imports.

### Fake data shared via imports from another test file
Extract the needed fake constants into the current test file. Remove the cross-file import.

---

## Scope Rules
- Fix only the reported failures or violations -- do not opportunistically refactor other parts of the file
- Do not modify the implementation files -- only the test files
- If a fix requires changing an implementation signature (e.g. a method was renamed), stop and report to the user instead of modifying implementation code

---

## After Fixing — TestRunner
After applying all fixes:
- Invoke **TestRunner** with:
  - The same test files that were failing
  - The attempt number received from the caller (1 if called by TestManager, N if called by TestRunner in retry loop)
- Do not consider the task complete until TestRunner reports its outcome

---

## Checklist
- [ ] Failing test output or violation description confirmed before making changes
- [ ] Target test file(s) read in full before making changes
- [ ] Only the reported failures/violations fixed -- no scope expansion
- [ ] No implementation files modified -- only test files
- [ ] If a fix requires an implementation change: stopped and reported to user
- [ ] TestRunner invoked with correct attempt number as final step
