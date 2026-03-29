---
name: analytics guidance
description: Project-specific analytics coding guidelines for Flutter. Covers AnalyticsRepository usage, BaseEvent, ScreenViewMixin, feature event helpers, cubit injection, and user properties. Referenced by analytics agents performing Flutter tasks.
---

# Analytics Guidance

> All analytics flows through `AnalyticsRepository`. Never call Firebase, Airship, or AppDynamics SDKs directly from features or cubits.

---

## 1. Architecture

Analytics is fanned out to multiple targets through a single entry point registered with GetIt.

```
feature widget / cubit
        │
        ▼
AnalyticsRepository          ← single entry point for all analytics
        │
        ├──► Firebase Analytics
        ├──► Airship
        └──► AppDynamics (breadcrumbs)
```

---

## 2. BaseEvent — The Event Model

All events are built using `BaseEvent` — a mutable data class with named fields for every standard analytics dimension.

| Field | Purpose |
|---|---|
| `pageName` | Screen/page identifier (e.g. `'store-finder'`) |
| `pageType` | Page type (e.g. `'Store'`) |
| `contentGroup` | Logical content group (e.g. `'StoreFinder'`) |
| `eventDetails` | Specific action detail (e.g. `'store locator : search'`) |
| `eventName` | Overrides the default event name when set |
| `searchTerm`, `itemId`, `price`, etc. | Domain-specific dimensions |

Build events by **populating fields** — never subclass `BaseEvent`:

```dart
final eventData = BaseEvent(
  eventDetails: 'store locator : search',
  pageName: kStoresPageName,
  pageType: kStoresPageType,
  contentGroup: kStoresContentGroup,
);
```

`BaseEvent` supports `copy()` to merge additional data onto a base event.

---

## 3. Screen View Tracking — ScreenViewMixin

Every routable page that needs a `screen_view` event must be a `StatefulWidget` and mix in `ScreenViewMixin`. The mixin fires automatically on `initState` and re-fires on back navigation.

**Contract:** implement `updateScreenViewEvent(BaseEvent event)` — never call `logScreenView()` manually.

```dart
class _MyPageState extends State<MyPage> with ScreenViewMixin {
  @override
  void updateScreenViewEvent(BaseEvent event) {
    event
      ..pageName = kMyFeaturePageName
      ..pageType = kMyFeaturePageType
      ..contentGroup = kMyFeatureContentGroup;
  }
}
```

To fire an additional event beyond `screen_view` on arrival (e.g. a filter applied), use `logAdditionalEvent`. The mixin merges the page's base data with `additionalData` automatically:

```dart
logAdditionalEvent(
  AnalyticsEvents.filterPlp,
  additionalData: BaseEvent(filterType: 'colour', filterValue: 'red'),
);
```

---

## 4. Page-Level Analytics Constants

Each feature defines its own page-level analytics constants as top-level `const` strings in `lib/features/<feature>/helpers/<feature>_event.dart`. Use these constants consistently across the feature's pages, widgets, and cubit.

```dart
// lib/features/my_feature/helpers/my_feature_event.dart

const kMyFeaturePageName = 'my-feature';
const kMyFeaturePageType = 'MyFeature';
const kMyFeatureContentGroup = 'MyFeatureGroup';

// Action-level detail strings
const kMyFeatureButtonTap = 'my-feature : button tap';
const kMyFeatureSearch = 'my-feature : search';
```

Never hardcode these strings at the call site — always reference the constants.

---

## 5. UI Interaction Events — Feature Event Helper

For UI interactions (button taps, toggles, etc.), create a feature-scoped static helper class in the same `helpers/` file. This avoids repeating `pageName`/`pageType`/`contentGroup` on every call:

```dart
class MyFeatureEvent {
  static void logUIEvent(String eventDetails, {BaseEvent? additionalData}) {
    final eventData = BaseEvent(
      eventDetails: eventDetails,
      pageName: kMyFeaturePageName,
      pageType: kMyFeaturePageType,
      contentGroup: kMyFeatureContentGroup,
    )..copy(additionalData);

    GetIt.I<AnalyticsRepository>().logEvent(
      AnalyticsEvents.uiInteraction,
      parameters: eventData,
    );
  }
}
```

