---
name: "routing-with-autoroute"
description: "Implements type-safe Flutter routing using AutoRoute v11 with code generation and @RoutePage annotations. Activates when setting up AutoRoute navigation, configuring AutoRouterConfig or RootStackRouter, generating route files with build_runner, using @RoutePage, @PathParam, or @QueryParam annotations, implementing AutoTabsRouter for bottom navigation tabs, adding navigation guards with AutoRouteGuard, setting up nested or child routes, handling type-safe route parameters, or migrating from string-based Navigator routing to compile-time-verified routes. Ideal for large apps requiring refactoring confidence and zero string-based routing errors."
metadata:
  last_modified: "2026-04-01 14:35:00 (GMT+8)"
---

# AutoRoute Type-Safe Navigation Guide (v11.x)

## Goal
Implement type-safe routing using AutoRoute with compile-time code generation. AutoRoute v11+ uses @RoutePage annotations to eliminate boilerplate and ensure type safety through generated route configurations.

## Process

### Phase 1: Install Dependencies

```yaml
dependencies:
  auto_route: ^11.1.0

dev_dependencies:
  auto_route_generator: ^11.1.0
  build_runner: ^2.4.0
```

### Phase 2: Annotate Pages

```dart
import 'package:auto_route/auto_route.dart';
import 'package:flutter/material.dart';

@RoutePage()
class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Home')),
      body: Center(
        child: ElevatedButton(
          onPressed: () => context.router.push(ProfileRoute()),
          child: Text('Go to Profile'),
        ),
      ),
    );
  }
}

@RoutePage()
class ProfileScreen extends StatelessWidget {
  const ProfileScreen({super.key});
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Profile')),
    );
  }
}

// Type-safe parameters
@RoutePage()
class UserScreen extends StatelessWidget {
  final String userId;
  final String? referrer;  // Optional parameter
  
  const UserScreen({
    @PathParam('id') required this.userId,
    @QueryParam('ref') this.referrer,
    super.key,
  });
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('User $userId')),
      body: Text('Referrer: ${referrer ?? "none"}'),
    );
  }
}
```

### Phase 3: Define Router Configuration

```dart
import 'package:auto_route/auto_route.dart';

part 'app_router.gr.dart';

@AutoRouterConfig()
class AppRouter extends RootStackRouter {
  @override
  List<AutoRoute> get routes => [
    AutoRoute(page: HomeRoute.page, initial: true),
    AutoRoute(page: ProfileRoute.page),
    AutoRoute(page: UserRoute.page, path: '/user/:id'),
    AutoRoute(page: LoginRoute.page, path: '/login'),
  ];
}
```

### Phase 4: Generate Routes

```bash
flutter pub run build_runner build --delete-conflicting-outputs

# Or watch mode for development
flutter pub run build_runner watch
```

### Phase 5: Initialize Router

```dart
void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  final _appRouter = AppRouter();
  
  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      routerConfig: _appRouter.config(),
    );
  }
}
```

### Phase 6: Navigation

```dart
// Push route (type-safe!)
context.router.push(UserRoute(userId: '123'));

// Push with query params
context.router.push(UserRoute(userId: '123', referrer: 'home'));

// Replace
context.router.replace(ProfileRoute());

// Pop
context.router.pop();

// Pop until
context.router.popUntil((route) => route.settings.name == HomeRoute.name);

// Navigate to nested route
context.router.navigate(ProfileRoute(children: [
  EditProfileRoute(),
]));
```

### Phase 7: Nested Navigation & Tabs

```dart
@AutoRouterConfig()
class AppRouter extends RootStackRouter {
  @override
  List<AutoRoute> get routes => [
    AutoRoute(
      page: DashboardRoute.page,
      initial: true,
      children: [
        AutoRoute(page: HomeRoute.page, initial: true),
        AutoRoute(page: SearchRoute.page),
        AutoRoute(page: ProfileRoute.page),
      ],
    ),
  ];
}

@RoutePage()
class DashboardScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return AutoTabsRouter(
      routes: const [
        HomeRoute(),
        SearchRoute(),
        ProfileRoute(),
      ],
      builder: (context, child) {
        final tabsRouter = AutoTabsRouter.of(context);
        return Scaffold(
          body: child,
          bottomNavigationBar: BottomNavigationBar(
            currentIndex: tabsRouter.activeIndex,
            onTap: tabsRouter.setActiveIndex,
            items: const [
              BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Home'),
              BottomNavigationBarItem(icon: Icon(Icons.search), label: 'Search'),
              BottomNavigationBarItem(icon: Icon(Icons.person), label: 'Profile'),
            ],
          ),
        );
      },
    );
  }
}
```

### Phase 8: Guards (Authentication)

```dart
class AuthGuard extends AutoRouteGuard {
  @override
  void onNavigation(NavigationResolver resolver, StackRouter router) {
    if (AuthService.instance.isLoggedIn) {
      resolver.next(); // Proceed
    } else {
      resolver.redirect(LoginRoute(
        onResult: (success) {
          if (success) {
            resolver.next();
          }
        },
      ));
    }
  }
}

// Apply guard
@AutoRouterConfig()
class AppRouter extends RootStackRouter {
  @override
  List<AutoRoute> get routes => [
    AutoRoute(page: LoginRoute.page),
    AutoRoute(
      page: ProfileRoute.page,
      guards: [AuthGuard()],  // Protected route
    ),
  ];
}
```

---

## Constraints

* **Breaking Changes in v11**: Old syntax from v10 is incompatible. Use @RoutePage() only.
* **Type Safety**: All route parameters are type-checked at compile time.
* **Code Generation**: Must run build_runner after route changes.
* **Unique Route Names**: Page names must be unique across the app.
* **Guard Order**: Guards execute in the order they're listed.
