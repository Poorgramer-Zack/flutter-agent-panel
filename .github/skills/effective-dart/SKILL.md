---
name: "applying-effective-dart"
description: "Dart 3 modern features (through 3.10) and Effective Dart best practices including records, patterns, sealed classes, dot shorthands, null-aware collection elements, and wildcard variables. Use when adopting Dart 3.7+ syntax, using dot shorthands for enums or static members, using null-aware elements in lists or maps, using wildcard _ parameters, improving code quality, or reviewing Dart code conventions."
metadata:
  last_modified: "2026-04-01 17:00:00 (GMT+8)"
---

# Dart 3 Latest Features and Effective Dart Best Practices Guide

## Goal
Covers Dart 3 features (through 3.10) and Effective Dart practices for writing safe, maintainable code.

## Instructions

### 1. Dart 3 Features

#### 1.1 Records
Records aggregate multiple typed values without requiring a dedicated class.
* **Best Practices**:
  * **Multiple Return Values**: Prefer Records over throwaway wrapper classes.
  * **Named Fields**: Use named fields when returning more than two values or when semantics are ambiguous.
  ```dart
  // ✅ Recommended Usage
  ({double lat, double lon}) getLocation(String city) {
    return (lat: 25.0330, lon: 121.5654);
  }
  ```

#### 1.2 Patterns and Destructuring
Patterns match and destructure data from Records, Lists, Maps, and custom objects.
* **Best Practices**:
  * **Destructuring on declaration**: Extract values from Records or Lists directly at assignment.
  * **Replace complex if-statements**: Use `if-case` to validate shape and extract variables simultaneously.
  ```dart
  // ✅ Recommended Usage
  final json = {'user': ['Zack', 25]};
  if (json case {'user': [String name, int age]}) {
    print('User $name is $age years old.');
  }
  ```

#### 1.3 `switch` Expressions and Exhaustive Checking
`switch` expressions return values and provide compile-time exhaustiveness for sealed classes and enums.
* **Best Practices**:
  * **Use as expression**: Assign diverging values concisely with switch expressions.
  * **Omit `default`**: On sealed/enum types, omit `default` so the compiler catches unhandled cases when new variants are added.

#### 1.4 Class Modifiers
Dart 3 class modifiers control inheritance and implementation boundaries.
* **Best Practices**:
  * **`sealed`**: For Algebraic Data Types (ADTs). Enables exhaustive switch checking. Ideal for State or Result/Error types.
  * **`interface`**: Allows `implements` only; prevents `extends`.
  * **`final`**: Prevents both `extends` and `implements` from external libraries.

#### 1.5 Wildcard Variables — Dart 3.7
Local variables and parameters named `_` are now **non-binding** — they can be declared multiple times without name collision, and their value cannot be read.

```dart
// Multiple _ params in constructors — no collision
Foo(_, this._, super._) {}

// Discard unused callback args cleanly
list.where((_) => true);

// Multiple discards in the same scope
var _ = expensiveCall1();
var _ = expensiveCall2(); // OK — no shadowing error
```

Top-level names, field names, and class names called `_` are unchanged — this only applies to local variables and parameters.

#### 1.6 Null-Aware Collection Elements — Dart 3.8
Prefix a nullable expression with `?` inside a collection literal to **omit it when null**. Works in lists, sets, and maps.

```dart
String? lunch = isTuesday ? 'tacos' : null;
int? count = hasItems ? items.length : null;

// ❌ Before 3.8
var old = [if (lunch != null) lunch, if (count != null) count];

// ✅ Dart 3.8+
var menu = [?lunch, ?count]; // nulls are silently excluded

// Works in maps too
final headers = <String, String>{
  'Content-Type': 'application/json',
  if (token != null) 'Authorization': 'Bearer $token', // old style
  ?'X-Custom': customHeader, // ✅ null-aware
};
```

#### 1.7 Dot Shorthands — Dart 3.10
Omit the type name when accessing a **static member or constructor** in a context where the expected type is already known.

```dart
// ❌ Before 3.10
Color color = Color.blue;
Widget w = Padding(padding: EdgeInsets.all(16), child: ...);
crossAxisAlignment: CrossAxisAlignment.start,

// ✅ Dart 3.10+
Color color = .blue;
Widget w = Padding(padding: .all(16), child: ...);  // EdgeInsets inferred
crossAxisAlignment: .start,
mainAxisSize: .min,

// Works in switch cases
switch (color) {
  case .blue: print('blue');
  case .red:  print('red');
}
```

Dot shorthands work for: `enum` values, `static` fields, `static` methods, and named constructors — anywhere the context type is known.

### 2. Effective Dart: Style
Consistent style enables collaboration.

* **Formatting**: Use `dart format` and enable `core` or `recommended` lints in `analysis_options.yaml`.
* **Naming Conventions**:
  * **Classes, Enums, Typedefs, Type parameters**: `UpperCamelCase`
  * **Libraries, Packages, Directories, Files**: `lowercase_with_underscores`
  * **Variables, Parameters, Functions, Methods**: `lowerCamelCase`
  * **Constants**: `lowerCamelCase` (e.g., `const defaultTimeout = 1000;`) — not SCREAMING_CAPS.

### 3. Effective Dart: Usage

#### 3.1 Variables and Types
* **Type Inference**: Use `var`/`final` for local variables; omit type annotations where the compiler infers correctly.
* **Collections**: Initialize with literals (`var list = [];`). Use spread operators and collection `if`/`for` instead of `.add()` calls.
* **`late`**: Use only when initialization must be deferred. Never access before assignment.

#### 3.2 Functions and Async
* **Arrow Functions**: Use `=>` for single-expression functions.
* **Named Parameters**: Prefer named parameters for multiple optional arguments; apply `required` as needed.
* **Async**: Prefer `async`/`await` over `.then()`/`.catchError()` chains.

#### 3.3 Null Safety
* **Minimize nullables**: Design variables as non-nullable where possible.
* **Avoid `!`**: Use type promotion (`if (value != null)`) or pattern matching instead of forced unwrapping.
* **Null-aware collection elements**: Prefer `[?nullable]` over `[if (x != null) x]` for cleaner collection literals (Dart 3.8+).

#### 3.4 Async Annotations — Dart 3.9
* **`@awaitNotRequired`**: Annotate a `Future`-returning method to suppress `unawaited_futures` lint at the call site when intentionally fire-and-forget.

### 4. Effective Dart: API Design
* **Minimal exposure**: Only expose members that require external access. Prefix private members with `_`.
* **Getters/Setters**: Use `get` for property-like reads with no side effects or expensive computation. Don't add setters for every property — prefer `final` constructor initialization.
* **Constructors**: Use named constructors (e.g., `User.fromJson`) and redirecting constructors for semantics. Provide `const` constructors wherever possible.

### 5. Recommended Lints (Dart 3.9+)

| Lint | Purpose |
|---|---|
| `switch_on_type` | Prefer `switch` over `is`-chain `if` for type checks |
| `unnecessary_unawaited` | Flag `unawaited()` calls where the future is already non-async |

## Constraints
* Use `Records` for composite returns instead of one-off data classes.
* Omit type annotations where the compiler can infer the type safely.
* Use dot shorthands (`.value`) when the context type is unambiguous — reduces enum and static verbosity.
* Use `?element` in collection literals instead of `if (x != null) x` — cleaner and more idiomatic (Dart 3.8+).
