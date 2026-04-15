# Architecture

**Analysis Date:** 2026-04-15

## Pattern Overview

**Overall:** Feature-based Clean Architecture with BLoC state management

**Key Characteristics:**
- Feature modules organized by business domain (workspace, terminal, settings)
- Persistent state via HydratedBloc for workspace/settings
- Ephemeral state via regular Bloc for terminal instances
- Type-safe routing via AutoRoute
- UI built with shadcn_ui components

## Layers

**Feature Layer:**
- Purpose: Business logic and UI organized by domain
- Location: `lib/features/{feature_name}/`
- Contains: bloc/, models/, views/, widgets/, services/
- Depends on: Core services, shared utilities
- Used by: App shell via routing

**Core Layer:**
- Purpose: Cross-cutting concerns (routing, services, l10n, extensions)
- Location: `lib/core/`
- Contains: services/, router/, l10n/, extensions/, constants/, types/
- Depends on: Nothing (base layer)
- Used by: All features

**Shared Layer:**
- Purpose: Reusable utilities and common widgets
- Location: `lib/shared/`
- Contains: widgets/, utils/, constants/
- Depends on: Core
- Used by: Features as needed

## Data Flow

**App Initialization:**
```
main.dart
  → HydratedStorage init (persistence)
  → WindowManager init (desktop)
  → Services init (logging, crash handling)
  → runApp(App)
    → MultiBlocProvider (WorkspaceBloc, SettingsBloc, TerminalBloc)
    → ShadApp.router with AutoRoute
    → AppShell (home route)
```

**Workspace → Terminal Relationship:**
```
WorkspaceBloc (HydratedBloc)
  └─ stores List<Workspace>
      └─ each Workspace contains List<TerminalConfig>

TerminalBloc (regular Bloc)
  └─ manages Map<String, TerminalNode> (PTY instances)
  └─ TerminalNode = Pty + Terminal + TerminalController bundle
```

**Routing Hierarchy:**
```
AppShellRoute /
  └─ WorkspaceWrapperRoute /workspace
      └─ WorkspaceRoute /workspace/:workspaceId
          └─ TerminalRoute /workspace/:workspaceId/terminal/:terminalId
```

## Key Abstractions

**HydratedBloc Pattern:**
- Used for: WorkspaceBloc, SettingsBloc (persistent state)
- Serialization: `fromJson()` / `toJson()` methods
- Storage: `HydratedStorage.build()` with config directory

**TerminalNode:**
- Purpose: Bundles native PTY process with terminal emulator
- Contains: Pty (process), Terminal (buffer), TerminalController (UI bridge)
- Files: `lib/features/terminal/models/terminal_node.dart`

**Workspace Model:**
- Purpose: Represents a collection of terminal sessions
- Contains: id, name, icon, tags, path, isPinned, terminals list
- File: `lib/features/workspace/models/workspace.dart`

## Entry Points

**main.dart:**
- Location: `lib/main.dart`
- Triggers: App launch
- Responsibilities: Error handling setup, HydratedBloc storage init, window config, service initialization

**app.dart:**
- Location: `lib/app.dart`
- Triggers: Run from main()
- Responsibilities: MultiBlocProvider setup, ShadApp theming, locale/l10n, router config

**AppShell:**
- Location: `lib/features/home/views/app_shell.dart`
- Triggers: Root route `/`
- Responsibilities: Main app layout scaffold

## Error Handling

**Strategy:** Multi-layered error catching

**Patterns:**
- `runZonedGuarded()` for zone-level async error catching
- `FlutterError.onError` for Flutter framework errors
- `PlatformDispatcher.instance.onError` for platform errors
- `AppBlocObserver` for BLoC event/transition logging
- CrashLogService persists errors to disk

## Cross-Cutting Concerns

**Logging:** AppLogger singleton wrapping `logger` package
**Localization:** flutter_localizations + ARB files, generated via `flutter gen-l10n`
**Routing:** AutoRoute with generated `app_router.gr.dart`
**Theming:** Shadcn_ui ShadThemeData with light/dark modes

---

*Architecture analysis: 2026-04-15*
