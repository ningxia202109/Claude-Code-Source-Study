# Summary: 19 — Feature Flags & Compile-Time Optimization: DCE, USER_TYPE, and GrowthBook

## Overview
Covers Claude Code's two-tier feature flag system — compile-time dead code elimination via `feature()` with Bun `--define` constants, and runtime A/B testing via GrowthBook — plus the `MACRO.*` constants for build-time value injection.

## Key Points
- **`feature()` + `require()` DCE**: Wrapping `require()` calls in `if (feature('FLAG'))` blocks allows Bun's bundler to evaluate the condition at compile time and eliminate the unreachable branch entirely, removing code and its transitive dependencies from the bundle.
- **`USER_TYPE` `--define`**: A Bun `--define` constant injected at build time that classifies the build variant (e.g., `internal`, `external`, `enterprise`); used in `feature()` checks to ship different capability sets in different builds.
- **`MACRO.*` constants**: Build-time constants (version number, build timestamp, API endpoint) injected via `--define`; accessing them compiles to literal values with zero runtime overhead.
- **GrowthBook runtime flags**: For gradual rollouts and A/B tests that can't be decided at build time, GrowthBook provides a runtime flag client that fetches flag states from a remote service; flags are evaluated per-user based on attributes.
- **Flag layering**: Compile-time `feature()` checks act as the outer gate (the code literally doesn't exist in the build); GrowthBook checks act as the inner gate (the code exists but is conditionally activated); this prevents feature leakage even if runtime flags malfunction.
- **Zero-cost compile-time flags**: Because `feature()` uses `require()` (not `import`), disabled code paths are fully tree-shaken from the bundle — no runtime boolean checks, no dead code in production.

## Transferable Patterns
1. **`if (CONSTANT) require()` for zero-cost DCE**: Wrap optional features in a constant-gated `require()` instead of a dynamic `import()`; the bundler will eliminate the dead branch and all its transitive deps.
2. **Layer compile-time and runtime flags**: Use compile-time flags for capability sets that differ by build variant (internal vs. external); use runtime flags for gradual rollouts and experiments — don't mix the two.
3. **Inject build metadata as `--define` constants**: Version, build timestamp, and environment are best injected at build time as literal constants rather than read from environment variables at runtime.
