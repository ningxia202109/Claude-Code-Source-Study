# Summary: 02 — Startup Optimization: From Cold Start to First Response in Milliseconds

## Overview
Analyzes the multi-layered startup optimization techniques that make Claude Code's CLI feel instant, covering fast-path routing, parallelized I/O, API preconnection, early input capture, dead code elimination, and memoization.

## Key Points
- **Fast-path routing**: `cli.tsx` intercepts version/help flags and simple non-interactive commands before loading the full UI stack, returning at the earliest possible layer.
- **Side-effect hoisting**: Async I/O calls (config reads, network preconnects) are placed between `import` statements so they execute in parallel with module evaluation, effectively hiding I/O latency behind module load time.
- **API preconnection**: Initiates an HTTPS connection to the Anthropic endpoint during startup before the user has even finished typing, eliminating TCP/TLS handshake latency from the first API call.
- **Early input capture**: Keyboard input buffering starts before the Ink UI is fully initialized, ensuring no keystrokes are lost during the rendering warmup period.
- **Dead code elimination (DCE)**: `feature()` paired with `require()` (not static `import`) allows Bun's bundler to eliminate entire code branches at compile time for unused build variants.
- **Memoization**: Expensive computations (system prompt assembly, settings resolution) are cached with `memoize()` so repeated calls within a session pay only the first-call cost.

## Transferable Patterns
1. **Hoist async calls between imports**: Place `const prefetchPromise = fetchSomething()` between import statements; await it lazily when the result is actually needed.
2. **Preconnect before user intent is known**: Establish network connections during startup for endpoints that are almost always needed, trading a small idle cost for near-zero connection latency.
3. **Buffer user input before UI is ready**: Capture and queue raw stdin events immediately on process start; replay them once the UI event loop is running.
