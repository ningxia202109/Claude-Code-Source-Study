# Summary: 06 — Context Management: Token Budget, Compaction, and the Circuit Breaker

## Overview
Explains how Claude Code monitors the conversation context window, triggers compaction when the token budget runs low, and uses a circuit breaker to prevent infinite compaction loops — along with the file state cache that makes re-reads cheap.

## Key Points
- **Token budget functions**: A set of pure functions compute remaining tokens, warn thresholds, and hard limits; these are called before every API request to decide whether compaction is needed.
- **Microcompact**: A lightweight summarization pass that truncates older tool results and collapses repeated assistant/user pairs, preserving recent context while reducing token count by ~30%.
- **Full Compact**: A heavier summarization that sends the entire conversation to the model with a "summarize this" instruction, then replaces the history with the summary — triggered when Microcompact is insufficient.
- **Circuit breaker**: Tracks consecutive compaction attempts; if compaction fails to reduce tokens below the threshold after N tries, it raises an error instead of looping indefinitely.
- **`FileStateCache`**: Caches file contents with modification-time keys so that repeated reads of unchanged files (e.g., CLAUDE.md) skip disk I/O, reducing latency during context reconstruction.
- **Progressive degradation**: The system tries Microcompact → Full Compact → error in sequence, always attempting the least-destructive option first.

## Transferable Patterns
1. **Budget-gated compaction**: Check token count before every API call; trigger compaction proactively rather than waiting for a context-length error from the server.
2. **Circuit breaker on recursive operations**: Any operation that can trigger itself (compact → still too large → compact again) needs an iteration counter and a hard stop condition.
3. **Cache external reads with mtime keys**: Use `{path, mtime}` as a cache key for file reads; this gives you correct invalidation without polling or filesystem watchers.
