# Summary: 09 — Tool System Design: The 30-Method Interface and Builder Pattern

## Overview
Describes the architecture of Claude Code's tool system — the `Tool` interface with 30+ methods, the `buildTool()` builder that reduces boilerplate, the `tools.ts` registry, and the deferred `ToolSearch` loading mechanism for optional tools.

## Key Points
- **`Tool` interface**: Defines 30+ methods covering the full tool lifecycle: `name`, `description`, `inputSchema` (Zod), `call()` (execution), `prompt` (system prompt contribution), `isEnabled()`, `isReadOnly()`, `needsPermission()`, `renderToolUse()`, `renderToolResult()`, and more.
- **`buildTool()` builder**: A factory function that accepts a partial tool definition and fills in sensible defaults for optional methods, reducing per-tool boilerplate from ~200 lines to ~50.
- **`tools.ts` registry**: A central file that imports all tools, runs `isEnabled()` checks at startup, and exports the active tool list to both the system prompt assembler and the dialog loop.
- **Deferred `ToolSearch` loading**: Optional or rarely-used tools are registered as "deferred" — their schemas are listed in a manifest but not loaded until `ToolSearch` is called, reducing startup memory and system prompt size.
- **Zod input schemas**: Every tool defines its input with a Zod schema, which serves triple duty: JSON Schema for the API, runtime validation of model output, and TypeScript type inference for the `call()` implementation.
- **Tool UI protocol**: Tools implement `renderToolUse()` and `renderToolResult()` to produce React-compatible terminal UI nodes, keeping rendering logic co-located with tool logic.

## Transferable Patterns
1. **Design a rich tool interface upfront**: Defining 30+ methods in the Tool interface — including rendering, permissions, and prompt contributions — prevents the ad-hoc additions that create inconsistency over time.
2. **Use a builder to enforce defaults**: A `buildTool()` factory with sensible defaults makes it easy to add new tools without duplicating boilerplate and ensures all optional methods have safe implementations.
3. **Defer loading of optional tools**: Register tool metadata (name, description) in a manifest; load the full implementation only when the tool is actually requested, keeping the common path lean.
