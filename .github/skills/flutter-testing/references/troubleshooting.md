---
metadata:
  last_modified: "2026-03-31 14:30:00 (GMT+8)"
---

# Testing Troubleshooting & Edge Cases

## Goal
Diagnose and resolve common production testing failures including async errors, golden test platform inconsistencies, missing widget dependencies, and mock configuration issues. This guide focuses on real-world edge cases encountered in CI/CD pipelines and cross-platform development.

## Instructions

### Common Widget Test Failures

#### 1. Async Operation Errors

**⚠️ CRITICAL: "Expected: \<Future\>" Error**

**Problem:** Tests fail with messages like "Expected: \<Future\<void\>\>" or assertions execute before async operations complete.

**Root Cause:** Missing `await` on `tester.pump()` or `tester.pumpAndSettle()` after triggering state changes.

**❌ Anti-pattern:**
```dart
testWidgets('Button tap updates counter', (tester) async {
  await tester.pumpWidget(MaterialApp(home: CounterWidget()));
  
  await tester.tap(find.byType(ElevatedButton));
  // ❌ Missing pump - test checks OLD state!
  expect(find.text('1'), findsOneWidget); // FAILS
});
```

**✅ Best Practice:**
```dart
testWidgets('Button tap updates counter', (tester) async {
  await tester.pumpWidget(MaterialApp(home: CounterWidget()));
  
  await tester.tap(find.byType(ElevatedButton));
  await tester.pumpAndSettle(); // ✅ Wait for all frames to complete
  expect(find.text('1'), findsOneWidget); // PASSES
});
```

**When to use each:**
- `pump()`: Single frame advance - use for precise timing control
- `pumpAndSettle()`: Wait until no more frames scheduled - use for animations/async state updates
- `pump(Duration)`: Advance virtual time by specific duration

---

#### 2. MediaQuery/Theme Missing Errors

**⚠️ CRITICAL: "MediaQuery.of() called with a context that does not contain a MediaQuery"**

**Problem:** Widgets crash during tests with errors about missing MediaQuery, Theme, or Directionality.

**Root Cause:** Test widgets not wrapped in MaterialApp or CupertinoApp, which provide these dependencies.

**❌ Anti-pattern:**
```dart
testWidgets('Widget renders correctly', (tester) async {
  // ❌ No MaterialApp wrapper
  await tester.pumpWidget(MyCustomWidget());
  // CRASH: MediaQuery.of() called with a context that does not contain a MediaQuery
});
```

**✅ Best Practice:**
```dart
testWidgets('Widget renders correctly', (tester) async {
  // ✅ Always wrap in MaterialApp
  await tester.pumpWidget(
    MaterialApp(
      home: Scaffold(
        body: MyCustomWidget(),
      ),
    ),
  );
  
  expect(find.byType(MyCustomWidget), findsOneWidget);
});
```

**For custom themes:**
```dart
await tester.pumpWidget(
  MaterialApp(
    theme: ThemeData(
      primarySwatch: Colors.blue,
      textTheme: customTextTheme,
    ),
    home: MyCustomWidget(),
  ),
);
```

---

#### 3. Finder Errors: Element Not Found

**Problem:** `expect(find.text('...'), findsOneWidget)` fails even though the text visibly exists.

**Common Causes:**

**Cause 1: Text wrapped in RichText/SelectableText**
```dart
// ❌ Won't find text inside RichText
expect(find.text('Hello'), findsOneWidget);

// ✅ Use descendant finders
expect(
  find.descendant(
    of: find.byType(RichText),
    matching: find.text('Hello'),
  ),
  findsOneWidget,
);
```

**Cause 2: Overflow widget hiding content**
```dart
// ✅ Provide explicit size constraints
await tester.pumpWidget(
  MaterialApp(
    home: SizedBox(
      width: 400,
      height: 600,
      child: MyWidget(),
    ),
  ),
);
```

