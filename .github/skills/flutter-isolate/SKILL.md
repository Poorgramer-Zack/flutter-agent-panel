---
name: "running-flutter-isolates"
description: "Background processing and heavy computations using Dart Isolates with Isolate.run(), compute() function, and platform channels. Use this skill when performing CPU-intensive operations without blocking UI (image processing, data parsing, encryption), implementing background tasks that prevent jank or frame drops, handling large file operations (JSON parsing, database migrations), running parallel computations, fixing UI freezes caused by main thread blocking, using compute() for one-off background work, creating long-lived Isolates with ReceivePort/SendPort, implementing multi-threaded algorithms, debugging isolate communication issues, or preventing memory leaks from unclosed ReceivePorts. Covers isolate spawn, message passing, ReceivePort cleanup (CRITICAL for leak prevention), SendPort communication patterns, error handling, serialization rules, and performance optimization for compute-heavy operations."
metadata:
  last_modified: "2026-04-01 14:35:00 (GMT+8)"
---

# Flutter Isolate

## Goal
Move heavy computations off the main isolate to prevent UI jank (>16ms frame gaps), using the simplest API that fits the use case.

## Decision: Which API to Use?

| Use `compute()` when: | Use `Isolate.spawn()` when: |
|---|---|
| ✅ One-off computation | ✅ Long-lived worker needed |
| ✅ Simple serializable data | ✅ Multiple messages over time |
| ✅ Auto-cleanup desired | ✅ Custom messaging protocol |
| ✅ Web compatibility required | ✅ Avoid repeated spawn overhead |
| ❌ NOT for long-running tasks | ❌ NOT for single one-shot tasks |

| Additional Scenarios | API |
|---|---|
| One-shot, single return value (Dart 2.19+) | `Isolate.run()` |
| Need platform plugins in background | `BackgroundIsolateBinaryMessenger` |

---

## 1. Short-lived Isolate: `Isolate.run()`

Best for single, one-off computations. The isolate spawns, runs the task, returns the value, then shuts down automatically.

```dart
// Decode a large JSON file without blocking the UI thread
Future<List<Photo>> getPhotos() async {
  // Load asset on the main isolate first (rootBundle not accessible in isolate)
  final String jsonString = await rootBundle.loadString('assets/photos.json');

  // Offload CPU-heavy decoding to a new isolate
  final List<Photo> photos = await Isolate.run<List<Photo>>(() {
    final List<Object?> photoData = jsonDecode(jsonString) as List<Object?>;
    return photoData.cast<Map<String, Object?>>().map(Photo.fromJson).toList();
  });

  return photos;
}
```

> **Key behavior**: The result is *transferred* (not copied) back to the main isolate via `Isolate.exit` internally — zero-copy for the return value.

---

## 2. Cross-platform: `compute()`

`compute()` is Flutter's wrapper that falls back gracefully on **Flutter Web** (runs on the main thread there, since web doesn't support isolates).

```dart
// Equivalent to Isolate.run on mobile/desktop, runs on main thread on web
Future<List<Photo>> getPhotos(String jsonString) async {
  return compute(_parsePhotos, jsonString);
}

// Top-level or static function only — closures are NOT supported by compute()
List<Photo> _parsePhotos(String jsonString) {
  final data = jsonDecode(jsonString) as List<Object?>;
  return data.cast<Map<String, Object?>>().map(Photo.fromJson).toList();
}
```

> **Constraint**: The callback must be a **top-level or static** function. Closures are not supported.

---

## 3. Long-lived Background Worker

Use `Isolate.spawn()` + ports when you need to send multiple requests to the same isolate over time (avoids spawn overhead per call). Implementation pattern:

Quick structure overview:

```dart
class Worker {
  final SendPort _commands;     // main → worker
  final ReceivePort _responses; // worker → main
  final Map<int, Completer<Object?>> _activeRequests = {};
  int _idCounter = 0;
  bool _closed = false;

  static Future<Worker> spawn() async { /* ... */ }
  Future<Object?> parseJson(String message) async { /* ... */ }
  void close() { _closed = true; _responses.close(); }
}
```

