---
name: test guidance
description: Rules and conventions for writing Flutter tests in this project. Covers approved packages, file structure, AAA comment requirements, success/failure case requirements, mock naming, repository tests, cubit tests with blocTest, widget tests with pumpFakeApp, mock cubit patterns, fake data conventions, and forbidden patterns. Referenced by TestPlanner, TestBuilder, TestUpdater, TestCorrector, and TestRunner.
---

# Test Guidance

> Only the approved packages below may be used. Do not introduce additional test dependencies without explicit approval.

---

## 1. Approved Test Packages

```yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
  mocktail: ^1.0.4
  bloc_test: ^10.0.0
  mocktail_image_network: ^1.2.0
  webview_flutter_platform_interface: ^2.10.0
  plugin_platform_interface: ^2.1.8
```


**Do not use:** `mockito`, `http_mock_adapter`, `dio` mocking, `golden_toolkit`, or any screenshot test library.

---

## 2. File Structure

Mirror `lib/` exactly under `test/`:

```
test/
  test_helpers.dart           ← shared test utilities (pumpFakeApp, global mocks)
  webview_test_helpers.dart   ← webview platform mock registration helpers
  features/
    <feature>/
      cubit/
        <feature>_cubit_test.dart
      pages/
        <page>_page_test.dart
      widgets/
        <widget>_test.dart
  repositories/
    <name>_repository_test.dart
  cubit/
    <name>_cubit_test.dart    ← for global/shared cubits
  widgets/
    <name>_test.dart
  router/
    router_test.dart
  models/
    <name>_test.dart
  helpers/
    <name>_test.dart
```

---

## 3. Conventions

- **AAA**: Every `test()` body must have `// Arrange`, `// Act`, `// Assert` comments. One-liner asserts may use just `// Assert`.
- **Success + failure cases**: Both are required for every feature under test -- never skip the failure case.
- **SUT variable**: In unit tests assign the subject under test to a variable named `sut`.
- **`group()` organisation**: Top-level group = class name; nested groups = method or scenario.
- **Test naming**: `[MethodUnderTest]_[Scenario]_[ExpectedBehavior]` -- e.g. `getThing_apiSucceeds_returnsModel`. Avoid prose descriptions.
- **Mock naming**: Declare mock classes as private to the test file: `class _MockThing extends Mock implements Thing {}`.
- **No GetIt for SUT injection**: Always inject via constructor. Use `GetIt.I.registerSingleton` only for internal dependencies the widget/cubit reads from GetIt; call `GetIt.I.reset()` in `tearDown`.
- **Fake data**: Top-level `const` constants, descriptively named (`_fakeThing`, `_nextFakeThing`), local to the test file -- never shared via imports.

---

## 4. Repository Tests

Use plain `test()` calls. Construct the SUT via constructor injection; assign to `sut`.

```dart
class _MockThingApi extends Mock implements ThingApi {}
const _fakeThing = ThingModel(id: '1');

void main() {
  group('ThingRepository', () {
    late _MockThingApi _mockApi;
    setUp(() => _mockApi = _MockThingApi());

    test('getThing_apiSucceeds_returnsModel', () async {
      // Arrange
      when(() => _mockApi.getThing()).thenAnswer((_) async => ApiSuccessResult(data: _fakeThing));
      final sut = ThingRepository(api: _mockApi);
      // Act + Assert
      expect(await sut.getThing(), equals(_fakeThing));
    });
    // Failure case: stub returns ApiErrorResult, expect null / error state
  });
}
```

---

## 5. Cubit Tests

Use `blocTest<CubitType, StateType>` from `bloc_test`.

| Parameter | Purpose |
|-----------|---------|
| `build` | Constructs the cubit under test with injected mock dependencies |
| `setUp` | Optional per-`blocTest` setup for stubs (runs before `build`) |
| `seed` | Optional initial state for tests starting from a non-initial state |
| `act` | Calls the method under test on the cubit |
| `expect` | Asserts the emitted state sequence |
| `verify` | Optionally verifies mock interactions after emission |

