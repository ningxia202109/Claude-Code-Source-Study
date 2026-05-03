# Summary: 21 — Ink Framework Deep Customization: Forked Reconciler, Yoga Layout, and Virtual Scrolling

## Overview
Analyzes the 96-file forked Ink framework inside Claude Code — the customized React reconciler, Yoga flexbox layout engine integration, double-buffer rendering pipeline, virtual scrolling for long outputs, and mouse event handling.

## Key Points
- **Forked React reconciler**: Claude Code ships a modified copy of Ink (the React-for-terminals library) with patches to the reconciler for performance and to support terminal-specific features not available upstream.
- **Yoga layout engine**: Facebook's Yoga (a C++ flexbox engine) computes layout for terminal UI nodes; Claude Code uses it to support `flexDirection`, `alignItems`, `justifyContent`, and `gap` in terminal components — the same flexbox model as CSS.
- **Double-buffer rendering**: The render pipeline maintains two terminal-cell buffers (current and next); each frame computes the diff between them and emits only the changed ANSI escape sequences, minimizing flicker and terminal I/O.
- **Virtual scrolling**: Very long content (e.g., large file contents, long tool outputs) is rendered with a virtual scroll window — only the visible rows are passed to the layout engine and rendered, keeping frame time O(viewport) not O(content).
- **Mouse event handling**: The forked Ink includes mouse event support (click, scroll, hover) using terminal mouse reporting escape sequences; this enables scrollable panes and clickable UI elements.
- **Measurement and intrinsics**: Custom components can implement a `measureNode()` method to report their preferred dimensions to Yoga, enabling text components to request exactly the width/height they need.

## Transferable Patterns
1. **Fork aggressively when upstream is too constrained**: If an upstream library's architecture prevents the optimizations you need (double buffering, virtual scroll), forking and patching is more maintainable than workarounds.
2. **Use double-buffering for any incremental renderer**: Maintain current and next state buffers; compute and emit only the diff — this is the right architecture for any output that updates in place (terminals, canvases, e-ink).
3. **Virtual scrolling is required for unbounded content**: Never pass unbounded lists to a layout engine; clip to the visible viewport before layout and rendering, regardless of the rendering target.
