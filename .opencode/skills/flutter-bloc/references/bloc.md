---
name: "bloc-best-practices"
description: "Best practices and guidelines for BLoC (Business Logic Component) Latest Version Best Practices Guide (v9.1.x). Use this skill when you need to write or review code related to BLoC (Business Logic Component) Latest Version Best Practices Guide (v9.1.x)."
metadata:
  last_modified: "2026-03-12 11:18:17 (GMT+8)"
---

# BLoC (Business Logic Component) Latest Version Best Practices Guide (v9.1.x)

## Goal
Implements BLoC as the state management solution in Flutter to strongly decouple the UI presentation layer from the business logic layer. As of now, the latest version on pub.dev is roughly **9.1.1**.

This document compiles the best practices based on the latest `flutter_bloc` ecosystem.

## Instructions

### 1. Core Concept: Unidirectional Data Flow of Event and State
Any operation should be abstracted as a unique event (**Event**) and map to the output of a new state (**State**).
Within the BLoC package, developers have two implementation choices:
1. **Bloc**: Complete event and state mapping. Suitable for high-complexity logic requiring tracking of all user behaviors and executing side effects (e.g., login authentication flows, shopping cart states).
2. **Cubit**: A lightweight Bloc. No need to define Events; states are modified and `emit`ted directly via calling methods (Functions). Suitable for handling simple behaviors (e.g., toggling dark mode, managing simple counters).

> ✅ **Best Practice**: Default to using `Cubit` primarily. Upgrade to a full `Bloc` only when you realize you need to track behavior via event logs, or require specific Event Transformers like `debounce` / `throttle`.

### 2. Dependency Setup
Add dependencies in `pubspec.yaml`:
*   `flutter_bloc`: Flutter integration layer.
*   `equatable`: (Strongly Recommended) Used to simplify Value Equality comparisons for States or Events. If equality comparison is not implemented correctly, `BlocBuilder` might issue unnecessary rebuilds when the state hasn't genuinely changed.

### 3. Practice and Syntax Tutorial: Cubit Syntax (Lightweight, No Events)
Cubit is optimally suited for simple scenarios lacking complex timing dependencies.

```dart
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:equatable/equatable.dart';

// State (It is recommended to always use Immutable classes)
class CounterState extends Equatable {
  final int value;

  const CounterState({required this.value});

  // Equatable requires overriding props, helping the package determine if the state has substantively changed
  @override
  List<Object?> get props => [value];
}

// Cubit
class CounterCubit extends Cubit<CounterState> {
  CounterCubit() : super(const CounterState(value: 0));

  // Directly use functions to trigger state changes
  void increment() {
    emit(CounterState(value: state.value + 1));
  }
}
```

### 4. Practice and Syntax Tutorial: Bloc Syntax (Powerful, Has Events, Supports Transformers)
Since BLoC 8.0, it is strongly advised to discard the legacy `mapEventToState` and adopt the new `on<Event>` API.

```dart
import 'package:flutter_bloc/flutter_bloc.dart';

// 1. Define Events
sealed class AuthEvent {}
class AuthLoginRequested extends AuthEvent {
  final String username;
  final String password;
  AuthLoginRequested(this.username, this.password);
}

// 2. Define States
sealed class AuthState {}
class AuthInitial extends AuthState {}
class AuthLoading extends AuthState {}
class AuthSuccess extends AuthState {}
class AuthFailure extends AuthState {
  final String error;
  AuthFailure(this.error);
}

// 3. Define Bloc
class AuthBloc extends Bloc<AuthEvent, AuthState> {
  final AuthenticationRepository authRepository;

  AuthBloc({required this.authRepository}) : super(AuthInitial()) {
    // Register Event Listener
    on<AuthLoginRequested>(_onLoginRequested);
  }

  // Extracting methods makes the code cleaner and facilitates testing
  Future<void> _onLoginRequested(
    AuthLoginRequested event, 
    Emitter<AuthState> emit
  ) async {
    // Emit Loading state
    emit(AuthLoading());
    try {
      await authRepository.logIn(
        username: event.username,
        password: event.password,
      );
      // Emit Success state
      emit(AuthSuccess());
    } catch (e) {
      emit(AuthFailure(e.toString()));
    }
  }
}
```

### 5. Advanced BLoC Architecture: Freezed Integration (Union Types)
When States and Events become extremely complex, manually writing `Equatable` and subclasses becomes tedious. `freezed` perfectly complements BLoC by generating Union Types, drastically reducing boilerplate and providing compile-time safety through `map` or `when` pattern matching.

#### 5.1 Defining Events and States with Freezed
Instead of creating multiple subclasses, define a single `@freezed` class with factory constructors for each variation:

```dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'auth_event.freezed.dart';
part 'auth_state.freezed.dart';

@freezed
class AuthEvent with _$AuthEvent {
  const factory AuthEvent.loginRequested(String username, String password) = _LoginRequested;
  const factory AuthEvent.logoutRequested() = _LogoutRequested;
}

@freezed
class AuthState with _$AuthState {
  const factory AuthState.initial() = _Initial;
  const factory AuthState.loading() = _Loading;
  const factory AuthState.success(User user) = _Success;
  const factory AuthState.failure(String error) = _Failure;
}
```

#### 5.2 Consuming Freezed in the UI
On the UI side, `BlocBuilder` can elegantly resolve the current state utilizing the generated `map` or `when` functions safely!

```dart
BlocBuilder<AuthBloc, AuthState>(
  builder: (context, state) {
    // 🌟 'when' forces you to handle ALL possible Union cases. If you add a new state later, the compiler throws an error here!
    return state.when(
      initial: () => const LoginButton(),
      loading: () => const CircularProgressIndicator(),
      success: (user) => Text('Welcome ${user.name}'),
      failure: (error) => Text('Error: $error'),
    );
  },
)
```

### 6. UI Layer Usage (Widgets) Best Practices
The `flutter_bloc` package provides a suite of Widgets. Please ensure they are used in the correct scenarios.

#### 6.1 Providing Dependency Injection (Providing)
*   **`BlocProvider` / `MultiBlocProvider`**: Always provide the Bloc at the lowest common ancestor node in the widget tree that requires it.
```dart
BlocProvider(
  create: (context) => AuthBloc(authRepository: RepositoryProvider.of(context)),
  child: const AppView(),
)
```

#### 6.2 Reacting to State Updates (Consuming)
*   **`BlocBuilder`**: This is the most frequently used Widget, employed for rendering the screen. It **triggers a rebuild** when a new state is emitted.
    *   **Best Practice: Keep the UI inside the Builder as minimal as possible.** If only a small section of the entire page relies on state, do not wrap the entire Scaffold with the Builder.
    *   You can utilize `buildWhen: (previous, current) => ...` to control rebuilding only under specific conditions.
*   **`BlocListener`**: Used to execute **one-shot** side effects, such as: displaying a SnackBar, Navigation page routing, or Dialogs. This **will not** trigger a UI rebuild.
*   **`BlocConsumer`**: Combines `BlocBuilder` and `BlocListener`. Use this when a distinct section simultaneously needs to render a new state and execute side effects.

```dart
BlocConsumer<AuthBloc, AuthState>(
  listener: (context, state) {
    // Execute side effects
    if (state is AuthFailure) {
      ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text(state.error)));
    } else if (state is AuthSuccess) {
      Navigator.of(context).pushReplacementNamed('/home');
    }
  },
  builder: (context, state) {
    // Render UI
    if (state is AuthLoading) {
      return const CircularProgressIndicator();
    }
    return ElevatedButton(
      onPressed: () => context.read<AuthBloc>().add(AuthLoginRequested('user', 'pass')),
      child: const Text('Login'),
    );
  },
)
```

#### 6.3 Reading/Dispatching Events (Context extensions)
*   `context.read<MyBloc>()`: The best practice during button click events or within `initState` (used to call functions or add events).
*   `context.watch<MyBloc>()`: Listens to changes in the entire state within a Widget's `build` method and returns the Bloc instance (generally less intuitive and more resource-intensive than directly using `BlocBuilder`; usage should be minimized).
*   `context.select<MyBloc, bool>((b) => b.state.isLoading)`: Use this when you only need to listen to a specific property rather than the entire state.

### 7. Advanced Ecosystem Packages (Indispensable Powerful Weapons)
For medium-to-large projects, official peripheral packages are provided to resolve specific pain points. It is highly recommended to introduce them based on your scenario.

#### 6.1 `bloc_test`: The Holy Grail of Unit Testing
(Latest version roughly `^10.0.0`)

Writing solid state tests in Flutter has never been easy, but `bloc_test` provides an elegant and intuitive API specifically designed for testing Bloc/Cubit behavior.

```dart
import 'package:bloc_test/bloc_test.dart';

void main() {
  group('CounterCubit', () {
    // blocTest encapsulates setup, act, expect, rendering the code exceptionally clean
    blocTest<CounterCubit, CounterState>(
      'State becomes CounterState(1) after dispatching increment',
      build: () => CounterCubit(),
      act: (cubit) => cubit.increment(),
      expect: () => [const CounterState(value: 1)], // Expected sequence of states
    );
  });
  
  group('AuthBloc', () {
    // Advanced Application: Seed state, Multiple operations, and Verification
    blocTest<AuthBloc, AuthState>(
      'Failed login should sequentially emit Loading and Error states',
      build: () {
        when(mockAuthRepo.login(any, any)).thenThrow(Exception('Failed'));
        return AuthBloc(mockAuthRepo);
      },
      seed: () => AuthUnauthenticated(), // Specify the initial state
      act: (bloc) => bloc.add(LoginRequested('user', 'pass')),
      expect: () => [
        isA<AuthLoading>(),
        isA<AuthError>().having((e) => e.message, 'message', 'Exception: Failed'),
      ],
      verify: (_) { // Verify if the repository was actually called
        verify(mockAuthRepo.login('user', 'pass')).called(1);
      },
    );
  });
}
```

