---
name: webview guidance
description: Guidelines for implementing, updating, and correcting Flutter WebView pages in this project. Covers WrappedWebViewPage usage, router-based construction, URL interception, JS messaging, SSO, and the rules that govern what feature code may and may not do. Referenced by WebViewPlanner, WebViewBuilder, WebViewUpdater, and WebViewCorrector.
---

# WebView Guidance

> Feature code must never instantiate `TKMaxxWebView` directly. All webview feature pages use `WrappedWebViewPage`. The architecture is a strict three-layer stack that feature agents interact with only at the top layer.

---

## 1. Architecture Layers

```
WrappedWebViewPage          ← feature code touches ONLY this layer
        │
        └──► TKMaxxWebView  ← core rendering — handles all lifecycle & behaviour
                  │
                  ├── NavigationDelegate       ← intercepts every navigation request
                  ├── WebViewInterceptHandler  ← decides what to do with intercepted URLs
                  ├── JsMessageHandler         ← receives JS→native messages
                  └── WebViewCleaner           ← clears cookies/storage on logout
```

---

## 2. WrappedWebViewPage — Standard Usage

Use `WrappedWebViewPage` for every feature-level page that renders web content.

### Minimal usage
```dart
WrappedWebViewPage(
  url: 'https://example.com/my-page',
  title: l10n.myPageTitle,
  accessibilityTitle: l10n.eaaMyPageTitle,
)
```

### Full options
```dart
WrappedWebViewPage(
  url: 'https://example.com/my-page',
  title: l10n.myPageTitle,
  accessibilityTitle: l10n.eaaMyPageTitle,
  analyticsPageName: kMyPageName,
  analyticsPageType: kMyPageType,
  analyticsContentGroup: kMyContentGroup,
  forceLoad: false,              // true = always reload, even if cached
  handleScreenView: true,        // false = opt out of auto screen_view
  onPageFinished: (controller) { ... },
  interceptHandler: MyCustomInterceptHandler(),  // optional — see section 4
)
```

### Key props
| Prop | Notes |
|------|-------|
| `url` | Must come from Routes constants + AppConfig — never hardcoded |
| `title` | App bar title |
| `accessibilityTitle` | Screen reader title |
| `analyticsPageName` / `analyticsPageType` / `analyticsContentGroup` | Analytics screen view dimensions |
| `forceLoad` | `true` to bypass cache |
| `handleScreenView` | `false` to opt out of auto screen_view |
| `interceptHandler` | Custom `WebViewInterceptHandler` subclass (see section 4) |
| `onPageFinished` | Callback after page load completes |

---

## 3. Router-Based Construction

For webview pages reached via a route, always use `WrappedWebViewPage.fromRouter`:

```dart
GoRoute(
  path: '/mypage',
  pageBuilder: (context, state) => MaterialPage(
    key: ValueKey(state.uri.toString()),   // ← REQUIRED so page remounts per URL
    child: WrappedWebViewPage.fromRouter(context, state),
  ),
),
```

`fromRouter` resolves `url`, `title`, `forceLoad`, analytics params, and `profilerKey` from query params automatically.

**Key query params read by `fromRouter`:**
| Param | Purpose |
|-------|--------|
| `url` | The web URL to load (required) |
| `forceLoad` | `'true'` to bypass cache |
| `pageName` / `pageType` / `contentGroup` | Analytics dimensions |
| `title` / `titleEn` / `titleDe` / `titleDeDE` / `titleDeAT` | Localised app bar title |
| `profilerKey` | Performance profiling label |

---

## 4. URL Interception — WebViewInterceptHandler

Every navigation request is intercepted before loading. Default interception priority order:

1. `.pkpass` file → AddToWallet native flow
2. External URL → launch in device browser
3. `tel:` URI → native phone call
4. `mailto:` URI → native email client
5. App deeplink → `go_router` navigation
6. `null` → allow webview to navigate normally