The two-way port handshake pattern:
1. Main creates `ReceivePort`, passes its `sendPort` to `Isolate.spawn()`
2. Worker creates its own `ReceivePort`, sends its `sendPort` back
3. Both sides now have a channel — main tracks requests with `Completer` + incrementing IDs

---

## 4. Platform Plugins in Background Isolates (Flutter 3.7+)

Since Flutter 3.7, you can call platform plugins (e.g., `shared_preferences`, native crypto APIs) from background isolates using `BackgroundIsolateBinaryMessenger`.

```dart
import 'dart:isolate';
import 'package:flutter/services.dart';
import 'package:shared_preferences/shared_preferences.dart';

void main() {
  // Must capture token on the main isolate before spawning
  final RootIsolateToken token = RootIsolateToken.instance!;
  Isolate.spawn(_isolateMain, token);
}

Future<void> _isolateMain(RootIsolateToken token) async {
  // Register BEFORE using any platform plugins
  BackgroundIsolateBinaryMessenger.ensureInitialized(token);

  final prefs = await SharedPreferences.getInstance();
  print(prefs.getBool('isDebug'));
}
```

---

## ⚠️ Memory Leak Prevention (CRITICAL)

### The #1 Isolate Mistake: Forgetting to Close ReceivePort

Every `ReceivePort` creates a native resource that **MUST** be explicitly closed. Forgetting this causes memory leaks that accumulate over time.

**❌ Memory Leak (BAD):**
```dart
// DANGEROUS: ReceivePort never closed!
Future<String> badExample() async {
  final receivePort = ReceivePort();
  await Isolate.spawn(_worker, receivePort.sendPort);
  final result = await receivePort.first; // Gets result but doesn't close port
  return result as String; // LEAK: receivePort still alive!
}
```

**✅ Always Close ReceivePort (GOOD):**
```dart
Future<String> goodExample() async {
  final receivePort = ReceivePort();
  try {
    await Isolate.spawn(_worker, receivePort.sendPort);
    final result = await receivePort.first; // Auto-closes after first message
    return result as String;
  } finally {
    receivePort.close(); // Explicit close for safety
  }
}
```

### Pattern: Using `receivePort.first` for Auto-Cleanup

The `.first` getter returns the first message then **automatically closes** the port. This is the safest pattern for single-response isolates:

```dart
Future<Map<String, dynamic>> parseInBackground(String json) async {
  final port = ReceivePort();
  await Isolate.spawn(_parse, (port.sendPort, json));
  return await port.first as Map<String, dynamic>; // Auto-closes
}
```

### Pattern: Manual Cleanup for Long-Lived Workers

For isolates that receive multiple messages, track the port lifecycle explicitly:

```dart
class WorkerPool {
  final ReceivePort _responses = ReceivePort();
  Isolate? _isolate;

  Future<void> start() async {
    _isolate = await Isolate.spawn(_worker, _responses.sendPort);
  }

  Future<void> dispose() async {
    _isolate?.kill(priority: Isolate.immediate);
    _responses.close(); // MUST close to prevent leak
  }
}
```

---

## Common Mistakes & Anti-Patterns

### ❌ Mistake #1: Spawning Too Many Isolates

Each isolate has ~2MB overhead. Spawning hundreds causes resource exhaustion.

**❌ BAD: Creates 100 isolates (200MB+):**
```dart
// TERRIBLE: Spawns new isolate for each item!
for (var i = 0; i < 100; i++) {
  await compute(processImage, images[i]); // 100 spawns!
}
```

**✅ GOOD: Reuse a Worker Pool:**
```dart
// Spawn once, send 100 tasks
final worker = await Worker.spawn();
for (var image in images) {
  await worker.processImage(image); // Reuses same isolate
}
worker.close();
```