**Mocking Blocs in Widget Tests**
When you need to test UIs that consume a Bloc without executing logic, you can use `MockBloc`.

```dart
class MockCounterBloc extends MockBloc<CounterEvent, int> implements CounterBloc {}

void main() {
  testWidgets('Render Counter, depend on MockBloc', (tester) async {
    final mockBloc = MockCounterBloc();
    // Hardcode the current state stream
    whenListen(
      mockBloc,
      Stream.fromIterable([42]),
      initialState: 42,
    );

    await tester.pumpWidget(
      BlocProvider<CounterBloc>.value(
        value: mockBloc,
        child: const MaterialApp(home: CounterPage()),
      ),
    );

    expect(find.text('42'), findsOneWidget);
  });
}
```

#### 6.2 `bloc_concurrency`: The Gatekeeper Regulating Event Timing
(Latest version roughly `^0.3.0`)

This is the biggest reason why you should choose `Bloc` over simplistic `Cubit`s or `Provider`s! When a user frantically clicks a button ten times in one second (e.g., hitting an API or submitting a form), you need precise control over how these Events are processed.

```dart
import 'package:bloc_concurrency/bloc_concurrency.dart';

class SearchBloc extends Bloc<SearchEvent, SearchState> {
  SearchBloc() : super(SearchInitial()) {
    // Introduce a transformer inside on<Event> to alter default concurrency behavior
    on<SearchKeywordChanged>(
      _onSearchChanged,
      // 👉 restartable: Cancels the previous uncompleted event when a new event enters (Optimal for Search bar typing)
      // 👉 droppable: Discards all newly injected events before the first event completes processing (Optimal for preventing Submit double-clicks)
      // 👉 sequential: Queues and processes them one by one
      transformer: restartable(), 
    );
  }
}
```

#### 6.3 `hydrated_bloc`: State Persistence (State Persistence)
(Latest version roughly `^10.1.1`)

If your App is killed by the system, and you want the user to instantly return to their previous state the next time they open it (e.g., maintaining shopping cart contents, dark mode preference). You no longer need to write SharedPreferences yourself!

Simply alter the original `Bloc` to inherit from `HydratedBloc`; it will automatically save your state to local disk and automatically read it upon reboot.

```dart
// 1. Change inheritance to HydratedCubit or HydratedBloc
class ThemeCubit extends HydratedCubit<ThemeMode> {
  ThemeCubit() : super(ThemeMode.system);
  void toggleTheme() => emit(state == ThemeMode.light ? ThemeMode.dark : ThemeMode.light);

  // 2. Implement logic restoring from JSON
  @override
  ThemeMode fromJson(Map<String, dynamic> json) => 
      ThemeMode.values[json['theme_index'] as int];

  // 3. Implement logic saving State into JSON
  @override
  Map<String, int> toJson(ThemeMode state) => {'theme_index': state.index};
}
```

#### 6.4 `replay_bloc`: Turning Back Time (Undo / Redo)
(Latest version roughly `^0.3.0`)

Adding **"Undo"** and **"Redo"** features to your App becomes incredibly easy. For instance, in drawing Apps or complex forms, just change your `Bloc` to inherit from `ReplayBloc`.

```dart
class FormBloc extends ReplayBloc<FormEvent, FormState> {
  // ...
}

// Within the UI layer, you can call at any time:
final formBloc = context.read<FormBloc>();

if (formBloc.canUndo) {
  formBloc.undo(); // State automatically reverts to the previous step!
}
```

### 8. Summary
1. Always implement **Cubit** as your default starting maneuver, switching to **Bloc** only when a clear need arises to log an event stream or utilize transformers.
2. Integrate heavily with **Equatable** to diminish invalid, redundant UI refreshes.
3. Consolidate all logic inside the bloc/cubit; **the UI is 100% responsible solely for reflecting results based on the `state`** (Do not write conditional logic or temporary variables in the UI layer).
4. The modernized `on<Event>` paradigm has superseded legacy manual generators (`yield`), rendering asynchronous syntax (`async/await`) infinitely more legible and intuitive for debugging.

## Constraints
* Priorize implementing `Cubit` structures globally over heavyweight `Bloc` architectures for straightforward component states to minimize boilerplate fatigue. 
* Strictly mandate utilizing `Equatable` mappings across ALL generated states.
* Strictly enforce using the `blocTest` syntax (`build`, `act`, `expect`, `verify`) rather than listening manually to Bloc streams out of context.
