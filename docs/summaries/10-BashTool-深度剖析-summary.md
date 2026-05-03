# Summary: 10 — BashTool Deep Dive: AST Security, Permission Chain, and Sandbox

## Overview
Provides a detailed analysis of BashTool — the most complex tool in the codebase at ~12,400 lines across 18 files — focusing on its tree-sitter AST-based security analysis, multi-stage permission chain, and optional macOS sandbox.

## Key Points
- **Scale**: BashTool spans 18 files and ~12,400 lines, making it the single largest module in Claude Code; its complexity reflects the security sensitivity of executing arbitrary shell commands.
- **Tree-sitter AST analysis**: Rather than regex-matching command strings, BashTool parses shell commands into an AST using tree-sitter, enabling precise detection of command injection, pipe chains, subshell escapes, and dangerous patterns like `rm -rf /`.
- **Multi-stage permission chain**: Every command passes through: (1) static allow/deny lists, (2) AST-based security analysis, (3) per-tool permission rules, (4) the global permission mode check, (5) optional user confirmation prompt — in order.
- **Sandbox (macOS only)**: On macOS, commands can be executed inside an `sandbox-exec` profile that restricts filesystem writes, network access, and process spawning; the profile is generated dynamically based on the tool's declared capabilities.
- **Persistent shell session**: BashTool maintains a long-lived shell process across calls within a session, enabling stateful workflows (e.g., `cd` persists, environment variables carry over) while tracking cwd changes.
- **Output capture and truncation**: stdout/stderr are captured with size limits; very large outputs are truncated with a summary, preventing context window overflow from runaway commands.

## Transferable Patterns
1. **Parse, don't pattern-match, for security analysis**: Use an AST (tree-sitter, esprima, etc.) to analyze code/commands; regex-based blocking is bypassable and produces false positives on legitimate code.
2. **Layer permissions from cheapest to most expensive**: Check static lists first, then programmatic rules, then AI classifiers, then user prompts — fail fast on the cheap checks before reaching expensive ones.
3. **Maintain a persistent subprocess with state tracking**: A long-lived shell process is more powerful than spawning a new process per command; track cwd and environment changes to keep the host process in sync.
