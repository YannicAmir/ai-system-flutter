---
name: flutter accessibility guidance
description: Best practices for implementing accessibility in Flutter. Covers Semantics, widget labelling, focus management, touch targets, and testing. Referenced by technology agents performing Flutter tasks.
---

# Flutter Accessibility Guidance

> Aim: WCAG 2.2 AA compliance. For WCAG criteria definitions see `wcag.guidance.instructions.md`.

---

## 1. Semantics Widget

`Semantics` is the primary tool for communicating widget meaning to assistive technologies (TalkBack / VoiceOver).

```dart
// Wrap non-semantic widgets with a meaningful label
Semantics(
  label: 'Add to favorites',
  button: true,
  child: GestureDetector(onTap: _onTap, child: const Icon(Icons.favorite_border)),
)
```

**Rules:**
- Set `label` on any interactive or informational widget that has no inherent text.
- Set `button: true` for tappable elements that are not `ElevatedButton`/`TextButton`/`IconButton`.
- Set `image: true` + `label` on decorative and informational images.
- Use `excludeSemantics: true` inside a parent `Semantics` to suppress redundant child labels.
- Use `Semantics(header: true)` for section headings so screen readers can navigate by heading.

---

## 2. Built-in Widget Properties

Prefer built-in accessibility properties over wrapping in `Semantics` where possible.

```dart
// ✅ Use semanticsLabel directly on Image and Icon
Image.asset('assets/logo.png', semanticsLabel: 'App logo')
Icon(Icons.close, semanticsLabel: 'Close dialog')

// ✅ Use tooltip on IconButton — announced by screen readers
IconButton(
  icon: const Icon(Icons.share),
  tooltip: 'Share',
  onPressed: _onShare,
)
```

**Rules:**
- Always set `semanticsLabel` on `Image` and standalone `Icon` widgets.
- Always set `tooltip` on `IconButton` — it doubles as the accessibility label.
- Set `TextField`'s `decoration.labelText` or provide a wrapping `Semantics(label:)` so the field is announced correctly when focused.

---

## 3. Touch Target Size

Minimum interactive target: **48 × 48 dp** (WCAG 2.5.8 / Material guidance).

```dart
// ✅ Use SizedBox or ConstrainedBox to enforce minimum size
SizedBox(
  width: 48,
  height: 48,
  child: IconButton(icon: const Icon(Icons.info), onPressed: _onInfo),
)
```

- `IconButton` defaults to 48 dp — do not shrink it with `visualDensity` or `padding` below threshold.
- Custom tappable widgets (GestureDetector, InkWell) must always be wrapped in a 48 dp minimum container.

---

## 4. Focus Management

```dart
// Request focus on first interactive element when a dialog or sheet opens
@override
void initState() {
  super.initState();
  WidgetsBinding.instance.addPostFrameCallback((_) {
    FocusScope.of(context).requestFocus(_firstFieldNode);
  });
}

// Trap focus inside a modal
FocusTrap(child: _ModalContent())
```

**Rules:**
- When a bottom sheet, dialog, or overlay opens, move focus to the first interactive element inside it.
- When a bottom sheet or dialog closes, return focus to the element that triggered it.
- Use `FocusTraversalGroup` to control tab order within complex layouts.
- Never suppress focus entirely — `Focus(canRequestFocus: false)` must be intentional and documented.

---

## 5. Text Scaling

- Never use fixed pixel font sizes (`TextStyle(fontSize: 14)` without respecting `MediaQuery.textScaleFactor`).
- Use `Theme.of(context).textTheme` styles — they scale automatically.
- Avoid `maxLines: 1` without an ellipsis on user-facing labels — prefer allowing wrapping.

---

## 6. Announcements & Live Regions

```dart
// Announce a dynamic status update to screen readers without moving focus
SemanticsService.announce('Item added to bag', TextDirection.ltr);
```

- Use `SemanticsService.announce` for transient status messages (success toasts, loading completion).
- Mark content that updates automatically with `Semantics(liveRegion: true)` so screen readers announce changes without user action.

---

## 7. Checklist

- [ ] All interactive elements have a screen-reader label (via `tooltip`, `semanticsLabel`, or `Semantics(label:)`)
- [ ] Touch targets are minimum 48 x 48 dp
- [ ] Focus moves into modals/sheets on open and returns on close
- [ ] Text uses theme styles (not fixed pixels); layout tested at 200% text scale
- [ ] Dynamic status updates use `SemanticsService.announce` or `liveRegion: true`
