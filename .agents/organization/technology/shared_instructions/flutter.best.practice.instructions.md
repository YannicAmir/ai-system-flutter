---
name: flutter best practice
description: General Flutter best practices for quality and performance. Covers widget design, build efficiency, state management fundamentals, and common violations. Referenced by technology agents performing Flutter tasks.
---

# Flutter Best Practice

> Scope: general quality and performance. For styling see `theme.styling.guidance.instructions.md`, for Dart language rules see `dart.best.practice.instructions.md`, for BLoC patterns see `flutter-bloc-best-practice.instructions.md` (forthcoming).

---

## 1. Stateless vs Stateful Widgets

**Use `StatelessWidget` when:**
- The widget has no mutable state of its own.
- All data is passed in via constructor (pure function of its inputs).

**Use `StatefulWidget` when:**
- The widget must react to user interaction, animations, or async results that change its appearance.
- Local UI-only state is needed (e.g., toggle, form field focus).

**Rules:**
- Default to `StatelessWidget`; only reach for `StatefulWidget` when local mutable state is genuinely required.
- Never store business or domain state inside a `StatefulWidget` — delegate that to a state management layer.
- Keep `State` classes lean: only UI-lifecycle logic and local ephemeral state belong there.

---

## 2. Widget Construction Violations

### Never use functions to return widgets
```dart
// ❌ Bad — defeats the widget tree; no element lifecycle, no keys, no rebuild optimisation
Widget _buildTitle() => Text('Hello');

// ✅ Good — extract as a proper widget class
class _TitleWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) => Text('Hello');
}
```
Functions that return widgets bypass Flutter's element tree, preventing proper reconciliation, key propagation, and performance optimization.

### Prefer `const` constructors
```dart
// ✅ Use const wherever the widget and its parameters are compile-time constants
const Text('Static label')
const SizedBox(height: 16)
const Padding(padding: EdgeInsets.all(8))
```
`const` widgets are canonicalized and never rebuilt, providing a free performance win.

### Avoid deep anonymous nesting
- Extract any subtree more than ~4 levels deep into a named widget class.
- Named extraction improves readability, enables isolated rebuilds, and simplifies testing.

---

## 3. Build Method Rules

- **Keep `build()` pure and fast.** It must be a side-effect-free function of state and context — no I/O, no async calls, no heavy computation.
- **Never trigger `setState` inside `build()`.**
- **Never instantiate controllers, streams, or listeners inside `build()`** — do this in `initState` / `didChangeDependencies`.
- **Minimize the rebuild scope.** Call `setState` only on the smallest widget that owns the changing state.

---

## 4. List & Scroll Performance

- Always use `ListView.builder` / `GridView.builder` for lists of unknown or large length. Never use `ListView(children: [...])` for more than a handful of static items.
- Set `itemExtent` on `ListView.builder` when all items share a fixed height — enables O(1) scroll offset calculations.
- Avoid placing an unbounded `ListView` inside a `Column` without a `Flexible` or fixed-height constraint.
- Use `const` for list item widgets wherever possible.

---

## 5. Images & Assets

- Always specify `width` and `height` on `Image` widgets to prevent layout jank during load.
- Use `cached_network_image` for network images to avoid redundant network requests.
- Compress and size assets for the actual display dimensions; do not load a 2000px image into a 60px avatar.
- Use `AssetImage` / `NetworkImage` appropriately; never load images inside `build()` synchronously.

---

## 6. RepaintBoundary

- Wrap widgets that repaint frequently but independently (e.g., animated progress indicators, real-time data displays) in a `RepaintBoundary` to isolate their repaint from the rest of the tree.
- Do not add `RepaintBoundary` indiscriminately — each one allocates an offscreen buffer.

---

## 7. Resource Disposal

- Every `AnimationController`, `TextEditingController`, `ScrollController`, `StreamSubscription`, and `FocusNode` created in a `State` must be disposed in `dispose()`.
- Failing to dispose causes memory leaks and stale listeners.

```dart
@override
void dispose() {
  _controller.dispose();
  _subscription.cancel();
  super.dispose();
}
```

---

## 8. Separation of Concerns

- Widgets are responsible for **layout and presentation only**.
- Business logic, API calls, and data transformation belong in a dedicated state management layer (e.g., BLoC, ViewModel, Provider).
- Never place `http` calls, database queries, or complex data processing directly inside `build()` or widget classes.

---

## 9. Performance Checklist

- [ ] No functions returning widgets — use widget classes.
- [ ] All static widgets use `const` constructors.
- [ ] `ListView.builder` used for dynamic/long lists.
- [ ] `build()` contains no I/O, async operations, or heavy computation.
- [ ] All controllers and subscriptions disposed in `dispose()`.
- [ ] `setState` scoped to the smallest applicable widget.
- [ ] Images have explicit dimensions.
