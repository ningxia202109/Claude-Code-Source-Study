# Summary: 01 — Project Overview: Technical Blueprint of an AI CLI Product

## Overview
Establishes a global understanding of Claude Code's architecture by examining its technology stack choices, startup chain, and module dependency graph across ~1,900 source files.

## Key Points
- **Tech stack**: Bun (runtime + bundler for fast startup and compile-time DCE), TypeScript + Zod (static + runtime type safety), forked Ink/React (declarative terminal UI), Commander.js (CLI parsing), Yoga layout engine.
- **Startup chain**: `cli.tsx` (bootstrap, fast-path) → `main.tsx` (orchestrator, 4,683 lines) → `init()` (core initialization via Commander `preAction` hook) → `setup.ts` (interactive session setup) → `replLauncher.tsx` (Ink REPL).
- **Module organization**: 10+ top-level modules — `query.ts` (dialog loop), `tools.ts` (tool registry), `commands.ts` (command aggregation), `state/` (state management), `services/` (MCP, compact, API), `components/` (380+ UI files), `ink/` (96-file forked framework), `utils/permissions/`, `utils/settings/`, `utils/hooks/`.
- **Core data flow**: `User input → query.ts assembles messages → Anthropic API → model returns tool_use → tool execution → results fed back → model continues/stops`.
- **Key architectural decisions**: Large single files (`main.tsx`, `query.ts`) are intentional strategy-concentration layers; dynamic `import()` used strategically for UI and optional features; lazy `require()` serves two purposes — breaking circular deps and enabling compile-time DCE with `feature()`.

## Transferable Patterns
1. **Layered startup + fast paths**: Each layer only loads the minimum required modules; simple commands return at the earliest possible layer.
2. **Side-effect hoisting**: Insert async I/O startup calls between `import` statements to parallelize I/O with module evaluation time.
3. **Compile-time feature flags + conditional `require()`**: Use `feature()` with `require()` (not static `import`) for zero-cost code elimination across build variants.
