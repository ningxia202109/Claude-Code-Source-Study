# 摘要：12 — Agent 系统：生命周期、隔离与子 Agent 编排

## 概述
解释 Claude Code 如何实现其 Agent 系统——`AgentDefinition` 类型层次、`runAgent()` 生命周期，以及 `createSubagentContext()` 隔离机制——使内置和用户定义的 Agent 都能在会话中作为独立子进程运行。

## 核心要点
- **`AgentDefinition` 类型**：三种变体——`built-in`（随 Claude Code 发布，以 TypeScript 定义）、`custom`（在 Agent 前置元数据 Markdown 文件中定义）、`plugin`（由已安装的插件贡献）——共享一个公共调用接口。
- **`runAgent()` 生命周期**：接受 `AgentDefinition` 和任务描述，创建隔离上下文，使用 Agent 的系统提示运行对话循环（通过 `query.ts`），并返回结构化结果或将输出流式传回父 Agent。
- **`createSubagentContext()` 隔离**：生成一个独立的上下文对象，拥有自己的对话历史、工具子集、权限范围和工作目录快照——防止子 Agent 污染父 Agent 状态。
- **工具子集**：每个 Agent 在其定义中声明 `tools` 白名单；`runAgent()` 将全局工具注册表过滤为仅声明的子集，将 Agent 的能力限制在所需范围内。
- **权限继承与覆盖**：子 Agent 默认继承父 Agent 的权限模式，但可在定义中声明更严格的模式（如 `read-only`），强制最小权限原则。
- **Agent 间通信**：结果以结构化 `AgentResult` 对象的形式返回；父 Agent 将输出作为工具结果接收，维护标准的 tool_use/tool_result 对话结构。

## 可迁移的设计模式
1. **完全隔离子 Agent 状态**：为每次子 Agent 调用创建新的上下文对象；永不在父子之间共享可变状态（对话历史、工具状态）。
2. **按 Agent 白名单工具**：在 Agent 配置中定义工具允许列表，而非暴露完整工具注册表；这强制最小权限并使 Agent 行为可预测。
3. **返回结构化结果而非原始文本**：将 Agent 输出设计为带类型的 `AgentResult` 对象，使调用代码能以编程方式处理成功、失败和部分结果。
