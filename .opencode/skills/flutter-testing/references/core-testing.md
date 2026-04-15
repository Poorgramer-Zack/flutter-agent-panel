---
metadata:
  last_modified: "2026-03-12 11:18:17 (GMT+8)"
---

# Core Testing (Core Testing Standard Guide)

## Goal
Implements robust baseline testing in the Flutter ecosystem. Establishing foundational tests is the first step to ensuring software quality. The official `test` and `flutter_test` packages, paired with powerful `matcher` components, are used to achieve coverage.

## Instructions

### Three Levels of the Testing Pyramid
Flutter defines three primary levels of testing:

1. **Unit Test**: Tests a single function, method, or class. They are the fastest and have the fewest dependencies.
2. **Widget Test**: Tests the layout and interactions of a single UI component. No physical device is needed; they run in a simulated testing environment (`TestWidgetsFlutterBinding`).
3. **Integration Test**: Tests the complete App flow or the interaction of most modules. Requires a physical device or simulator. They are the slowest but closest to real user behavior.

### Unit Test and Matcher
Use the native Dart `test` package (`import 'package:test/test.dart';`) or `flutter_test`.

#### 2.1 Basic Structure
```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:my_app/calculator.dart';

void main() {
  // Use group to categorize related tests
  group('Calculator Basic Operations Test', () {
    
    // setUp is called before each test runs (ideal for resetting states)
    setUp(() { /* initialization */ });
    
    // tearDown is called after each test finishes (ideal for cleaning up resources)
    tearDown(() { /* disposal */ });

    test('Addition computes correctly', () {
      // 1. Arrange 
      final calc = Calculator();
      
      // 2. Act 
      final result = calc.add(2, 3);
      
      // 3. Assert 
      expect(result, 5);
    });
  });
}
```

#### 2.2 Powerful Matcher Best Practices
Don't just use `expect(a_bool, true)`. Leverage Matchers to make error messages more precise when a test fails:

```dart
expect(result, isTrue); // Boolean
expect(list, isEmpty);  // Collection
expect(list, contains('apple')); // Contains specific element
expect(user.age, greaterThanOrEqualTo(18)); // Value comparison
expect(
  () => calc.divide(10, 0), 
  throwsA(isA<ArgumentError>()), // Throws specific error
);
```

### Widget Test
Use the `testWidgets` environment provided by `flutter_test`.

#### 3.1 Basic UI Testing
```dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';

void main() {
  testWidgets('Test MyWidget displays title and a clickable button', (WidgetTester tester) async {
    // 1. Load Widget (Must be wrapped in a MaterialApp to provide Directionality and other base configs)
    await tester.pumpWidget(const MaterialApp(home: MyWidget(title: 'T')));

    // 2. Find Elements (Finders)
    final titleFinder = find.text('T');
    final buttonFinder = find.byType(ElevatedButton);
    final iconFinder = find.byIcon(Icons.add);
    final keyFinder = find.byKey(const Key('my-unique-key'));

    // 3. Verify initial state
    expect(titleFinder, findsOneWidget); // Ensure exactly one is found
    expect(find.text('0'), findsOneWidget); 
    expect(find.text('1'), findsNothing); // Ensure it's not found

    // 4. Interact with UI (You must call pump to trigger a screen redraw after a deliberate action)
    await tester.tap(buttonFinder);
    await tester.pump(); // Or pumpAndSettle() to wait for all animations to conclude

    // 5. Verify new state
    expect(find.text('1'), findsOneWidget);
  });
}
```

### Integration Test
Use the official `integration_test` package to run tests on a real device.

```yaml
dev_dependencies:
  integration_test:
    sdk: flutter
  flutter_test:
    sdk: flutter
```

#### 4.1 Structure and Execution
Create `integration_test/app_test.dart` in the project root:

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:integration_test/integration_test.dart';
import 'package:my_app/main.dart' as app;