**Cause 3: Animation/Transition in progress**
```dart
// ❌ Element not yet visible
await tester.tap(find.text('Next'));
expect(find.text('Page 2'), findsOneWidget); // FAILS

// ✅ Wait for animations
await tester.tap(find.text('Next'));
await tester.pumpAndSettle();
expect(find.text('Page 2'), findsOneWidget); // PASSES
```

---

### Golden Test Cross-Platform Reliability

#### 4. Font Rendering Inconsistencies

**⚠️ CRITICAL: Pixel Differences Between Local and CI**

**Problem:** Golden tests pass locally but fail in CI/CD with pixel diffs in text rendering.

**Root Cause:** Different font rendering engines, antialiasing, and default fonts across macOS/Linux/Windows/Docker.

**✅ Best Practice: Use `golden_toolkit` with Font Preloading**

**Step 1: Add dependency**
```yaml
dev_dependencies:
  golden_toolkit: ^0.15.0
```

**Step 2: Create test configuration**
```dart
// test/flutter_test_config.dart
import 'dart:async';
import 'package:golden_toolkit/golden_toolkit.dart';

Future<void> testExecutable(FutureOr<void> Function() testMain) async {
  // ✅ Load fonts BEFORE any tests run
  await loadAppFonts();
  return testMain();
}
```

**Step 3: Use `testGoldens` instead of `testWidgets`**
```dart
import 'package:golden_toolkit/golden_toolkit.dart';

void main() {
  testGoldens('Login screen golden test', (tester) async {
    await tester.pumpWidgetBuilder(
      LoginScreen(),
      surfaceSize: Size(375, 667), // ✅ Explicit size for consistency
    );
    
    await screenMatchesGolden(tester, 'login_screen');
  });
}
```

---

#### 5. Pixel Difference Tolerance

**Problem:** Minor subpixel antialiasing differences fail tests unnecessarily.

**✅ Best Practice: Custom Comparator with Threshold**

```dart
// test/golden_test_helper.dart
import 'dart:io';
import 'package:flutter_test/flutter_test.dart';

class LocalFileComparatorWithThreshold extends LocalFileComparator {
  final double threshold;
  
  LocalFileComparatorWithThreshold(Uri testFile, this.threshold)
      : super(testFile);
  
  @override
  Future<bool> compare(Uint8List imageBytes, Uri golden) async {
    final ComparisonResult result = await GoldenFileComparator.compareLists(
      imageBytes,
      await File.fromUri(golden).readAsBytes(),
    );
    
    if (!result.passed && result.diffPercent <= threshold) {
      print('Diff ${result.diffPercent}% within threshold $threshold%');
      return true;
    }
    
    return result.passed;
  }
}
```

**Usage in tests:**
```dart
void main() {
  setUp(() {
    // ✅ Set 1% tolerance for antialiasing differences
    final testUri = Uri.parse(Platform.script.toString());
    goldenFileComparator = LocalFileComparatorWithThreshold(testUri, 0.01);
  });

  testWidgets('Button golden with tolerance', (tester) async {
    await tester.pumpWidget(MaterialApp(home: MyButton()));
    await expectLater(
      find.byType(MyButton),
      matchesGoldenFile('button.png'),
    );
  });
}
```

**Recommended thresholds:**
- **0.01 (1%)**: Text-heavy UIs with antialiasing variations
- **0.05 (5%)**: Complex layouts with shadows/gradients
- **0.001 (0.1%)**: Strict visual regression (icons, solid colors)

---

#### 6. Device Configuration Best Practices

**Problem:** Goldens differ based on device pixel ratio, screen size, or platform defaults.

**✅ Best Practice: Always Specify Surface Size and Device**

```dart
import 'package:golden_toolkit/golden_toolkit.dart';

void main() {
  testGoldens('Multi-device golden test', (tester) async {
    final builder = DeviceBuilder()
      ..overrideDevicesForAllScenarios(devices: [
        Device.phone, // iPhone 11 (414x896)
        Device.iphone11,
        Device.tabletPortrait, // iPad (768x1024)
      ])
      ..addScenario(
        widget: LoginScreen(),
        name: 'default state',
      )
      ..addScenario(
        widget: LoginScreen(hasError: true),
        name: 'error state',
      );

    await tester.pumpDeviceBuilder(builder);
    
    await screenMatchesGolden(tester, 'login_multi_device');
  });
}
```

