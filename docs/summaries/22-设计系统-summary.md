# Summary: 22 — Design System: 80+ Semantic Tokens, 6 Themes, and the Tool UI Protocol

## Overview
Describes the terminal design system that unifies Claude Code's visual language — a `Theme` type with 80+ semantic color tokens, six built-in themes, fifteen shared components, and the Tool UI protocol that lets each tool control its own rendering.

## Key Points
- **`Theme` type**: An 80+ field object mapping semantic names (e.g., `toolResultBorder`, `warningText`, `successBackground`) to ANSI color codes; components consume semantic tokens, never raw ANSI codes directly.
- **6 built-in themes**: `default`, `dark`, `light`, `solarized-dark`, `solarized-light`, `high-contrast` — each is a full `Theme` object; switching themes is a single context value change with no component rewrites.
- **15 shared components**: Includes `Box`, `Text`, `Badge`, `Spinner`, `ProgressBar`, `CodeBlock`, `Divider`, `StatusLine`, and others — all built on top of the forked Ink primitives and styled via the Theme context.
- **Tool UI protocol**: Each tool in the `Tool` interface implements `renderToolUse()` (shows what the tool is about to do) and `renderToolResult()` (shows the output) — 10 render methods total covering normal, error, compact, and streaming variants.
- **Theme context**: The active `Theme` is provided via React context; any component in the tree can access it via `useTheme()` without prop drilling, following the standard React context pattern.
- **Responsive layout**: Components use Yoga flexbox to adapt to the terminal width; narrow terminals collapse multi-column layouts to single-column automatically.

## Transferable Patterns
1. **Semantic tokens over raw values**: Define a `Theme` object with named semantic tokens; never hardcode colors or sizes in components — this makes theming a single-file change.
2. **Co-locate tool rendering with tool logic**: Implement `renderToolUse()` and `renderToolResult()` methods on the tool object itself so that when tool behavior changes, its UI representation changes with it.
3. **Theme via React context, not props**: Provide the active theme as a context value at the root; components access it via `useTheme()` — prop-drilling themes through 5+ component layers is always a mistake.
