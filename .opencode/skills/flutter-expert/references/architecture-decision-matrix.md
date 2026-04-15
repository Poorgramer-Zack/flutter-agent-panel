---
name: "architecture-decision-matrix"
description: "Comprehensive decision framework for choosing Flutter architecture patterns"
metadata:
  last_modified: "2026-03-31 10:45:00 (GMT+8)"
---

# Flutter Architecture Decision Matrix

## Overview

Choosing the right architecture is critical for long-term maintainability, scalability, and team productivity. This matrix provides a structured decision framework based on project complexity, team size, and business requirements.

---

## Quick Decision Tree

```
Start Here
    |
    ├─ Team Size < 3 + Simple CRUD App?
    │   └─> Provider + Feature-First (Simple MVC)
    │
    ├─ Medium Complexity + Testability Important?
    │   └─> Clean Architecture + BLoC/Riverpod
    │
    ├─ Large Team + Multiple Features + Microservices?
    │   └─> Modular Monolith / Multi-Package Architecture
    │
    └─ Extreme Scale + Domain Complexity?
        └─> Domain-Driven Design (DDD) + Hexagonal Architecture
```

---

## Architecture Pattern Comparison Matrix

| Pattern | Team Size | Complexity | Learning Curve | Testability | Scalability | Best For |
|---------|-----------|------------|----------------|-------------|-------------|----------|
| **Simple MVC** | 1-3 | Low | Low | Medium | Low | MVPs, prototypes, simple CRUD |
| **Provider + Repository** | 2-5 | Medium | Medium | High | Medium | Small-medium apps, startups |
| **Clean Architecture** | 3-10 | High | High | Very High | High | Enterprise apps, long-term projects |
| **Feature-First Modular** | 5-15 | Medium-High | Medium | High | Very High | Multi-team development |
| **Hexagonal (Ports & Adapters)** | 5-20 | Very High | Very High | Very High | Very High | Complex domain logic, banking/fintech |
| **DDD + Microservices** | 10+ | Extreme | Extreme | Very High | Maximum | Large enterprises, multi-platform ecosystems |

---

## Detailed Pattern Breakdown

### 1. Simple MVC (Model-View-Controller)

**Structure:**
```
lib/
  ├─ models/           # Data classes
  ├─ views/            # UI widgets
  ├─ controllers/      # Business logic
  └─ services/         # API/Database
```

**When to Use:**
- ✅ Team: 1-3 developers
- ✅ Timeline: < 3 months
- ✅ Scope: Simple CRUD operations
- ✅ Requirements: MVP, prototype, proof of concept

**Pros:**
- Fastest time to market
- Minimal boilerplate
- Easy to understand for beginners
- Low cognitive overhead

**Cons:**
- Poor separation of concerns
- Hard to test business logic
- Tight coupling between layers
- Difficult to scale beyond medium complexity

**Example Use Cases:**
- Personal projects
- Hackathon apps
- Internal tools
- Simple utility apps

---

### 2. Provider + Repository Pattern

**Structure:**
```
lib/
  ├─ models/              # Entities & DTOs
  ├─ repositories/        # Data abstraction layer
  ├─ providers/           # State management
  ├─ services/            # API clients
  └─ ui/
      ├─ screens/
      └─ widgets/
```

**When to Use:**
- ✅ Team: 2-5 developers
- ✅ Timeline: 3-6 months
- ✅ Scope: Medium complexity with moderate business logic
- ✅ Requirements: Good testability without extreme overhead

**Pros:**
- Balanced complexity vs. benefits
- Repository pattern enables easy testing
- Provider is officially supported
- Good for incremental migration

**Cons:**
- Can become messy without discipline
- Provider can lead to boilerplate in complex apps
- No enforced layer separation

**Example Use Cases:**
- E-commerce apps
- Social media clients
- Content management apps
- Educational platforms

**State Management Options:**
- Provider (official, simple)
- Riverpod (type-safe, compile-time DI)
- GetX (all-in-one, rapid development)

---

### 3. Clean Architecture (Layered)

