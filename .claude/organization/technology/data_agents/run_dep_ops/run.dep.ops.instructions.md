---
name: run dep ops instructions
description: Rules and procedures for the RunDepOps agent when running post-implementation dependency and code generation operations for the Flutter data layer.
---

# RunDepOps Instructions

## Purpose
RunDepOps runs the necessary build and code generation commands after a data layer change. It is always the final step in a DataBuilder, DataUpdater, or DataCorrector workflow.

---

## Decision — When to Run build_runner

Run `dart run build_runner build --delete-conflicting-outputs` if **any** of the following are true:
- A new `json_serializable` model was added (`.g.dart` file does not yet exist)
- An existing `json_serializable` model was modified (fields added, removed, or renamed)
- A new `freezed` class was added or modified

Do **not** run build_runner if:
- Only Api class methods were added (no model changes)
- Only `bootstrap.dart` registration was changed
- Only `AppApiConfig` / `AppConfig` URL constants were added
- Only `AppLocalDataStore` typed properties were modified

---

## Commands

### If build_runner is required
```
dart run build_runner build --delete-conflicting-outputs
```
Run from the root of the Flutter app package. Report whether it succeeded or failed with the relevant output.

### If build_runner is not required
Report: "build_runner not required — no json_serializable or freezed models were added or modified."

---

## Reporting

After completing, always report:
1. Whether build_runner was run (yes/no) and why
2. If run: success or failure, and any relevant output lines (errors, generated file names)
3. If failed: the exact error and what likely needs to be fixed

---

## Checklist
- [ ] Determined whether build_runner is required based on what was changed
- [ ] If required: ran `dart run build_runner build --delete-conflicting-outputs` from the correct package root
- [ ] Reported outcome clearly (run/not run, success/failure, generated files or errors)
