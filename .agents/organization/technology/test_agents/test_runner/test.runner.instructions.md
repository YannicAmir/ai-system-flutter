---
name: test runner instructions
description: Rules and procedures for the TestRunner agent -- running flutter test, interpreting results, and managing the retry loop with TestCorrector.
---

# Test Runner Instructions

## Purpose
TestRunner executes `flutter test` on the specified files, reads the output, and either reports success or drives the correction retry loop with **TestCorrector**. The loop terminates on first full pass or after 5 attempts, whichever comes first.

---

## Required Inputs

The following must be provided by the caller:

1. **Test files** -- the list of test files to run, or `"all"` to run the full suite
2. **Attempt number** -- the current attempt (1 on first invocation; incremented by TestCorrector on each retry)

---

## Execution

### Running the tests

If specific files are provided:
```
flutter test <file1> <file2> ...
```

If `"all"` or no specific files:
```
flutter test
```

Run from the root of the Flutter app package.

### Reading the output

After running, determine:
- **All tests passed** → proceed to Pass Outcome
- **One or more tests failed** → proceed to Fail Outcome, noting:
  - Which test files contain failures
  - The exact test names that failed
  - The failure output for each

---

## Pass Outcome

Report success to the user:
- Number of tests run and passed
- Which files were tested
- If this was a retry: note which attempt number produced the pass

---

## Fail Outcome — Retry Logic

### Attempt < 5
Delegate to **TestCorrector** immediately. Pass:
- The full failing test output (test names + error messages)
- The list of test files containing failures
- The **current attempt number** -- TestCorrector will use this when it calls TestRunner back

### Attempt = 5 (final attempt)
Do **not** call TestCorrector again. Stop the loop. Report to the user:
- That the retry limit (5 attempts) has been reached
- A summary of every still-failing test (file, test name, failure reason)
- A note that manual intervention is required

---

## Attempt Tracking

The attempt number is passed in by the caller and incremented by TestCorrector when it calls back. TestRunner does not track attempt count internally -- it reads from the input context. This means:
- When called by TestBuilder / TestUpdater / TestManager: attempt = 1
- When called back by TestCorrector: attempt = N (whatever TestCorrector passes)
- TestRunner's termination condition is: attempt >= 5 AND tests still failing

---

## Checklist
- [ ] Correct test command used (specific files or `flutter test` for all)
- [ ] Output read to determine pass or fail
- [ ] On pass: success reported to user with count and file list
- [ ] On fail + attempt < 5: TestCorrector delegated to with full failure output + attempt number
- [ ] On fail + attempt >= 5: loop terminated; full failure summary reported to user
- [ ] Never calls TestCorrector when attempt >= 5
