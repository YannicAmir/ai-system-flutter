---
name: asset handling guidance
description: Project-specific asset coding guidelines for Flutter. Covers the assets/ folder structure, AppIconPaths constants class, AppIcon widget usage, pubspec.yaml registration, and naming conventions. Referenced by asset and branding agents.
---

# Asset Handling Guidance

> Asset paths are NEVER hardcoded in widgets. All paths come from `AppIconPaths` constants. All icon rendering uses the `AppIcon` widget.

---

## 1. Folder Structure

```
assets/
├── icons/                      ← UI icons (SVG + PNG), grouped by section
│   ├── nav/                    ← bottom navigation tab icons
│   ├── general/                ← app-wide shared icons
│   ├── more/                   ← settings / more section icons
│   ├── shop/                   ← shop/PLP icons
│   ├── stores/                 ← store locator icons
│   ├── treasure/               ← loyalty programme icons
│   ├── feedback/               ← feedback icons
│   ├── bag/                    ← shopping bag icons
│   ├── delete_account/         ← account deletion icons
│   ├── country/                ← country flag icons
│   ├── launcher/               ← app icon source files (not referenced in code)
│   └── splash/                 ← splash screen assets
│
├── images/                     ← larger illustration/bitmap images
└── fonts/                      ← custom font files (.otf)
```

All asset folders are explicitly declared in `pubspec.yaml` under `flutter.assets`.

---

## 2. AppIconPaths — The Path Constants Class

All asset paths (icons and images) are declared as `static String get` properties in a single `AppIconPaths` class in `lib/theme/<app>_icon_paths.dart`. Path prefixes are private constants; individual path getters compose from them.

```dart
class AppIconPaths {
  // Private path prefixes — one per folder
  static const _navIconsPath = 'assets/icons/nav/';
  static const _generalIconsPath = 'assets/icons/general/';
  static const _generalImagesPath = 'assets/images/';

  // Icons
  static String get close => '${_generalIconsPath}close.png';
  static String get search => '${_generalIconsPath}search.png';
  static String get filter => '${_generalIconsPath}filter.svg';

  // Images
  static String get welcomeImage => '${_generalImagesPath}welcome.png';
}
```

### Naming Conventions
- Use `lowerCamelCase` for getter names
- Group paths with a section comment header matching the folder name
- Active/inactive pairs for nav icons: `shopActive` / `shopInactive`
- Locale/region variants use a suffix: `bigDealsWelcome` / `bigDealsWelcomeDe`

---

## 3. AppIcon Widget — Rendering Icons

`AppIcon` is the single widget for rendering all icon assets (SVG and PNG). It detects the file type by extension and delegates to `SvgPicture.asset` or `Image.asset` internally.

```dart
// CORRECT
AppIcon(
  path: AppIconPaths.filter,
  color: AppColors.primary,
  height: 24,
  width: 24,
)

// WRONG — never call these directly for icon assets
SvgPicture.asset('assets/icons/general/filter.svg')
Image.asset('assets/icons/general/close.png')
```

### AppIcon Parameters

| Parameter | Default | Purpose |
|---|---|---|
| `path` | required | Asset path from `AppIconPaths` |
| `color` | `IconTheme` color | Tint — applied via `ColorFilter` for SVG, `color` for PNG |
| `height` / `width` | 24 | Size |
| `useRawColor` | `false` | Set `true` to render original colours without tinting |

### Color Tinting
- By default all icons inherit the ambient `IconTheme` color
- Set `useRawColor: true` for multi-colour icons (flag icons, branded illustrations) that must not be tinted

```dart
// Multi-colour flag — preserve original colours
AppIcon(
  path: AppIconPaths.ukFlag,
  useRawColor: true,
  height: 20,
  width: 28,
)
```

---

## 4. Large Images — When to Use Image.asset Directly

`Image.asset` is used directly (not via `AppIcon`) only for large bitmap illustrations where tinting is undesirable. The path must still come from `AppIconPaths`:

```dart
// CORRECT — large illustration, path from constants
Image.asset(
  AppIconPaths.welcomeImage,
  height: 200,
  fit: BoxFit.contain,
)

// WRONG — hardcoded path
Image.asset('assets/images/welcome.png')
```

---

## 5. Locale / Region-Specific Images

Resolve the path at the call site based on locale or region — do not create separate widgets per variant:

```dart
// Via locale code
Image.asset(
  l10n.locale.languageCode == 'de'
      ? AppIconPaths.bigDealsWelcomeDe
      : AppIconPaths.bigDealsWelcome,
)

// Via cubit state
final region = context.select((AppCubit c) => c.state.region);
Image.asset(region.isUk ? AppIconPaths.someUkImage : AppIconPaths.someDEImage)
```

---

## 6. Registering New Assets

When adding a new asset file:

1. **Place the file** in the correct folder under `assets/` — create a new subfolder only if the section is genuinely new
2. **Register in `pubspec.yaml`** only if it is a new subfolder (existing folders with a trailing `/` pick up all files automatically):
   ```yaml
   flutter:
     assets:
       - assets/icons/my_new_section/
   ```
3. **Add a path getter** to `AppIconPaths` in the correct section with the appropriate private prefix constant

---

## 7. Rules

| Rule | Detail |
|---|---|
| Asset paths | NEVER hardcoded in widgets — ALWAYS via `AppIconPaths` constants |
| Icon rendering | ALWAYS via `AppIcon` widget — NEVER `SvgPicture.asset` / `Image.asset` for icons |
| Icon color | Pass `color` param or inherit from `IconTheme`; `useRawColor: true` for multi-colour assets |
| Large illustrations | `Image.asset()` acceptable; path MUST still come from `AppIconPaths` |
| New assets | File → correct `assets/` subfolder; new folder → register in `pubspec.yaml`; path → add to `AppIconPaths` |

---

## Checklist
- [ ] New asset file placed in correct `assets/` subfolder
- [ ] If new subfolder: registered in `pubspec.yaml` under `flutter.assets`
- [ ] Path getter added to `AppIconPaths` in the correct section using the correct private prefix
- [ ] Getter name uses `lowerCamelCase` and follows naming conventions (active/inactive, region suffix)
- [ ] No asset path string hardcoded in any widget file
- [ ] Icons rendered via `AppIcon` widget — not `SvgPicture.asset` or `Image.asset` directly
- [ ] `useRawColor: true` set for multi-colour / branded assets that must not be tinted
