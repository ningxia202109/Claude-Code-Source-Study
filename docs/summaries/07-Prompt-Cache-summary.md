# Summary: 07 — Prompt Cache: Byte-Exact Stability and the Latch Pattern

## Overview
Details Claude Code's prompt caching strategy — how `cache_control` markers are placed, how byte-exact stability is enforced across turns, how fork prompts are threaded for sub-agents, and the latch pattern used to prevent cache invalidation.

## Key Points
- **`cache_control` placement**: Breakpoints are inserted at the end of the static system prompt prefix and at the end of large stable message blocks (e.g., file contents injected early in the conversation).
- **`CacheSafeParams`**: A wrapper type that enforces that any value placed before a cache breakpoint passes a byte-exact stability check — if the value could change between turns, the type system rejects it at compile time.
- **Byte-exact stability requirement**: Anthropic's cache matches on exact byte sequences; even a single character change invalidates the cached prefix. Claude Code uses frozen objects and content-addressed keys to guarantee stability.
- **Fork prompt threading**: When a sub-agent is spawned, it inherits a "fork prompt" snapshot of the parent's stable system prompt prefix so the cache hit carries over to the sub-agent's first call.
- **Latch pattern**: Once a cache breakpoint is written into a message, it is never moved or removed during the session — this is the "latch." Moving a breakpoint would invalidate all downstream cache entries.
- **Cache hit metrics**: Per-turn metadata tracks `cache_read_input_tokens` and `cache_creation_input_tokens`; these are surfaced in the status line so engineers can verify cache effectiveness.

## Transferable Patterns
1. **Type-enforce cache stability**: Create a `CacheSafe<T>` wrapper that only accepts values known to be byte-stable; don't rely on runtime assertions alone.
2. **Latch breakpoints, never move them**: Treat cache breakpoint positions as append-only commitments for the session lifetime; redesign your prompt structure so breakpoints never need to move.
3. **Propagate cache context to sub-agents**: When forking a conversation, pass the parent's stable prefix snapshot so the child's first API call benefits from the parent's cache warm-up.
