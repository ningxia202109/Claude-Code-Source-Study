# Summary: 15 — MCP Protocol Implementation: 8 Server Types, 6 Transports, OAuth Auth

## Overview
Analyzes Claude Code's implementation of the Model Context Protocol (MCP) — covering the 8 server configuration types, 7 configuration scopes, 6 transport types, and the OAuth/XAA authentication flow for connecting to remote MCP servers.

## Key Points
- **8 server config types**: `stdio`, `sse`, `http`, `websocket`, `docker`, `npx`, `uvx`, `node` — each specifying how to launch or connect to an MCP server, with type-specific fields validated by Zod schemas.
- **7 configuration scopes**: Server configs can be declared at plugin / user-global / project / local / environment / CLI-flag / policy levels, following the same priority hierarchy as the Settings system.
- **6 transport types**: The underlying communication channel (stdio pipes, SSE stream, HTTP polling, WebSocket, in-process IPC, mock) is abstracted behind a `McpTransport` interface, making servers transport-agnostic.
- **OAuth/XAA authentication**: Remote MCP servers requiring auth trigger a browser-based OAuth flow; tokens are stored in the user-level credential store and refreshed automatically with exponential backoff.
- **Tool discovery**: After connecting, Claude Code calls `tools/list` to enumerate the server's tools and injects them into the active tool registry alongside built-in tools, making them indistinguishable from native tools.
- **Health monitoring**: Each connected server is polled with periodic pings; disconnected servers trigger reconnect logic with configurable retry limits before being marked unavailable.

## Transferable Patterns
1. **Unify local and remote tools under one interface**: Wrap MCP-sourced tools in the same `Tool` interface as built-in tools; callers never need to know whether a tool is local or remote.
2. **Scope-layered server configuration**: Allow server definitions at multiple scopes (user / project / local) so that different environments can use different servers without editing shared config files.
3. **Lazy OAuth with automatic token refresh**: Don't pre-authenticate all servers at startup; trigger the OAuth flow on first use and refresh tokens transparently so the user authenticates at most once per server.
