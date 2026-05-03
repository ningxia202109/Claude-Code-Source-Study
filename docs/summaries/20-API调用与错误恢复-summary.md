# Summary: 20 — API Calls & Error Recovery: withRetry, 529/429 Logic, and Multi-Provider

## Overview
Documents Claude Code's API call layer — the `withRetry` AsyncGenerator wrapper, specific handling for 529 (overloaded) and 429 (rate-limit) errors, the streaming/non-streaming dual mode, and the multi-provider fallback chain.

## Key Points
- **`withRetry` AsyncGenerator**: Wraps an API call generator with retry logic; on retryable errors it re-invokes the generator with exponential backoff, transparently yielding previously-streamed tokens from the new attempt to maintain stream continuity.
- **529 vs. 429 handling**: 529 (server overloaded) uses aggressive short-interval retries (1–4s) since overload is typically transient; 429 (rate limit) uses the `Retry-After` header value or a longer backoff since the limit is quota-based.
- **Streaming/non-streaming dual mode**: The same `query.ts` logic handles both SSE streaming responses and full JSON responses; a thin adapter normalizes both into an AsyncIterable of `ContentBlock` chunks.
- **Multi-provider support**: The API layer abstracts over Anthropic direct, AWS Bedrock, and Google Vertex AI; provider selection is config-driven; authentication and endpoint differences are handled in per-provider adapters.
- **Jitter on backoff**: Retry delays include random jitter (±20%) to prevent thundering-herd behavior when many Claude Code instances back off simultaneously after a rate-limit event.
- **Non-retryable error passthrough**: Authentication errors (401), invalid requests (400), and content policy violations are not retried and surface immediately to the caller with the original error code.

## Transferable Patterns
1. **Wrap streaming generators with retry logic at the generator boundary**: Implement retry in a wrapper generator that re-invokes the inner generator; this keeps retry logic out of the inner generator and handles partial-stream recovery cleanly.
2. **Distinguish 429 and 529 retry strategies**: Rate-limit errors need `Retry-After`-aware backoff; overload errors need aggressive short retries — using the same strategy for both wastes time or causes quota burn.
3. **Abstract provider differences in thin adapters**: Keep the dialog loop provider-agnostic; push Bedrock/Vertex/direct authentication and endpoint differences into per-provider adapter classes that share a common interface.
