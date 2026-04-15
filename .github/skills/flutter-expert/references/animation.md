---
name: "flutter-animation"
description: "Add animated effects to your Flutter app"
metadata:
  last_modified: "2026-03-12 11:18:17 (GMT+8)"

---
# Flutter Animations Implementation

## Goal
Implements and manages Flutter animations, selecting the appropriate animation strategy (implicit, explicit, tween, physics, hero, or staggered) based on UI requirements. Assumes a working Flutter environment, stateful/stateless widget competence, and a standard widget tree structure.

## Instructions

### 1. Determine Animation Strategy (Decision Logic)
Evaluate the UI requirement using the following decision tree to select the correct animation approach:
1. **Is the animation a simple property change (color, size, alignment) on a single widget?**
   * YES: Use **Implicit Animations** (e.g., `AnimatedContainer`).
   * NO: Proceed to 2.
2. **Does the animation model real-world movement (springs, gravity, velocity)?**
   * YES: Use **Physics-based animation** (`SpringSimulation`, `animateWith`).
   * NO: Proceed to 3.
3. **Does the animation involve a widget flying between two different screens/routes?**
   * YES: Use **Hero Animations** (`Hero` widget).
   * NO: Proceed to 4.
4. **Is it a continuous, infinite loop (e.g., loading spinner, pulse)?**
   * YES: Use **RepeatingAnimationBuilder** (Flutter 3.41+).
   * NO: Proceed to 5.
5. **Does the animation involve multiple sequential or overlapping movements?**
   * YES: Use **Staggered Animations** (Single `AnimationController` with multiple `Tween`s and `Interval` curves).
   * NO: Use **Standard Explicit Animations** (`AnimationController`, `Tween`, `AnimatedBuilder` / `AnimatedWidget`).

**STOP AND ASK THE USER:** If the requirement is ambiguous, pause and ask the user to clarify the desired visual effect before writing implementation code.

### 2. Implement Implicit Animations
For simple transitions between values, use implicit animation widgets. Do not manually manage state or controllers.

```dart
AnimatedContainer(
  duration: const Duration(milliseconds: 500),
  curve: Curves.bounceIn,
  width: _isExpanded ? 200.0 : 100.0,
  height: _isExpanded ? 200.0 : 100.0,
  decoration: BoxDecoration(
    color: _isExpanded ? Colors.green : Colors.blue,
    borderRadius: BorderRadius.circular(_isExpanded ? 50.0 : 8.0),
  ),
  child: const FlutterLogo(),
)
```

### 3. Implement Continuous/Repeating Animations (Flutter 3.41+)
For simple, infinite loop animations (like throbbing buttons, loading spinners, or continuous rotations) where you previously needed a full `AnimationController`, use `RepeatingAnimationBuilder`. It automatically manages the lifecycle and disposal.

```dart
RepeatingAnimationBuilder<double>(
  // Define how the value transitions (0.0 to 1.0)
  animatable: Tween<double>(begin: 0.8, end: 1.2),
  duration: const Duration(milliseconds: 1000),
  curve: Curves.easeInOut,
  // Use RepeatMode.reverse for a ping-pong effect (e.g., breathing/pulse)
  // Use RepeatMode.restart for continuous identical loops (e.g., a radar sweep)
  repeatMode: RepeatMode.reverse,
  builder: (context, value, child) {
    return Transform.scale(
      scale: value,
      child: child,
    );
  },
  // Pass static widgets to child for performance (avoids rebuilding the icon)
  child: const Icon(Icons.favorite, color: Colors.red, size: 50),
)
```

### 3.1 Material 3 Overscroll Behavior (Flutter 3.41+)

With `ThemeData.useMaterial3` set to `true`, Flutter defaults to using the `StretchingOverscrollIndicator` (a stretchy, iOS-like bounce effect) instead of the legacy Android `GlowingOverscrollIndicator` (the semi-circular glow) when a user scrolls past the boundaries of a `ScrollView` (like `ListView` or `GridView`).

#### Manual Wrapping
For highly custom scrollable areas, you can manually wrap a scroll-generating widget to explicitly control its stretch axis or clip behavior:
```dart
StretchingOverscrollIndicator(
  axisDirection: AxisDirection.down,
  clipBehavior: Clip.hardEdge,
  child: ListView.builder(
    // ...
  ),
)
```

#### Disabling the Stretch Effect
If the bouncy stretch effect interferes with your custom UI, intercept the `OverscrollIndicatorNotification` using a `NotificationListener` to completely disable the indicator:
```dart
NotificationListener<OverscrollIndicatorNotification>(
  onNotification: (OverscrollIndicatorNotification overscroll) {
    // Completely disables the stretch/glow effect
    overscroll.disallowIndicator(); 
    return true; 
  },
  child: ListView.builder(
    // ...
  ),
)
```

### 4. Implement Explicit Animations (Tween-based)
When you need to control the animation (play, pause, reverse), use an `AnimationController` with a `Tween`. Separate the transition rendering from the state using `AnimatedBuilder`.

