---
name: theme and styling guidance
description: Flutter theme and styling rules for all technology agents. Governs colors, typography, spacing, component theming, and theme access patterns for Flutter mobile apps.
---

# Theme & Styling Guidance

## 1. Colors

- All colors must be declared in the centralized `AppColors` constants class. Never write raw hex values (`Color(0xff...)`) or `Color.fromRGBO(...)` inline inside a widget.
- Alpha-derived variants (overlays, dividers, tints) must be computed and named in `AppColors` using `.withValues(alpha: ...)` — not inline at the call site.
- Prefer `Theme.of(context).colorScheme` semantic roles (`primary`, `surface`, `onSurface`, `error`, etc.) over direct `AppColors` references wherever a role maps clearly. Use named constants only for colors with no matching semantic role.
- Never use `Colors.white`, `Colors.black`, or any other `Colors.*` constant inline — reference `AppColors` instead. `Colors.transparent` is the only exception.

```dart
// Bad
color: const Color(0xff404040)
color: Colors.white

// Good
color: AppColors.textPrimary
color: Theme.of(context).colorScheme.surface
```

---

## 2. Typography

- The font family must be stored as a single named constant and set once in `ThemeData.fontFamily`. Never hardcode the font family string in a widget.
- Define all text styles in `ThemeData.textTheme` mapped to M3 semantic roles: `displayLarge/Medium/Small`, `headlineLarge/Medium/Small`, `titleLarge/Medium/Small`, `bodyLarge/Medium/Small`, `labelLarge/Medium/Small`.
- In widgets, access styles via `Theme.of(context).textTheme.<role>` and use `.copyWith(...)` to override only what differs. Never construct `TextStyle(...)` from scratch inline in new code.
- Font sizes must not be magic numbers — they must trace back to a `textTheme` entry or a named constant.
- Line height (`height`) when set must use the ratio form (e.g. `height: 22 / 14`), not an absolute value. M3 guidance: ~1.2× for display/headline/title, ~1.5× for body/label.
- Never set `fontFamilyFallback` per-widget — set it once globally in `ThemeData`.

```dart
// Bad
style: const TextStyle(fontSize: 14, color: Color(0xff404040))

// Good
style: Theme.of(context).textTheme.bodyMedium
// Good — override only what differs
style: Theme.of(context).textTheme.bodyMedium!.copyWith(fontWeight: FontWeight.bold)
```

---

## 3. Spacing & Layout

- Common spacing values (horizontal screen padding, bottom padding, app bar height, etc.) must be named constants — not repeated magic numbers.
- Any spacing value used in more than two places must be extracted as a named constant in the theme constants file.
- Do not use `MediaQuery.of(context).textScaleFactor` to clamp or override font sizes — accessibility text scaling must function naturally.

---

## 4. Component Theming

- Component-level styles (`BottomNavigationBarThemeData`, `DialogThemeData`, `BottomSheetThemeData`, `IconThemeData`, `DividerThemeData`, etc.) are set once in `ThemeData` and consumed globally via `Theme.of(context)`. Do not re-declare them locally unless intentionally scoping an override to a specific subtree.
- When overriding for a subtree, wrap in a `Theme` widget using `.copyWith()` — do not propagate style overrides by setting properties on every individual child widget.

```dart
// Bad — re-declaring component style locally
BottomNavigationBar(
  selectedItemColor: const Color(0xffE71324), // should come from ThemeData
)

// Good — subtree override
Theme(
  data: Theme.of(context).copyWith(...),
  child: ...,
)
```

---

## 5. Theme Access Pattern — Quick Reference

```dart
// Correct
color: Theme.of(context).colorScheme.primary
style: Theme.of(context).textTheme.bodyMedium

// Correct — partial override
style: Theme.of(context).textTheme.bodyMedium!.copyWith(fontWeight: FontWeight.bold)

// Wrong — inline hex
color: const Color(0xff404040)

// Wrong — inline TextStyle with magic numbers
style: const TextStyle(fontSize: 14, color: Color(0xff404040))
```

---

## 6. Constraints

- Only a light theme is supported unless dark mode has been explicitly scoped into the project. Do not introduce `darkTheme` as a side-effect of feature work.
- Changes to shared theme files (color constants, spacing constants, `ThemeData`) must be intentional and isolated — never a side-effect of feature work.
- Do not introduce a new color constant that is a near-duplicate of an existing one — check `AppColors` first.

---

## 7. Checklist

- [ ] No raw hex values or `Color.fromRGBO` inline in widgets.
- [ ] No `Colors.white`, `Colors.black`, or other `Colors.*` inline (except `Colors.transparent`).
- [ ] All text styles accessed via `Theme.of(context).textTheme.<role>`.
- [ ] No `TextStyle(...)` constructed from scratch inline.
- [ ] No magic number font sizes or spacing values.
- [ ] Line height uses ratio form (`height: 22 / 14`), not absolute.
- [ ] No component themes re-declared locally when `ThemeData` already covers them.
- [ ] No `darkTheme` introduced as a side-effect.
- [ ] No near-duplicate color constants added without checking `AppColors` first.
