# 摘要：14 — 任务系统：本地、Agent、远程与 Dream 任务类型

## 概述
记录 Claude Code 的任务系统——`TaskType`/`TaskState` 类型层次、四种具体任务实现（LocalShellTask、LocalAgentTask、RemoteAgentTask、DreamTask）、`framework.ts` 编排，以及 `DiskTaskOutput` 持久化。

## 核心要点
- **`TaskType` 联合类型**：四种变体——`local-shell`（运行 Shell 命令）、`local-agent`（进程内运行内置或自定义 Agent）、`remote-agent`（委托给远程 Claude Code 实例）、`dream`（在后台运行的探索性/推测性任务，不阻塞主流程）。
- **`TaskState` 生命周期**：任务经历 `pending → running → completed | failed | cancelled`；状态转换是原子的，并记录到 `DiskTaskOutput`，因此可在进程重启后恢复。
- **`framework.ts` 编排**：管理任务队列、并发限制、依赖排序和并行任务扇出的中央调度器。
- **`DreamTask`**：用于长期后台探索的新型任务类型；以较低优先级运行，可暂停/恢复，生成用户可异步查看的推测性输出。
- **`DiskTaskOutput`**：实时将任务结果流式写入磁盘的文件支持输出存储；允许不会溢出内存的大型输出，并支持在任务仍在运行时检查结果。
- **依赖图**：任务可以声明对其他任务的依赖；`framework.ts` 对图进行拓扑排序并按正确顺序执行任务，支持复杂的多步工作流。

## 可迁移的设计模式
1. **从一开始就将任务状态持久化到磁盘**：在发生时原子地将任务状态转换写入磁盘；这为你免费提供崩溃恢复和可观测性。
2. **将长期后台工作建模为一等任务类型**：不要将后台工作强塞进前台任务抽象；专用的 `dream` 风格类型，配合低优先级和异步输出，让用户保持在工作流中。
3. **拓扑调度而非手动排序**：将任务依赖表达为 DAG 并让调度器推导执行顺序；这比手写的 `await task1; await task2` 链更易维护。
