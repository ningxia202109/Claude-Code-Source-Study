# Summary: 05 — Dialog Loop: The AsyncGenerator State Machine at the Heart of Claude Code

## Overview
Dissects `query.ts`, the central dialog loop that drives every conversation turn — from message preprocessing and streaming API calls to tool execution, error recovery, and the 7+ `continue` paths that keep the loop alive across failures.

## Key Points
- **AsyncGenerator architecture**: `query()` is implemented as an `async function*` that yields `AssistantMessage` chunks to the caller (the REPL), allowing the UI to stream tokens while the loop manages state internally.
- **Preprocessing pipeline**: Each user message passes through an ordered pipeline of transformers (mention expansion, file injection, context trimming) before being sent to the API.
- **Tool execution cycle**: When the model returns `tool_use` blocks, `query.ts` executes each tool, collects `tool_result` blocks, appends them to the conversation, and loops back to the API — fully transparent to the caller.
- **7+ recovery `continue` paths**: Handles `overloaded_error`, `rate_limit_error`, `context_length_exceeded`, empty response, interrupted streams, and more — each with a specific recovery strategy (retry, compact, truncate, resume).
- **Streaming vs. non-streaming dual mode**: Supports both Server-Sent Events streaming (default) and non-streaming (fallback/batch mode) through the same generator interface, hiding the transport difference from callers.
- **Turn metadata tracking**: Collects per-turn metrics (input tokens, output tokens, cache hits, tool calls) and emits them as a final yield at the end of each turn for logging and display.

## Transferable Patterns
1. **Model the dialog loop as an AsyncGenerator**: Yields let you stream partial results to the UI while keeping all state-machine logic inside a single function — cleaner than callbacks or event emitters.
2. **Enumerate and handle every failure mode explicitly**: List all API error codes your system can encounter and give each a named recovery path; don't use a single catch-all retry.
3. **Separate transport from conversation logic**: The streaming/non-streaming duality lives in a thin adapter layer; `query.ts` itself is transport-agnostic.
