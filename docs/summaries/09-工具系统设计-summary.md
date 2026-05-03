# 摘要：09 — 工具系统设计：30 方法接口与构建器模式

## 概述
描述 Claude Code 工具系统的架构——拥有 30+ 个方法的 `Tool` 接口、减少样板代码的 `buildTool()` 构建器、`tools.ts` 注册表，以及可选工具的延迟加载 `ToolSearch` 机制。

## 核心要点
- **`Tool` 接口**：定义涵盖完整工具生命周期的 30+ 个方法：`name`、`description`、`inputSchema`（Zod）、`call()`（执行）、`prompt`（系统提示贡献）、`isEnabled()`、`isReadOnly()`、`needsPermission()`、`renderToolUse()`、`renderToolResult()` 等。
- **`buildTool()` 构建器**：接受部分工具定义并为可选方法填充合理默认值的工厂函数，将每个工具的样板代码从约 200 行压缩到约 50 行。
- **`tools.ts` 注册表**：导入所有工具、在启动时运行 `isEnabled()` 检查并导出活跃工具列表的中心文件，供系统提示组装器和对话循环使用。
- **延迟加载 `ToolSearch`**：可选或不常用的工具注册为"延迟加载"——其 Schema 列在清单中，但直到调用 `ToolSearch` 时才加载，减少启动内存和系统提示大小。
- **Zod 输入 Schema**：每个工具使用 Zod Schema 定义输入，一式三份：API 的 JSON Schema、模型输出的运行时验证，以及 `call()` 实现的 TypeScript 类型推断。
- **工具 UI 协议**：工具实现 `renderToolUse()` 和 `renderToolResult()` 以生成兼容 React 的终端 UI 节点，使渲染逻辑与工具逻辑共同存放。

## 可迁移的设计模式
1. **预先设计丰富的工具接口**：在 Tool 接口中预先定义 30+ 个方法（包括渲染、权限和提示贡献）——否则临时添加会产生不一致。
2. **用构建器强制默认值**：带合理默认值的 `buildTool()` 工厂使添加新工具变得简单，无需复制样板代码，并确保所有可选方法都有安全实现。
3. **延迟加载可选工具**：在清单中注册工具元数据（名称、描述）；仅在工具实际被请求时加载完整实现，保持常见路径的精简。
