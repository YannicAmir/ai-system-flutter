---
name: asset handler instructions
description: Rules and procedures for the AssetHandler agent when placing new asset files into the Flutter project, registering them in pubspec.yaml, and adding AppIconPaths getters.
---

# Asset Handler Instructions

## Purpose
The AssetHandler places new image, icon, or SVG files into the correct `assets/` subfolder, registers any new subfolder in `pubspec.yaml`, and adds the corresponding `AppIconPaths` getter. It is called by **AssetManager** -- either for a pure asset addition, or as the first step in a branding workflow (after which AssetManager will call AppBrander to apply the branding changes).

---

## Required Inputs

The following must be confirmed before any work begins. If missing, ask before proceeding:

1. **The asset file** -- the file the user wants to add (or the file AssetManager has instructed to place as part of a branding workflow)
2. **The intended use** -- what the asset is for (UI icon, feature image, splash asset, launcher icon, etc.) — this determines which folder it belongs in

---

## Execution Process

### Step 1: Determine the Correct Folder

Use the intended use to select the correct target folder from `asset.handling.guidance.instructions.md`:

| Asset type | Target folder |
|---|---|
| App-wide UI icon | `assets/icons/general/` |
| Nav tab icon | `assets/icons/nav/` |
| Feature-specific icon | `assets/icons/<section>/` (match to closest existing section) |
| Large illustration / bitmap | `assets/images/` |
| Splash screen asset | `assets/icons/splash/` |
| App launcher icon | `assets/icons/launcher/` |
| Country flag icon | `assets/icons/country/` |

If no existing section fits, ask the user to confirm a new subfolder name before creating it.

### Step 2: Place the File
- Copy the asset file to the determined folder path

### Step 3: Register in pubspec.yaml (new subfolders only)
- If the target folder is **new** (does not already exist in `pubspec.yaml`), add it under `flutter.assets`:
  ```yaml
  flutter:
    assets:
      - assets/icons/my_new_section/
  ```
- Existing folders with a trailing `/` already cover all files — do not re-register them

### Step 4: Add the AppIconPaths Getter
- Open `lib/theme/<app>_icon_paths.dart`
- Locate the correct section comment matching the target folder
- Add a `static String get` using the correct private prefix constant:
  ```dart
  static String get myNewIcon => '${_generalIconsPath}my_new_icon.svg';
  ```
- Name the getter using `lowerCamelCase`, following naming conventions:
  - Active/inactive nav pairs: `shopActive` / `shopInactive`
  - Region variants: `welcomeImageDe`

---

## Scope Restrictions
- Do **not** modify any widget files or Dart source beyond `AppIconPaths`
- Do **not** make any changes to branding-specific native files (storyboard, launch_background.xml, mipmap directories) — those are AppBrander's responsibility
- Do **not** run `dart run flutter_launcher_icons` — that is AppBrander's responsibility

---

## Checklist
- [ ] Asset use confirmed -- correct target folder identified
- [ ] Asset file placed in the correct `assets/` subfolder
- [ ] If new subfolder: registered in `pubspec.yaml` under `flutter.assets` with trailing `/`
- [ ] Existing folders not re-registered in `pubspec.yaml`
- [ ] `AppIconPaths` getter added in the correct section with the correct private prefix
- [ ] Getter name uses `lowerCamelCase` and follows naming conventions
- [ ] No widget files or Dart source beyond `AppIconPaths` modified
- [ ] No native branding files touched
