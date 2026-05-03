# Summary: 25 — Architecture Patterns Summary: Seven Transferable Patterns from Claude Code

## Overview
Synthesizes the entire 25-chapter series into seven transferable architectural patterns distilled from Claude Code's production codebase, each with a concrete implementation recipe applicable to any AI-integrated application.

## Key Points
- **Pattern 1 — Compile-time DCE via `feature()` + `require()`**: Gate optional features with a constant-checked `require()` so the bundler eliminates dead code at build time; produces zero-overhead build variants without runtime boolean checks.
- **Pattern 2 — Layered startup with fast paths**: Structure initialization as layers (bootstrap → CLI parse → optional UI → full session); return at the earliest layer that satisfies the request; never load what isn't needed.
- **Pattern 3 — Minimal reactive Store with `useSyncExternalStore`**: A 35-line custom store satisfies React 18's tear-free rendering contract without Redux or Zustand; only reach for a library when you need its specific features (devtools, middleware).
- **Pattern 4 — `onChange` side-effects for derived state**: React to state changes via `onChange` subscriptions rather than computing derived state inline; this decouples the trigger from the effect and supports async side-effects cleanly.
- **Pattern 5 — Context isolation for sub-agents**: Every sub-agent invocation gets a fresh context object (conversation history, tool subset, permissions); shared mutable state between parent and child agents causes subtle, hard-to-reproduce bugs.
- **Pattern 6 — Deferred tool loading**: Publish tool metadata (name, description, schema) in a manifest; load implementations only when first invoked; this keeps startup memory and system prompt size proportional to what the user actually uses.
- **Pattern 7 — Defense-in-depth for AI actions**: Layer static rules → programmatic analysis → AI classification → user confirmation for any action with side effects; never rely on a single gate; the cheapest gate goes first.

## Transferable Patterns
1. **Apply patterns at the right layer**: Each pattern targets a specific concern — build size, startup latency, rendering, state, security — resist applying a pattern beyond its intended layer.
2. **Prefer proven primitives over frameworks**: `useSyncExternalStore`, `AsyncGenerator`, `require()` — Claude Code's patterns are built on stable language and platform primitives, not framework-specific abstractions.
3. **Security as a layered system, not a checklist**: Defense-in-depth means each layer assumes the previous layer may fail; design every security-sensitive path to be safe even if upstream checks are bypassed.
