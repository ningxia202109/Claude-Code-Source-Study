# 摘要：24 — Skill/Plugin 开发实战：四个扩展点与前置元数据格式

## 概述
扩展 Claude Code 的四个扩展点实践指南——Hooks（Shell 脚本）、Skills（SKILL.md 命令）、Agents（前置元数据 Markdown）和 Plugins（完整包）——以及每层的实现模式。

## 核心要点
- **四层扩展层次**：Hook（通过 Shell 脚本响应事件）→ Skill（通过 Markdown 添加斜杠命令）→ Agent（通过前置元数据 Markdown 添加专业子 Agent）→ Plugin（通过完整包添加命令 + Agent + 工具）——每层提供更多能力，同时也更复杂。
- **SKILL.md 格式**：带 YAML 前置元数据的 Markdown 文件，声明 `name`（斜杠命令触发词）和自然语言提示体；放置在 `.claude/skills/` 或 `~/.claude/skills/`；无需编译。
- **Agent 前置元数据**：带 YAML 前置元数据的 Markdown 文件，声明 `name`、`description`、`tools`（白名单）、`model`，以及可选的 `system_prompt`；Markdown 正文成为 Agent 的系统提示；放置在 `.claude/agents/`。
- **Plugin 结构**：`~/.claude/plugins/<插件名>/` 目录，包含可选的 `package.json`、`skills/`、`agents/` 和 `hooks/` 子目录；插件可将多种扩展类型打包在一起便于发布。
- **通过 MCP 扩展工具**：需要添加新工具（而非仅命令/Agent）的插件在其 `package.json` 中注册 MCP 服务器；Claude Code 将其作为 Sidecar 启动，通过 MCP `tools/list` 协议发现其工具。
- **本地测试扩展**：Skills 和 Agents 在文件创建后无需重启即可立即测试（热重载）；Plugins 需要一次性的 `claude plugin install <路径>` 步骤。

## 可迁移的设计模式
1. **按复杂度分层扩展 API**：提供基于文件的快速入门（Skills/SKILL.md）、结构化中间层（带前置元数据的 Agents）和完整程序化层（Plugins）——用户可以在不重写简单扩展的情况下逐步获得更多能力。
2. **前置元数据存配置，Markdown 正文存提示**：使用 YAML 前置元数据存储结构化配置（名称、工具、模型），使用 Markdown 正文存储自由形式的系统提示文本；这将机器可读配置与人类可读指令分离。
3. **以 MCP 作为工具扩展边界**：不要从头构建工具插件 API；使用 MCP 作为添加新工具的标准协议——这使你的扩展与更广泛的 MCP 生态系统互操作。
