---
name: flutter bloc best practice
description: Project-specific BLoC/Cubit coding guidelines. Covers the two state patterns, constructor pattern, SafeCubitEmissions, providing cubits, consuming state, and shared helper widgets. Referenced by all state management agents.
---

# Flutter BLoC Best Practice

> The app uses **Cubit only** (no raw Bloc/events).

---

## 1. Two State Patterns

Choose based on the nature of the state.

### Pattern A -- Sealed Class Hierarchy
Use when state has fundamentally different shapes requiring different data. Exhaustive `switch` in the UI enforces completeness at compile time.

```dart
// <feature>_state.dart
part of '<feature>_cubit.dart';

sealed class FeatureState extends Equatable {
  const FeatureState();
  @override
  List<Object?> get props => [];
}

class FeatureInitial extends FeatureState { const FeatureInitial(); }
class FeatureLoading extends FeatureState { const FeatureLoading(); }

class FeatureLoaded extends FeatureState {
  const FeatureLoaded({required this.items});
  final List<Item> items;
  @override
  List<Object?> get props => [items];
}

class FeatureError extends FeatureState {
  const FeatureError({required this.message});
  final String message;
  @override
  List<Object?> get props => [message];
}
```

### Pattern B -- Single Immutable Class with copyWith
Use when state is a collection of properties that evolve incrementally.

```dart
// <feature>_state.dart
part of '<feature>_cubit.dart';

@immutable
class FeatureState extends Equatable {
  const FeatureState({this.busy = false, this.items = const [], this.query = ''});

  final bool busy;
  final List<Item> items;
  final String query;

  FeatureState copyWith({bool? busy, List<Item>? items, String? query}) =>
      FeatureState(
        busy: busy ?? this.busy,
        items: items ?? this.items,
        query: query ?? this.query,
      );

  @override
  List<Object?> get props => [busy, items, query];
}
```

---

## 2. Cubit Constructor Pattern

All dependencies are **optional named parameters with a `GetIt.I()` fallback** -- this lets widget tests inject mocks without configuring GetIt.

```dart
class FeatureCubit extends Cubit<FeatureState> with SafeCubitEmissions<FeatureState> {
  FeatureCubit({
    FeatureRepository? featureRepository,
    AnalyticsRepository? analyticsRepository,
  })  : _featureRepository = featureRepository ?? GetIt.I(),
        _analyticsRepository = analyticsRepository ?? GetIt.I(),
        super(const FeatureInitial());

  final FeatureRepository _featureRepository;
  final AnalyticsRepository _analyticsRepository;
}
```

- All deps stored as `final` private fields.
- **Never** call `GetIt.I()` inside methods -- resolve at construction only.
- State file must be `part of` its cubit file.

---

## 3. SafeCubitEmissions

Mix in `SafeCubitEmissions` on **any cubit that performs async operations** to guard against emitting after `close()`.

```dart
class FeatureCubit extends Cubit<FeatureState> with SafeCubitEmissions<FeatureState> {
  Future<void> load() async {
    emit(const FeatureLoading());
    final result = await _repository.fetch();
    emit(FeatureLoaded(items: result)); // safe -- won't throw if widget is gone
  }
}
```

---

## 4. Side Effects via onChange

Use `onChange` for reactions that must fire on every state change (e.g. analytics, messaging identity sync). Do **not** use it for navigation or UI -- those belong in the widget layer via `BlocListener`.

```dart
@override
void onChange(Change<FeatureState> change) {
  super.onChange(change);
  _analyticsRepository.updateUserProperties(change.nextState);
}
```

---

## 5. Providing Cubits

- **Global cubits** (user session, app config, feature flags) -- provided once at app root via `MultiBlocProvider` in `<app>_app.dart`.
- **Feature cubits** -- provided at the page level. The page creates its own cubit.

```dart
// FeaturePage.build()
return BlocProvider(
  create: (_) => FeatureCubit(),
  child: const _FeatureView(),
);
```

Never pass a cubit instance down through widget constructors -- use `context.read<T>()` anywhere in the subtree.

---

## 6. Consuming State in Widgets

Use the **narrowest API** that fits to minimise unnecessary rebuilds.

| API | Use when |
|---|---|
| `context.read<T>()` | Calling a method or reading state once in a callback -- no rebuild |
| `context.watch<T>()` | Full rebuild on every state change -- use sparingly |
| `context.select<T, R>()` | Rebuild only when a specific field changes -- **preferred** |
| `BlocBuilder<T, S>` | Rebuild with `buildWhen` to filter irrelevant changes |
| `BlocListener<T, S>` | Side effects only (navigation, sheets) -- no rebuild |
| `BlocConsumer<T, S>` | Both rebuild and side effect in the same widget |
| `MultiBlocListener` | Multiple listeners with a single shared child |

```dart
// Most efficient -- rebuilds only when the selected field changes
final region = context.select((AppCubit c) => c.state.region);

// BlocBuilder with buildWhen
BlocBuilder<FeatureCubit, FeatureState>(
  buildWhen: (prev, cur) => prev.items != cur.items,
  builder: (context, state) => ...,
)

// Navigation side effect
BlocListener<FeatureCubit, FeatureState>(
  listenWhen: (prev, cur) => cur is FeatureLoaded && prev is! FeatureLoaded,
  listener: (context, state) => context.go(Routes.someRoute),
  child: ...,
)

// Calling a method -- no rebuild needed
onPressed: () => context.read<FeatureCubit>().submit(),
```

---

## 7. Shared Bloc Helper Widgets

Use the app's `bloc_helpers` widgets rather than reimplementing common patterns.

| Widget | Purpose |
|---|---|
| `BlocErrorListener<TBloc, TState, TError>` | Listens for an error flag and shows an error sheet. State must implement `BlocErrorState<TError>` |
| `BlocFormListener<TBloc, TState, TError>` | Success callback + error sheet display for form submission flows |
| `BlocFormField<TBloc, TState>` | Text field wired to a cubit -- handles value display and change callbacks |

---

## 8. Rules

```
Cubit          -- NO BuildContext, NO navigation, NO UI imports
State file     -- MUST be `part of` its cubit file
State          -- MUST extend Equatable with complete props
Dependencies   -- MUST be optional constructor params with GetIt fallback
Async methods  -- MUST use SafeCubitEmissions mixin
Navigation     -- MUST be in widget layer via BlocListener
Cross-cubit    -- MUST be in widget layer via context.read<OtherCubit>()
GetIt access   -- constructor only, never inside methods
```
