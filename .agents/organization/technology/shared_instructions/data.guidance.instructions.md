---
name: data guidance
description: Guidelines for implementing data sources in this Flutter app. Covers the ApiClient hierarchy, AppApiClient authenticated headers, domain Api classes, ApiResult handling, fromJson helpers, AppLocalDataStore typed access, bootstrap registration order, and the rules for what may and may not be done in the data layer. Referenced by DataPlanner, DataBuilder, DataUpdater, and DataCorrector.
---

# Data Guidance

> All data layer additions flow through the `ApiClient` base class. Nothing reaches HTTP, Realm, or SharedPreferences directly from repositories or features. The entire data layer is registered once in `bootstrap.dart`.

---

## 1. Architecture Overview

```
bootstrap.dart
        │  registers all as GetIt singletons (once, at startup)
        │
        ├── AppApiClient             ← main REST client (authenticated)
        │       │
        │       └── XxxApi           ← one class per domain (AuthApi, ProductApi…)
        │
        ├── ThirdPartyApiClient      ← separate client for unauthenticated 3rd-party APIs
        │
        ├── AppLocalDataStore        ← local persistence (Realm + SharedPreferences)
        │
        └── Platform SDKs            ← wrapped in packages/ — never called directly
```

---

## 2. ApiClient — Base HTTP Client

All REST communication extends the abstract `ApiClient` base from the internal API package. The single entry point for all HTTP calls:

```dart
Future<ApiResult<T>> makeRequest<T>({
  required Uri url,
  required HttpMethod method,
  required ResultFromJson<T> fromJson,   // String → T
  Map<String, String> headers = const {},
  Object? body,
  bool authorized = true,               // false = skip auth header
})
```

**Never call `http.Client` directly** from repositories or features.

---

## 3. AppApiClient — Authenticated App Client

`AppApiClient` extends `ApiClient` and injects default headers on every request:

| Header | Source |
|--------|--------|
| `x-app-banner` | Brand/region identifier |
| `x-app-install-id` | Unique device install ID from local store |
| `x-app-tx-id` | Per-session transaction UUID (rotated on logout) |
| `x-app-identifier` | App identifier string |
| `Authorization: Bearer <token>` | Auth token from `TokenManager` when `authorized: true` |

The `tokenProvider` is wired after all singletons are registered:

```dart
// bootstrap.dart — after all singletons registered
GetIt.I<AppApiClient>().tokenProvider = GetIt.I<TokenManager>().getToken;
```

The base URL is resolved at runtime via a `baseUrlProvider` closure:

```dart
baseUrlProvider: () => GetIt.I<EnvironmentRepository>().current.apiBaseUrl,
```

Individual endpoint URLs are built via:

```dart
url: _client.fromBaseUrl(_client.config.someEndpointUrl)
```

All endpoint URL strings come from `AppConfig` (remote config) -- **never hardcoded in Api classes**.

---

## 4. Domain Api Classes

One class per domain area, in `packages/<api_package>/lib/src/apis/`. These classes:
- Accept `AppApiClient` via constructor -- **never GetIt** (the package has no app dependency)
- Call `makeRequest()` with URL, method, body, and `fromJson`
- Return `ApiResult<T>` only -- **zero business logic inside**

```dart
class FeatureApi {
  const FeatureApi(this._client);
  final AppApiClient _client;

  Future<ApiResult<FeatureModel>> getFeature(String id) {
    return _client.makeRequest(
      url: _client.fromBaseUrl(_client.config.backendFeatureUrl),
      method: HttpMethod.get,
      fromJson: FeatureModel.fromJson.fromMapOrThrow,
    );
  }

  Future<ApiResult<void>> updateFeature({required String id}) {
    return _client.makeRequest(
      url: _client.fromBaseUrl(_client.config.backendFeatureUpdateUrl),
      method: HttpMethod.post,
      fromJson: (_) {},
      body: {'id': id},
    );
  }
}
```

### Adding a new endpoint to an existing Api class
1. Add the URL constant to `AppApiConfig` interface and `AppConfig`
2. Add the method to the relevant `XxxApi` class

### Adding a new domain Api class
1. Create `packages/<api_package>/lib/src/apis/<domain>_api.dart`
2. Register it as a singleton in `bootstrap.dart`, passing `GetIt.I<AppApiClient>()`
3. Export it from the package barrel file

---

## 5. fromJson Helpers

The `fromJson` parameter is a `String → T` function. Use the extension helpers:

