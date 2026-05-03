# 摘要：18 — Hooks 系统：27 个生命周期事件与 Shell 执行模型

## 概述
解释 Claude Code 的 Hooks 系统——27 种生命周期事件类型（PreToolUse、PostToolUse、SessionStart、Stop 等）、基于 Shell 的 Hook 脚本执行模型，以及 Hook 如何实现用户定义的自动化而无需修改核心代码库。

## 核心要点
- **27 个生命周期事件**：事件覆盖完整的会话和工具生命周期：`SessionStart`、`SessionEnd`、`PreToolUse`、`PostToolUse`、`PreAgentCall`、`PostAgentCall`、`Stop`、`Notification` 等——每个在执行流中定义明确的触发点。
- **Shell 执行模型**：Hook 处理器是在设置中定义的任意 Shell 命令；Claude Code 将它们作为子进程派生，通过环境变量注入相关上下文（工具名称、参数、结果、会话 ID）。
- **`PreToolUse` 阻塞**：`PreToolUse` Hook 可返回非零退出码来阻止工具调用；这支持自定义安全策略（如阻止所有 `git push` 命令），无需修改 Claude Code 源码。
- **`PostToolUse` 观测**：`PostToolUse` Hook 接收工具结果，可记录日志、发出警告或触发副作用；不能修改结果，但可通过 stdout 注入后续消息。
- **stdin/stdout 协议**：Hook 通过 stdout 上的结构化 JSON 与 Claude Code 通信；这允许 Hook 向对话注入消息、设置环境变量或发出错误信号。
- **设置定义的 Hook 注册表**：Hook 在 `settings.json` 的 `hooks` 键下注册，将事件名映射到 Shell 命令数组；每个事件支持多个 Hook，按顺序运行。

## 可迁移的设计模式
1. **将 Hook 建模为生命周期事件 → Shell 命令**：避免插件 API；以 Shell 进程的形式触发 Hook，通过环境变量传递上下文——这让用户可以用任何语言编写 Hook，无需 SDK。
2. **用 `PreToolUse` 实现自定义安全策略**：阻塞式前置 Hook 是组织安全规则的正确扩展点；它将策略保持在核心代码库之外，并在设置文件中可审计。
3. **Hook→宿主通信使用 stdout JSON**：定义一个简单的 JSON 协议供 Hook 向宿主发回消息；这与语言无关，不需要共享库或 SDK。
