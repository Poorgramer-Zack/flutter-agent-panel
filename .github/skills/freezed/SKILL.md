---
name: "generating-freezed-models"
description: "Generates immutable Dart data classes, sealed union types, and deep copyWith using Freezed (v3.2.x) with optional json_serializable integration. Use when creating DTOs, API response models, app state classes, or any model requiring immutability, pattern matching, or JSON serialization. Activates on: @freezed annotation, freezed_annotation import, immutable data class request, copyWith deep nesting, union/sealed types for loading/data/error states, generic response wrappers, or build_runner code generation for models."
metadata:
  last_modified: "2026-04-01 14:35:00 (GMT+8)"
---

# Freezed Guide (v3.2.x)

## Goal
Generate immutable data classes, union types, and deep `copyWith` using Freezed v3.2.x. Complements Dart 3 `sealed` classes by adding `copyWith`, `==`/`hashCode`, and JSON serialization via `json_serializable`.

## Instructions

### 1. Dependencies and Environment Setup
Because Freezed relies heavily on code generation, ensure your `pubspec.yaml` is fully configured:

```yaml
dependencies:
  freezed_annotation: ^3.2.5
  json_annotation: ^4.9.0 # If mutual conversion with JSON is required

dev_dependencies:
  build_runner: ^2.4.0
  freezed: ^3.2.5
  json_serializable: ^6.8.0 # If mutual conversion with JSON is required
```

### 2. Defining an Immutable Data Class

```dart
import 'package:freezed_annotation/freezed_annotation.dart';

// You MUST declare part files; the name must exactly match the current file
part 'user_model.freezed.dart';
part 'user_model.g.dart'; // If utilizing JSON serialization

// Freezed 3.x: prefer `sealed` modifier for exhaustive pattern matching
@freezed
sealed class UserModel with _$UserModel {
  const factory UserModel({
    required String id,
    required String name,
    @Default(18) int age,
    @JsonKey(name: 'is_active') @Default(true) bool isActive,
  }) = _UserModel;

  // Required for JSON serialization
  factory UserModel.fromJson(Map<String, Object?> json) => 
      _$UserModelFromJson(json);
}
```

> After every model change, run `dart run build_runner build -d`.

### 3. Custom Getters and Methods
To add computed properties or methods, add a private parameterless constructor.

```dart
@freezed
abstract class Product with _$Product {
  // private constructor required to add custom methods
  const Product._();
  
  const factory Product({
    required double price,
    required double discountRate,
  }) = _Product;

  // Custom Getter
  double get discountedPrice => price * (1 - discountRate);

  // Custom Method
  bool isFree() => discountedPrice == 0;
}
```

### 4. Generic Types Support

```dart
@freezed
@JsonSerializable(genericArgumentFactories: true) // required for generic JSON parsing
sealed class PaginatedResponse<T> with _$PaginatedResponse<T> {
  const factory PaginatedResponse({
    required int currentPage,
    required int totalPages,
    required List<T> data,
  }) = _PaginatedResponse<T>;

  // fromJson for generics requires a conversion function parameter
  factory PaginatedResponse.fromJson(
    Map<String, dynamic> json,
    T Function(Object? json) fromJsonT,
  ) => _$PaginatedResponseFromJson(json, fromJsonT);
}
```

### 5. Union Types and Dart 3 Pattern Matching
Use Freezed union types for sealed state modeling with automatic subtypes.

```dart
@freezed
sealed class ApiResponse with _$ApiResponse {
  const factory ApiResponse.loading() = ApiResponseLoading;
  const factory ApiResponse.data(List<String> items) = ApiResponseData;
  const factory ApiResponse.error(String message) = ApiResponseError;
}
```

**Use Dart 3 native pattern matching with Freezed union types:**

```dart
Widget buildStatus(ApiResponse response) {
  return switch (response) {
    ApiResponseLoading() => const CircularProgressIndicator(),
    ApiResponseData(:final items) => Text('Item Count: ${items.length}'),
    ApiResponseError(:final message) => Text('Error: $message'),
  };
}
```
> Prefer native `switch` expressions over `.when()` / `.map()` — better compiler exhaustiveness checking.

### 6. Deep Copy (`copyWith`) vs `@unfreezed`

#### 6.1 Automatic Deep CopyWith
When a Freezed class contains another Freezed class, deep `copyWith` chaining is generated automatically.

```dart
// Instead of: user.copyWith(address: user.address.copyWith(city: 'Taipei'))
final newUser = user.copyWith.address(city: 'Taipei');
```

#### 6.2 When to use `@unfreezed`
- **Default:** Always use `@freezed` (immutable).
- `@unfreezed` allows mutable fields (`user.age = 20`) but skips `==`/`hashCode` generation.
- **Anti-pattern** in Riverpod/BLoC — breaks unidirectional data flow. Only consider for large mutable forms where deep-copying causes measurable performance issues.

### 7. JSON Serialization with `@JsonKey`
Use `@JsonKey` to customize field serialization:

- **Ignore a field:** `@JsonKey(includeFromJson: false, includeToJson: false)`
- **Custom converter** for non-standard formats (e.g., date strings):

```dart
class MyDateConverter implements JsonConverter<DateTime, String> {
  const MyDateConverter();
  @override
  DateTime fromJson(String json) => DateTime.parse(json.replaceAll('/', '-'));
  @override
  String toJson(DateTime object) => object.toIso8601String();
}

@freezed
sealed class Event with _$Event {
  const factory Event({
    @MyDateConverter() required DateTime eventDate,
  }) = _Event;
  // ... fromJson ...
}
```

### 8. Summary
1. Use Freezed for all DTOs and app state classes.
2. Add a private constructor (`const ClassName._()`) to enable custom getters/methods.
3. Use `switch` expressions instead of `.when()` / `.map()`.
4. Default to `@freezed` (immutable); avoid `@unfreezed` in app state.

## Constraints
* Generics with JSON: always add `@JsonSerializable(genericArgumentFactories: true)`.
* Avoid `@unfreezed` in app state; use only for mutable form models where deep-copy performance is proven problematic.