**Structure:**
```
lib/
  ├─ core/                    # Shared utilities
  ├─ features/
  │   └─ auth/
  │       ├─ domain/
  │       │   ├─ entities/    # Pure Dart models
  │       │   ├─ repositories/# Abstract contracts
  │       │   └─ usecases/    # Business logic
  │       ├─ data/
  │       │   ├─ models/      # JSON DTOs
  │       │   ├─ datasources/ # API/DB implementations
  │       │   └─ repositories/# Concrete implementations
  │       └─ presentation/
  │           ├─ bloc/        # State management
  │           ├─ pages/       # Screens
  │           └─ widgets/     # Reusable components
```

**When to Use:**
- ✅ Team: 3-10 developers
- ✅ Timeline: 6+ months
- ✅ Scope: High complexity with evolving requirements
- ✅ Requirements: Maximum testability, long-term maintenance

**Pros:**
- **Enforced separation of concerns**
- **Domain layer is pure Dart** (no Flutter dependencies)
- **Highly testable** (mock all dependencies)
- Industry-standard pattern
- Excellent for onboarding new developers

**Cons:**
- High initial boilerplate
- Steeper learning curve
- Can feel over-engineered for simple features
- Requires discipline to maintain

**Example Use Cases:**
- Enterprise mobile applications
- Banking/Fintech apps
- Healthcare platforms
- Long-term SaaS products

**Recommended State Management:**
- BLoC (official recommendation for Clean Architecture)
- Riverpod (modern alternative with less boilerplate)

**Key Principles:**
1. **Dependency Rule:** Dependencies point INWARD only
2. **Entities are independent** of frameworks/databases/UI
3. **UseCases** encapsulate single business rules
4. **Repository interfaces** in domain, implementations in data

---

### 4. Feature-First Modular Architecture

**Structure:**
```
lib/
  ├─ features/
  │   ├─ auth/
  │   │   ├─ data/
  │   │   ├─ domain/
  │   │   ├─ presentation/
  │   │   └─ auth.dart       # Feature barrel file
  │   ├─ cart/
  │   ├─ products/
  │   └─ profile/
  ├─ core/
  │   ├─ network/
  │   ├─ storage/
  │   └─ theme/
  └─ app.dart
```

**When to Use:**
- ✅ Team: 5-15 developers (multiple squads)
- ✅ Timeline: 12+ months
- ✅ Scope: Large apps with independent feature teams
- ✅ Requirements: Parallel development, feature isolation

**Pros:**
- **Teams can work independently** on features
- **Feature toggles** easy to implement
- **Code ownership** is clear
- Easy to extract features into packages

**Cons:**
- Potential code duplication across features
- Requires careful management of shared code
- Can lead to inconsistent patterns if not standardized

**Example Use Cases:**
- Super apps (like WeChat, Grab)
- Multi-tenant platforms
- Apps with frequent feature releases

---

### 5. Hexagonal Architecture (Ports & Adapters)

**Structure:**
```
lib/
  ├─ domain/                 # Core business logic (center)
  │   ├─ models/
  │   ├─ ports/              # Interfaces (contracts)
  │   └─ services/           # Domain services
  ├─ application/            # Use cases / orchestration
  ├─ adapters/               # Outer layer (implementations)
  │   ├─ driven/             # Outbound (DB, API)
  │   │   ├─ repositories/
  │   │   └─ api/
  │   └─ driving/            # Inbound (UI, CLI)
  │       └─ ui/
```

**When to Use:**
- ✅ Team: 5-20 developers
- ✅ Timeline: 18+ months
- ✅ Scope: Complex domain logic, multiple integration points
- ✅ Requirements: Swap implementations easily (e.g., REST → GraphQL)

**Pros:**
- **Extreme flexibility** for changing external dependencies
- **Technology-agnostic core**
- Perfect for apps with multiple data sources
- Ideal when business rules are complex

**Cons:**
- Very high initial setup cost
- Requires deep architectural understanding
- Can be overkill for standard CRUD apps

