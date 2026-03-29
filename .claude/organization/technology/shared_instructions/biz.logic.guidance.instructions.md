---
name: biz logic guidance
description: Guidelines for writing and placing business logic in this Flutter app. Covers the three-layer split (repositories, cubits, helpers), what each layer owns, return type patterns, in-memory state, specialist repositories, and the decision guide for where new logic should go. Referenced by BizLogicPlanner, BizLogicBuilder, BizLogicUpdater, and BizLogicCorrector.
---

# Business Logic Guidance

> There is no dedicated domain layer. Business logic is split across three places by concern: `lib/repositories/`, `lib/features/*/cubit/`, and `lib/helpers/`. Writing new logic means understanding which layer owns which kind of responsibility.

---

## 1. Layer Responsibilities

### `lib/repositories/` — Data + Lightweight Business Logic
The closest thing to a domain layer. Owns:
- Translating `ApiResult<T>` into domain-meaningful return types
- Error mapping (deciding what an API error means to the app)
- In-memory state that doesn't belong in the UI (auth token, cached lists, user profile fields)
- Composing multiple data sources (API + local DB + platform SDK) into a single call
- Constructing request parameters from business rules (consent dates, registration sources, region-specific config)
- Retry logic and token refresh
- Specialist subsystems used by other repositories (e.g. `TokenManager`, `AppConfig`)

Does **not** own: UI state flags, navigation decisions, analytics calls, cubit-level orchestration.

### `lib/features/*/cubit/` — Orchestration Logic
Sits above the repository. Owns:
- Calling multiple repositories in sequence and composing their results into a single state emission
- Deciding which state to emit based on repository outcomes
- Handling async dependencies (e.g. "don't load products until region is known")
- Cross-repository coordination (e.g. after login: update user state, load profile, trigger analytics)
- UI-facing derived values (e.g. `canLoadMore`, `startupComplete`)

Does **not** own: raw HTTP calls, token refresh logic, JSON parsing, persistent storage reads/writes.

### `lib/helpers/` — Pure Utilities
Stateless pure functions and extensions with no dependencies on repositories, cubits, or Flutter. Safe to call from anywhere.

Feature-scoped utilities that don't fit a repository go in `lib/features/<feature>/helpers/`.

Does **not** own: anything with external dependencies or mutable state.

---

## 2. Return Type Patterns

Repository methods follow consistent return type conventions -- use the pattern that matches the call's semantics:

```dart
// 1. Success/failure tuple — (bool, String?)
// Use when the caller only needs to know if it worked and optionally why not
Future<(bool, String?)> register() async { ... }

// 2. Nullable domain type — T?
// Use when absence is a valid non-error state
Future<TreasureEligibility?> getEligibility() async { ... }

// 3. Named tuple — (T? data, ErrorEnum? error)
// Use when the caller needs to distinguish between error types
typedef GetProductsResponse = (ProductListResponse?, ProductRepositoryError?);
Future<GetProductsResponse> getProductsForCategory(...) async { ... }

// 4. Error enum mapping
ProductRepositoryError _mapError(ApiErrorResult result) => switch (result.code) {
  404 => ProductRepositoryError.notFound,
  _ => ProductRepositoryError.unknown,
};
```

---

## 3. In-Memory State Pattern

Use private fields in a repository for data that needs to survive navigation within a session but does not belong in global cubit state. Initialise eagerly on app start, not lazily on first use:

```dart
class UserRepository {
  UserStatus? _status;        // computed once, cached in memory
  String? firstName;          // profile fields loaded after login
  bool isLoyaltyMember = false;

  // Eager initialisation on app start
  Future<void> initStatus() async { ... }
}
```

---

## 4. Orchestration in Cubits

Cubits sequence repository calls and decide state. They do not duplicate logic already in a repository:

```dart
Future<void> onAppStart() async {
  await _userRepository.initStatus();
  if (_userRepository.status == UserStatus.loggedIn) {
    final res = await _userRepository.relogin();
    emit(LoggedInUserState(
      mustReauthenticate: !res,
      firstName: _userRepository.firstName ?? '',
    ));
  } else {
    emit(const GuestUserState());
  }
}
```

---

## 5. Specialist Repositories

Some repositories own a specific subsystem with no direct API involvement:

| Repository | Owns |
|------------|------|
| `TokenManager` | Token refresh with exponential backoff, de-duplication via `Completer` |
| `AppConfig` | Remote config values — feature flags, backend URLs, whitelists |
| `DeeplinkRepository` | Web URL → app route translation, external URL detection |
| `RegionRepository` | Current region, persistence |
| `PermissionsRepository` | Platform permission request/check logic |

Specialist repositories are injected into other repositories via constructor -- they are **not** accessed directly from cubits via `GetIt`:

```dart
class UserRepository {
  UserRepository({TokenManager? tokenManager, ...})
      : _tokenManager = tokenManager ?? GetIt.I();
}
```

---

## 6. Layer Violation Rules

| Location | Must NOT contain |
|----------|-----------------|
| Feature widgets / pages | Any logic beyond calling `context.read<Cubit>().method()` |
| Cubit | Token refresh, JSON parsing, retry logic, storage reads/writes |
| Repository | UI state flags, navigation calls, analytics calls |
| Helper | External dependencies, mutable state, side effects |
| API package | Anything beyond HTTP transport and JSON ↔ Dart type conversion |

---

## 7. Decision Guide — Where Does New Logic Go?

```
Is it a pure transformation, calculation, or string operation
with no dependencies?
        └──► lib/helpers/ or lib/features/<feature>/helpers/

Does it involve calling an API, local DB, or platform SDK?
        └──► lib/repositories/<domain>/

Does it need to sequence multiple repository calls or decide
which UI state to show?
        └──► lib/features/<feature>/cubit/

Does it apply app-wide (auth, config, region, deeplinks)?
        └──► lib/cubit/ (global cubit) or a specialist repository

Is it a self-contained subsystem used by other repositories
(token management, config values)?
        └──► New specialist class in lib/repositories/<domain>/
             injected via constructor — not accessed directly from cubits
```

---

## Checklist
- [ ] New logic placed in the correct layer per the decision guide
- [ ] Repository methods use the appropriate return type pattern (tuple, nullable, named tuple)
- [ ] In-memory state initialised eagerly (not lazily) if it must be ready on app start
- [ ] Specialist repositories injected via constructor, not accessed from cubits via GetIt
- [ ] No layer violations: no HTTP in cubits, no UI state in repositories, no side effects in helpers
- [ ] Cubit orchestration sequences existing repository methods -- not re-implementing repository logic
