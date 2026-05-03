# Summary: 04 — System Prompt Engineering: The Static/Dynamic Boundary

## Overview
Examines how Claude Code constructs its system prompt from static and dynamic sections, explains the `SYSTEM_PROMPT_DYNAMIC_BOUNDARY` marker that separates cacheable from non-cacheable content, and covers the escape hatch for injecting uncached content.

## Key Points
- **Static/dynamic boundary**: The system prompt is split at `SYSTEM_PROMPT_DYNAMIC_BOUNDARY`. Everything before the marker is stable across turns and eligible for prompt caching; everything after contains per-session dynamic data (working directory, current time, tool list).
- **`DANGEROUS_uncachedSystemPromptSection`**: An escape hatch for injecting content that must always be fresh (e.g., real-time environment state) without poisoning the cache of the static prefix.
- **Layered assembly**: The system prompt is built by composing named sections (`corePersonality`, `toolDescriptions`, `memoryInstructions`, etc.) in a defined order, making individual sections independently testable and replaceable.
- **Tool description injection**: Each tool contributes its own prompt section via `Tool.prompt`; the registry merges these into the system prompt at build time, keeping tool descriptions co-located with tool logic.
- **Session-level memoization**: Static sections are computed once per session and cached; only dynamic sections are regenerated per turn, minimizing redundant string operations.

## Transferable Patterns
1. **Explicit cache boundary marker**: Place a sentinel string in your system prompt to document exactly where the cacheable prefix ends; test that the static section never changes between turns.
2. **Compose prompts from named sections**: Build system prompts as an ordered list of independently maintainable sections rather than one monolithic string; this enables unit-testing individual instructions.
3. **Keep tool descriptions next to tool logic**: Have each tool export its own prompt text so that when the tool changes, its description stays in sync automatically.