**Example Use Cases:**
- Fintech platforms with regulatory requirements
- Healthcare systems with multiple data sources
- IoT platforms with varied device protocols

---

### 6. Domain-Driven Design (DDD)

**Structure:**
```
lib/
  ├─ bounded_contexts/       # Separate domains
  │   ├─ payment/
  │   │   ├─ domain/
  │   │   │   ├─ aggregates/
  │   │   │   ├─ entities/
  │   │   │   ├─ value_objects/
  │   │   │   └─ domain_events/
  │   │   ├─ application/
  │   │   └─ infrastructure/
  │   └─ inventory/
  ├─ shared_kernel/          # Cross-domain code
  └─ anti_corruption_layer/  # Integration boundaries
```

**When to Use:**
- ✅ Team: 10+ developers (multiple specialized teams)
- ✅ Timeline: Multi-year projects
- ✅ Scope: Extremely complex domain with rich business rules
- ✅ Requirements: Multiple bounded contexts, microservices

**Pros:**
- **Aligns code with business domains**
- **Ubiquitous language** improves communication
- Scales to massive complexity
- Prevents coupling between unrelated features

**Cons:**
- Extreme complexity
- Requires domain experts
- Long ramp-up time
- Massive overhead for simple apps

**Example Use Cases:**
- Enterprise resource planning (ERP) systems
- Large-scale e-commerce platforms
- Multi-tenant SaaS with complex workflows

---

## State Management Decision Matrix

| Pattern | Complexity | Boilerplate | Type Safety | Testing | Best Architecture |
|---------|------------|-------------|-------------|---------|-------------------|
| **setState** | Low | Minimal | Low | Hard | Simple MVC |
| **Provider** | Medium | Low | Medium | Medium | Provider + Repo |
| **Riverpod** | Medium | Low | High | High | Clean, Feature-First |
| **BLoC** | High | High | High | Very High | Clean Architecture |
| **GetX** | Low | Minimal | Low | Medium | Simple MVC, Rapid prototyping |
| **MobX** | Medium | Medium | Medium | High | Feature-First |
| **Redux** | High | Very High | High | Very High | Large-scale apps |

---

## File Organization Strategies

### Layer-First (Traditional Clean Architecture)
```
lib/
  ├─ data/
  ├─ domain/
  └─ presentation/
```
**Best for:** Small-medium apps, clear layer boundaries

### Feature-First (Modern Recommendation)
```
lib/
  ├─ features/
  │   └─ auth/
  │       ├─ data/
  │       ├─ domain/
  │       └─ presentation/
```
**Best for:** Medium-large apps, team scalability

### Hybrid (Best of Both Worlds)
```
lib/
  ├─ features/              # Feature modules
  ├─ core/                  # Shared layer
  │   ├─ domain/
  │   ├─ data/
  │   └─ presentation/
```
**Best for:** Enterprise apps with shared/feature-specific code

---

## Migration Strategies

### From Simple MVC → Clean Architecture

**Phase 1:** Introduce Repository Pattern
- Extract data logic into repositories
- Keep existing controllers

**Phase 2:** Extract Domain Layer
- Create entities from models
- Define repository interfaces
- Move business logic to use cases

**Phase 3:** Implement Clean Presentation
- Migrate to BLoC/Riverpod
- Remove business logic from UI

**Timeline:** 2-4 sprints depending on app size

---

### From Monolith → Feature Modules

**Phase 1:** Identify Feature Boundaries
- Map business domains
- Define shared core

**Phase 2:** Create Feature Packages
- Extract one feature at a time
- Create package dependencies

**Phase 3:** Establish Feature Contracts
- Define feature communication protocols
- Implement dependency injection

**Timeline:** 6-12 months for large apps

---

## Decision Criteria Checklist

### Choose Simple MVC if:
- [ ] App will be < 20 screens
- [ ] Timeline is < 3 months
- [ ] Team is 1-2 developers
- [ ] It's a prototype/MVP
- [ ] Business logic is trivial