```dart
// Single JSON object { ... }
fromJson: MyModel.fromJson.fromMapOrThrow

// JSON array [ ... ]
fromJson: MyModel.fromJson.fromListOrThrow

// No body to parse
fromJson: (_) {}
```

All models have `fromJson(Map<String, dynamic>)` constructors generated with `json_serializable`. After adding or modifying a model, `build_runner` must be run.

---

## 6. ApiResult<T> — Handling in Repositories

Every `makeRequest()` call returns `ApiResult<T>`, a sealed class. Repositories pattern-match exhaustively:

```dart
switch (res) {
  case ApiSuccessResult(:final data):
    return (data, null);
  case ApiErrorResult(:final code, :final message):
    _logger.warning('Failed: $code $message');
    return (null, _mapError(res));
}
```

**Never propagate `ApiResult` out of a repository** -- it is an API-package type. Always translate to a domain return type before returning to the cubit.

Convenience accessors available but not preferred over exhaustive switch:
```dart
res.maybeData   // T? — null if error
res.isSuccess   // bool
```

---

## 7. AppLocalDataStore — Local Persistence

The single local persistence class, composing:

| Storage | Used for |
|---------|----------|
| Realm (encrypted) | Structured objects persisted across restarts |
| SharedPreferences | Simple key-value data (region, install ID, build version) |

Access is always via **typed properties** -- never raw string keys from outside the store:

```dart
// ✅ Typed access
store.authToken    // String? — encrypted Realm
store.region       // String? — SharedPreferences
store.installId    // String — auto-generates UUID on first access

// ✅ Realm write pattern
set authToken(String? value) {
  writeToRealm<AuthStorage>((realm) {
    realm.add(AuthStorage(primaryKey, authToken: value));
  });
}
```

- Realm encryption key managed by `DbEncryption` helper -- never hardcoded
- The store is created once in `bootstrap.dart` via async `getInstance()` factory
- Passed into repositories via constructor injection -- **never retrieved via GetIt inside a method**

---

## 8. Bootstrap Registration Order

All singletons are registered in `bootstrap.dart` in this strict order:

1. Third-party SDK init (Firebase, APM, etc.)
2. `AppConfig` — remote config fetch (async)
3. `AppLocalDataStore` — reads disk (async)
4. `AppInfo` — reads from store
5. Core singletons: `AnalyticsRepository`, `RegionRepository`
6. `AppApiClient` — depends on `AppConfig` + `AppInfo` + `EnvironmentRepository`
7. Domain Api classes — depend on `AppApiClient`
8. `TokenManager` — depends on `AppLocalDataStore` + `AuthApi`
9. Repositories — depend on Api classes + `TokenManager` + `AppLocalDataStore`
10. Wire `tokenProvider` onto `AppApiClient`
11. `runApp()`

`GetIt.I<T>()` lookups are safe only after their registration point. **Never register singletons from features or cubits.**

---

## 9. Rules Summary

| Concern | Rule |
|---------|------|
| New endpoint | URL constant in `AppApiConfig` + `AppConfig`; method in existing `XxxApi`; return `ApiResult<T>` only -- zero logic |
| New domain Api class | New file in `packages/<api_package>/src/apis/`; register in `bootstrap.dart`; export from barrel |
| `fromJson` | Use `.fromMapOrThrow` / `.fromListOrThrow`; never parse JSON manually in a repository |
| `ApiResult` | Exhaustive switch in repository only; never returned to a cubit |
| Local storage | Typed properties on `AppLocalDataStore` only; never raw SharedPreferences / Realm from features |
| HTTP client | Never use `http.Client` directly; always via `makeRequest()` |
| Bootstrap | All singletons registered in `bootstrap.dart` only; never from features or cubits |
| Code generation | Run `dart run build_runner build` after adding or modifying `json_serializable` models |

---

## Checklist
- [ ] New endpoint URL added to `AppApiConfig` and `AppConfig` -- not hardcoded in the Api class
- [ ] Api class method returns `ApiResult<T>` only -- no business logic inside
- [ ] `fromJson` uses `.fromMapOrThrow` or `.fromListOrThrow` helpers -- no manual JSON parsing
- [ ] `ApiResult` exhaustively switched in the repository -- not propagated to a cubit
- [ ] Local storage accessed via typed properties on `AppLocalDataStore` -- no raw keys
- [ ] New singleton registered in `bootstrap.dart` at the correct position in the dependency order
- [ ] If new domain Api class: registered in bootstrap and exported from package barrel
- [ ] If `json_serializable` model added or changed: `build_runner` run after implementation
