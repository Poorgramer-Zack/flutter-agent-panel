---
name: "flutter-testing"
description: "Covers Flutter testing across all three layers: unit tests (business logic, ViewModels, repositories), widget tests (UI rendering, interactions, golden regression), and integration/E2E tests (full app flows, native OS interactions). Use this skill when writing any Flutter test, setting up mocking with mocktail or mockito, implementing golden file tests, debugging async failures (pumpAndSettle timeouts, Timer leaks, FakeAsync deadlocks), fixing cross-platform golden pixel diffs in CI/CD, testing state management (Riverpod/BLoC/Provider), handling permission dialogs or native system UI with Patrol, or troubleshooting MediaQuery/Theme missing errors in widget tests."
metadata:
  last_modified: "2026-04-01 14:35:00 (GMT+8)"
---
# Flutter Testing Guide

## Overview
Flutter testing follows a three-layer pyramid: **Unit** (fast, pure Dart logic) → **Widget** (UI component rendering) → **Integration/E2E** (full device flows). Each layer has distinct tooling and tradeoffs. Load the relevant reference based on what you need to test.

## Process

### Phase 1: Choose the Right Layer
Identify the testing tier before writing any code. Wrong tier = wasted effort.
- [🧱 Testing Fundamentals](./references/testing-fundamentals.md) — Decision tree for choosing unit/widget/integration, MVVM isolation patterns, and project structure.

### Phase 2: Write Tests with Correct APIs
Apply AAA (Arrange-Act-Assert), proper pump usage, and matcher selection.
- [🛠️ Core Testing](./references/core-testing.md) — Unit test groups/setUp/tearDown, widget test pump patterns, integration test binding setup, and async troubleshooting (FakeAsync, runAsync, Timer leaks).

### Phase 3: Mocking, Goldens, and E2E
Isolate external dependencies, lock UI visuals, and validate native OS interactions.
- [🧪 Advanced Tools](./references/advanced-tools.md) — Mocktail/Mockito, golden tests with cross-platform tolerance, Patrol 4.0 E2E (native permission dialogs, notifications), patrol_finders for widget tests.

### Phase 4: Diagnose Failures
Resolve production testing failures in CI/CD pipelines.
- [🔧 Troubleshooting](./references/troubleshooting.md) — Async errors, golden pixel diffs, MediaQuery crashes, Timer memory leaks, flaky mock configurations.

---

## 📚 Reference Library

- [🧱 Testing Fundamentals](./references/testing-fundamentals.md)
- [🛠️ Core Testing](./references/core-testing.md)
- [🧪 Advanced Tools (Mocking, Goldens, Patrol E2E)](./references/advanced-tools.md)
- [🔧 Troubleshooting & Edge Cases](./references/troubleshooting.md)
