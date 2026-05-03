# Summary: 03 — State Management: A Three-Layer Reactive Architecture

## Overview
Describes Claude Code's three-layer state management architecture — bootstrap constants, reactive session state, and ephemeral tool context — and explains why a custom 35-line Store outperforms Redux/Zustand for this use case.

## Key Points
- **Three layers**: (1) `bootstrap/state.ts` — immutable constants set at process start (paths, user info, initial config); (2) `Store + AppState` — reactive mutable session state using a custom observable; (3) `ToolUseContext` — ephemeral per-tool-call context passed as function arguments.
- **Custom Store (~35 lines)**: Implements the `useSyncExternalStore` contract with `getSnapshot()`, `subscribe()`, and `setState()`. No external dependency, no boilerplate, zero overhead from selectors or middleware.
- **`useSyncExternalStore` integration**: React 18's built-in hook ensures tear-free reads during concurrent rendering; the custom Store hooks directly into this protocol without a library.
- **Immutability at the right layer**: Bootstrap state is frozen after initialization; AppState updates are done via shallow-merge `setState()` to trigger re-renders; ToolUseContext is never stored globally — it flows down the call stack.
- **Avoiding over-reactivity**: Components subscribe only to the slice of state they read; coarse subscriptions are avoided to prevent unnecessary re-renders in the terminal UI.

## Transferable Patterns
1. **Classify state before choosing a store**: Distinguish immutable bootstrap data (plain object), reactive shared state (custom Store or Zustand), and per-call context (function parameter) — each layer deserves a different solution.
2. **Roll a minimal Store when libraries add overhead**: A 35-line store that satisfies `useSyncExternalStore` is sufficient for most React apps; reach for Redux/Zustand only when you need time-travel, devtools, or middleware.
3. **Pass ephemeral context as arguments, not globals**: Tool execution context (current working directory, permission state, conversation turn) lives on the call stack, not in a global store, preventing subtle concurrency bugs.
