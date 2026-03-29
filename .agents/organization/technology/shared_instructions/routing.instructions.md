---
name: routing instructions
description: Flutter routing rules for all technology agents. Governs route registration, navigation patterns, data passing, and page types using go_router.
---

# Routing Instructions

---

## 1. Setup

- Routing is handled entirely by `go_router` via a dedicated `buildRouter()` function passed into `MaterialApp.router`.
- Route path strings live exclusively in `routes.dart` — never inline in the router definition, widgets, or cubits.
- A single `_rootNavKey` (`GlobalKey<NavigatorState>`) is defined at the top of the router file and used for overlays that must appear above the tab bar.

---

## 2. Route Tree Structure

```
GoRouter
├── StatefulShellRoute.indexedStack  (tab bar shell)
│   ├── Branch: /tab-one
│   │   └── nested routes...
│   ├── Branch: /tab-two
│   │   └── nested routes...
│   └── Branch: /tab-n
│       └── nested routes...
│
└── Top-level routes (outside shell)
    ├── /landing
    ├── /signin
    ├── /register
    ├── /maintenance         ← fullscreenDialog
    ├── /some-sheet          ← ModalBottomSheetPage, parentNavigatorKey: _rootNavKey
    └── ...
```

Routes outside the tab shell (auth flows, full-screen modals, app-level overlays) are declared at the top level, after the shell.

---

## 3. Route Path Constants

Never hardcode path strings. Always use the `Routes` constants class.

```dart
// Correct
context.go(Routes.home);
context.push(Routes.signIn(redirect: somePath));

// Wrong
context.go('/home');
context.push('/auth/signin?redirect=/home');
```

Parameterised routes are static methods that build the URI:

```dart
static const home = '/home';
static String featureDetail({required String id}) =>
    _routeWithQuery('/feature/detail', {'id': id});
```

---

## 4. Page Types

| Scenario | Use |
|---|---|
| Standard full-screen page | `builder:` |
| Must remount on each visit (e.g. WebView) | `pageBuilder: MaterialPage(key: ValueKey(state.uri))` |
| Animated transition | `pageBuilder: FadeTransitionPage(...)` |
| Modal bottom sheet | `pageBuilder: ModalBottomSheetPage(...)` |
| Full-screen modal (iOS style) | `MaterialPage(fullscreenDialog: true)` |
| System dialog | `pageBuilder: DialogPage(...)` |

**Modal bottom sheets above the tab bar** require `parentNavigatorKey: _rootNavKey`. Sheets scoped inside a tab omit it.

```dart
// Above tab bar
GoRoute(
  parentNavigatorKey: _rootNavKey,
  path: '/mysheet',
  pageBuilder: (context, state) => ModalBottomSheetPage(builder: (_) => const MySheet()),
),

// Within a tab
GoRoute(
  path: 'mysheet',
  pageBuilder: (context, state) => ModalBottomSheetPage(builder: (_) => const MySheet()),
),
```

---

## 5. Router-Level Redirects

Top-level redirect priority order:
1. App-unavailable states (maintenance, force upgrade) → dedicated page
2. Required setup incomplete (region/onboarding) → setup page
3. Deep-link resolution → resolved route or `null`

Per-route auth-gating uses a shared `requireAuth` helper:

```dart
GoRoute(
  path: 'protected',
  redirect: requireAuth,
  builder: ...,
),
```

---

## 6. Navigation in Widgets

```dart
context.go(Routes.home);               // replace entire stack
context.push(Routes.detail);           // push onto current stack
context.pushReplacement(Routes.next);  // replace current page
context.pop();                         // go back
context.canPop();                      // check before popping
```

**Cubits never navigate.** State-triggered navigation is handled in the widget layer via `BlocListener`:

```dart
// Correct
BlocListener<MyFeatureCubit, MyFeatureState>(
  listener: (context, state) {
    if (state is MyFeatureSuccess) context.go(Routes.home);
  },
  child: ...,
)
// Wrong — never call context.go / context.push inside a cubit
```

App-level routing reactions (deeplinks, forced logout, global redirects) are handled in the app delegate class via `router.go()` / `router.push()`.

---

## 7. Passing Data Between Routes

| Method | Survives deep-links? | Use when |
|---|---|---|
| Query parameters | Yes | Default — any routable data |
| Path parameters | Yes | Resource identifiers (IDs, slugs) |
| `extra` | No | In-memory only, non-serialisable objects |

Prefer query or path parameters over `extra` for any route reachable via deep-link.

---

## 8. `fromRouter` Factory Pattern

Pages that receive data from the router implement a `fromRouter` factory. This keeps route-wiring co-located with the page and out of `router.dart`.

```dart
class MyPage extends StatelessWidget {
  const MyPage({required this.id, super.key});

  factory MyPage.fromRouter(BuildContext _, GoRouterState state) =>
      MyPage(id: state.pathParameters['id'] ?? '');

  static Page<void> pageFromRouter(BuildContext context, GoRouterState state) =>
      MaterialPage<void>(
        key: ValueKey(state.uri.toString()),
        child: MyPage.fromRouter(context, state),
      );
}

// In router.dart:
GoRoute(
  path: '/mypage/:id',
  builder: MyPage.fromRouter,         // or:
  pageBuilder: MyPage.pageFromRouter,
),
```

---

## 9. Checklist — Adding a New Route

- [ ] Path constant(s) added to `routes.dart`
- [ ] `GoRoute` registered in `router.dart` at correct nesting level (shell branch or top-level)
- [ ] `parentNavigatorKey: _rootNavKey` added if sheet/overlay must appear above the tab bar
- [ ] `pageBuilder: ModalBottomSheetPage(...)` used for bottom sheets
- [ ] `fromRouter` / `pageFromRouter` implemented on the page widget
- [ ] Query/path parameters used (not `extra`) for data that must survive deep-links
- [ ] `redirect: requireAuth` added if the route requires an authenticated user
- [ ] Navigation via `context.go()` / `context.push()` in widget layer only — never inside a cubit
