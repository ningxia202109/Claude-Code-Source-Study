# Summary: 24 — Skill/Plugin Development: Four Extension Points and the Frontmatter Format

## Overview
Practical guide to extending Claude Code through its four extension points — Hooks (shell scripts), Skills (SKILL.md commands), Agents (frontmatter markdown), and Plugins (full packages) — with implementation patterns for each tier.

## Key Points
- **Four extension tiers**: Hook (react to events via shell scripts) → Skill (add slash commands via markdown) → Agent (add specialized sub-agents via frontmatter markdown) → Plugin (add commands + agents + tools via a full package) — each tier offering more capability with more complexity.
- **SKILL.md format**: A markdown file with YAML front-matter declaring `name` (the slash command trigger) and a natural-language prompt body; placed in `.claude/skills/` or `~/.claude/skills/`; no compilation required.
- **Agent frontmatter**: A markdown file with YAML front-matter declaring `name`, `description`, `tools` (whitelist), `model`, and optionally `system_prompt`; the markdown body becomes the agent's system prompt; placed in `.claude/agents/`.
- **Plugin structure**: A directory in `~/.claude/plugins/<plugin-name>/` containing an optional `package.json`, `skills/`, `agents/`, and `hooks/` subdirectories; plugins can bundle multiple extension types together for distribution.
- **Tool extension via MCP**: Plugins that need to add new tools (not just commands/agents) register an MCP server in their `package.json`; Claude Code launches it as a sidecar and discovers its tools via the MCP `tools/list` protocol.
- **Testing extensions locally**: Skills and agents can be tested immediately after file creation without restart (hot-reloaded); plugins require a one-time `claude plugin install <path>` step.

## Transferable Patterns
1. **Tier your extension API by complexity**: Offer a file-based quick-start (Skills/SKILL.md), a structured middle tier (Agents with frontmatter), and a full programmatic tier (Plugins) — users can grow into more power without rewriting simpler extensions.
2. **Frontmatter for metadata, markdown body for prompts**: Use YAML front-matter for structured configuration (name, tools, model) and the markdown body for free-form system prompt text; this separates machine-readable config from human-readable instructions.
3. **Use MCP as the tool extension boundary**: Don't build a tool plugin API from scratch; use MCP as the standard protocol for adding new tools — this gives your extensions inter-operability with the broader MCP ecosystem.
