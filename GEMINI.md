# Flutter Agent Panel - Project Context

## Project Overview
**Name:** `flutter_agent_panel`
**Type:** Flutter Application (Desktop Focus)
**Description:** A cross-platform terminal aggregator and AI agent panel. It appears to be a Flutter implementation inspired by or migrating from the co-located `better-agent-terminal` Electron project. It features multi-workspace support, terminal emulation, and integration with AI agents.

## Architecture & Tech Stack

### Core Frameworks
- **Flutter:** SDK ^3.5.0
- **State Management:** `flutter_bloc`, `hydrated_bloc` (Persistent state)
- **Navigation:** `auto_route`
- **UI Components:** `shadcn_ui` (Shadcn port for Flutter)
- **Responsive Design:** `flutter_screenutil`
- **Localization:** `flutter_localizations` (ARB files in `assets/l10n/`)

### Key Dependencies
- **Terminal:** `xterm` (local package), `flutter_pty` (local package) for terminal emulation and pseudo-terminal management.
- **Desktop Integration:** `window_manager` for window control, `path_provider` for file system access.
- **Utilities:** `uuid`, `gap`.

### Directory Structure
The project follows a **Feature-First** architecture:

```
lib/
├── app.dart                  # Root widget, providers, global configuration
├── main.dart                 # Entry point, initialization (Window, Storage)
├── core/                     # Core utilities and shared infrastructure
│   ├── router/               # AutoRoute configuration (AppRouter)
│   ├── services/             # Global services (UserConfigService)
│   └── l10n/                 # Localization generated files
├── features/                 # Feature modules
│   ├── terminal/             # Terminal logic (Bloc, Views, Services)
│   ├── workspace/            # Workspace management
│   ├── settings/             # App settings and theme
│   ├── home/                 # Home screen
│   └── info/                 # Information/About
└── shared/                   # Shared widgets and utilities across features
```

### Internal Packages
- `packages/xterm`: Custom or vendored xterm implementation.
- `packages/flutter_pty`: PTY (Pseudo Terminal) interface for Flutter.

## Development Conventions

### Build & Run
- **Run App:** `flutter run` (Targeting Windows/Linux/macOS recommended)
- **Code Generation:** `dart run lean_builder build` (Required for `auto_route`)
- **Linting:** Follows `flutter_lints` with custom rules in `analysis_options.yaml` (e.g., `prefer_single_quotes`, `always_declare_return_types`).

### Coding Style
- **Pattern:** BLoC pattern for logic. UI talks to BLoC, BLoC talks to Services/Repositories.
- **Theming:** Uses `ShadThemeData` from `shadcn_ui`, can access via `context.theme` extension.
- **Localization:** Use `context.t` or `AppLocalizations.of(context)` for strings.
- **Imports:** Prefer relative imports within features, absolute imports for core/shared.
- **Gap over SizedBox:** Use `Gap` instead of `SizedBox` for spacing in `Column`, `Row`, and `ListView.separated`.
  ```dart
  // Avoid
  Column(
    children: [
      Text('Top'),
      SizedBox(height: 10),
      Text('Bottom'),
    ],
  )

  // Preferred
  Column(
    children: [
      Text('Top'),
      const Gap(10),
      Text('Bottom'),
    ],
  )
  ```
- **Responsive Design:** Use `flutter_screenutil` extensions (`.w`, `.h`, `.sp`, `.r`) for responsive sizing.
  ```dart
  Container(
    width: 100.w,
    height: 50.h,
    padding: EdgeInsets.all(8.r),
    child: Text(
      'Label',
      style: TextStyle(fontSize: 16.sp),
    ),
  )
  ```
- **Widgets over Methods:** Prefer creating a `Widget` class over a `_buildWidget` helper method for better performance and readability.
  ```dart
  // Avoid
  Widget _buildProfile() {
    return Column(...);
  }

  // Preferred
  class ProfileSection extends StatelessWidget {
    @override
    Widget build(BuildContext context) {
      return Column(...);
    }
  }
  ```

## Common Commands
- **Analyze:** `flutter analyze`
- **Format:** `dart format .`
- **Fix:** `dart fix --apply`

## Note
When modifying terminal behavior, check `packages/flutter_pty` and `packages/xterm` if the changes involve low-level emulation or process management. For UI/UX changes, refer to `lib/features/terminal`.
Always launch the app with `dart_mcp_server_launch_app` and call `dart_mcp_server_hot_reload` or `dart_mcp_server_hot_restart` to apply the changes.

## External documents
- [shadcn_ui](https://flutter-shadcn-ui.mariuti.com/llms.txt)