void main() {
  // 🌟 You must ensure initialization bindings are configured
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  testWidgets('End-to-End Test: Login Flow', (WidgetTester tester) async {
    // Start the whole App
    app.main();
    // Wait for App to fully launch
    await tester.pumpAndSettle();

    // Execute a series of actions
    await tester.enterText(find.byType(TextField).first, 'test@example.com');
    await tester.enterText(find.byType(TextField).last, 'password123');
    await tester.tap(find.text('Login'));
    await tester.pumpAndSettle(); // Wait for APIs and animations

    // Verify navigation to Homepage
    expect(find.text('Welcome, User!'), findsOneWidget);
  });
}
```

**How to run**: You need to specify a target device in your terminal:
```bash
flutter test integration_test/app_test.dart -d <device_id>
```

> **Note**: Native `integration_test` cannot interact with native system dialogs (e.g., notification permissions, camera permissions, WebViews). Use full `patrol` package with `patrol test` CLI for these cases. See [Advanced Tools](./advanced-tools.md#patrol).

### Common Pain Points and Async Troubleshooting Guide
In StackOverflow and GitHub communities, the most common issues when testing Flutter are Timeouts or Memory Leak warnings caused by `Timer` or `Future.delayed`.

#### 5.1 Pain Point 1: "A Timer is still pending even after the widget tree was disposed"
**Cause**:
A `Timer` (or `AnimationController`) was started in a Widget's `State`, but it wasn't `cancel()`ed during `dispose()`. The testing framework (`testWidgets`) runs by default in a virtual time manipulation environment called `FakeAsync`. When the test concludes, it strictly checks for any unexecuted schedules. If any are found, it determines it's a potential Memory Leak and throws an error.

**Correct Solutions**:
1. **Always clean up in dispose**:
   ```dart
   Timer? _timer;
   @override
   void dispose() {
     _timer?.cancel();
     super.dispose();
   }
   ```
2. **Advance virtual time in tests**: If the Timer must complete execution, advance time manually before the test ends.
   ```dart
   // Advance by 5 seconds to let the Timer fire and conclude
   await tester.pump(const Duration(seconds: 5)); 
   ```

#### 5.2 Pain Point 2: Timeout on `pumpAndSettle` (Default is 10 Minutes)
**Cause**:
The underlying logic of `pumpAndSettle` is "constantly calling `pump` until **no new Frames are scheduled** on the screen".
If you have **infinite loop animations** on the screen (e.g., `CircularProgressIndicator`, `LinearProgressIndicator`, or Rive/Lottie animations), the UI will never "Settle", inevitably resulting in a Timeout error.

**Correct Solution**:
Never use `pumpAndSettle` on screens featuring infinite animations. Utilize precise `pump`ing for controlling time instead:
```dart
// ❌ Incorrect: Getting stuck on a loading spinner leading to a Timeout
await tester.tap(find.text('Submit'));
await tester.pumpAndSettle(); 

// ✅ Correct: Precisely simulate the time spent awaiting an API or spinner
await tester.tap(find.text('Submit'));
await tester.pump(); // Triggers redraw (display loading)
await tester.pump(const Duration(seconds: 2)); // Simulates a 2-second wait for data load completion
expect(find.byType(CircularProgressIndicator), findsNothing);
```

#### 5.3 Pain Point 3: Real Async Tasks vs `tester.runAsync`
**Cause**:
`testWidgets` intercepts Dart's native time mechanisms creating the `FakeAsync` boundary. All `Future.delayed` instances are instantly accelerated. However, if your code natively relies on **real system asynchronous operations** (e.g., reading/writing real files, complex Isolate computations, or un-Mocked HTTP requests), `FakeAsync` cannot control them. This will freeze tests resulting in Deadlocks.

**Correct Solution (`runAsync`)**:
When you must wait for real system tasks "outside Flutter's control framework," use `tester.runAsync` to temporarily escape the time illusion and return to true Wall-clock time:
```dart
testWidgets('Test true underlying async behaviors', (tester) async {
  await tester.pumpWidget(const MyApp());

  // Temporarily escape FakeAsync, awaiting in real-world time
  await tester.runAsync(() async {
    // This Future.delayed is a real 1-second wait
    // Appropriate for waiting on true Database initializations or I/O
    await Future.delayed(const Duration(seconds: 1));
  });

  await tester.pumpAndSettle();
});
```
> **Strong Recommendation**: Attempt to Mock time-consuming services using Dependency Injection (DI) as much as possible, keeping `runAsync` as an absolute last resort. A healthy Widget Test should execute entirely within `FakeAsync` to ensure blazing-fast speed.

## Constraints
* Ensure that you avoid using `pumpAndSettle` whenever infinite animations are executing in an active widget test.
* Manage timeouts appropriately regarding `Timer` utilization by enforcing `cancel()` mechanisms inside of `dispose()`.
