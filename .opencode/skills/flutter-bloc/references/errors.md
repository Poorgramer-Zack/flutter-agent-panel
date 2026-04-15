# BLoC Common Errors & Solutions

## ToC
1. [Emitting After Bloc Close](#1-emitting-after-bloc-close)
2. [Mutable State Causing UI Bugs](#2-mutable-state-causing-ui-bugs)
3. [Missing Centralized Error Handling](#3-missing-centralized-error-handling)

---

## 1. Emitting After Bloc Close

**Problem**: Async operations may complete after the Bloc is disposed, crashing with
`"Cannot emit new states after calling close"`.

```dart
// ❌ BAD: Bloc might be closed before the delay completes
Future<void> _onEvent(MyEvent event, Emitter<MyState> emit) async {
  await Future.delayed(Duration(seconds: 5));
  emit(NewState()); // Bloc might already be closed
}
```

**Solution**:
```dart
// ✅ GOOD: Guard with emit.isDone (preferred over isClosed inside handlers)
Future<void> _onEvent(MyEvent event, Emitter<MyState> emit) async {
  await Future.delayed(Duration(seconds: 5));
  if (!emit.isDone) {
    emit(NewState());
  }
}
```

**Always close Blocs in StatefulWidget**:
```dart
@override
void dispose() {
  myBloc.close(); // Prevents memory leaks
  super.dispose();
}
```

> `BlocProvider` handles `.close()` automatically when the widget is removed from the tree.
> Only call `.close()` manually when managing Blocs outside of `BlocProvider`.

---

## 2. Mutable State Causing UI Bugs

**Problem**: Mutating state in-place means `BlocBuilder` won't rebuild because the object
reference hasn't changed.

```dart
// ❌ BAD: In-place mutation — BlocBuilder ignores this
void _onUserAdded(UserAdded event, Emitter<UserState> emit) {
  state.users.add(event.user); // Same list reference
  emit(state);                 // Equatable sees no change → no rebuild
}
```

**Solution with Equatable** (spread into new list):
```dart
// ✅ GOOD: New list instance triggers Equatable change detection
void _onUserAdded(UserAdded event, Emitter<UserState> emit) {
  emit(UserState([...state.users, event.user]));
}
```

**Solution with Freezed** (recommended for complex states):
```dart
// ✅ BEST: copyWith guarantees a new immutable instance
void _onUserAdded(UserAdded event, Emitter<UserState> emit) {
  emit(state.copyWith(
    users: [...state.users, event.user],
  ));
}
```

---

## 3. Missing Centralized Error Handling

**Problem**: Duplicated `try/catch` in every event handler leads to inconsistent error
reporting and bloated handlers.

```dart
// ❌ BAD: Scattered error handling
void _onEvent1(Event1 event, Emitter<MyState> emit) async {
  try { /* logic */ } catch (e) { emit(ErrorState(e.toString())); }
}
void _onEvent2(Event2 event, Emitter<MyState> emit) async {
  try { /* logic */ } catch (e) { emit(ErrorState(e.toString())); }
}
```

**Solution**: Override `onError` for unexpected errors; use typed catches for user-facing errors.

```dart
// ✅ GOOD: Centralized unexpected-error handling
class MyBloc extends Bloc<MyEvent, MyState> {
  @override
  void onError(Object error, StackTrace stackTrace) {
    logger.error('Bloc error', error, stackTrace);
    crashReporter.report(error, stackTrace);
    super.onError(error, stackTrace);
  }

  // Handlers stay focused on logic
  Future<void> _onLoginEvent(LoginEvent event, Emitter<AuthState> emit) async {
    emit(AuthLoading());
    try {
      await authRepository.login(event.username, event.password);
      emit(AuthSuccess());
    } on InvalidCredentialsException catch (e) {
      emit(AuthFailure(e.message)); // User-facing: emit to UI
    } catch (e) {
      addError(e); // Unexpected: routes to onError
    }
  }
}
```