### Choose Clean Architecture if:
- [ ] App will be maintained for 2+ years
- [ ] High test coverage required (>80%)
- [ ] Team is 3+ developers
- [ ] Business logic is moderately complex
- [ ] Multiple data sources needed

### Choose Feature-First Modular if:
- [ ] Multiple teams working simultaneously
- [ ] Features are relatively independent
- [ ] Need feature toggles/A-B testing
- [ ] Planning to extract features to packages
- [ ] App has 50+ screens

### Choose Hexagonal/DDD if:
- [ ] Extremely complex domain logic
- [ ] Multiple external system integrations
- [ ] Need to swap implementations frequently
- [ ] Regulatory/compliance requirements
- [ ] Enterprise-level application

---

## Common Anti-Patterns to Avoid

### ❌ The God UseCase
```dart
// BAD: UseCase doing too much
class UserUseCase {
  Future<User> login() {}
  Future<User> register() {}
  Future<void> logout() {}
  Future<void> updateProfile() {}
  Future<void> deleteAccount() {}
}
```

### ✅ Single Responsibility UseCases
```dart
// GOOD: One use case per action
class LoginUseCase {
  Future<User> call(String email, String password) {}
}

class RegisterUseCase {
  Future<User> call(UserRegistration data) {}
}
```

---

### ❌ Domain Layer Importing Flutter
```dart
// BAD: Domain depends on Flutter
import 'package:flutter/material.dart';

class User {
  final Color favoriteColor; // ❌ Flutter dependency in Domain
}
```

### ✅ Pure Dart Domain
```dart
// GOOD: Domain is pure Dart
class User {
  final int favoriteColorHex; // ✅ Pure Dart representation
}
```

---

### ❌ Business Logic in UI
```dart
// BAD: Business logic mixed with UI
class LoginPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () async {
        // ❌ Business logic directly in UI
        if (email.isEmpty || password.length < 6) {
          showError();
        }
        final response = await dio.post('/login');
        if (response.statusCode == 200) {
          Navigator.push(...);
        }
      },
    );
  }
}
```

### ✅ Business Logic in UseCase/BLoC
```dart
// GOOD: UI only dispatches events
class LoginPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () {
        context.read<LoginBloc>().add(LoginSubmitted(email, password));
      },
    );
  }
}
```

---

## Performance Considerations

| Architecture | Build Time | Hot Reload Speed | App Size | Memory Overhead |
|--------------|------------|------------------|----------|-----------------|
| Simple MVC | Fast | Fast | Small | Low |
| Clean Architecture | Medium | Medium | Medium | Medium |
| Feature Modules | Slow (1st build) | Fast (incremental) | Large | Medium-High |
| DDD/Microservices | Very Slow | Fast (per module) | Very Large | High |

**Optimization Tips:**
- Use code generation (freezed, json_serializable) sparingly
- Enable tree shaking in release builds
- Split large features into separate packages for better build caching
- Use const constructors aggressively

---

## Recommended Resources

### Books
- **Clean Architecture** by Robert C. Martin
- **Domain-Driven Design** by Eric Evans
- **Implementing Domain-Driven Design** by Vaughn Vernon

### Flutter-Specific
- [Reso Coder's Clean Architecture Series](https://resocoder.com/flutter-clean-architecture-tdd/)
- [Flutter Architecture Samples (Official)](https://github.com/brianegan/flutter_architecture_samples)
- [Very Good Ventures Architecture Guide](https://verygood.ventures/blog/very-good-flutter-architecture)

---

## Conclusion

**For most production Flutter apps (3-10 person teams):**  
→ **Clean Architecture + Feature-First structure + BLoC/Riverpod**

**Key Principles:**
1. Start simple, add complexity only when needed
2. Optimize for **team scalability**, not just code scalability
3. Prioritize **consistent patterns** over perfect architecture
4. Invest in architecture when the cost of change becomes high
5. **Test-drive** your architecture decision with a small feature first

**Remember:** The best architecture is the one your team can execute consistently. A simpler architecture executed well beats a complex one executed poorly.
