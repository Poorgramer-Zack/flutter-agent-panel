---
name: "implementing-flutter-bloc"
description: "Implements BLoC (v9.x) event-driven state management for Flutter with unidirectional data flow using the flutter_bloc package. Use when implementing Bloc/Cubit classes, mapping Events to States, consuming state with BlocBuilder/BlocListener/BlocConsumer, applying event transformers (restartable/droppable/sequential) for concurrency control, setting up BlocProvider/MultiBlocProvider DI, integrating Freezed for sealed union types and exhaustive pattern matching, or testing with blocTest. Ideal for enterprise apps requiring business logic separation, precise concurrency control (search debouncing, form deduplication), or event replay debugging."
metadata:
  last_modified: "2026-04-01 14:35:00 (GMT+8)"
---

# BLoC State Management (v9.x)

## Goal

Strict separation of UI from business logic via event-driven unidirectional data flow.
Choose **Cubit** (lightweight, no events) for simple state, or full **Bloc** (events + transformers) for complex flows.

## Process

### 1. Install Dependencies

```yaml
dependencies:
  flutter_bloc: ^9.1.1
  equatable: ^2.0.5
  bloc_concurrency: ^0.3.0  # For event transformers

dev_dependencies:
  bloc_test: ^9.1.0
```

### 2. Choose Pattern

| Pattern | When to Use |
|---------|-------------|
| **Cubit** | Simple state (toggle, counter, form validation) |
| **Bloc** | Event tracking, transformers, complex flows (login, search debounce) |

### 3A. Implement Cubit

```dart
class CounterState extends Equatable {
  final int value;
  const CounterState({required this.value});
  @override
  List<Object?> get props => [value];
}

class CounterCubit extends Cubit<CounterState> {
  CounterCubit() : super(const CounterState(value: 0));
  void increment() => emit(CounterState(value: state.value + 1));
  void decrement() => emit(CounterState(value: state.value - 1));
}
```

### 3B. Implement Bloc

```dart
abstract class CounterEvent extends Equatable {
  const CounterEvent();
  @override
  List<Object?> get props => [];
}
class CounterIncremented extends CounterEvent {}
class CounterDecremented extends CounterEvent {}

class CounterBloc extends Bloc<CounterEvent, CounterState> {
  CounterBloc() : super(const CounterState(value: 0)) {
    on<CounterIncremented>((e, emit) => emit(CounterState(value: state.value + 1)));
    on<CounterDecremented>((e, emit) => emit(CounterState(value: state.value - 1)));
  }
}
```

### 4. Provide to Widget Tree

```dart
MultiBlocProvider(
  providers: [
    BlocProvider(create: (context) => CounterCubit()),
    BlocProvider(create: (context) => AuthBloc()),
  ],
  child: MyApp(),
)
```

### 5. Consume State

| Widget | Use For |
|--------|---------|
| `BlocBuilder` | Rebuild UI on state changes |
| `BlocListener` | Side effects only (navigation, snackbars) |
| `BlocConsumer` | Rebuild + side effects together |

```dart
// Trigger events
context.read<CounterBloc>().add(CounterIncremented());
context.read<CounterCubit>().increment(); // Cubit: direct method call
```

See [BLoC Best Practices](./references/bloc.md) for full widget examples with `buildWhen`/`listenWhen`.

### 6. Event Transformers

| Transformer | Behavior | Use Case |
|-------------|----------|----------|
| `concurrent` (default) | All events in parallel | Independent API calls |
| `sequential` | One at a time, queued | Multi-step transactions |
| `restartable` | Cancel previous, start fresh | Search bar, autocomplete |
| `droppable` | Ignore while processing | Submit button deduplication |

```dart
import 'package:bloc_concurrency/bloc_concurrency.dart';

on<SearchQueryChanged>(_onQueryChanged, transformer: restartable());
on<FormSubmitted>(_onFormSubmitted, transformer: droppable());
```

See [BLoC Best Practices](./references/bloc.md) for custom debounce transformer.

### 7. Freezed Integration

Use Freezed for sealed union states with exhaustive `when`/`map` pattern matching.
See [BLoC Best Practices](./references/bloc.md) for full Freezed setup and examples.

### 8. Common Errors

See [Error Handling & Common Pitfalls](./references/errors.md) for solutions to:
- Emitting after Bloc close (`emit.isDone` guard)
- Mutable state causing silent UI bugs (spread into new instances)
- Scattered `try/catch` — use `onError` + `addError` instead

---

## Reference Documentation

- [BLoC Best Practices (v9.x)](./references/bloc.md) — Transformers, Freezed, testing with blocTest, hydrated/replay packages
- [Error Handling & Common Pitfalls](./references/errors.md) — Three common errors with before/after fixes
- [State Management Overview](./references/state-management-overview.md) — Cubit vs Provider/MVVM comparison

---

## Constraints

* **Immutable States**: All states MUST be immutable. Use `Equatable` or `Freezed`.
* **No UI in Bloc**: Blocs/Cubits MUST NOT import Flutter widgets or `BuildContext`.
* **Single Responsibility**: Each Bloc handles one feature domain.
* **Event Naming**: Past tense nouns (`UserLoggedIn`, not `UserLogin`).
* **State Naming**: Nouns or adjectives (`AuthSuccess`, not `AuthSucceeded`).
* **Always Close**: Blocs/Cubits MUST be closed on dispose to prevent memory leaks.