**For single-device tests:**
```dart
testGoldens('Dashboard golden', (tester) async {
  await tester.pumpWidgetBuilder(
    DashboardScreen(),
    surfaceSize: Size(1024, 768), // ✅ Explicit tablet size
    wrapper: materialAppWrapper(
      theme: ThemeData.light(),
      platform: TargetPlatform.android, // ✅ Lock platform
    ),
  );
  
  await screenMatchesGolden(tester, 'dashboard');
});
```

---

#### 7. CI/CD Golden Test Best Practices

**⚠️ CRITICAL: Ensure Consistent CI Environment**

**Checklist for CI/CD:**

✅ **Use identical Docker image across all runners**
```dockerfile
# Dockerfile for CI
FROM cirrusci/flutter:stable

# Install fonts matching development environment
RUN apt-get update && apt-get install -y fonts-roboto fonts-noto

WORKDIR /app
COPY . .
RUN flutter pub get
```

✅ **Always call `loadAppFonts()` in test config**
```dart
// test/flutter_test_config.dart REQUIRED for CI
Future<void> testExecutable(FutureOr<void> Function() testMain) async {
  await loadAppFonts();
  return testMain();
}
```

✅ **Set tolerance threshold (1-5% for most apps)**
```dart
goldenFileComparator = LocalFileComparatorWithThreshold(testUri, 0.02);
```

✅ **Use `pumpAndSettle()` before capturing**
```dart
await tester.pumpWidget(MaterialApp(home: MyWidget()));
await tester.pumpAndSettle(); // ✅ Wait for animations
await expectLater(find.byType(MyWidget), matchesGoldenFile('widget.png'));
```

✅ **Regenerate goldens on single platform only**
```bash
# Generate goldens on Linux (CI environment)
flutter test --update-goldens --tags golden
```

**❌ Anti-pattern: Mixing macOS/Linux goldens**
```bash
# ❌ Developer on macOS updates goldens
flutter test --update-goldens # Creates macOS-specific antialiasing

# ❌ CI on Linux fails with pixel diffs
flutter test # FAILS: Linux rendering != macOS
```

---

### Mock Dependencies & Flaky Tests

#### 8. External Dependency Isolation

**Problem:** Tests are flaky due to real network calls, database queries, or file I/O.

**✅ Best Practice: Use Mocktail/Mockito with Dependency Injection**

**Example: Mocking API Client**

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';

// 1. Create mock
class MockApiClient extends Mock implements ApiClient {}
class FakeUser extends Fake implements User {}

void main() {
  setUpAll(() {
    // ✅ Register fallback for custom types
    registerFallbackValue(FakeUser());
  });

  group('UserRepository tests', () {
    late MockApiClient mockClient;
    late UserRepository repository;

    setUp(() {
      mockClient = MockApiClient();
      repository = UserRepository(apiClient: mockClient);
    });

    test('fetchUser returns user on success', () async {
      // 2. Stub the response
      final expectedUser = User(id: 1, name: 'Alice');
      when(() => mockClient.getUser(any()))
          .thenAnswer((_) async => expectedUser);

      // 3. Act
      final result = await repository.fetchUser(1);

      // 4. Assert
      expect(result, expectedUser);
      verify(() => mockClient.getUser(1)).called(1);
    });

    test('fetchUser throws on network error', () async {
      // ✅ Test error paths
      when(() => mockClient.getUser(any()))
          .thenThrow(NetworkException('Connection failed'));

      expect(
        () => repository.fetchUser(1),
        throwsA(isA<NetworkException>()),
      );
    });
  });
}
```

**Example: Mocking SharedPreferences**

```dart
import 'package:shared_preferences/shared_preferences.dart';

