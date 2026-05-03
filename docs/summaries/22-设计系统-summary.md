# 摘要：22 — 设计系统：80+ 语义 Token、6 个主题与工具 UI 协议

## 概述
描述统一 Claude Code 视觉语言的终端设计系统——具有 80+ 个语义颜色 Token 的 `Theme` 类型、六个内置主题、十五个共享组件，以及让每个工具控制自身渲染的工具 UI 协议。

## 核心要点
- **`Theme` 类型**：一个 80+ 字段的对象，将语义名称（如 `toolResultBorder`、`warningText`、`successBackground`）映射到 ANSI 颜色码；组件消费语义 Token，永不直接使用原始 ANSI 码。
- **6 个内置主题**：`default`、`dark`、`light`、`solarized-dark`、`solarized-light`、`high-contrast`——每个都是完整的 `Theme` 对象；切换主题只需改变一个上下文值，无需重写任何组件。
- **15 个共享组件**：包括 `Box`、`Text`、`Badge`、`Spinner`、`ProgressBar`、`CodeBlock`、`Divider`、`StatusLine` 等——全部构建在 forked Ink 原语之上，并通过 Theme 上下文样式化。
- **工具 UI 协议**：`Tool` 接口中每个工具实现 `renderToolUse()`（展示工具即将做什么）和 `renderToolResult()`（展示输出）——共 10 个渲染方法，涵盖普通、错误、紧凑和流式变体。
- **Theme 上下文**：活跃的 `Theme` 通过 React 上下文提供；树中任何组件都可以通过 `useTheme()` 访问，无需 prop 透传，遵循标准 React 上下文模式。
- **响应式布局**：组件使用 Yoga Flexbox 适配终端宽度；窄终端自动将多列布局折叠为单列。

## 可迁移的设计模式
1. **语义 Token 而非原始值**：定义带命名语义 Token 的 `Theme` 对象；永不在组件中硬编码颜色或尺寸——这使主题化成为单文件的改动。
2. **将工具渲染与工具逻辑共同存放**：在工具对象本身上实现 `renderToolUse()` 和 `renderToolResult()` 方法，当工具行为变化时，其 UI 表示自动随之更新。
3. **通过 React 上下文而非 props 传递主题**：在根节点将活跃主题作为上下文值提供；组件通过 `useTheme()` 访问——将主题透传 5+ 层组件始终是个错误。
