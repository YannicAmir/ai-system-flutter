---
name: localization guidance
description: Project-specific localisation coding guidelines for Flutter. Covers TJXLocalizations usage, feature l10n extensions, accessibility strings, remote config strings, region variants, and formatting utilities. Referenced by localisation agents performing Flutter tasks.
---

# Localization Guidance

> All user-facing strings are resolved through `TJXLocalizations` — never hardcode strings in widget files. This project does NOT use Flutter's `.arb` / code-gen approach.

---

## 1. Architecture

Localisation is handled by `TJXLocalizations`, which resolves strings at runtime based on the current `Locale` using named `getString()` calls directly in Dart.

```
TJXLocalizations.of(context)           ← access point in widgets
        │
        ├── getString(en:, de:)          ← resolves to correct language
        ├── getStringGroup<T>(en:, de:)  ← resolves non-String types
        └── getStringFromMap(map)        ← resolves from a Map (remote config)
```

Extensions on `TJXLocalizations`:

| File | Purpose |
|---|---|
| `lib/l10n/common_localizations.dart` | App-wide shared UI strings |
| `lib/l10n/common_semantics_localizations.dart` | Shared accessibility strings |
| `lib/features/<feature>/l10n.dart` | Feature-specific UI strings |
| `lib/accessibility_l10n/pages/<feature>/` | Feature-specific accessibility strings |

---

## 2. Accessing Localisation in Widgets

Always use `TJXLocalizations.of(context)`. Assign it to a local variable named `l10n` at the top of `build()`. Never store it as an instance variable — it must be retrieved fresh from context per build.

```dart
@override
Widget build(BuildContext context) {
  final l10n = TJXLocalizations.of(context);

  return Text(l10n.myFeatureTitle);
}
```

---

## 3. Adding Strings — Feature l10n.dart

Every feature owns its UI strings in an extension on `TJXLocalizations` at `lib/features/<feature>/l10n.dart`.

**Simple string (no parameters) — use a getter:**

```dart
// lib/features/my_feature/l10n.dart
import 'package:tkmaxx/l10n/tjx_localizations.dart';

extension MyFeatureLocalizations on TJXLocalizations {
  String get myFeatureTitle => getString(
        en: 'My Feature',
        de: 'Mein Feature',
      );
}
```

**String with parameters — use a method, not a getter:**

```dart
String itemsFound(int count) => getString(
      en: '$count item${count == 1 ? '' : 's'} found',
      de: '$count Artikel gefunden',
    );

String welcomeMessage(String name) => getString(
      en: 'Welcome, $name',
      de: 'Willkommen, $name',
    );
```

**Pluralisation / conditional content — use Dart `switch` inside the string argument:**

```dart
String storesNearYou(int count) => getString(
      en: switch (count) {
        0 => 'No stores near you',
        _ => '$count stores near you',
      },
      de: switch (count) {
        0 => 'Keine Stores in deiner Nähe',
        _ => '$count Stores in deiner Nähe',
      },
    );
```

---

## 4. Language / Region Variants

`getString` supports region-specific overrides when DE and AT copy genuinely differs. Use `deDE` and `deAT` only when the copy is actually different — do not add them speculatively.

```dart
String get myString => getString(
      en: 'My string',
      de: 'Mein Text',               // used for both DE and AT by default
      deDE: 'Nur für Deutschland',   // optional: DE-specific override
      deAT: 'Nur für Österreich',    // optional: AT-specific override
    );
```

---

## 5. Grouped / Non-String Localisations

Use `getStringGroup<T>` when a localised value is not a plain `String` — for example, a data class holding multiple related strings:

```dart
PermissionStrings get locationPermissionStrings => getStringGroup(
      en: const PermissionStrings(
        title: 'Location access',
        description: 'We need your location...',
      ),
      de: const PermissionStrings(
        title: 'Standortzugriff',
        description: 'Wir benötigen deinen Standort...',
      ),
    );
```

---

