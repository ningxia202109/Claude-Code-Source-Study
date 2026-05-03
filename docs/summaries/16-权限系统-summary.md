# Summary: 16 — Permission System: 7 Modes, Rule Engine, and AI Classifier

## Overview
Details Claude Code's multi-layered permission system — seven permission modes, a rule engine for declarative allow/deny lists, and an AI-based classifier for ambiguous cases — all composing into a defense-in-depth architecture.

## Key Points
- **7 permission modes**: `default` (prompt for writes), `plan` (read-only, shows plan before acting), `acceptEdits` (auto-approve file edits), `bypassPermissions` (skip all checks — CI/automation use), `dontAsk` (never prompt, use rules only), `auto` (heuristic-based auto-approval), `bubble` (delegate decisions to parent agent).
- **Rule engine**: Per-project and per-user `allow`/`deny` rule lists in settings; rules match on tool name, path patterns (glob), and command patterns; evaluated in priority order before any prompt is shown.
- **AI classifier**: For tool calls that don't match any static rule and aren't in an auto-approval mode, an AI classifier scores the action's risk level (safe/warn/dangerous) and chooses whether to auto-approve, prompt, or block.
- **`bubble` mode for sub-agents**: Sub-agents running under `bubble` escalate permission decisions to the parent agent rather than prompting the user directly, enabling centralized permission management in multi-agent workflows.
- **Audit trail**: Every permission decision (approved/denied/auto-approved) is logged with tool name, arguments, decision source, and timestamp, providing a full audit trail for security review.
- **Permission caching**: Within a session, approval decisions for identical tool+argument pairs are cached so the user isn't re-prompted for repetitive actions.

## Transferable Patterns
1. **Enumerate permission modes as a named type**: Replace boolean `autoApprove` flags with a named mode enum; this makes the permission surface explicit and enables future modes without refactoring call sites.
2. **Layer static rules before AI classifiers**: Check cheap deterministic rules (allow/deny lists) before invoking a model-based classifier; reserve the AI call for genuinely ambiguous cases.
3. **Escalate rather than block in sub-agent contexts**: Give sub-agents a `bubble` mode that passes permission decisions up the call stack; this prevents sub-agents from either blocking silently or over-prompting.
