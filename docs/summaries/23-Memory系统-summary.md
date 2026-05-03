# Summary: 23 — Memory System: Five Layers from CLAUDE.md to Relevant Memory Injection

## Overview
Maps Claude Code's five-layer memory system — CLAUDE.md project instructions, Auto Memory (memdir), Session Memory, Agent Memory, and Relevant Memories injection — explaining how each layer is written, stored, and surfaced to the model.

## Key Points
- **CLAUDE.md**: Project-level instructions checked into the repository; read at session start and injected into the static system prompt prefix; the primary mechanism for project-specific knowledge that all users share.
- **Auto Memory (memdir)**: A `~/.claude/memory/` directory of short markdown files automatically created and updated by the model when it learns something worth remembering across sessions (user preferences, project conventions, frequently-used commands).
- **Session Memory**: In-session facts stored in a temporary map that lives only for the current conversation; used for intra-session recall without persisting to disk.
- **Agent Memory**: Per-agent memory stores that allow sub-agents to maintain their own knowledge base separate from the parent agent's memory, supporting multi-agent workflows without cross-contamination.
- **Relevant Memories injection**: Before each turn, a retrieval step scores stored memories by relevance to the current query (using embedding similarity or keyword matching) and injects the top-k results into the conversation context.
- **Memory file format**: Auto Memory files are plain markdown with optional YAML front-matter for metadata (date, source agent, relevance tags); no special encoding required.

## Transferable Patterns
1. **Separate project memory from personal memory**: CLAUDE.md is project-scoped (committed to git, shared by team); memdir is user-scoped (personal preferences, private context) — keep these stores separate and never mix their content.
2. **Retrieval-augmented memory injection**: Don't inject all memories on every turn; score them for relevance and inject only the top-k — this keeps context usage low while surfacing the most useful memories.
3. **Plain markdown for memory files**: Store memories as plain markdown files on disk rather than in a database; this makes memories human-readable, editable, and version-controllable without special tooling.
