# Summary: 11 — Command System: 70+ Built-ins, Skills, Plugins, and Workflows

## Overview
Maps the architecture of Claude Code's slash-command system — the `Command` union type, how `commands.ts` aggregates 70+ built-in commands with Skills, Plugins, and Workflows, and the dispatch logic that routes user input to the right handler.

## Key Points
- **`Command` union type**: Three variants — `prompt` (sends text to the model), `local` (executes a TypeScript function directly), `local-jsx` (renders a React component in the terminal) — each with different execution paths and return types.
- **`commands.ts` aggregation**: A single file that imports all built-in command definitions plus dynamically discovered Skills, Plugins, and Workflows, merging them into one flat command registry.
- **70+ built-in commands**: Includes `/help`, `/clear`, `/compact`, `/model`, `/memory`, `/cost`, `/status`, `/config`, and many others — each defined in its own file and registered via the aggregator.
- **Skills (SKILL.md)**: User-defined commands loaded from `SKILL.md` files in the project or home directory; each skill is a markdown file with a front-matter command name and a natural-language prompt body.
- **Plugins**: Third-party command packages discovered via `~/.claude/plugins/`; each plugin can contribute multiple commands and custom agents through a standard directory structure.
- **Dispatch logic**: Command input is matched against the registry by prefix; ambiguous matches are resolved by specificity; unmatched input falls through to the model as a regular message.

## Transferable Patterns
1. **Union-type command variants**: Model your command system as a union of distinct execution strategies (text-prompt / function / component) rather than a single interface with nullable fields.
2. **Aggregate commands at a single entry point**: Maintain one authoritative registry file that merges built-ins, user extensions, and plugins; this makes the full command surface discoverable and debuggable.
3. **File-based skill definition**: Allow users to define commands as plain markdown files with front-matter; this lowers the barrier to extending the CLI without requiring code compilation.
