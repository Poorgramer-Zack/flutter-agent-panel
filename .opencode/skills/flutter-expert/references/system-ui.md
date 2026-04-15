---
name: "flutter-system-ui"
description: "Expert-level guide for controlling system UI using SystemChrome, including system bar styling, immersive modes, and device-specific edge case handling."
metadata:
  last_modified: "2026-03-12 11:18:17 (GMT+8)"
---

# System UI Control (SystemChrome)

## Goal
Master the implementation of advanced System UI controls in Flutter. This includes configuring status and navigation bar aesthetics, implementing full-screen "immersive" modes, and resolving complex hardware-specific display issues such as notch padding and background segments on specific Android builds (OPPO, Xiaomi, etc.).

## Instructions

### 1. Basic System UI Style
Control the appearance of the status bar and navigation bar using `SystemUiOverlayStyle`.

```dart
import 'package:flutter/services.dart';

SystemChrome.setSystemUIOverlayStyle(
  const SystemUiOverlayStyle(
    statusBarColor: Colors.transparent, // iOS: none, Android: transparent
    statusBarIconBrightness: Brightness.dark, // Android: dark icons
    statusBarBrightness: Brightness.light,    // iOS: dark icons
    systemNavigationBarColor: Colors.white,   // Android only
    systemNavigationBarIconBrightness: Brightness.dark,
  ),
);
```

### 2. Immersive Modes & System Overlays
Use `setEnabledSystemUIMode` to control full-screen experiences.

- **Immersive Sticky (`immersiveSticky`)**: Hides status & navigation bars. Users can swipe from edges to show them temporarily (translucent), then they auto-hide. Recommended for games/video players.
- **Manual (`manual`)**: Manually specify which overlays are visible (status bar vs navigation bar).

```dart
// Enter Sticky Immersive Mode
SystemChrome.setEnabledSystemUIMode(SystemUiMode.immersiveSticky);

// Restore default mode
SystemChrome.setEnabledSystemUIMode(SystemUiMode.edgeToEdge);

// Manual control
SystemChrome.setEnabledSystemUIMode(SystemUiMode.manual, overlays: [
  SystemUiOverlay.top,
]);
```

### 3. Edge Case Handling

#### A. Persistent Black Bar (OPPO/Xiaomi/Vivo)
Some Android vendors retain a black background for the status bar area even in immersive mode.
*   **Recommendation**: Instead of full contraction of the status bar, keep it visible but set it to `Colors.transparent`. Avoid frequent `SystemUiMode` toggling to prevent flickering/re-layout during app usage.

#### B. AppBar Clickability in Immersive Mode
On Android, the system reserves about 24dp of the top edge to detect the pull-down gesture.
*   **Avoidance**: Do NOT place critical interactive elements at the absolute top 24dp when in immersive mode.

#### C. Keyboard Interaction Issues
The Android system often resets UI overlays when a keyboard pops up.
*   **Fix**: Restore UI settings after the keyboard closes.
```dart
WidgetsBinding.instance.addPostFrameCallback((_) {
    // Restore or re-apply system UI settings
    SystemChrome.restoreSystemUIOverlays();
});
```

## Constraints
*   **Main Method Initialization**: Always call `WidgetsFlutterBinding.ensureInitialized()` before attempting `SystemChrome` calls in `main()`.
*   **Real-Device Testing**: System UI behavior varies significantly by manufacturer. Testing on actual OPPO, Xiaomi, and Samsung devices is mandatory for production.
*   **Redundant Layouts**: Excessive switching of `SystemUiMode` triggers expensive UI repaints. Stick to one mode per screen when possible.
