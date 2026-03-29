---
name: refactor guidance
description: Guidelines for how refactoring agents should approach, scope, and execute refactoring work. Covers violation identification, behavioural safety, scope constraints, and the categories of violations that warrant a refactor. Referenced by RefactorPlanner and RefactorBuilder.
---

# Refactor Guidance

> Refactoring is **behaviour-preserving**. The external behaviour of refactored code must be identical before and after. If a change alters what the code does, it is not a refactor.

---

## 1. What Qualifies as a Refactor

A refactor addresses code that works correctly but violates the project's coding standards. Valid refactor triggers:

- Flutter widget patterns that violate `flutter.best.practice.instructions.md` (e.g., StatefulWidget used where StatelessWidget is sufficient, missing `const`, improper widget decomposition)
- Dart idiom violations that violate `dart.best.practice.instructions.md` (e.g., unnecessary dynamic types, missing null safety patterns, verbose constructs with idiomatic alternatives)
- Architecture violations that violate `architecture.guidance.instructions.md` (e.g., business logic in widgets, wrong layer access, feature code outside its bounded context)
- Software development best practice violations that violate `software.dev.best.practice.instructions.md` (e.g., functions too long, duplicated logic, poor naming, missing abstraction)
- Restriction violations that violate `restrictions.instructions.md` (e.g., reserved patterns used incorrectly)

A refactor does **not** add new features, fix bugs in logic, or change the user-visible behaviour of the application.

---

## 2. Violation Identification

When auditing code for refactor candidates, evaluate against each instruction set systematically:

### Flutter violations
- `StatefulWidget` where no mutable state exists
- Widget `build()` methods doing work that belongs in a cubit or repository
- Missing `const` constructors on widgets and their subtrees
- Widgets that should be extracted (build methods exceeding ~50 lines, deeply nested subtrees)
- Inline styles or values that should reference theme tokens

### Dart violations
- Use of `dynamic` or `var` where a type can be explicit
- Long method chains that should be broken into named intermediate variables
- Missing or incorrect use of `final` / `const`
- Functions longer than ~30 lines that can be meaningfully decomposed
- Duplicated logic that should be extracted into a shared utility or extension

### Architecture violations
- Business logic or data transformation inside widget `build()` methods
- Direct access to a repository from a widget (should go through a cubit)
- Feature code importing from another feature's internal files
- Code placed in the wrong layer (e.g., UI logic in a repository, API calls in a widget)

### Software development violations
- Functions / methods with unclear or misleading names
- Magic numbers or strings that should be named constants
- Long parameter lists that should be grouped into a data class
- Deep nesting (3+ levels) that should be flattened with early returns or extracted methods

---

## 3. Behavioural Safety Rules

These rules are non-negotiable:

- **No logic changes.** Do not change conditions, data transformations, calculations, or control flow.
- **No API changes.** Do not rename public methods or change their signatures without updating all call sites in the same operation.
- **No state management changes.** Widget restructuring must not change how state is provided, consumed, or mutated.
- **Rename only within scope.** If renaming a private symbol, confirm no call sites outside the file are affected.
- **One violation category at a time per step.** Do not mix architectural restructuring with Dart idiom fixes in the same step -- keep changes atomic and reviewable.

---

## 4. Scope Constraints

- Refactor only the files and symbols explicitly identified in the plan
- Do not refactor adjacent code that was not flagged, even if violations are visible
- Do not change test files unless the refactor renames a symbol that tests reference
- If a refactor requires touching more than 5 files to remain behaviour-preserving, flag it in the plan for human confirmation before proceeding

---

## 5. Plan Format

Every refactoring plan must contain:

### Violation Summary
A table listing each violation: file, line range (approximate), violation type (Flutter / Dart / Architecture / SoftDev), and the specific rule broken.

### Affected Files
List of files to be modified.

### Refactoring Steps
Numbered, sequenced steps. Each step must specify:
- The file
- The violation being addressed
- The specific change (e.g., "extract `_buildHeader()` method", "convert `StatefulWidget` to `StatelessWidget`", "replace magic number `24` with `AppSpacing.md`")
- Confirmation that behaviour is preserved

### Risks & Notes
Flag any step that touches a public API, involves renaming, or affects more than 3 files -- these require extra care during execution.

---

## Checklist
- [ ] All violations identified and mapped to the specific rule they violate
- [ ] No logic, behaviour, or user-visible output will change as a result of this refactor
- [ ] Refactor scope limited to flagged files only -- no opportunistic out-of-scope changes
- [ ] Each plan step is atomic and addresses a single violation category
- [ ] Public API / symbol renames flagged in Risks & Notes with all call sites accounted for
- [ ] Steps involving more than 5 files flagged for human confirmation
