# Summary: 12 — Agent System: Lifecycle, Isolation, and Sub-agent Orchestration

## Overview
Explains how Claude Code implements its agent system — the `AgentDefinition` type hierarchy, the `runAgent()` lifecycle, and `createSubagentContext()` isolation — enabling both built-in and user-defined agents to run as self-contained sub-processes within a session.

## Key Points
- **`AgentDefinition` type**: Three variants — `built-in` (shipped with Claude Code, defined in TypeScript), `custom` (defined in agent frontmatter markdown files), `plugin` (contributed by installed plugins) — sharing a common interface for invocation.
- **`runAgent()` lifecycle**: Accepts an `AgentDefinition` and a task description, creates an isolated context, runs a dialog loop (via `query.ts`) with the agent's system prompt, and returns a structured result or streams output back to the parent.
- **`createSubagentContext()` isolation**: Produces a fresh context object with its own conversation history, tool subset, permission scope, and working directory snapshot — preventing sub-agents from polluting parent state.
- **Tool subsetting**: Each agent declares a `tools` whitelist in its definition; `runAgent()` filters the global tool registry to only the declared subset, limiting the agent's capabilities to what it needs.
- **Permission inheritance vs. override**: Sub-agents inherit the parent's permission mode by default but can declare a stricter mode (e.g., `read-only`) in their definition, enforcing least privilege.
- **Agent-to-agent communication**: Results flow back as structured `AgentResult` objects; the parent agent receives the output as a tool result, maintaining the standard tool_use/tool_result conversation structure.

## Transferable Patterns
1. **Isolate sub-agent state completely**: Create a new context object for every sub-agent invocation; never share mutable state (conversation history, tool state) between parent and child.
2. **Whitelist tools per agent**: Define a tools allowlist in the agent's configuration rather than exposing the full tool registry; this enforces least privilege and makes agent behavior predictable.
3. **Return structured results, not raw text**: Design agent output as a typed `AgentResult` object so that calling code can handle success, failure, and partial results programmatically.
