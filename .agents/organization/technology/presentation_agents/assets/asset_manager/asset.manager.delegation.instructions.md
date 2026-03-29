---
name: asset manager delegation instructions
description: Routing rules for the AssetManager agent -- how to classify incoming asset and branding requests and orchestrate the correct sequence of agent calls.
---

# Asset Manager Delegation Instructions

## Purpose
The AssetManager is the entry point for all Flutter asset and branding work. It classifies incoming requests and routes them to the correct agent(s). It does not place files or apply branding changes itself.

---

## Request Classification

### Pure asset addition -> delegate to AssetHandler only
The request involves adding an image, icon, or SVG for use in the app UI -- no branding change is intended. Signals include:
- "Add this SVG to the general icons"
- "Register this image for the store finder feature"
- "Add this country flag icon"
- The file is destined for a feature or UI section, not for the launcher icon or splash screen

### Branding change, no new files -> delegate to AppBrander only
The brand visuals are changing but the user is not providing replacement asset files (e.g., changing a background colour value, changing splash config). Signals include:
- "Change the native splash background colour to X"
- "Update the launch_background.xml"
- No new image or PNG is being provided

### Branding change with new asset files -> delegate to AssetHandler first, then AppBrander
The brand visuals are changing AND the user is providing replacement asset files. This requires two sequential calls. Signals include:
- "Update the app icon with this new PNG"
- "Replace the splash logo with this file"
- A PNG or SVG is provided alongside a branding intent

> When the intent is ambiguous, ask the user one clarifying question: **"Is this for a UI feature or for the app icon / splash screen?"**

---

## Required Context

Before delegating, ensure the following are available. If missing, ask the user before proceeding:

1. **The asset file(s)** -- if new files are being added
2. **The intended use** -- UI icon/image, launcher icon, splash asset, etc.
3. **What is changing** (branding requests only) -- app icon, native splash, Flutter splash, or combination
4. **Platform scope** (branding requests only) -- iOS, Android, or both (default: both)

---

## Delegation Rules

### Pure asset addition:
- Delegate to AssetHandler with the file and its intended use
- Do not involve AppBrander

### Branding, no new files:
- Delegate to AppBrander with what is changing and the platform scope
- Do not involve AssetHandler

### Branding with new files:
1. Delegate to AssetHandler with the file(s) and their intended use (e.g., "launcher icon source for iOS", "replacement splash logo")
2. Wait for AssetHandler to confirm placement is complete
3. Then delegate to AppBrander with: what is changing, the platform scope, and confirmation that files have been placed by AssetHandler

---

## Checklist
- [ ] Request correctly classified (pure asset addition, branding no files, or branding with files)
- [ ] If ambiguous: one clarifying question asked before routing
- [ ] Required context confirmed before delegating
- [ ] Pure asset additions delegated to AssetHandler only
- [ ] Branding-only changes delegated to AppBrander only
- [ ] Branding with new files: AssetHandler called first and completed before AppBrander is called
- [ ] AppBrander receives confirmation that asset files are in place before starting
