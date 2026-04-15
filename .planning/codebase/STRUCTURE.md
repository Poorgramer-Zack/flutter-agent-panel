# Codebase Structure

**Analysis Date:** 2026-04-15

## Directory Layout

```
flutter_agent_panel/
├── lib/
│   ├── main.dart                  # Entry point with error handling
│   ├── app.dart                   # Root widget, MultiBlocProvider, theming
│   ├── core/                      # Cross-cutting concerns
│   │   ├── services/              # Singleton services
│   │   ├── router/                # AutoRoute config
│   │   ├── l10n/                  # Generated localizations
│   │   ├── extensions/            # Context extensions
│   │   ├── constants/            # Asset paths
│   │   └── types/                 # Type definitions
│   ├── features/                  # Feature modules (BLoC pattern)
│   │   ├── home/                  # App shell layout
│   │   ├── info/                  # Help/About dialogs
│   │   ├── settings/              # App configuration
│   │   ├── terminal/              # PTY management, terminal rendering
│   │   └── workspace/             # Multi-workspace organization
│   └── shared/                    # Reusable widgets and utilities
├── packages/                      # Local Flutter packages
│   ├── flutter_pty/               # Native PTY FFI bindings
│   └── xterm/                     # Terminal emulator core
├── assets/                        # Images, agent logos
└── .planning/codebase/            # This documentation
```

## Directory Purposes

**lib/core/:**
- Purpose: Infrastructure and cross-cutting services
- Contains: Services (singleton pattern), Router (AutoRoute), L10n (generated), Extensions
- Key files: `app_router.dart`, `app_logger.dart`, `context_extension.dart`

**lib/features/:**
- Purpose: Business domain modules
- Structure per feature: `bloc/`, `models/`, `views/`, `widgets/`, `services/`
- Features: home, info, settings, terminal, workspace

**lib/shared/:**
- Purpose: Code shared across features
- Contains: Common widgets, utilities, constants
- Key files: `app_scaffold.dart`, `settings_helpers.dart`, `system_fonts.dart`

**packages/:**
- Purpose: Local Flutter packages (workspace monorepo)
- xterm: Forked/customized terminal emulator with 60fps rendering
- flutter_pty: Native PTY bindings via FFI (ConPTY on Windows, POSIX on Unix)

## Key File Locations

**Entry Points:**
- `lib/main.dart`: App bootstrap, error handling, HydratedStorage init
- `lib/app.dart`: Widget tree root, MultiBlocProvider, ShadApp theming

**Configuration:**
- `lib/core/router/app_router.dart`: Route definitions
- `lib/features/settings/models/app_settings.dart`: Settings model
- `pubspec.yaml`: Dependencies and workspace config

**Core Logic:**
- `lib/features/workspace/bloc/workspace_bloc.dart`: Workspace persistence
- `lib/features/terminal/bloc/terminal_bloc.dart`: PTY lifecycle management
- `lib/features/settings/bloc/settings_bloc.dart`: App settings

**Testing:**
- Test files co-located with feature (e.g., `lib/features/workspace/**/*_test.dart`)

## Naming Conventions

**Files:**
- Feature folders: lowercase with underscores (e.g., `workspace`, `terminal`)
- Dart files: snake_case (e.g., `workspace_bloc.dart`, `terminal_config.dart`)
- BLoC files: `{feature}_{name}.dart` (e.g., `workspace_event.dart`)
- Generated files: `*.gr.dart`, `*.freezed.dart`, `*_test.dart`

**Directories:**
- Feature directories: lowercase with underscores
- BLoC subdir: `bloc/`
- Models subdir: `models/`
- Views subdir: `views/`
- Widgets subdir: `widgets/`

**Classes:**
- BLoC: `{Feature}Bloc`, `{Feature}Event`, `{Feature}State`
- Models: PascalCase (e.g., `Workspace`, `TerminalConfig`)
- Views: `{Name}View`, `{Name}Dialog`, `{Name}Page`
- Widgets: PascalCase (e.g., `WorkspaceDrawer`, `TerminalSearchBar`)

## Where to Add New Code

**New Feature Module:**
- Create directory: `lib/features/{new_feature}/`
- Add subdirectories: `bloc/`, `models/`, `views/`, `widgets/`, `services/`
- Register BLoC in `lib/app.dart` MultiBlocProvider
- Add route in `lib/core/router/app_router.dart`
- Run `dart run lean_builder build` to regenerate routes

**New Model:**
- Location: `lib/features/{feature}/models/`
- Follow existing model patterns (Equatable, JSON serialization)

**New Service:**
- Location: `lib/core/services/` (if cross-cutting) or `lib/features/{feature}/services/`
- Use singleton pattern: `MyService._(); static final MyService instance = MyService._();`

**New Shared Utility:**
- Location: `lib/shared/utils/` or `lib/shared/widgets/`
- Avoid feature-specific logic

## Special Directories

**packages/xterm/:**
- Purpose: Customized terminal emulator rendering engine
- Generated: No (committed to repo)
- Committed: Yes

**packages/flutter_pty/:**
- Purpose: Native PTY FFI bindings
- Generated: Yes (via ffigen, `flutter_pty_bindings_generated.dart`)
- Committed: Yes, but generated files excluded from analysis

**lib/core/l10n/:**
- Purpose: Internationalization
- Generated: Yes (via `flutter gen-l10n`)
- Committed: Yes (generated .dart files)

**lib/core/router/:**
- Purpose: Type-safe routing
- Generated: Yes (via `lean_builder`)
- Committed: Yes (`app_router.gr.dart` excluded from analysis)

---

*Structure analysis: 2026-04-15*
