# Summary: 08 — Thinking & Reasoning Control: Adaptive Depth from Ultrathink to Disabled

## Overview
Explains how Claude Code controls the model's extended thinking capability through a `ThinkingConfig` with three modes (adaptive/enabled/disabled), four effort levels, the special "ultrathink" keyword, and an Advisor server-tool pattern for lightweight reasoning.

## Key Points
- **`ThinkingConfig` modes**: `adaptive` (default — enables thinking when the query is judged complex), `enabled` (always on), `disabled` (always off, for speed-sensitive paths like autocompletion).
- **Effort levels**: Four levels (low/medium/high/max) map to increasing `budget_tokens` values; higher effort gives the model more thinking tokens but increases latency and cost.
- **Ultrathink keyword**: When the user types "ultrathink" in their message, Claude Code detects it and upgrades the current turn to `max` effort, effectively asking the model to think as deeply as possible.
- **Adaptive complexity heuristics**: In `adaptive` mode, a set of heuristics (message length, presence of code blocks, tool invocation history) estimates task complexity to choose an effort level automatically.
- **Advisor server-tool pattern**: For tasks where full extended thinking is too expensive, a lightweight "advisor" tool sends a separate, smaller model call to get a reasoned recommendation — a cheaper form of structured reasoning.
- **Thinking token budget isolation**: Thinking tokens are counted separately from the conversation context budget so that extended reasoning doesn't crowd out tool results or file contents.

## Transferable Patterns
1. **Gate extended thinking on complexity signals**: Don't pay for thinking on every turn; classify query complexity with cheap heuristics and enable thinking only above a threshold.
2. **Expose a "max effort" keyword**: Give power users a simple token ("ultrathink", "think harder") that overrides the default effort level without requiring a settings change.
3. **Use a smaller advisor model for structured reasoning**: When a full thinking turn is too expensive, a separate call to a smaller model with a structured reasoning prompt can approximate the benefit at lower cost.