### ❌ Mistake #2: Sending Non-Serializable Data

Only primitive types and certain objects can cross isolate boundaries.

**✅ Allowed Types:**
- Primitives: `int`, `double`, `String`, `bool`, `null`
- Collections: `List`, `Map`, `Set` (of serializable types)
- Special: `SendPort`, `Capability`, `TransferableTypedData`

**❌ Forbidden (Will Throw):**
- Classes/objects (unless implementing `Sendable` in Dart 3.3+)
- Functions (except top-level/static)
- Closures

**❌ BAD:**
```dart
class User {
  final String name;
  User(this.name);
}

// CRASH: User is not serializable
await compute(_process, User('Alice'));
```

**✅ GOOD:**
```dart
// Send primitive data
await compute(_process, {'name': 'Alice'});

// In worker, reconstruct object
User _process(Map<String, dynamic> data) {
  return User(data['name'] as String);
}
```

### ❌ Mistake #3: Using `compute()` for Long-Running Tasks

`compute()` spawns and tears down the isolate on every call. For repeated work, this overhead kills performance.

**❌ BAD: Repeated `compute()` calls:**
```dart
// Spawns 50 times!
for (var i = 0; i < 50; i++) {
  await compute(heavyTask, data[i]);
}
```

**✅ GOOD: Use `Isolate.spawn()` for repeated work:**
```dart
final worker = await Worker.spawn();
for (var i = 0; i < 50; i++) {
  await worker.runTask(data[i]); // Same isolate, 50 tasks
}
worker.close();
```

---

## Best Practices for Reusing Isolates

When you need to process multiple items over time, reuse a single isolate instead of spawning repeatedly.

### Pattern: Worker Pool with Request Tracking

```dart
class Worker {
  final SendPort _commands;
  final ReceivePort _responses;
  final Map<int, Completer<Object?>> _activeRequests = {};
  int _idCounter = 0;

  Worker._(this._commands, this._responses) {
    _responses.listen((message) {
      final (id, result) = message as (int, Object?);
      _activeRequests.remove(id)?.complete(result);
    });
  }

  static Future<Worker> spawn() async {
    final receivePort = ReceivePort();
    await Isolate.spawn(_workerMain, receivePort.sendPort);
    final sendPort = await receivePort.first as SendPort;
    return Worker._(sendPort, ReceivePort()..listen(...));
  }

  Future<T> execute<T>(Object? message) {
    final id = _idCounter++;
    final completer = Completer<T>();
    _activeRequests[id] = completer;
    _commands.send((id, message));
    return completer.future;
  }

  void dispose() {
    _responses.close(); // CRITICAL: Prevent leak
  }
}
```

### When to Use Single Isolate vs Pool

| Scenario | Pattern |
|---|---|
| Process items sequentially | Single long-lived worker |
| Process items in parallel (up to CPU cores) | Worker pool (2-4 isolates) |
| UI blocking < 16ms | No isolate needed (runs fast enough) |
| Occasional heavy task | `compute()` or `Isolate.run()` |

---

## Message Passing Rules

- **Mutable objects are copied** when sent via `SendPort.send()` — mutating them in the worker does not affect the main isolate.
- **Immutable objects** (e.g., `String`, unmodifiable `Uint8List`) send a *reference* for performance.
- `Isolate.exit()` transfers ownership (zero-copy) — used internally by `Isolate.run()` and `compute()`.

---

## Limitations

| Limitation | Detail |
|---|---|
| **Web** | Isolates not supported on Flutter Web; use `compute()` as a cross-platform shim |
| **rootBundle / dart:ui** | Not accessible inside background isolates; load assets on main isolate first |
| **UI operations** | No widget or rendering calls allowed in background isolates |
| **Plugin push messages** | Cannot receive *unsolicited* messages from host platform (e.g., no Firestore listener in background isolate); you can *query* but not *subscribe* |
| **Shared mutable state** | Global variables are *copied* at spawn time — changes in the worker never reflect back |

---
