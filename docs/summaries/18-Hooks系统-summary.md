# Summary: 18 — Hooks System: 27 Lifecycle Events and the Shell Execution Model

## Overview
Explains Claude Code's hooks system — 27 lifecycle event types (PreToolUse, PostToolUse, SessionStart, Stop, etc.), the shell-based execution model for hook scripts, and how hooks enable user-defined automation without modifying the core codebase.

## Key Points
- **27 lifecycle events**: Events span the full session and tool lifecycle: `SessionStart`, `SessionEnd`, `PreToolUse`, `PostToolUse`, `PreAgentCall`, `PostAgentCall`, `Stop`, `Notification`, and more — each fired at a well-defined point in the execution flow.
- **Shell execution model**: Hook handlers are arbitrary shell commands defined in settings; Claude Code spawns them as child processes with relevant context injected via environment variables (tool name, arguments, result, session ID).
- **`PreToolUse` blocking**: `PreToolUse` hooks can return a non-zero exit code to block the tool call; this enables custom security policies (e.g., block all `git push` commands) without modifying Claude Code's source.
- **`PostToolUse` observation**: `PostToolUse` hooks receive the tool result and can log, alert, or trigger side effects; they cannot modify the result but can inject follow-up messages via stdout.
- **Stdin/stdout protocol**: Hooks communicate back to Claude Code via structured JSON on stdout; this allows hooks to inject messages into the conversation, set environment variables, or signal errors.
- **Settings-defined hook registry**: Hooks are registered in `settings.json` under a `hooks` key mapping event names to shell command arrays; multiple hooks per event are supported and run in order.

## Transferable Patterns
1. **Model hooks as lifecycle event → shell command**: Avoid a plugin API; fire hooks as shell processes with context in environment variables — this lets users write hooks in any language without an SDK.
2. **`PreToolUse` for custom security policies**: A blocking pre-hook is the right extension point for organizational security rules; it keeps policy out of the core codebase and makes it auditable in settings files.
3. **JSON-on-stdout for hook→host communication**: Define a simple JSON protocol for hooks to send messages back; this is language-agnostic and doesn't require a shared library or SDK.
