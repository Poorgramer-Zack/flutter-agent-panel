---
name: "implementing-responsive-layouts"
description: "Implements responsive and adaptive layouts in Flutter using flutter_screenutil v5.9.x (pixel-perfect scaling from Figma), sizer v3.1.x (percentage-based viewport), and responsive_framework v1.1.x (breakpoint-driven structure changes). Use this skill when building apps for multiple screen sizes (phones, tablets, foldables, desktop, web), fixing layout overflow or sizing issues, implementing breakpoint-based adaptive UI, creating pixel-perfect designs from Figma specs, handling orientation changes, supporting Flutter web or desktop responsive layouts, or any time the words 'responsive', 'screenutil', 'sizer', 'breakpoint', 'adaptive layout', or 'screen size' appear in the request. Includes a decision matrix for selecting the correct package per scenario."
metadata:
  last_modified: "2026-04-01 14:35:00 (GMT+8)"
---

# Flutter Responsive Implementations (v5.9.x / v3.1.x / v1.1.x)

## Goal
Implement layout adaptability and responsive UI across varying device screens. This skill provides a decision matrix to choose the right responsiveness package and strict instructions on how to use `flutter_screenutil`, `sizer`, and `responsive_framework` natively.

## Instructions

### 1. Package Decision Matrix
Before writing layout code, strictly evaluate the project's target platforms and design requirements to select the appropriate library:

*   **Scenario A (Strict Mobile UI Replication):** The project targets **only** iOS/Android, and the designers have provided a strict Figma draft (e.g., 375x812) that must look *exactly* proportional on all phones (mini to pro max).
    *   **Decision:** Use `flutter_screenutil`. (Math-based density scaling)
*   **Scenario B (Percentage-based Mobile UI):** The project targets mobile, but layout is dictated by percentage of screen real estate rather than fixed Figma pixels.
    *   **Decision:** Use `sizer`. (Percentage-based viewport scaling)
*   **Scenario C (True Cross-Platform / Web / Desktop):** The project spans Mobile, Tablet, Web, and Desktop. UI components must fundamentally change their structure (e.g. BottomNav -> SideRail) based on available space, rather than just zooming in.
    *   **Decision:** Use `responsive_framework`. (Breakpoint-based rendering)

---

### 2. Implementing `flutter_screenutil`
Best for mathematically scaling a fixed design draft across different physical device sizes.

**Setup (`main.dart`):**
Wrap the `MaterialApp` in `ScreenUtilInit` and provide the `designSize` from the design specs (Figma/AdobeXD).

```dart
Widget build(BuildContext context) {
  return ScreenUtilInit(
    designSize: const Size(375, 812), // The Figma draft size
    minTextAdapt: true,
    splitScreenMode: true,
    builder: (context, child) {
      return MaterialApp(
        // ...
      );
    },
  );
}
```

**Usage Constraints:**
*   Width: Use `.w` (e.g., `100.w`)
*   Height: Use `.h` (e.g., `200.h`)
*   Square dimensions / Radius: Use `.r` (e.g., `Radius.circular(16.r)`) or `100.w` for both width and height to prevent stretching. **Do not use `.h` for width.**
*   Fonts: Use `.sp` (e.g., `16.sp`). Note: `sp` changes with system accessibility font sizes.

```dart
Container(
  width: 50.w,
  height: 200.h,
  child: Text('Responsive Text', style: TextStyle(fontSize: 14.sp)),
)
```

---

### 3. Implementing `sizer`
Best for scenarios where defining UI in viewport percentages is easier than absolute dp conversions.

**Setup (`main.dart`):**
Wrap the `MaterialApp` in `Sizer`.

```dart
Widget build(BuildContext context) {
  return Sizer(
    builder: (context, orientation, screenType) {
      return MaterialApp(
        // ...
      );
    },
  );
}
```

**Usage Constraints:**
*   Percentages: Use `.w` and `.h` as raw screen percentages. `100.w` is exactly the full width of the current screen.
*   Fonts/Density: Use `.dp` (Density Pixel) instead of `.sp` if you want text to scale strictly by physical display density rather than user accessibility settings.
*   **Crucial Difference:** `50.w` in `sizer` means 50% of screen. `50.w` in `ScreenUtil` means 50 logical pixels relative to the 375 Figma draft. Be acutely aware of which package is active.

```dart
Container(
  width: 20.w, // Takes exactly 20% of screen width
  height: 30.5.h, // Takes exactly 30.5% of screen height
  child: Text('Sizer Text', style: TextStyle(fontSize: 15.dp)),
)
```

---

### 4. Implementing `responsive_framework`
Best for Tablet/Desktop/Web where components shift layout structures at specific breakpoints. 

**Setup (`main.dart`):**
Wrap the router or `MaterialApp` builder with `ResponsiveBreakpoints.builder()`. Do NOT use the deprecated `ResponsiveWrapper`.

```dart
return MaterialApp(
  builder: (context, child) => ResponsiveBreakpoints.builder(
    child: child!,
    breakpoints: [
      const Breakpoint(start: 0, end: 450, name: MOBILE),
      const Breakpoint(start: 451, end: 800, name: TABLET),
      const Breakpoint(start: 801, end: 1920, name: DESKTOP),
      // Custom breakpoint labels can be added here
      const Breakpoint(start: 1921, end: double.infinity, name: '4K'),
    ],
  ),
);
```

**Usage Constraints (Conditionals):**
Use `ResponsiveBreakpoints.of(context)` to query the current environment to determine layout structures.

```dart
// Check size logic to conditionally render navigation styles
bool showSideNav = ResponsiveBreakpoints.of(context).largerThan(MOBILE);

if (showSideNav) {
  // Render Desktop layout
} else {
  // Render Mobile layout
}
```

**Usage Constraints (Value Scaling):**
If you need specific padding or constraints to change per breakpoint, use `ResponsiveValue`.

```dart
double padding = ResponsiveValue(
    context, 
    defaultValue: 10.0,
    conditionalValues: [
        Condition.equals(name: MOBILE, value: 10.0),
        Condition.largerThan(name: MOBILE, value: 30.0),
    ]
).value;
```

## Constraints
1.  **Never Mix Scaling Libraries:** Do not use `sizer` and `flutter_screenutil` in the same project. They conflict conceptually.
2.  **Web/Desktop Prohibition:** NEVER use `flutter_screenutil` to blindly scale a mobile UI up to a Desktop screen. A 375-width design scaled to a 1920-width monitor results in cartoonishly massive text and buttons. Use `responsive_framework` for Desktop/Web.
3.  **No `MediaQuery` Boilerplate:** When utilizing these libraries, do not write raw `MediaQuery.of(context).size.width * 0.5`. Use the library's native syntax (e.g., `50.w` in `sizer`) to keep code declarative and completely eliminate boilerplate.
