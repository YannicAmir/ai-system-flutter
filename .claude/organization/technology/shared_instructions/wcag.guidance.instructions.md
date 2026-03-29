---
name: wcag guidance
description: WCAG 2.2 Level AA compliance reference. Covers the criteria that apply to mobile Flutter apps. Used alongside flutter.accessibility.guidance.instructions.md. Referenced by technology agents performing Flutter tasks.
---

# WCAG 2.2 AA Guidance

> Scope: WCAG 2.2 Level AA criteria relevant to mobile Flutter applications. Level AAA and desktop-only criteria are excluded.

---

## 1. Perceivable

### 1.1 Text Alternatives
- **1.1.1 Non-text Content (A):** Every non-decorative image, icon, and graphic must have a text alternative. Purely decorative images must be hidden from assistive technologies.

### 1.3 Adaptable
- **1.3.1 Info and Relationships (A):** Structure and relationships conveyed visually must be available programmatically (e.g., headings, lists, grouped controls labelled as a group).
- **1.3.3 Sensory Characteristics (A):** Instructions must not rely solely on shape, colour, size, or position (e.g., "tap the red button" is non-compliant).
- **1.3.4 Orientation (AA):** Content must not be locked to portrait or landscape unless essential.
- **1.3.5 Identify Input Purpose (AA):** Programmatically identify the purpose of common input fields (name, email, phone, address) so autofill and assistive tools can assist.

### 1.4 Distinguishable
- **1.4.1 Use of Colour (A):** Colour must not be the only visual means of conveying information.
- **1.4.3 Contrast (Minimum) (AA):** Normal text ≥ 4.5:1. Large text (≥ 18pt / 14pt bold) ≥ 3:1.
- **1.4.4 Resize Text (AA):** Text must be resizable up to 200% without loss of content or functionality.
- **1.4.10 Reflow (AA):** Content must reflow to a single column at 320 CSS px equivalent without horizontal scrolling (except for content that requires two-dimensional layout, e.g., maps, data tables).
- **1.4.11 Non-text Contrast (AA):** UI components (buttons, inputs, focus indicators) and informational graphics must have a contrast ratio of at least 3:1 against adjacent colours.
- **1.4.12 Text Spacing (AA):** No loss of content or functionality when text spacing is adjusted (line height ≥ 1.5×, letter spacing ≥ 0.12em, word spacing ≥ 0.16em). Applies to system-level overrides.
- **1.4.13 Content on Hover or Focus (AA):** If additional content appears on hover/focus (tooltips, popovers), it must be dismissible, hoverable, and persistent.

---

## 2. Operable

### 2.1 Keyboard Accessible (Switch / External Keyboard)
- **2.1.1 Keyboard (A):** All functionality must be operable via an external keyboard or switch device. No keyboard traps.
- **2.1.4 Character Key Shortcuts (A):** If single-character keyboard shortcuts are implemented, provide a mechanism to remap or disable them.

### 2.4 Navigable
- **2.4.3 Focus Order (A):** Focus order must be logical and consistent with the visual reading order.
- **2.4.6 Headings and Labels (AA):** Headings and labels must be descriptive.
- **2.4.7 Focus Visible (AA):** Any keyboard-focusable interface component must have a visible focus indicator.
- **2.4.11 Focus Not Obscured (Minimum) (AA) — new in 2.2:** When a component receives keyboard focus, it must not be entirely hidden by other content (e.g., a sticky header).

### 2.5 Input Modalities
- **2.5.1 Pointer Gestures (A):** Functionality using multi-point or path-based gestures must have a single-pointer alternative.
- **2.5.3 Label in Name (A):** For UI components with a visible text label, the accessible name must contain the visible label text.
- **2.5.4 Motion Actuation (A):** Functionality triggered by device motion (shake, tilt) must have a UI alternative, and be disableable.
- **2.5.7 Dragging Movements (AA) — new in 2.2:** All drag functionality must have a single-pointer alternative.
- **2.5.8 Target Size (Minimum) (AA) — new in 2.2:** Touch targets must be at least 24 × 24 CSS px. Recommended minimum is **48 × 48 dp** (meets the spirit of this criterion and Material guidance).

---

## 3. Understandable

### 3.1 Readable
- **3.1.1 Language of Page (A):** The primary language of the UI must be set programmatically.

### 3.2 Predictable
- **3.2.1 On Focus (A):** Focusing a component must not trigger an unexpected context change.
- **3.2.2 On Input (A):** Changing a setting (e.g., toggling a switch) must not automatically navigate away or submit unless the user is warned in advance.

### 3.3 Input Assistance
- **3.3.1 Error Identification (A):** Input errors must be identified in text and described to the user.
- **3.3.2 Labels or Instructions (A):** Labels or instructions must be provided for all user input.
- **3.3.7 Redundant Entry (A) — new in 2.2:** Information previously entered by the user must not be re-requested in the same process unless re-entry is essential.
- **3.3.8 Accessible Authentication (Minimum) (AA) — new in 2.2:** Authentication steps must not require a cognitive function test (e.g., transcribing a CAPTCHA) unless an alternative is provided (e.g., biometric, copy-paste support).

---

## 4. Robust

### 4.1 Compatible
- **4.1.2 Name, Role, Value (A):** All UI components must have a programmatic name, role, and state/property that can be determined by assistive technologies.
- **4.1.3 Status Messages (AA):** Status messages (success, error, loading) must be programmatically determinable (via `liveRegion` or `SemanticsService.announce`) without requiring focus.

---

## 5. Quick Reference -- Contrast Ratios

| Content type | Minimum ratio (AA) |
|---|---|
| Normal text (< 18pt / < 14pt bold) | 4.5 : 1 |
| Large text (≥ 18pt or ≥ 14pt bold) | 3 : 1 |
| UI components & informational graphics | 3 : 1 |
| Inactive / disabled components | No requirement |

---

## 6. WCAG 2.2 New Criteria Summary (AA)

| Criterion | Summary |
|---|---|
| 2.4.11 Focus Not Obscured (Minimum) | Focused element must not be entirely hidden |
| 2.5.7 Dragging Movements | Drag actions need single-pointer alternative |
| 2.5.8 Target Size (Minimum) | Targets ≥ 24 × 24 CSS px |
| 3.3.7 Redundant Entry | Don't re-ask for info already provided in same session |
| 3.3.8 Accessible Authentication (Min) | No cognitive CAPTCHA without alternative |
