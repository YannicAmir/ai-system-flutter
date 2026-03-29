---
name: app brander instructions
description: Rules and procedures for the AppBrander agent when updating the Flutter app icon, native splash screens, or Flutter splash screen assets.
---

# App Brander Instructions

## Purpose
The AppBrander applies brand-level visual changes: the app icon, native splash screens (iOS and Android), and the Flutter splash. It is called by **AssetManager**. When the request includes new asset files, AssetManager will have already delegated to AssetHandler to place those files before calling AppBrander -- asset placement is complete by the time AppBrander starts.

---

## Required Inputs

The following will be provided by AssetManager. If any are missing, ask before proceeding:

1. **What is changing** -- app icon, native splash, Flutter splash, or some combination
2. **Platform scope** -- iOS only, Android only, or both (default: both)
3. **Confirmation that asset files are in place** -- for requests with new files, AssetManager confirms AssetHandler has completed placement

---

## Execution Process

### Step 1: Apply Branding Changes

Work through only the sections that are relevant to what is changing:

#### App Icon — iOS
1. Confirm `launcher-icon.png` has been placed by AssetHandler in `assets/icons/launcher/`
2. Confirm `remove_alpha_ios: true` is present in `pubspec.yaml` — add it if missing
3. Run `dart run flutter_launcher_icons` to regenerate the iOS icon set
4. Do **not** manually edit any generated files in `AppIcon.appiconset`

#### App Icon — Android
1. Confirm `launcher-icon-foreground.png` has been placed by AssetHandler
2. Replace `ic_launcher_foreground.png` at every mipmap density (`mdpi`, `hdpi`, `xhdpi`, `xxhdpi`, `xxxhdpi`)
3. If the background colour is changing, update the colour value in `res/values/`
4. Do **not** run `flutter_launcher_icons` for Android

#### Native Splash — iOS
1. If changing the background colour: edit the normalised `<color>` value in `LaunchScreen.storyboard`
2. If adding or replacing a logo: replace the PNG files in `LaunchImage.imageset` at `@1x`, `@2x`, and `@3x`

#### Native Splash — Android
1. If replacing the background image: replace `background.png` in `res/drawable/`
2. If adding a centred logo over a colour: update `launch_background.xml` with a second `<item>` containing a centred `<bitmap>`

#### Flutter Splash
1. Confirm replacement splash asset(s) have been placed by AssetHandler in `assets/icons/splash/`
2. Verify `AppIconPaths.splashlogo` and `AppIconPaths.eclipse` point to the correct file names — update `AppIconPaths` if file names have changed
3. Do **not** hardcode any splash path strings in the `AppLogo` widget or any other widget

### Step 2: Verify Completeness
When a brand change touches the splash, confirm both native and Flutter splash layers have been updated.

---

## Scope Restrictions
- Do **not** modify any feature UI widgets or business logic
- Do **not** hardcode asset paths -- all paths must come from `AppIconPaths`
- Do **not** run `flutter_launcher_icons` for Android
- Do **not** manually edit generated iOS icon files in `AppIcon.appiconset`

---

## Checklist
- [ ] Required inputs confirmed from AssetManager (what is changing, platform scope, confirmation that asset files are placed)
- [ ] If iOS app icon: `launcher-icon.png` replaced, `remove_alpha_ios: true` present, `dart run flutter_launcher_icons` run
- [ ] If Android app icon: `ic_launcher_foreground.png` replaced at every mipmap density, background colour updated if needed
- [ ] If iOS native splash: `LaunchScreen.storyboard` colour updated and/or `LaunchImage.imageset` replaced at all scales
- [ ] If Android native splash: `background.png` replaced and/or `launch_background.xml` updated
- [ ] If Flutter splash: asset files replaced in `assets/icons/splash/`, `AppIconPaths` getters updated if file names changed
- [ ] If brand splash changed: BOTH native splash AND Flutter splash updated
- [ ] No asset paths hardcoded in any widget
- [ ] No feature UI, business logic, or state management code modified
