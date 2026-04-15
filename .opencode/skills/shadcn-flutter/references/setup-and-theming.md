---
name: "shadcn-flutter-setup-and-theming"
description: "Core setup and customization guide for Shadcn UI Flutter, including app initialization, responsive design, and advanced theming."
metadata:
  last_modified: "2026-03-12 11:18:17 (GMT+8)"
---

# Shadcn UI Setup & Theming Guide

## App Setup

### Pure Shadcn (No Material/Cupertino)
```dart
import 'package:shadcn_ui/shadcn_ui.dart';

void main() => runApp(ShadApp(child: HomePage()));
```

### Shadcn + Material Integration
```dart
return ShadApp.custom(
  appBuilder: (context) => MaterialApp(
    theme: Theme.of(context),
    builder: (context, child) => ShadAppBuilder(child: child!),
  ),
);
```

## Theme Customization

### Color Schemes
Supported: `ShadBlue`, `ShadGray`, `ShadGreen`, `ShadNeutral`, `ShadOrange`, `ShadRed`, `ShadRose`, `ShadSlate`, `ShadStone`, `ShadViolet`, `ShadYellow`, `ShadZinc`.

```dart
ShadApp(
  darkTheme: ShadThemeData(
    brightness: Brightness.dark,
    colorScheme: const ShadSlateColorScheme.dark(),
  ),
  child: HomePage(),
)
```

### Custom Colors Extension
```dart
extension CustomColorExtension on ShadColorScheme {
  Color get myCustomColor => custom['myCustomColor']!;
}
```

## Typography
Access styles via `ShadTheme.of(context).textTheme`:
- `h1Large`, `h1`, `h2`, `h3`, `h4`
- `p`, `blockquote`, `table`, `list`
- `lead`, `large`, `small`, `muted`

## Responsive Design
Breakpoints: `tn` (0), `sm` (640), `md` (768), `lg` (1024), `xl` (1280), `xxl` (1536).

```dart
final sm = context.breakpoint >= ShadTheme.of(context).breakpoints.sm;

ShadResponsiveBuilder(
  builder: (context, breakpoint) => switch (breakpoint) {
    ShadBreakpointTN() => const Text('Tiny'),
    ShadBreakpointSM() => const Text('Small'),
    _ => const Text('Large'),
  },
);
```

## Form Validation
```dart
ShadForm(
  key: formKey,
  child: Column(children: [
    ShadInputFormField(
      id: 'username',
      validator: (v) => v.length < 2 ? 'Too short' : null,
    ),
    ShadButton(onPressed: () => formKey.currentState!.saveAndValidate()),
  ]),
)
```

## Common Patterns

### Dialog
```dart
showShadDialog(
  context: context,
  builder: (context) => ShadDialog(title: Text('Hi')),
);
```

### Toast
```dart
ShadToaster.of(context).show(ShadToast(title: Text('Success')));
```