## 6. Remote-Config / Dynamic Strings

When a string comes from a remote config map (e.g. `Map<String, String>` with `en`/`de` keys), use `getStringFromMap`:

```dart
// Remote config returns: {'en': 'Sale on now', 'de': 'Jetzt im Sale'}
final bannerText = l10n.getStringFromMap(config.plpBannerText);

// With a fallback if the map is empty
final text = l10n.getStringFromMap(config.someText, fallback: 'Default text');
```

---

## 7. Accessibility Strings

Accessibility strings (screen reader labels) are separate from UI strings. They live in `lib/accessibility_l10n/` with the `eaa` prefix convention:

```
lib/accessibility_l10n/
├── common/                         ← shared semantic labels
├── pages/<feature>/                ← page-level semantic labels
│   └── <feature>_l10n.dart
└── widgets/<widget>/               ← widget-level semantic labels
```

```dart
// lib/accessibility_l10n/pages/my_feature/my_feature_l10n.dart
extension MyFeatureSemanticsLocalizations on TJXLocalizations {
  String get eaaMyFeatureTitle => getString(
        en: 'My Feature page',
        de: 'Mein Feature Seite',
      );

  String get eaaMyButton => getString(
        en: 'tap to confirm',
        de: 'Zum Bestätigen tippen',
      );
}
```

Accessibility strings are used exclusively within `Semantics` widgets — never as visible text.

---

## 8. Common / Shared Strings

Before adding a new string to a feature's `l10n.dart`, check the shared files first to avoid duplicating strings like "Cancel", "Close", or "Search" that are already defined globally:

- `common_localizations.dart` — shared UI strings (button labels, error messages, navigation)
- `common_semantics_localizations.dart` — shared accessibility labels

---

## 9. Formatting Utilities

`TJXLocalizations` provides locale-aware formatting utilities. Use these instead of constructing `Intl` formatters manually:

```dart
// Currency
l10n.formatCurrency(19.99)                              // → '£19.99' or '19,99 €'
l10n.formatCurrency(19.99, showSymbol: false)
l10n.currencySymbol                                     // → '£' or '€'

// Date
l10n.formatDate(DateTime.now())
l10n.formatDate(DateTime.now(), pattern: DateFormat.MONTH_DAY)
```

---

## 10. Rules

| Rule | Detail |
|---|---|
| User-facing strings | NEVER hardcoded in widget files — ALWAYS in an `l10n` extension |
| Feature strings | `lib/features/<feature>/l10n.dart` |
| Shared UI strings | `lib/l10n/common_localizations.dart` — check here before adding feature strings |
| Accessibility strings | `lib/accessibility_l10n/` with `eaa` prefix — NEVER used as visible text |
| All supported locales | MUST provide both `en` and `de` |
| Region variants | Use `deDE` / `deAT` only when copy genuinely differs |
| `TJXLocalizations.of()` | Assigned to local `l10n` in `build()` — NEVER stored as instance variable |
| Formatting | Use `TJXLocalizations` utility methods — NEVER construct `Intl` formatters inline |

---

## Checklist
- [ ] No user-facing strings hardcoded in widget files
- [ ] All new feature strings added to `lib/features/<feature>/l10n.dart` as an extension on `TJXLocalizations`
- [ ] Common strings checked in `common_localizations.dart` before adding a new feature string
- [ ] Both `en` and `de` values provided for every string
- [ ] `deDE` / `deAT` only used where copy genuinely differs between regions
- [ ] Parameterised strings use a method (not a getter)
- [ ] Pluralisation uses Dart `switch` inside the string argument
- [ ] Accessibility strings placed in `lib/accessibility_l10n/` with `eaa` prefix
- [ ] Accessibility strings only used inside `Semantics` widgets — not as visible text
- [ ] `TJXLocalizations.of(context)` assigned to local `l10n` in `build()` — not stored as instance variable
- [ ] Currency, date, and number formatting uses `TJXLocalizations` utility methods — no inline `Intl` formatters