```dart
blocTest<ThingCubit, ThingState>(
  'load_repositoryReturnsModel_emitsLoadingThenLoaded',
  build: () => ThingCubit(repo: _mockRepo),
  act: (cubit) => cubit.load(),
  expect: () => [const ThingLoading(), ThingLoaded(model: _fakeThing)],
  verify: (_) => verify(() => _mockRepo.getThing()).called(1),
);
// Failure case: override stub in per-test setUp, assert error state
// seed: supply pre-existing state when the action requires it (e.g. seed: () => ThingLoaded(...))
```

---

## 6. Widget Tests

Use `testWidgets` with `TestHelpers.pumpFakeApp()` as the standard pump helper. This wraps the widget in `MaterialApp` with localisation, `InheritedGoRouter`, and global mock cubits (`MockAppCubit`, `MockUserCubit`, etc.).

```dart
testWidgets('build_stateIsThingLoaded_showsContent', (tester) async {
  // Arrange
  whenListen<ThingState>(mockCubit, const Stream.empty(), initialState: ThingLoaded(model: _fakeThing));
  // Act
  await TestHelpers.pumpFakeApp(
    tester,
    BlocProvider<ThingCubit>.value(value: mockCubit, child: const ThingWidget()),
  );
  // Assert
  expect(find.text('Expected Text'), findsOneWidget);
});
// Failure case: change initialState to error state, assert error widget visible
```

### Widget Test Rules
- Always use `whenListen(mockCubit, stream, initialState: ...)` -- never emit states directly
- Pass feature cubits via `BlocProvider<X>.value(value: mock)` wrapping the widget; `pumpFakeApp` merges them with global mocks
- Network images are already suppressed inside `pumpFakeApp` via `mockNetworkImages` -- no extra setup needed

---

## 7. Shared Mock Cubits

When a mock cubit is needed across multiple test files, declare it in `test_helpers.dart` or a feature-level `mock_cubits.dart`:

```dart
class MockThingCubit extends MockCubit<ThingState> implements ThingCubit {
  MockThingCubit() {
    whenListen(this, const Stream<ThingState>.empty(), initialState: const ThingInitial());
  }
}
```

- Provide a sensible `initialState` in the constructor so the cubit is usable without extra setup
- Override the stream in individual tests with `whenListen(mockThingCubit, ...)` as needed

---

## 8. Forbidden Patterns

- Do **not** use `mockito` -- use `mocktail` only
- Do **not** use `http_mock_adapter` or `dio` mocking -- mock the repository or API class directly
- Do **not** use `golden_toolkit` or screenshot tests
- Do **not** call `GetIt.I<X>()` inside a test to obtain the SUT -- always use constructor injection
- Do **not** write tests that test implementation details (private methods, internal counters) -- test observable state and output only
- Do **not** skip the failure case because "it's obvious" -- it is required

---

## Checklist
- [ ] Only approved packages used -- no additional test dependencies introduced
- [ ] Test file mirrors the `lib/` path under `test/`
- [ ] Every `test()` body has `// Arrange`, `// Act`, `// Assert` comments
- [ ] Both success and failure paths covered for every feature under test
- [ ] SUT assigned to variable named `sut` in unit tests
- [ ] Tests organised with `group()` by class at top level
- [ ] Test names follow `[MethodUnderTest]_[Scenario]_[ExpectedBehavior]` convention
- [ ] Mocks declared as private classes with `_` prefix local to the test file
- [ ] No GetIt used to obtain or inject the SUT -- constructor injection only
- [ ] `GetIt.I.reset()` called in `tearDown` if GetIt was used for any dependency
- [ ] Widget tests use `whenListen(...)` + `TestHelpers.pumpFakeApp()` -- no direct state emission
- [ ] Fake data is `const`, locally scoped, and descriptively named
