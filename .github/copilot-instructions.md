# Copilot Instructions for `flutter_agent_panel`

## Build, lint, and test commands

### Root app (desktop Flutter)
```bash
fvm flutter pub get
fvm dart run lean_builder build --delete-conflicting-outputs
fvm flutter gen-l10n
fvm dart format --set-exit-if-changed .
fvm flutter analyze
fvm flutter build windows --release
fvm flutter build macos --release
fvm flutter build linux --release
```

### Pre-push checks used by repository hook
```bash
fvm flutter analyze
fvm dart format --set-exit-if-changed lib/
```

### Tests
- Root `test/` is currently empty.
- Active tests are in `packages/xterm/test/`.

```bash
# Run all xterm tests
cd packages/xterm
fvm flutter test

# Run a single test file
fvm flutter test test/src/core/escape/parser_test.dart
```

## High-level architecture

- App bootstrap (`lib/main.dart`) initializes config/log directories, crash logging, app logger, global Flutter/zone error handlers, `window_manager`, and HydratedBloc storage at `~/.flutter-agent-panel/storage` before `runApp`.
- App composition (`lib/app.dart`) uses three core blocs:
  - `WorkspaceBloc` (Hydrated): persisted workspaces and serialized `TerminalConfig`.
  - `SettingsBloc` (Hydrated): persisted theme/font/locale/shell/agent settings.
  - `TerminalBloc` (non-hydrated): live PTY + terminal runtime state.
- Routing (`lib/core/router/app_router.dart`) is nested: `AppShellView (/)` → `WorkspaceWrapperView (/workspace)` → `WorkspaceView (/:workspaceId)` → `TerminalView (terminal/:terminalId)`.
- Cross-feature runtime flow:
  - `WorkspaceView` issues `SyncWorkspaceTerminals` from persisted workspace configs.
  - `TerminalBloc` creates/disposes/restarts `TerminalNode` (xterm `Terminal` + `IsolatePty`) and tracks pending/restarting/error states.
  - `WorkspaceBloc` stays as source of truth for terminal metadata; `TerminalBloc` owns non-serializable runtime objects.
- Repo is a Flutter workspace with local packages:
  - `packages/flutter_pty`: native PTY bindings (ConPTY/POSIX).
  - `packages/xterm`: terminal emulation/rendering engine.

## Key conventions specific to this repository

- Do not edit generated files directly:
  - `lib/core/router/app_router.gr.dart` (regenerate with `fvm dart run lean_builder build`)
  - `lib/core/l10n/app_localizations*.dart` (regenerate with `fvm flutter gen-l10n`)
  - `packages/flutter_pty/lib/src/flutter_pty_bindings_generated.dart` (regenerate with `fvm flutter pub run ffigen --config ffigen.yaml` from `packages/flutter_pty`)
  - `*.mocks.dart` files
- State updates are immutable: copy list/map first, then emit with `copyWith`; avoid direct list mutation in bloc handlers.
- `AppSettings.copyWith` uses explicit clear flags (`clearCustomTerminalThemeJson`, `clearSelectedCustomShellId`, `clearAppFontFamily`) for nullable fields.
- UI code relies on context extensions (`context.t`, `context.theme`, `context.colorScheme`) from `lib/core/extensions/context_extension.dart`.
- UI strings should go through localization (`assets/l10n/*.arb` + `fvm flutter gen-l10n`), not hardcoded text.
- Layout/style conventions used across feature guides:
  - use `Gap` instead of `SizedBox` for spacing
  - use `flutter_screenutil` (`.w/.h/.sp/.r`) for fixed dimensions
- Terminal spawning conventions are centralized in `TerminalBloc._createTerminalNode`:
  - environment merge includes `TERM=xterm-256color` and `COLORTERM=truecolor`
  - shell-specific wrapping differs for PowerShell / cmd / WSL / bash
  - PTY I/O runs through `IsolatePty` (avoid blocking main isolate)
- User config and runtime artifacts are stored under `~/.flutter-agent-panel` (themes, schema, logs, hydrated storage).
- Import style in existing project docs: prefer relative imports inside feature folders; use absolute imports for `core/` and `shared/`.