**To customise interception for a specific feature**, subclass `WebViewInterceptHandler` and pass it to `WrappedWebViewPage`:

```dart
class MyFeatureInterceptHandler extends WebViewInterceptHandler {
  @override
  bool shouldHandleDeeplink(BuildContext context, String deeplink) {
    return !deeplink.contains('/my-special-path');
  }

  @override
  void handleRouting(BuildContext context, String deeplink) {
    context.push(deeplink);
  }
}
```

**Never** add `if/else` URL interception logic inside feature widgets. Always subclass.

The `allowAll: true` flag disables the "open externally" check — use only for webviews where all navigation must stay in-app.

---

## 5. JS→Native Messaging — JsMessageHandler

The web page communicates with the app via `window.TKMaxxWebView.postMessage(JSON.stringify({...}))`. These are dispatched by `JsMessageHandler.handleMessage()`.

**Built-in message types (handled automatically — no feature code needed):**
| `eventType` | What it does |
|-------------|-------------|
| `AddToLoveList` with `details: productCode` | Adds product to lovelist |
| `RemoveFromLoveList` with `details: productCode` | Removes from lovelist |
| `addToBag` with `quantity`, `isSuccess` | Shows add-to-bag confirmation or error |
| `TWLID` with `id: token` | Triggers bidirectional SSO login |
| `GA4` with `details: {event, ...params}` | Forwards analytics event to AnalyticsRepository |

**To add a new message type**, subclass `JsMessageHandler` and pass it to `TKMaxxWebView`:

```dart
class MyWebViewMessageHandler extends JsMessageHandler {
  const MyWebViewMessageHandler();

  @override
  void handleMessage(BuildContext context, JavaScriptMessage message, String id, {TJXLogger? logger}) {
    final data = jsonDecode(message.message);
    if (data case {'eventType': 'MyCustomEvent', 'details': final String val}) {
      // handle it
      return;
    }
    super.handleMessage(context, message, id, logger: logger);
  }
}
```

---

## 6. SSO and Auth

SSO token injection is handled automatically by `TKMaxxWebView` via `SsoCubit`. On `onPageFinished`, the Gigya auth token is injected as a `GJWT_UID` request header.

**Never inject auth tokens manually from feature code.**

---

## 7. Logout Cleanup — WebViewCleaner

`WebViewCleaner().cleanUpForLogout()` is already called in the logout flow (UserRepository). It clears all webview cookies and local storage.

**Never call `WebViewCleaner` from feature code.**

---

## 8. Rules Summary

| Concern | Rule |
|---------|------|
| Feature webview pages | ALWAYS use `WrappedWebViewPage` — NEVER instantiate `TKMaxxWebView` directly |
| Route-based pages | ALWAYS use `MaterialPage(key: ValueKey(uri))` so the page remounts when the URL changes |
| URLs | ALWAYS via Routes constants + AppConfig — NEVER hardcode URLs in widgets |
| URL interception | Subclass `WebViewInterceptHandler` to customise — NEVER add if/else logic in feature widgets |
| JS messages | Subclass `JsMessageHandler` to add new types — NEVER handle JS messages outside the handler |
| SSO / auth headers | Handled automatically — NEVER inject tokens manually |
| Logout cleanup | `WebViewCleaner` called in logout flow only — NEVER from feature code |

---

## Checklist
- [ ] `WrappedWebViewPage` used — `TKMaxxWebView` not instantiated directly from feature
- [ ] `url` prop sourced from Routes constants + AppConfig — not hardcoded
- [ ] If routed: `MaterialPage(key: ValueKey(state.uri.toString()))` and `fromRouter` used
- [ ] If custom URL interception needed: `WebViewInterceptHandler` subclassed — no if/else in feature widget
- [ ] If custom JS messages needed: `JsMessageHandler` subclassed — not handled inline
- [ ] No auth token injection from feature code
- [ ] No `WebViewCleaner` call from feature code
