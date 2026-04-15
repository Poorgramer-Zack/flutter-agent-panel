---
name: "flutter-architecture"
description: "Implement highly scalable Clean Architecture in Flutter projects"
metadata:
  last_modified: "2026-03-12 11:18:17 (GMT+8)"
---
# Flutter Clean Architecture Implementation

## Goal
Implements a highly scalable, maintainable, and testable Flutter application architecture adhering strictly to **Clean Architecture** principles (by Robert C. Martin). Enforces rigid separation of concerns across three distinct layers: **Presentation**, **Domain**, and **Data**. Guarantees unidirectional data flow and ensures business logic remains entirely decoupled from UI frameworks, external APIs, and database implementations.

## Architecture Layers

Clean Architecture strictly enforces the **Dependency Rule**: Source code dependencies must only point INWARD. Inner layers know absolutely nothing about outer layers.

### 1. Domain Layer (The Core)
The absolute center of the architecture. It is written in **pure Dart** and contains absolutely ZERO Flutter dependencies (`package:flutter`).
*   **Entities**: Pure data structures or enterprise business objects.
*   **Repositories (Interfaces/Contracts)**: Abstract classes defining the expected behaviors (CRUD operations) without exposing implementation details.
*   **UseCases (Interactors)**: Classes encapsulating single, specific business rules or actions (e.g., `LoginUserUseCase`, `GetCartItemsUseCase`). They orchestrate the flow of data to and from the entities using the repository interfaces.

### 2. Data Layer (The Outer Shell - Bottom)
Responsible for all data fetching and system boundaries. It implements the contracts defined by the Domain layer.
*   **Data Sources**: Classes responsible for raw data fetching (e.g., `RemoteDataSource` using `dio`, `LocalDataSource` using `shared_preferences` or `sqflite`).
*   **Models (DTOs)**: Data Transfer Objects. These represent the raw JSON/SQL schemas and include `fromJson`/`toJson` serialization logic. They subclass or convert into Domain Entities.
*   **Repository Implementations**: Concrete implementations of the Domain layer's repository interfaces. They decide whether to fetch data from the remote or local data source, handle raw exception catching, and map Data Models into Domain Entities.

### 3. Presentation Layer (The Outer Shell - Top)
Responsible exclusively for the UI and state manipulation.
*   **State Management (ViewModels / Blocs / Controllers)**: Classes that execute UseCases, manage loading/error states, and expose final immutable states to the UI.
*   **UI (Widgets/Pages)**: Pure Flutter widgets that passively observe the State classes and render pixels. They dispatch user intents (button clicks) back to the State classes.

---

## Instructions

### 1. Define the Domain Layer (Pure Dart)

First, define the Entity and the Repository Interface.
```dart
// domain/entities/user.dart
class User {
  final String id;
  final String name;
  const User({required this.id, required this.name});
}

// domain/repositories/auth_repository.dart
abstract class AuthRepository {
  Future<User> login(String username, String password);
}
```

Next, define the UseCase. The UseCase executes the specific business logic.
```dart
// domain/usecases/login_usecase.dart
class LoginUseCase {
  final AuthRepository repository;

  LoginUseCase(this.repository);

  Future<User> call(String username, String password) async {
    // 🌟 Potential business logic (e.g., password validation) can reside here natively before hitting the repository
    if (password.length < 6) throw Exception("Invalid password length");
    return await repository.login(username, password);
  }
}
```

### 2. Implement the Data Layer

Create the specific Data Source treating the network or database.
```dart
// data/datasources/auth_remote_data_source.dart
class AuthRemoteDataSource {
  final Dio client;
  AuthRemoteDataSource(this.client);

  Future<UserModel> login(String username, String password) async {
    final response = await client.post('/login', data: {'u': username, 'p': password});
    return UserModel.fromJson(response.data);
  }
}

// data/models/user_model.dart
class UserModel extends User {
  const UserModel({required super.id, required super.name});

  factory UserModel.fromJson(Map<String, dynamic> json) {
    return UserModel(id: json['id'], name: json['name']);
  }
}
```

Implement the Repository Interface bridging the Data and Domain layers.
```dart
// data/repositories/auth_repository_impl.dart
class AuthRepositoryImpl implements AuthRepository {
  final AuthRemoteDataSource remoteDataSource;

  AuthRepositoryImpl(this.remoteDataSource);

  @override
  Future<User> login(String username, String password) async {
    try {
      // 🌟 The Implementation fetches the explicit Model, but returns the abstracted Entity upwards natively!
      final userModel = await remoteDataSource.login(username, password);
      return userModel; 
    } on DioException catch (e) {
      throw ServerException(e.message);
    }
  }
}
```

### 3. Implement the Presentation Layer

The ViewModel/Bloc takes the UseCase and manages UI state.
```dart
// presentation/bloc/login_bloc.dart
class LoginBloc extends Bloc<LoginEvent, LoginState> {
  final LoginUseCase loginUseCase; // 🌟 Directly depends on the Domain UseCase, completely ignorant of APIs or JSON!

  LoginBloc({required this.loginUseCase}) : super(LoginInitial()) {
    on<SubmitLoginEvent>((event, emit) async {
      emit(LoginLoading());
      try {
        final user = await loginUseCase(event.username, event.password);
        emit(LoginSuccess(user));
      } catch (e) {
        emit(LoginFailure(e.toString()));
      }
    });
  }
}
```

Finally, the UI consumes the presentation state.
```dart
// presentation/pages/login_page.dart
class LoginPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocConsumer<LoginBloc, LoginState>(
      listener: (context, state) {
        if (state is LoginSuccess) context.go('/home');
        if (state is LoginFailure) showSnackbar(state.error);
      },
      builder: (context, state) {
        if (state is LoginLoading) return const CircularProgressIndicator();
        return ElevatedButton(
          onPressed: () => context.read<LoginBloc>().add(SubmitLoginEvent('user', 'pass')),
          child: const Text('Login'),
        );
      },
    );
  }
}
```

## Constraints
* **The Dependency Rule Strictness:** The `domain/` directory MUST NEVER import anything from `data/` or `presentation/`. It must never import `package:flutter/material.dart` or any GUI frameworks.
* **UseCase Granularity:** Do not create massive `GodUseCases`. Every UseCase should ideally have a single `call()` method representing exactly one user action or business flow.
* **Model vs Entity Isolation:** The Presentation layer must only receive Domain `Entities` from UseCases. The UI should never directly interact with `Models` (DTOs) preventing JSON structures from bleeding into standard UI components natively.
* **No Logic in Views:** Flutter Widgets must remain completely passive. They solely dispatch events to the State Management layer and render states received.