Usage at the call site — in a widget's `onTap` or in a cubit method:

```dart
MyFeatureEvent.logUIEvent(kMyFeatureButtonTap);
MyFeatureEvent.logUIEvent(kMyFeatureSearch, additionalData: BaseEvent(searchTerm: query));
```

---

## 6. Logging Events from Cubits

Cubits receive `AnalyticsRepository` as an injected constructor dependency using the same GetIt fallback pattern used across the project:

```dart
class MyFeatureCubit extends Cubit<MyFeatureState> {
  MyFeatureCubit({AnalyticsRepository? analyticsRepository})
      : _analyticsRepository = analyticsRepository ?? GetIt.I(),
        super(const MyFeatureInitial());

  final AnalyticsRepository _analyticsRepository;

  Future<void> submit() async {
    final result = await _repository.submit();
    _analyticsRepository.logEvent(
      AnalyticsEvents.account,
      parameters: BaseEvent(
        eventDetails: result ? 'submit : success' : 'submit : failed',
        pageName: kMyFeaturePageName,
        pageType: kMyFeaturePageType,
        contentGroup: kMyFeatureContentGroup,
      ),
    );
  }
}
```

---

## 7. User Properties

User property updates are centralised in the **global `UserCubit` only** via its `onChange` override — they fire automatically on every user state change. Never set user properties from feature cubits or widgets.

```dart
// Only in the global UserCubit
@override
void onChange(Change<UserState> change) {
  super.onChange(change);
  updateAnalyticsUserProperties(change.nextState);
}

void updateAnalyticsUserProperties(UserState state) {
  _analyticsRepository
    ..setUserProperty(AnalyticsUserProperties.userId, _userRepository.uid)
    ..setUserProperty(AnalyticsUserProperties.loginStatus,
        state is LoggedInUserState ? 'login' : 'logout');
}
```

User property keys must come from `AnalyticsUserProperties` constants — never hardcode strings.

---

## 8. Event Names and Targets

All event name strings come from `AnalyticsEvents` constants — never hardcode them at call sites:

```dart
// CORRECT
AnalyticsEvents.uiInteraction
AnalyticsEvents.search
AnalyticsEvents.screenView

// WRONG
'ui_interaction'
'screen_view'
```

By default, `logEvent` fans out to all three targets. Restrict targets only when there is a deliberate reason:

```dart
_analyticsRepository.logEvent(
  AnalyticsEvents.search,
  parameters: eventData,
  targets: [AnalyticsTarget.firebase],
);
```

---

## 9. Rules

| Rule | Detail |
|---|---|
| Platform SDKs | NEVER called directly from features or cubits — always via `AnalyticsRepository` |
| Event names | MUST come from `AnalyticsEvents` constants |
| User property keys | MUST come from `AnalyticsUserProperties` constants |
| Page constants | MUST be top-level `const` strings in `features/*/helpers/<feature>_event.dart` |
| User properties | MUST only be set from the global `UserCubit` |
| Screen view | MUST use `ScreenViewMixin` — never log manually |
| UI interactions | MUST use a feature-scoped event helper class |
| Cubit analytics | Injected via constructor with GetIt fallback (`?? GetIt.I()`) |

---

## Checklist
- [ ] No platform SDK (Firebase, Airship, AppDynamics) called directly — only `AnalyticsRepository`
- [ ] All event names reference `AnalyticsEvents` constants
- [ ] All user property keys reference `AnalyticsUserProperties` constants
- [ ] Page analytics constants defined as top-level `const` strings in `helpers/<feature>_event.dart`
- [ ] No analytic strings hardcoded at call sites
- [ ] Routable pages requiring screen view use `ScreenViewMixin` with `updateScreenViewEvent` implemented
- [ ] `logScreenView()` never called manually
- [ ] UI interaction events routed through a feature-scoped event helper class
- [ ] Cubit receives `AnalyticsRepository` via constructor with `?? GetIt.I()` fallback
- [ ] User properties set only in the global `UserCubit.onChange`
