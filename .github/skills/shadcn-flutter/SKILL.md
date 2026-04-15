---
name: "building-shadcn-flutter-ui"
description: "shadcn_flutter UI component library for building accessible, themed Flutter interfaces. Use when integrating shadcn components, configuring design system theming, or building consistent widget layouts."
metadata:
  last_modified: "2026-04-01 14:35:00 (GMT+8)"
---

# Shadcn UI Architecture Guide

## Goal
Build beautiful, accessible, and highly customizable user interfaces in Flutter using the Shadcn UI design system. This skill ensures proper framework integration (Pure, Material, or Cupertino), consistent theming, and effective use of the 30+ responsive components available in the library.

## Process

### 🚀 High-Level Workflow
Implementing Shadcn UI involves setting up the design tokens first, then consuming specific components for layout and logic.

### Phase 1: Foundation & Theming
- **Install**: Add `shadcn_ui` to dependencies.
- **Initialize**: Choose between `ShadApp` (Standalone) or `ShadApp.custom` (Material/Cupertino bridge).
- **Configure**: Define `ShadColorScheme` and customize `ShadThemeData`.

### Phase 2: Component Implementation
- **Typography**: Strictly use `ShadTextTheme` styles for consistent font scaling and semantics.
- **Forms**: Use `ShadForm` and specialized `Shad*FormField` widgets for unified validation.
- **Interactions**: Leverage `Sonner`, `Dialog`, and `Sheet` for standardized overlays and user feedback.

---

## 📚 Documentation Library
Refer to these detailed guides for implementation:

- [🎨 Setup & Theming Guide](./references/setup-and-theming.md) - Framework integration, dynamic color schemes, and responsive breakpoints.
- [🧱 Component Reference](./references/components.md) - Detailed API and code examples for all 30+ Shadcn UI components.

## Constraints
- **Theme Integrity**: Never bypass the `ShadTheme` variables by hardcoding magic colors or spacing values.
- **Form Safety**: Always use `ShadForm` and unique `id`s for form fields to ensure reliable state management and validation.
- **Accessibility**: Provide proper labels and descriptions for all interactive elements using built-in properties.
- **Native Alignment**: When using Material/Cupertino integration, always wrap with `ShadAppBuilder` to ensure overlays (Toasts/Dialogs) render correctly.
