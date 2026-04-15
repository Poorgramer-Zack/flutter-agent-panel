---
name: "flutter-http-and-json"
description: "Make HTTP requests and encode / decode JSON in a Flutter app"
metadata:
  last_modified: "2026-03-12 11:18:17 (GMT+8)"

---
# flutter-http-json-networking

## Goal
Manages HTTP networking and JSON data handling in Flutter applications. Implements secure, asynchronous REST API calls (GET, POST, PUT, DELETE) using the `dio` package. `dio` is the absolute standard networking framework offering robust interceptors, global configurations, and form-data uploads which basic `http` intrinsically severely lacks. Handles JSON serialization, background parsing via isolates for large datasets, and structured JSON schemas for AI model integrations. Assumes the `dio` package is added to `pubspec.yaml` and the environment supports Dart 3 pattern matching and null safety.

## Decision Logic
When implementing JSON parsing and serialization, evaluate the following decision tree:
1. **Payload Size:** 
   * If the JSON payload is small, parse synchronously on the main thread.
   * If the JSON payload is large (takes >16ms to parse), use background parsing via `compute()` to avoid UI jank.
2. **Model Complexity:**
   * If the data model is simple or a quick prototype, use manual serialization (`dart:convert`).
   * If the data model is highly nested or part of a large production app, **STOP AND ASK THE USER:** "Should we configure `json_serializable` and `build_runner` for automated code generation?"

## Instructions

### 1. Configure Platform Permissions
Before making network requests, ensure the target platforms have the required internet permissions.

**Android (`android/app/src/main/AndroidManifest.xml`):**
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- Required to fetch data from the internet. -->
    <uses-permission android:name="android.permission.INTERNET" />
    <application ...>
</manifest>
```

**macOS (`macos/Runner/DebugProfile.entitlements` and `Release.entitlements`):**
```xml
<dict>
    <!-- Required to fetch data from the internet. -->
    <key>com.apple.security.network.client</key>
    <true/>
</dict>
```

### 2. Define the JSON Data Model
Create a strongly typed Dart class to represent the JSON data. Use factory constructors for deserialization and a `toJson` method for serialization.

```dart
import 'dart:convert';

class ItemModel {
  final int id;
  final String title;

  const ItemModel({required this.id, required this.title});

  // Deserialize using Dart 3 pattern matching
  factory ItemModel.fromJson(Map<String, dynamic> json) {
    return switch (json) {
      {'id': int id, 'title': String title} => ItemModel(id: id, title: title),
      _ => throw const FormatException('Failed to parse ItemModel.'),
    };
  }

  // Serialize to JSON
  Map<String, dynamic> toJson() => {
        'id': id,
        'title': title,
      };
}
```

### 3. Implement HTTP Operations (CRUD)
Use the `dio` package to perform network requests. `dio` excels because it automatically handles JSON decoding natively, eliminating exhaustive manual `jsonDecode` parsing boilerplate universally.

```dart
import 'package:dio/dio.dart';

class ApiService {
  final Dio client;

  // 🌟 Best Practice: Instantiate Dio with base global configurations
  ApiService() : client = Dio(
    BaseOptions(
      baseUrl: 'https://api.example.com',
      connectTimeout: const Duration(seconds: 5),
      receiveTimeout: const Duration(seconds: 3),
      headers: {'Accept': 'application/json'},
    ),
  ) {
    // 🌟 Optional Interceptor integrations mapping tokens globally
    client.interceptors.add(LogInterceptor(responseBody: true));
  }

  // GET Request
  Future<ItemModel> fetchItem(int id) async {
    try {
      final response = await client.get('/items/$id');
      // 🌟 Dio natively parses JSON; response.data is already a Map<String, dynamic>
      return ItemModel.fromJson(response.data as Map<String, dynamic>);
    } on DioException catch (e) {
      throw Exception('Failed fetching item: ${e.message}');
    }
  }

  // POST Request
  Future<ItemModel> createItem(String title) async {
    try {
      final response = await client.post(
        '/items',
        data: {'title': title}, // 🌟 Dio natively serializes Maps to JSON bodies automatically
      );
      return ItemModel.fromJson(response.data as Map<String, dynamic>);
    } on DioException catch (e) {
      throw Exception('Failed creating item: ${e.message}');
    }
  }

  // DELETE Request
  Future<void> deleteItem(int id) async {
    try {
      await client.delete('/items/$id');
    } on DioException catch (e) {
      throw Exception('Failed deleting item: ${e.message}');
    }
  }
}
```

### 4. Implement Background Parsing for Large JSON Arrays
If fetching a large list of objects, move the JSON decoding and mapping sequentially to a separate isolate using `compute()`. Since `dio` automatically decodes JSON, we can configure interceptors to outsource this parsing to isolates seamlessly, keeping the main thread permanently fluid.

```dart
import 'package:flutter/foundation.dart';
import 'package:dio/dio.dart';

// 1. Establish the Top-level isolated decoding function 
_parseAndDecode(String response) {
  return jsonDecode(response);
}

_parseJson(String text) {
  return compute(_parseAndDecode, text);
}

void setupDioWithIsolate(Dio dio) {
  // 🌟 Inject the computing background function entirely restructuring Dio's default parser
  (dio.transformer as DefaultTransformer).jsonDecodeCallback = _parseJson;
}

// 2. Map the resulting isolated collections
Future<List<ItemModel>> fetchLargeItemList(Dio client) async {
  try {
    final response = await client.get('/items');
    // response.data is ALREADY mapped via the isolate background parser flawlessly
    final parsedList = (response.data as List<dynamic>).cast<Map<String, dynamic>>();
    return parsedList.map<ItemModel>(ItemModel.fromJson).toList();
  } on DioException catch (e) {
    throw Exception('Failed to load vast items array: ${e.message}');
  }
}
```

### 5. Define Structured JSON Output for AI Models
When integrating LLMs (like Gemini), enforce reliable JSON output by passing a strict schema in the generation configuration and system instructions.

```dart
import 'package:firebase_vertexai/firebase_vertexai.dart';

// Define the expected JSON schema
final _responseSchema = Schema(
  SchemaType.object,
  properties: {
    'width': Schema(SchemaType.integer),
    'height': Schema(SchemaType.integer),
    'items': Schema(
      SchemaType.array,
      items: Schema(
        SchemaType.object,
        properties: {
          'id': Schema(SchemaType.integer),
          'name': Schema(SchemaType.string),
        },
      ),
    ),
  },
);

// Initialize the model with the schema
final model = FirebaseAI.googleAI().generativeModel(
    generationConfig: GenerationConfig(
    responseMimeType: 'application/json',
    responseSchema: _responseSchema,
  ),
);

Future<Map<String, dynamic>> analyzeData(String prompt) async {
  final content = [Content.text(prompt)];
  final response = await model.generateContent(content);
  
  // Safely decode the guaranteed JSON response
  return jsonDecode(response.text!) as Map<String, dynamic>;
}
```

## Constraints
* **Immutable Base Configurations:** Always utilize `BaseOptions` inside `Dio` defining default timeouts preventing infinite hanging request loops explicitly.
* **Error Handling:** Never return `null` on a failed network request. Always intercept throwing a `DioException` natively handling specific cases (`e.type == DioExceptionType.connectionTimeout`) rendering accurate UI error displays.
* **Status Code Validation:** Dio naturally throws exceptions bridging failing status architectures, removing generic `if (statusCode == 200)` boilerplate intrinsically.
* **Library Restriction:** Do not use `dart:io` `HttpClient` nor the primitive `http` package directly for standard modern app networking; universally deploy `dio` cementing global enterprise scalability seamlessly.