void main() {
  setUp(() {
    // ✅ Use in-memory fake for SharedPreferences
    SharedPreferences.setMockInitialValues({
      'user_token': 'mock_token_12345',
      'theme_mode': 'dark',
    });
  });

  test('AuthService reads token correctly', () async {
    final prefs = await SharedPreferences.getInstance();
    final authService = AuthService(prefs);
    
    expect(authService.getToken(), 'mock_token_12345');
  });
}
```

---

#### 9. Async Test Timeouts

**Problem:** Tests hang or timeout waiting for async operations.

**Common Causes:**

**Cause 1: Infinite animations**
```dart
// ❌ pumpAndSettle hangs on CircularProgressIndicator
await tester.tap(find.text('Submit'));
await tester.pumpAndSettle(); // TIMEOUT after 10 minutes

// ✅ Use precise pump durations
await tester.tap(find.text('Submit'));
await tester.pump(); // Trigger loading state
await tester.pump(Duration(seconds: 2)); // Simulate API delay
expect(find.byType(CircularProgressIndicator), findsNothing);
```

**Cause 2: Real async without `runAsync`**
```dart
// ❌ Real file I/O not intercepted by FakeAsync
testWidgets('File upload', (tester) async {
  await tester.pumpWidget(MyApp());
  await tester.tap(find.text('Upload'));
  await tester.pumpAndSettle(); // Hangs - real I/O blocks
});

// ✅ Use runAsync for real system operations
testWidgets('File upload', (tester) async {
  await tester.pumpWidget(MyApp());
  
  await tester.runAsync(() async {
    // Escape FakeAsync boundary for real I/O
    await tester.tap(find.text('Upload'));
    await Future.delayed(Duration(seconds: 1)); // Real delay
  });
  
  await tester.pumpAndSettle();
});
```

---

#### 10. Memory Leaks in Tests

**⚠️ CRITICAL: "A Timer is still pending even after the widget tree was disposed"**

**Problem:** Tests fail with errors about pending Timers or uncanceled subscriptions.

**Root Cause:** Resources not cleaned up in widget `dispose()` method.

**❌ Anti-pattern:**
```dart
class MyWidget extends StatefulWidget {
  @override
  _MyWidgetState createState() => _MyWidgetState();
}

class _MyWidgetState extends State<MyWidget> {
  Timer? _timer;
  
  @override
  void initState() {
    super.initState();
    _timer = Timer.periodic(Duration(seconds: 1), (_) {
      // Update state
    });
  }
  
  // ❌ No dispose - timer leaks!
}
```

**✅ Best Practice:**
```dart
class _MyWidgetState extends State<MyWidget> {
  Timer? _timer;
  
  @override
  void initState() {
    super.initState();
    _timer = Timer.periodic(Duration(seconds: 1), (_) {
      setState(() {/* update */});
    });
  }
  
  @override
  void dispose() {
    _timer?.cancel(); // ✅ Always cancel in dispose
    super.dispose();
  }
}
```

**Testing Timer-based widgets:**
```dart
testWidgets('Timer widget cleans up', (tester) async {
  await tester.pumpWidget(MaterialApp(home: TimerWidget()));
  
  // Advance virtual time
  await tester.pump(Duration(seconds: 5));
  
  // Dispose widget
  await tester.pumpWidget(SizedBox.shrink());
  
  // ✅ No pending timer error if dispose() implemented correctly
});
```

---

## Constraints

* **Always wrap test widgets in MaterialApp/CupertinoApp** unless explicitly testing widgets that don't require material/cupertino dependencies.
* **Use `loadAppFonts()` in CI/CD** to eliminate font rendering inconsistencies.
* **Mock all external dependencies** (network, database, file I/O) to avoid flaky tests.
* **Set explicit surface sizes** in golden tests for cross-platform consistency.
* **Never use `pumpAndSettle()` with infinite animations** - use `pump(Duration)` instead.
* **Always implement `dispose()`** for Timers, StreamSubscriptions, and AnimationControllers.
