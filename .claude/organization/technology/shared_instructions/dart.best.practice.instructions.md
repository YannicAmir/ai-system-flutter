---
name: dart best practice
description: General Dart best practices for quality, performance, and code clarity. Referenced by technology agents performing any Dart or Flutter task.
---

# Dart Best Practice

> Scope: general Dart quality and performance. Flutter-specific rules live in `flutter.best.practice.instructions.md`. Styling/theme rules in `theme.styling.guidance.instructions.md`. BLoC patterns in `flutter-bloc-best-practice.instructions.md` (forthcoming).

---

## 1. Null Safety

- Always write null-safe code. Never use `!` (null assertion) unless a null value is genuinely impossible and you can prove it at the call site.
- Prefer `??` and `?.` over null assertions.
- Use `late` only when a non-nullable variable is guaranteed to be initialized before first access (e.g., in `initState`). Do not use `late` as a workaround for nullable design.
- Model optionality explicitly with nullable types (`String?`) rather than sentinel values (empty string, `-1`, `null` as "not found").

```dart
// Bad
final name = user!.name;

// Good
final name = user?.name ?? 'Unknown';
```

---

## 2. `final` and `const`

- Declare every variable `final` by default. Only use `var` or mutable declarations when reassignment is genuinely required.
- Use `const` for values that are known at compile time — collections, objects, and primitives.
- `const` collections are deeply immutable and canonicalized; prefer them over `final` for static data.

```dart
// Good
const defaultPadding = 16.0;
final userId = user.id;      // assigned once, not reassigned
```

---

## 3. Collections

- Prefer collection literals (`[]`, `{}`, `<K,V>{}`) over constructor forms (`List()`, `Map()`).
- Use spread operators (`...`) and collection-if / collection-for instead of imperative `add`/`addAll` loops when building collections.
- Never cast a collection type with `as` — use `.cast<T>()` or rebuild with the correct type.
- Use `const []` / `const {}` for empty collections passed as default parameter values.

```dart
// Bad
final items = List<String>();
items.addAll(baseItems);
if (condition) items.add(extra);

// Good
final items = [
  ...baseItems,
  if (condition) extra,
];
```

---

## 4. Async / Await

- Always `await` a `Future` rather than accessing `.then()` chains except when intentionally fire-and-forget.
- Never discard a `Future` silently — if fire-and-forget is intentional, use `unawaited()` from `dart:async` to make it explicit.
- Catch errors at the appropriate level; avoid empty `catch` blocks.
- Prefer `Future<void>` over `Future<Null>` for async functions with no return value.
- Do not use `async` on a function that contains no `await` — it wraps the return value unnecessarily.

```dart
// Bad
void load() async {
  fetchData(); // unawaited, silent
}

// Good
void load() {
  unawaited(fetchData());
}
```

---

## 5. Naming Conventions

| Construct | Convention | Example |
|---|---|---|
| Classes, enums, typedefs | `UpperCamelCase` | `UserProfile`, `AuthState` |
| Variables, functions, params | `lowerCamelCase` | `userName`, `fetchData()` |
| Constants | `lowerCamelCase` | `defaultTimeout` |
| Private members | `_lowerCamelCase` | `_controller` |
| Files | `snake_case` | `user_profile.dart` |
| Packages / directories | `snake_case` | `auth_service/` |

- Names must be descriptive and unambiguous. Never use single-letter names outside loop indices.
- Boolean variables and getters should read as a predicate: `isLoading`, `hasError`, `canSubmit`.

---

## 6. Functions & Methods

- Keep functions single-purpose. If a function does more than one thing, split it.
- Limit function length to ~30 lines. If longer, extract logical units.
- Use named parameters for functions with more than two parameters or any boolean parameter.
- Never use positional booleans — they are unreadable at the call site.

```dart
// Bad
void configure(true, false, 10);

// Good
void configure({required bool enableCache, required bool verbose, required int timeout});
```

---

## 7. Error Handling

- Use typed `catch` clauses — catch `Exception` or specific subtypes, never bare `catch (e)` unless rethrowing.
- Do not swallow exceptions with empty catch blocks.
- Throw `Exception` or custom exception classes; do not throw raw strings or generic `Object`s.
- Use `Result`/`Either` patterns (or equivalent) at service boundaries rather than letting exceptions propagate across layers.

---

## 8. Code Quality Rules

- No dead code — remove unused variables, imports, parameters, and functions.
- No commented-out code — delete it; version control preserves history.
- Avoid magic numbers and magic strings — assign them to named `const` values.
- One class per file as a general rule; closely related small classes (e.g., sealed subclasses) may share a file.
- Imports must be ordered: dart: → package: → relative, each group separated by a blank line.

---

## 9. Enums

- Use enums to represent any fixed set of named values. Never use raw strings, integers, or boolean flags as a substitute for an enum.
- Prefer `enum` with methods or properties (enhanced enums) over parallel maps or switch-heavy helper functions.
- Exhaustive `switch` on enums without a `default` clause is required — it lets the compiler catch unhandled cases when new values are added.

```dart
// Bad — raw strings as state
String status = 'loading'; // unchecked, typo-prone
if (status == 'laoding') { ... } // silent bug

// Good — enum
enum FetchStatus { loading, success, failure }

FetchStatus status = FetchStatus.loading;

switch (status) {
  case FetchStatus.loading: showSpinner();
  case FetchStatus.success: showData();
  case FetchStatus.failure: showError();
  // compiler error if a new value is added and not handled
}
```

---

## 10. Performance

- Avoid creating objects inside loops or in hot paths — pre-allocate outside the loop.
- Prefer `where`, `map`, `fold`, and `any` over manual loops for collection transformations — they express intent clearly and the compiler can optimize them.
- Use `StringBuffer` for string concatenation in loops; never use `+=` on a `String` in a loop.
- Avoid `dynamic` — it disables type checking and hinders tree-shaking and AOT optimization.

```dart
// Bad
String result = '';
for (final s in items) { result += s; }

// Good
final buffer = StringBuffer();
for (final s in items) { buffer.write(s); }
final result = buffer.toString();
```

---

## 11. Quality Checklist

- [ ] All variables declared `final` unless reassignment is required.
- [ ] No `!` null assertions without proof of non-null at call site.
- [ ] No unawaited `Future`s without explicit `unawaited()`.
- [ ] No positional boolean parameters.
- [ ] No magic numbers or magic strings — use named `const`s.
- [ ] No dead code, unused imports, or commented-out code.
- [ ] No `dynamic` usage without justification.
- [ ] `StringBuffer` used for loop-based string concatenation.
- [ ] Enums used for all fixed sets of named values — no raw strings or integers as state flags.
