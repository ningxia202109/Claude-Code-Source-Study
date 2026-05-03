# Summary: 14 — Task System: Local, Agent, Remote, and Dream Task Types

## Overview
Documents Claude Code's task system — the `TaskType`/`TaskState` type hierarchy, the four concrete task implementations (LocalShellTask, LocalAgentTask, RemoteAgentTask, DreamTask), `framework.ts` orchestration, and `DiskTaskOutput` persistence.

## Key Points
- **`TaskType` union**: Four variants — `local-shell` (runs a shell command), `local-agent` (runs a built-in or custom agent in-process), `remote-agent` (delegates to a remote Claude Code instance), `dream` (an exploratory/speculative task that runs in the background without blocking).
- **`TaskState` lifecycle**: Tasks move through `pending → running → completed | failed | cancelled`; state transitions are atomic and logged to `DiskTaskOutput` so they survive process restarts.
- **`framework.ts` orchestration**: The central scheduler that manages task queues, concurrency limits, dependency ordering, and fan-out for parallel task execution.
- **`DreamTask`**: A novel task type for long-running background explorations; runs at lower priority, can be paused/resumed, and produces speculative outputs that the user can review asynchronously.
- **`DiskTaskOutput`**: A file-backed output store that streams task results to disk in real time; allows large outputs that would overflow memory and enables result inspection even if the task is still running.
- **Dependency graph**: Tasks can declare dependencies on other tasks; `framework.ts` topologically sorts the graph and executes tasks in the correct order, enabling complex multi-step workflows.

## Transferable Patterns
1. **Persist task state to disk from the start**: Write task state transitions to disk atomically as they happen; this gives you crash recovery and observability for free.
2. **Model long-running background work as a first-class task type**: Don't force background work into foreground task abstractions; a dedicated `dream`-style type with lower priority and async output lets users stay in flow.
3. **Topological scheduling over manual sequencing**: Express task dependencies as a DAG and let a scheduler derive execution order; this is more maintainable than hand-coded `await task1; await task2` chains.
