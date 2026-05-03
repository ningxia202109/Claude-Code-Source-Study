# 摘要：01 — 项目全景：AI CLI 产品的技术蓝图

## 概述
通过分析约 1,900 个源文件，建立对 Claude Code 架构的全局认知，涵盖技术栈选型、启动链路和模块依赖图。

## 核心要点
- **技术栈**：Bun（运行时 + 打包器，快速启动与编译期 DCE）、TypeScript + Zod（静态 + 运行时类型安全）、forked Ink/React（声明式终端 UI）、Commander.js（CLI 解析）、Yoga 布局引擎。
- **启动链路**：`cli.tsx`（引导，快速路径）→ `main.tsx`（协调器，4,683 行）→ `init()`（通过 Commander `preAction` 钩子完成核心初始化）→ `setup.ts`（交互式会话配置）→ `replLauncher.tsx`（Ink REPL）。
- **模块组织**：10+ 个顶层模块 —— `query.ts`（对话循环）、`tools.ts`（工具注册）、`commands.ts`（命令聚合）、`state/`（状态管理）、`services/`（MCP、压缩、API）、`components/`（380+ 个 UI 文件）、`ink/`（96 个文件的 forked 框架）、`utils/permissions/`、`utils/settings/`、`utils/hooks/`。
- **核心数据流**：`用户输入 → query.ts 组装消息 → Anthropic API → 模型返回 tool_use → 工具执行 → 结果反馈 → 模型继续/停止`。
- **关键架构决策**：大型单文件（`main.tsx`、`query.ts`）是有意为之的策略集中层；动态 `import()` 用于 UI 和可选功能；懒加载 `require()` 有两个用途——打破循环依赖，并通过 `feature()` 实现编译期 DCE。

## 可迁移的设计模式
1. **分层启动 + 快速路径**：每层只加载所需的最少模块；简单命令在最早的层返回。
2. **副作用提升**：在 `import` 语句之间插入异步 I/O 启动调用，使 I/O 与模块求值并行执行。
3. **编译期特性开关 + 条件式 `require()`**：配合 `feature()` 使用 `require()`（而非静态 `import`），实现跨构建变体的零成本代码消除。