```dart
class _MyAnimatedWidgetState extends State<MyAnimatedWidget> with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _animation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(seconds: 2),
      vsync: this,
    );
    
    _animation = Tween<double>(begin: 0, end: 300).animate(
      CurvedAnimation(parent: _controller, curve: Curves.easeOut),
    )..addStatusListener((status) {
        if (status == AnimationStatus.completed) {
          _controller.reverse();
        } else if (status == AnimationStatus.dismissed) {
          _controller.forward();
        }
      });
      
    _controller.forward();
  }

  @override
  void dispose() {
    _controller.dispose(); // STRICT REQUIREMENT
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return AnimatedBuilder(
      animation: _animation,
      builder: (context, child) {
        return SizedBox(
          height: _animation.value,
          width: _animation.value,
          child: child,
        );
      },
      child: const FlutterLogo(), // Passed as child for performance
    );
  }
}
```

### 4. Implement Page Route Transitions
To animate transitions between routes, use `PageRouteBuilder` and chain a `CurveTween` with a `Tween<Offset>`.

```dart
Route<void> _createRoute() {
  return PageRouteBuilder(
    pageBuilder: (context, animation, secondaryAnimation) => const DestinationPage(),
    transitionsBuilder: (context, animation, secondaryAnimation, child) {
      const begin = Offset(0.0, 1.0);
      const end = Offset.zero;
      const curve = Curves.ease;

      final tween = Tween(begin: begin, end: end).chain(CurveTween(curve: curve));

      return SlideTransition(
        position: animation.drive(tween),
        child: child,
      );
    },
  );
}
```

### 5. Implement Physics-Based Animations
For realistic motion (e.g., snapping back after a drag), calculate velocity and apply a `SpringSimulation`.

```dart
void _runSpringAnimation(Offset pixelsPerSecond, Size size, Alignment dragAlignment) {
  _animation = _controller.drive(
    AlignmentTween(begin: dragAlignment, end: Alignment.center),
  );

  final unitsPerSecondX = pixelsPerSecond.dx / size.width;
  final unitsPerSecondY = pixelsPerSecond.dy / size.height;
  final unitsPerSecond = Offset(unitsPerSecondX, unitsPerSecondY);
  final unitVelocity = unitsPerSecond.distance;

  const spring = SpringDescription(mass: 1, stiffness: 1, damping: 1);
  final simulation = SpringSimulation(spring, 0, 1, -unitVelocity);

  _controller.animateWith(simulation);
}
```

### 6. Implement Hero Animations (Shared Element)
To fly a widget between routes, wrap the identical widget tree in both routes with a `Hero` widget using the exact same `tag`.

```dart
// Source Route
Hero(
  tag: 'unique-photo-tag',
  child: Image.asset('photo.png', width: 100),
)

// Destination Route
Hero(
  tag: 'unique-photo-tag',
  child: Image.asset('photo.png', width: 300),
)
```

### 8. Implement Staggered Animations
For sequential or overlapping animations, use a single `AnimationController` and define multiple `Tween`s with `Interval` curves.

```dart
class StaggerAnimation extends StatelessWidget {
  StaggerAnimation({super.key, required this.controller}) :
    opacity = Tween<double>(begin: 0.0, end: 1.0).animate(
      CurvedAnimation(
        parent: controller,
        curve: const Interval(0.0, 0.100, curve: Curves.ease),
      ),
    ),
    width = Tween<double>(begin: 50.0, end: 150.0).animate(
      CurvedAnimation(
        parent: controller,
        curve: const Interval(0.125, 0.250, curve: Curves.ease),
      ),
    );

  final AnimationController controller;
  final Animation<double> opacity;
  final Animation<double> width;

  @override
  Widget build(BuildContext context) {
    return AnimatedBuilder(
      animation: controller,
      builder: (context, child) {
        return Opacity(
          opacity: opacity.value,
          child: Container(width: width.value, height: 50, color: Colors.blue),
        );
      },
    );
  }
}
```

### 9. Validate-and-Fix Loop
After generating animation code, verify the following:
1. Does the `State` class use `SingleTickerProviderStateMixin` (or `TickerProviderStateMixin` for multiple controllers)?
2. Is `_controller.dispose()` explicitly called in the `dispose()` method?
3. If using `AnimatedBuilder`, is the static widget passed to the `child` parameter rather than rebuilt inside the `builder` function?
If any of these are missing, fix the code immediately before presenting it to the user.

## Constraints
* **Strict Disposal:** You MUST include `dispose()` methods for all `AnimationController` instances to prevent memory leaks.
* **No URLs:** Do not include external links or URLs in the output or comments.
* **Immutability:** Treat `Tween` and `Curve` classes as stateless and immutable. Do not attempt to mutate them after instantiation.
* **Performance:** Always use `AnimatedBuilder` or `AnimatedWidget` instead of calling `setState()` inside a controller's `addListener` when building complex widget trees.
* **Hero Tags:** Hero tags MUST be identical and unique per route transition. Do not use generic tags like `'image'` if multiple heroes exist.
