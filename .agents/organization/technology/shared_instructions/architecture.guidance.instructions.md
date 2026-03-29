---
name: architecture guidance
description: Architecture standards and guidelines for the Bizzie Flutter application. All developers (Human or AI) must adhere to these rules to maintain a scalable, maintainable, and testable codebase.
---

# Architecture Instructions

This document outlines the architectural standards and guidelines for the Bizzie Flutter application. All developers (Human or AI) must adhere to these rules to maintain a scalable, maintainable, and testable codebase.

## Architecture Overview

This app uses **Clean Architecture** combined with **Feature-Driven Architecture**. This ensures separation of concerns, independency of frameworks, and testability.

### Core Principles

1.  **Separation of Concerns:** The code is divided into distinct layers (Data, Domain, Presentation) with specific responsibilities.
2.  **Dependency Rule:** Source code dependencies only point inwards. Inner layers (Domain) know nothing about outer layers (Data, Presentation).
3.  **Feature Autonomy:** Each feature is self-contained in `lib/features/[feature_name]`.

## Folder Structure

The project follows a strict directory structure.

### Full Project Architecture Reference

Use this full project tree as the context for where features fit in and strictly follow this structure:

```plaintext
lib/
├── l10n/                  # Localization
│   ├── app_en.arb         # English Strings
│   └── l10n.dart          # Helper configuration
├── app/
│   ├── app.dart           # Root widget
│   ├── router.dart        # App-level routing
│   └── themes/            # AppTheme, AppColors, AppTextStyles, AppAssets
├── bootstrap/             # Entry point setup
│   └── bootstrap.dart
├── core/                  # Shared business logic
│   ├── error/             # Failures & Exceptions
│   ├── usecase/           # Abstract Base UseCase
│   ├── network/           # Interceptors & Network Info
│   └── enums/             # Global Enums
├── di/                    # Dependency Injection
│   ├── injection.dart     # GetIt/Injectable setup
│   └── injectable.config.dart
├── services/              # Third-party wrapper services
│   ├── notification_service.dart
│   ├── analytics_service.dart
│   ├── permission_service.dart
│   └── auth_service.dart
├── features/              # Modular Features
│   └── [feature_name]/
│       ├── data/
│       │   ├── dtos/
│       │   ├── datasources/
│       │   └── repositories/
│       ├── domain/
│       │   ├── models/
│       │   ├── interfaces/
│       │   └── usecases/
│       ├── presentation/
│       │   ├── bloc/
│       │   ├── views/
│       │   ├── widgets/
│       │   └── routes/
│       └── [feature_name].dart
├── shared/                # Global UI components & Utils
│   ├── widgets/
│   ├── utils/
│   └── constants/
└── main.dart              # App entry point
```

### Feature Structure (`lib/features/`)

Each feature typically contains the following structure:

```plaintext
lib/features/[feature_name]/
├── data/
│   ├── dtos/          # Data Transfer Objects (JSON parsing, from/to Map)
│   ├── datasources/   # Remote (API clients) & Local (Database DAOs) sources
│   └── repositories/  # Implementation of Domain Repositories (implements interfaces)
├── domain/
│   ├── models/        # Pure Dart classes (Business Entities)
│   ├── interfaces/    # Abstract Repositories (Contracts)
│   └── usecases/      # Single-responsibility business logic classes (Interactors)
├── presentation/
│   ├── bloc/          # State Management (Blocs/Cubits)
│   ├── views/         # Screens (Pages)
│   └── widgets/       # Feature-specific, reusable widgets
└── [feature].dart     # Barrel file / Export
```

### Core Structure (`lib/core/`)

Shared logic and utilities that are used across multiple features reside here.

-   `error/`: Failure definitions and exception handling.
-   `usecases/`: Base UseCase interface.
-   `utils/`: General utility functions.
-   `constants/`: App-wide constants.

## Boundaries & Rules

1.  **Rule 1:** The `domain` layer must **NOT** depend on `data` or `presentation` layers. It should contain only pure Dart code and no Flutter dependencies (except for basic types if absolutely necessary and abstract).
2.  **Rule 2:** `presentation` talks to `domain`, which talks to `data` (via interfaces). `presentation` **NEVER** talks to `data` directly. `presentation` must go through `domain` (UseCases).
3.  **Rule 3:** The `data` layer implements the interfaces defined in the `domain` layer. This allows for easy swapping of data sources and mocking for tests.
4.  **Rule 4:** Business logic should reside in UseCases, and UI logic in BLoCs/Cubits. Widgets should be dumb and only display state.
5.  **Rule 5:** **Routes MUST be defined in `lib/app/routes/app_routes.dart` and referenced via static constants.** Hardcoded route strings (e.g., `'/login'`) are forbidden in `GoRoute` definitions or navigation calls (`context.push()`).

## Shared Logic

-   Shared logic should be placed in `lib/core` (e.g., Failure classes, UseCase base classes).
-   **Strictly Enforced:** Use `lib/shared/utils/validators.dart` for all form validation (Email, Password, etc.). Do not duplicate regex logic.

## Code Reviews

All code changes involving structural modifications must be reviewed against these architectural guidelines.
