---
metadata:
  last_modified: "2026-03-12 11:18:17 (GMT+8)"
---

# Advanced Testing Tools and Frameworks (Advanced Testing Tools)

## Goal
Implements advanced testing tools and frameworks to resolve common pain points in the native Flutter testing environment: dependency decoupling (Mocking), native system interaction (Native E2E), and test debugging (Test Debugging).

## Instructions

### Mockito (Mock Objects and Dependency Inversion)
[mockito](https://pub.dev/packages/mockito) is the most mainstream Mocking tool. When paired with `build_runner`, it can automatically generate fake classes.

#### 1.1 Installation and Configuration
```yaml
dependencies:
  mockito: ^5.4.4

dev_dependencies:
  build_runner: ^2.4.0
```

#### 1.2 Generating Mock Classes
Use the `@GenerateNiceMocks` annotation to generate Mock classes. The advantage of `NiceMocks` is that when it encounters an undefined method, it returns a safe null or default value instead of crashing.

```dart
import 'package:mockito/annotations.dart';
import 'package:mockito/mockito.dart';
import 'api_client.dart';

// This will generate api_client.mocks.dart
@GenerateNiceMocks([MockSpec<ApiClient>()])
import 'api_client.mocks.dart';

void main() {
  test('Test API Call', () async {
    final mockClient = MockApiClient();
    
    // 🌟 Stubbing: Set the default return value when fetchUser is called
    when(mockClient.fetchUser(any))
        .thenAnswer((_) async => User(id: 1, name: 'Zack'));
        
    final repo = UserRepository(apiClient: mockClient);
    final user = await repo.getUser(1);
    
    expect(user.name, 'Zack');
    
    // 🌟 Verify the call count is as expected
    verify(mockClient.fetchUser(1)).called(1);
  });
}
```
Run `dart run build_runner build` to generate the files.

### Mocktail (Code-generation-free Mock Alternative)
[mocktail](https://pub.dev/packages/mocktail) is another highly popular choice comparable to `mockito`. Its greatest advantage is that **it absolutely does not rely on `build_runner` for code generation**, making it particularly suitable for developers who hate long build times.

#### 2.1 Installation and Basic Configuration
```yaml
dev_dependencies:
  mocktail: ^1.0.4
```

#### 2.2 Core Syntax and Usage
The underlying API of `mocktail` borrows heavily from `mockito`, but the implementation is different.

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';

// 1. Directly declare a class that extends Mock and implements the target interface
class MockApiClient extends Mock implements ApiClient {}

// 🌟 Fallback for Custom Types (The core of Mocktail solving Null Safety)
class FakeUser extends Fake implements User {}

void main() {
  setUpAll(() {
    // If your method parameters use custom objects and any(), you MUST register the fallback in setUpAll
    registerFallbackValue(FakeUser());
  });

  test('Test using Mocktail', () async {
    final mockClient = MockApiClient();

    // 2. Stubbing (Identical to mockito)
    when(() => mockClient.fetchUser(any()))
        .thenAnswer((_) async => User(id: 2, name: 'Alice'));

    // 3. Act
    final repo = UserRepository(apiClient: mockClient);
    final user = await repo.getUser(2);

    // 4. Assert & Verify
    expect(user.name, 'Alice');
    verify(() => mockClient.fetchUser(2)).called(1);
  });
}
```

> **Comparison**:
> - **Mockito**: Requires `build_runner`. The advantage is all fallbacks and type-safety are handled automatically during the generation phase.
> - **Mocktail**: Write and test instantly, no generation wait time. The trade-off is manually writing many `Fake` classes to `registerFallbackValue()` when encountering `any()` with custom objects.

### Patrol (Native E2E Testing Framework) - 🌟 Industry Preferred E2E
The most fatal flaw of Flutter's native `integration_test` is the **inability to tap native system dialogs** (e.g., requesting location permissions, camera authorization, interacting with WebViews, push notifications). [Patrol](https://patrol.leancode.co/) solves this with a native automator bridge.

**Two packages — choose based on your needs:**
- `patrol` — full framework with native OS interaction, requires `patrol_cli` and platform setup
- `patrol_finders` — lightweight custom finder API only, works with standard `flutter test`, no native setup

#### 3.1 patrol_finders (Widget Test Enhancement — No Native Setup Required)

Drop-in enhancement for widget tests. Use `patrolWidgetTest` instead of `testWidgets`:

```yaml
dev_dependencies:
  patrol_finders: ^3.2.0
```

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:patrol_finders/patrol_finders.dart';

void main() {
  patrolWidgetTest('login flow', ($) async {
    await $.pumpWidgetAndSettle(const MyApp());

    // Chain selectors — far more readable than nested find.descendant()
    await $(#emailInput).enterText('user@test.com');
    await $(#passwordInput).enterText('secret');
    await $(Scaffold).$(#loginBox).$('Log in').tap();

    // Contextual scroll finders
    await $(Scrollable).containing(ElevatedButton).tap();

    expect($(Text).containing('Welcome'), findsOneWidget);
  });
}
```

Run with standard `flutter test` — no extra CLI needed.

#### 3.2 patrol (Full E2E with Native Interactions)

Use `patrolTest` for flows that cross the Flutter/OS boundary:

```yaml
dev_dependencies:
  patrol: ^4.5.0
```

```bash
# Install patrol CLI
dart pub global activate patrol_cli

# Run patrol tests (NOT flutter test)
patrol test --target integration_test/app_test.dart
```

```dart
import 'package:patrol/patrol.dart';

void main() {
  patrolTest(
    'notification permission flow',
    ($) async {
      await $.pumpWidgetAndSettle(const MyApp());

      await $(#enableNotifications).tap();

      // Platform API (patrol 4.0+) — replaces deprecated $.native.*
      await $.platform.mobile.grantPermissionWhenInUse();

      // Press Home, open notification tray, tap notification
      await $.platform.mobile.pressHome();
      await $.platform.mobile.openNotifications();
      await $.platform.tap(Selector(text: 'New message'));

      expect($('Chat'), findsOneWidget);
    },
  );
}
```

#### 3.3 Patrol 4.0 Platform API Reference

Patrol 4.0 reorganized native interactions into semantic categories. `$.native` is **deprecated**.

```dart
// Mobile (both Android + iOS)
await $.platform.mobile.pressHome();
await $.platform.mobile.pressBack();
await $.platform.mobile.openNotifications();
await $.platform.mobile.grantPermissionWhenInUse();
await $.platform.mobile.grantPermissionAlways();
await $.platform.mobile.denyPermission();
await $.platform.mobile.pressDoubleRecentApps();

// Android-specific
await $.platform.android.openNotifications();
await $.platform.android.tap(Selector(text: 'Allow'));

// iOS-specific
await $.platform.iOS.dismissSafariPrompt();

// Web
await $.platform.web.clearCookies();

// Cross-platform (same as old $.native at the top level)
await $.platform.openNotifications();
await $.platform.tap(Selector(text: 'OK'));
```

> **When to use each:**
> - `patrol_finders` — widget tests needing cleaner finder syntax, no CI complexity
> - `patrol` (full) — integration tests requiring permission dialogs, push notifications, WebViews, deep links

### Convenient Test (Enhancing Development and CI Experience)
[convenient_test](https://pub.dev/packages/convenient_test) is a heavy-duty testing plugin designed to solve the issues of "debugging difficulties" and "slow execution" in Flutter testing.

#### 4.1 Core Features
1. **Time Travel + Screenshots**: Automatically records video and captures screenshots for every step of your test. When a test fails, you can open the GUI and drag the timeline to see exactly what the screen looked like at that moment.
2. **5x+ Speed on Desktop**: Allows heavy Integration tests normally bound to mobile simulators to run at ultra-high speed on Host (Mac/Windows) desktop environments.
3. **Isolation Mode**: Ensures tests are completely isolated (via automatic Hot Restart), preventing a dirty state from the previous test affecting the next.
4. **Built-in Auto Retry Mechanism**: Say goodbye to endless `pumpAndSettle`. It continuously automatically tries to find elements before timing out.

#### 4.2 Basic Setup
```dart
import 'package:convenient_test/convenient_test.dart';
import 'package:flutter_test/flutter_test.dart';

void main() {
  // Use tTestWidgets instead of testWidgets
  tTestWidgets('Tap button', (t) async {
    
    // No longer need to manually call pumpAndSettle, the framework automatically handles waiting!
    await t.get(ElevatedButton).tap();
    
    // Automatically generate Logs with friendly descriptions, displayed on the GUI panel
    t.log('Button tap concluded, preparing to verify');
    
    await t.get(find.text('Success')).should(findsOneWidget);
  });
}
```

> **Best Practice Workflow**:
> - Everyday small modules: Use `flutter_test` (Unit/Widget).
> - Core fool-proofing automation and debugging: Introduce `convenient_test` during daily development to accelerate verification.
> - Pre-submission/Final CI line of defense: Use `Patrol` to deploy the App onto Firebase Test Lab for brutal testing on real phones end-to-end (Including permission dialogs).

### Golden Tests (Visual Regression Testing)

Golden tests (also known as snapshot tests) capture pixel-perfect screenshots of widgets and compare them against baseline images to detect unintended visual regressions. They are particularly valuable for:
- Design system components (buttons, cards, typography)
- Complex layouts sensitive to CSS-like changes
- Cross-platform UI consistency verification
- Preventing accidental visual breaking changes

#### 5.1 Basic Golden Test Workflow

**Step 1: Add `golden_toolkit` dependency**
```yaml
dev_dependencies:
  golden_toolkit: ^0.15.0
  flutter_test:
    sdk: flutter
```

**Step 2: Create test configuration for font loading**
```dart
// test/flutter_test_config.dart
import 'dart:async';
import 'package:golden_toolkit/golden_toolkit.dart';

Future<void> testExecutable(FutureOr<void> Function() testMain) async {
  // ⚠️ CRITICAL: Load fonts BEFORE tests to ensure consistency
  await loadAppFonts();
  return testMain();
}
```

**Step 3: Write golden tests**
```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:golden_toolkit/golden_toolkit.dart';

void main() {
  testGoldens('Login button golden test', (tester) async {
    await tester.pumpWidgetBuilder(
      LoginButton(label: 'Sign In'),
      surfaceSize: Size(200, 50), // ✅ Explicit size for consistency
    );
    
    await screenMatchesGolden(tester, 'login_button_default');
  });

  testGoldens('Login button states', (tester) async {
    final builder = GoldenBuilder.column()
      ..addScenario('Default', LoginButton(label: 'Sign In'))
      ..addScenario('Disabled', LoginButton(label: 'Sign In', enabled: false))
      ..addScenario('Loading', LoginButton(label: 'Sign In', isLoading: true));

    await tester.pumpWidgetBuilder(builder.build());
    await screenMatchesGolden(tester, 'login_button_states');
  });
}
```

**Step 4: Generate baseline goldens**
```bash
flutter test --update-goldens --tags golden
```

**Step 5: Run golden tests**
```bash
flutter test --tags golden
```

#### 5.2 Cross-Platform Reliability Best Practices

**⚠️ CRITICAL: Font Rendering Consistency**

Golden tests are notorious for producing pixel differences across different operating systems (macOS vs Linux vs Windows) and CI/CD environments due to font rendering, antialiasing, and subpixel rendering variations.

**✅ Best Practice: Use Custom Comparator with Tolerance Threshold**

```dart
// test/golden_test_helper.dart
import 'dart:io';
import 'dart:typed_data';
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
      print('✅ Golden diff ${result.diffPercent}% within threshold $threshold%');
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

  testGoldens('Button with tolerance', (tester) async {
    await tester.pumpWidgetBuilder(
      ElevatedButton(onPressed: () {}, child: Text('Click Me')),
      surfaceSize: Size(150, 50),
    );
    await screenMatchesGolden(tester, 'button_golden');
  });
}
```

**Recommended Tolerance Thresholds:**
- **0.001 (0.1%)**: Strict mode for solid colors, icons, geometric shapes
- **0.01 (1%)**: Standard for text-heavy UIs with minor antialiasing variations
- **0.05 (5%)**: Permissive for complex layouts with shadows, gradients, images

#### 5.3 Device Configuration for Multi-Platform Testing

**✅ Best Practice: Always Specify Device and Surface Size**

```dart
import 'package:golden_toolkit/golden_toolkit.dart';

void main() {
  testGoldens('Multi-device dashboard', (tester) async {
    final builder = DeviceBuilder()
      ..overrideDevicesForAllScenarios(devices: [
        Device.phone,           // iPhone 11: 414x896
        Device.iphone11,
        Device.tabletPortrait,  // iPad: 768x1024
        Device.tabletLandscape, // iPad landscape: 1024x768
      ])
      ..addScenario(
        widget: DashboardScreen(),
        name: 'default',
      )
      ..addScenario(
        widget: DashboardScreen(hasNotifications: true),
        name: 'with_notifications',
      );

    await tester.pumpDeviceBuilder(builder);
    await screenMatchesGolden(tester, 'dashboard_multi_device');
  });
}
```

**For single-device tests with custom theme:**
```dart
testGoldens('Profile card golden', (tester) async {
  await tester.pumpWidgetBuilder(
    ProfileCard(user: mockUser),
    surfaceSize: Size(400, 600), // ✅ Explicit dimensions
    wrapper: materialAppWrapper(
      theme: ThemeData.light(),
      platform: TargetPlatform.android, // ✅ Lock platform behavior
    ),
  );
  
  await screenMatchesGolden(tester, 'profile_card');
});
```

#### 5.4 CI/CD Golden Test Configuration

**⚠️ CRITICAL: Ensure Consistent CI Environment**

**Problem:** Golden tests pass locally (macOS) but fail in CI (Linux Docker) with pixel diffs.

**✅ Solution Checklist:**

1. **Use identical Docker image for all CI runners**
```dockerfile
# Dockerfile
FROM cirrusci/flutter:stable

# Install fonts matching development environment
RUN apt-get update && apt-get install -y \
    fonts-roboto \
    fonts-noto \
    fonts-liberation

WORKDIR /app
COPY . .
RUN flutter pub get
```

2. **ALWAYS call `loadAppFonts()` in test config** (already shown in 5.1)

3. **Set tolerance threshold in CI configuration**
```dart
// test/flutter_test_config.dart
Future<void> testExecutable(FutureOr<void> Function() testMain) async {
  await loadAppFonts();
  
  // ✅ Set tolerance for CI environment
  if (Platform.environment['CI'] == 'true') {
    final testUri = Uri.parse(Platform.script.toString());
    goldenFileComparator = LocalFileComparatorWithThreshold(testUri, 0.02);
  }
  
  return testMain();
}
```

4. **Use `pumpAndSettle()` before golden capture**
```dart
testGoldens('Animated widget golden', (tester) async {
  await tester.pumpWidget(MaterialApp(home: AnimatedWidget()));
  await tester.pumpAndSettle(); // ✅ Wait for all animations
  await screenMatchesGolden(tester, 'animated_widget');
});
```

5. **Regenerate goldens on CI platform ONLY**
```yaml
# .github/workflows/test.yml
- name: Generate Golden Baselines
  run: flutter test --update-goldens --tags golden
  if: github.event_name == 'workflow_dispatch' # Manual trigger only
```

**❌ Anti-pattern: Mixing Platform Goldens**
```bash
# ❌ Developer on macOS updates goldens
flutter test --update-goldens

# ❌ CI on Linux detects pixel diffs
# FAILURE: Expected pixel-perfect match, got 0.8% difference
```

**✅ Best Practice: Single Platform for Goldens**
```bash
# ✅ Use Docker locally to match CI environment
docker run --rm -v $(pwd):/app -w /app cirrusci/flutter:stable \
  flutter test --update-goldens --tags golden

# ✅ CI uses same image - no diffs!
```

## Constraints
* Prefer `Mocktail` when rapid iteration speed is the top priority and manual fallback configuration is acceptable. Use `Mockito` when type-safety and automatic generation are preferred.
* Use `patrol_finders` (`patrolWidgetTest`) for widget tests that need cleaner finder syntax — no native setup required, runs with `flutter test`.
* Use full `patrol` package (with `patrol test` CLI) **only** when the test flow requires native OS interaction (permissions, push notifications, WebViews, system dialogs).
* **NEVER use `$.native.*`** — deprecated since Patrol 4.0. Use `$.platform.mobile.*`, `$.platform.android.*`, `$.platform.iOS.*`, or `$.platform.web.*` instead.
* **ALWAYS use `loadAppFonts()` in `test/flutter_test_config.dart`** when implementing golden tests to prevent font rendering inconsistencies.
* **Set explicit `surfaceSize`** in golden tests to ensure cross-platform pixel-perfect consistency.
* **Use tolerance thresholds (0.01-0.05)** for golden comparators to handle minor antialiasing differences across platforms.
