---
name: asset enforcer instructions
description: Domain-specific checks for AssetEnforcer. Violations to look for when measuring asset handling implementation against asset.handling.guidance.instructions.md. References qa.enforcement.pattern.instructions.md for retry loop rules.
---

# Asset Enforcer Instructions

## Guidance Source
Verify all changed files against every checklist item in `asset.handling.guidance.instructions.md`.

## Key Violation Categories

### File placement violations
- Asset file placed in the wrong subfolder (e.g., icon placed in images/, image placed in icons/)
- Asset file name not following the project's snake_case naming convention

### pubspec.yaml violations
- New asset folder not registered in pubspec.yaml
- Registered path has trailing slash missing or incorrect format

### AppIconPaths violations
- Getter added to the wrong class or section
- Getter name not matching the asset file name in a clear, descriptive way
- Getter missing for a newly added asset
- Path string in getter does not exactly match the registered pubspec path

## Enforcement Loop
Follow the retry loop rules in `qa.enforcement.pattern.instructions.md`.

## Checklist
- [ ] All changed files read (asset file location, pubspec.yaml, AppIconPaths)
- [ ] Every item in `asset.handling.guidance.instructions.md` checklist verified
- [ ] Violations reported with file, rule, location, and detail
- [ ] Retry loop applied per `qa.enforcement.pattern.instructions.md`
