---
name: tech stack
description: Approved technology stack and package list for all Bizzie technology agents. No package outside this list may be introduced without explicit approval.
---

# Bizzie Technology Stack

This document serves as the **Single Source of Truth** for the tools, libraries, and services used in the Bizzie application. All Agents (Architects, Builders, Testers) must reference this file before adding new dependencies.

## 1. Development Environment
* **IDEs:** Google Antigravity, Claude Code, VS Code
* **Language:** Dart (Latest Stable, SDK ^3.8.0)
* **Framework:** Flutter (Latest Stable)

## 2. Core Architecture & Frontend
* **Architecture Pattern:** Feature-Driven Clean Architecture (Data/Domain/Presentation)
* **State Management:** `flutter_bloc` (v9+) + `freezed` (State Unions & Pattern Matching)
* **Functional Programming:** `dartz` (Either, Option)
* **Data Models:** `freezed` + `json_annotation` + `json_serializable`
* **Navigation/Routing:** `go_router` (v17+)
* **Dependency Injection:** `get_it` + `injectable`
* **Charts & Graphs:** `syncfusion_flutter_charts` (Standard for all data visualization)
* **Generative UI:** `genui` + `genui_firebase_ai`
* **Local Storage:** `shared_preferences`
* **Cloud Functions:** Firebase Cloud Functions `cloud_functions`
* **Networking:** `dio` (v5+), `connectivity_plus`, `internet_connection_checker_plus`

## 3. Backend & Infrastructure (Firebase)
* **Base Core:** `firebase_core`
* **Authentication:** `firebase_auth` (Email), `google_sign_in`, `sign_in_with_apple`
* **Database:** Cloud Firestore (`cloud_firestore`)
* **Cloud Functions/Tasks:** Google Cloud Platform (Scheduled Tasks)
* **Storage:** Firebase Storage (`firebase_storage`)

## 4. Security & Secrets
* **RASP (Runtime App Self Protection):** `freerasp` (Detects rooting, tampering, hooks)
* **Secrets Management:** `envied` & `envied_generator` (Obfuscated API keys in binary)

## 5. Operations & Telemetry
* **Analytics:** `firebase_analytics` & Google Analytics
* **Crash Reporting:** `firebase_crashlytics`
* **Logging:** `logging`
* **A/B Testing:** `firebase_remote_config`
* **Notifications:** `firebase_messaging` (FCM), `flutter_local_notifications`

## 6. External Integrations & APIs
* **Financial Data:** Financial Modeling Prep API
* **Deep Linking:** Branch.io (`url_launcher`)
* **AI/LLM:** Google Gemini API (`firebase_ai`), `genui`, `genui_firebase_ai`
* **In-App Purchases / Subscriptions:** RevenueCat (`purchases_flutter`)
* **Review Prompt:** `in_app_review`

## 7. CI/CD Pipeline
* **Continuous Integration (CI):** GitHub Actions
* **Continuous Delivery (CD):** Firebase App Distribution

## 8. Styling & Design System
* **Theme Source:** `lib/app/themes/app_theme.dart`
* **Color Palette:** `lib/app/themes/app_colors.dart` (Strict Source of Truth)
* **Typography:** `lib/app/themes/app_text_styles.dart` (Strict Source of Truth)
    * **Rule:** Do NOT use `GoogleFonts` or `TextStyle` inline. Always define a constant in `AppTextStyles` and reference it.
* **UI Components:** `flutter_svg`, `cached_network_image`, `smooth_page_indicator`, `custom_refresh_indicator`, `flutter_native_splash`

## 9. Utilities & Helpers
* **Device Info:** `device_info_plus`, `package_info_plus`
* **Permissions:** `permission_handler`
* **Date & Time:** `intl`, `timeago`, `timezone`
* **Collections & Async:** `collection`, `async`, `rxdart`, `bloc_concurrency`
* **Others:** `path_provider`, `nanoid`, `equatable` (Legacy/Specific cases only)

## 10. Development & Testing
* **Linting:** `flutter_lints`
* **Code Generation:** `build_runner`, `freezed`, `json_serializable`, `injectable_generator`, `envied_generator`
* **Unit/Widget Testing:** `flutter_test`, `mocktail`, `bloc_test`
* **Integration/Fake:** `fake_cloud_firestore`, `fake_async`
* **Asset/Icons:** `flutter_launcher_icons`

## Agent Guidelines
* **StateArchitect:** Use `flutter_bloc` with `freezed` unions for all logic. Prefer `freezed` over `Equatable` for state/events.
* **MockBuilder:**
    * Use `go_router` for all navigation.
    * Use `syncfusion_flutter_charts` for any chart/graph requirements. Do not use `fl_chart` or `charts_flutter`.
    * Use `cached_network_image` for all remote images.
* **BackendConnector:** All data persists to Firestore unless it is financial data (fetch from API).
* **SecOps:** Ensure all keys are stored in `Envied` configs, never hardcoded.
