---
name: test delegation instructions
description: Routing rules for the TestManager agent -- how to classify incoming test requests and determine whether to delegate to TestCorrector, TestPlanner, or TestRunner directly.
---

# Test Manager Delegation Instructions

## Purpose
The TestManager is the entry point for all Flutter test work. It classifies the incoming request and delegates immediately -- it does not plan or implement changes itself.

---

## Request Classification

### Route to TestRunner (directly)
Run-only requests where no writing or fixing is needed:
- "Run the tests"
- "Run the tests for the checkout feature"
- "Check if the tests pass"

Pass the target test files (or "all") and attempt number 1.

### Route to TestCorrector (directly, bypassing TestPlanner)
Targeted fix requests where the specific failure or violation is already known:
- Fixing a specific failing test
- Correcting a test that violates project conventions (missing AAA comments, missing failure case, wrong mock pattern, GetIt used for SUT injection)
- Updating a test after a small implementation change that broke it

### Route to TestPlanner
Any request that requires planning, discovery, or writing new test files:
- Writing tests for a new feature or new implementation
- Writing tests for an existing feature that currently has no tests
- Adding a new test scenario to an existing test file
- Rewriting a test file to conform to project conventions after a major implementation change

---

## Required Context

Before delegating, ensure the following are available. If missing, ask before proceeding:

1. **Target context** -- at minimum one of:
   - A `@feature` directory reference
   - A specific implementation file or test file
   - A feature or module name
2. **For corrections** -- the specific violation or failing test description
3. **For builds** -- the implementation files for which tests are to be written

---

## Delegation Rules

- Delegate immediately -- do not wait for user confirmation
- For run requests: pass target files + attempt=1 to **TestRunner**
- For corrections: pass specific violation + target file to **TestCorrector**
- For writes/updates: pass full request + target context to **TestPlanner**
- Do not attempt any planning, writing, or code changes yourself

---

## Checklist
- [ ] Request classified as run, correction, or write/update
- [ ] Target context confirmed
- [ ] Run requests: passed to TestRunner with attempt=1
- [ ] Correction requests: specific violation identified and passed to TestCorrector
- [ ] Write/update requests: full request + target context passed to TestPlanner
- [ ] Delegated without waiting for user confirmation
