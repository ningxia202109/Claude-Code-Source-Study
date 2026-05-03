# 摘要：03 — 状态管理：三层响应式架构

## 概述
介绍 Claude Code 的三层状态管理架构——引导常量、响应式会话状态和临时工具上下文——并解释为何一个 35 行的自定义 Store 优于 Redux/Zustand。

## 核心要点
- **三层架构**：① `bootstrap/state.ts`——进程启动时设置的不可变常量（路径、用户信息、初始配置）；② `Store + AppState`——使用自定义可观察对象的响应式可变会话状态；③ `ToolUseContext`——作为函数参数传递的每次工具调用的临时上下文。
- **自定义 Store（约 35 行）**：实现 `useSyncExternalStore` 协议，包含 `getSnapshot()`、`subscribe()` 和 `setState()`。无外部依赖，无样板代码，无选择器或中间件开销。
- **`useSyncExternalStore` 集成**：React 18 的内置 Hook 确保并发渲染期间读取无撕裂；自定义 Store 直接接入此协议，无需库。
- **在正确层级保证不可变性**：引导状态初始化后冻结；AppState 通过浅合并 `setState()` 更新以触发重渲染；ToolUseContext 永不存储为全局——它沿调用栈向下传递。
- **避免过度响应**：组件只订阅它们读取的那部分状态；粗粒度订阅会导致终端 UI 中不必要的重渲染。

## 可迁移的设计模式
1. **在选择 Store 前先对状态分类**：区分不可变引导数据（普通对象）、响应式共享状态（自定义 Store 或 Zustand）和每次调用的上下文（函数参数）——每层需要不同的解决方案。
2. **当库带来额外开销时，手写最小化 Store**：满足 `useSyncExternalStore` 的 35 行 Store 对大多数 React 应用已经足够；只有在需要时间旅行、DevTools 或中间件时才引入 Redux/Zustand。
3. **通过参数传递临时上下文，而非全局存储**：工具执行上下文（当前工作目录、权限状态、对话轮次）存活在调用栈上，而非全局 Store 中，可防止隐蔽的并发 Bug。
