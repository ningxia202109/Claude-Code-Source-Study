# Summary: 13 — Built-in Agent Design Patterns: Six Agents, Six Specializations

## Overview
Analyzes the six built-in agents shipped with Claude Code — General-purpose, Statusline-setup, Explore, Plan, Guide, and Verification — extracting the design patterns that make each agent effective at its specific task.

## Key Points
- **General-purpose agent**: The default sub-agent for complex multi-step tasks; has access to all tools and uses full extended thinking; serves as the template for custom agent definitions.
- **Statusline-setup agent**: A narrow, single-purpose agent with read-only access to settings files; demonstrates how to build focused agents that cannot cause unintended side effects.
- **Explore agent**: A read-only search agent optimized for codebase navigation; uses a specialized system prompt that biases it toward breadth-first search and concise reporting rather than deep analysis.
- **Plan agent**: A planning-only agent that produces structured implementation plans; explicitly prohibited from writing code, keeping the plan/implement separation clean.
- **Guide agent (claude-code-guide)**: Answers questions about Claude Code's own features using web search and documentation reads; demonstrates meta-agents that reason about the tool they run inside.
- **Verification agent**: Runs tests and checks after an implementation; designed to be spawned post-implementation and return a pass/fail verdict with evidence, not suggestions.
- **Frontmatter configuration**: All built-in agents are defined in markdown files with YAML frontmatter specifying `name`, `description`, `tools`, `model`, and `system_prompt` — the same format available to users for custom agents.

## Transferable Patterns
1. **Design agents around capability boundaries, not task types**: The most effective agents are defined by what they *cannot* do (no writes, no network, no code execution) as much as what they can.
2. **Separate planning from implementation at the agent level**: Use distinct agents for planning and implementing; this prevents the model from collapsing into implementation before the plan is fully formed.
3. **Write agent system prompts that bias output format**: Specify in the system prompt exactly what the agent should return (a bulleted plan, a pass/fail verdict, a JSON object) so the calling code can parse results reliably.
