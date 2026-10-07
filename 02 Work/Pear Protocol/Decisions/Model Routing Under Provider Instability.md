---
type: decision
area: pear
status: active
created: 2026-08-13
---

# Decision: Model Routing Under Provider Instability

## Context

Three different upstream models (Gemini, Luna, MiMo) hit provider-side rate limits or capacity walls within the same production window, each requiring an emergency swap.

## Problem

Deciding, under time pressure and repeatedly, whether a given model failure calls for a swap at all, and how to trust a replacement fast without shipping a worse regression.

## Decision

For every swap: first distinguish rate-limiting (HTTP 429, retriable) from upstream deprecation (a flat 502, a different failure mode) before choosing a replacement model. Then live-verify the replacement before merging — a bakeoff run, a concurrent-burst smoke test, or a tool-loop latency check, depending on the node — every time, including the third and fourth swap of the same week.

## Why

Treating rate-limiting and deprecation as the same problem risks swapping away from a model that would have recovered on its own. Skipping live verification under pressure is exactly when a bad swap does the most damage, since pressure is highest right when the temptation to skip the check is highest.

## Tradeoffs

This produced several same-week reversals (Luna was the emergency replacement for Gemini, then itself needed replacing days later; Gemini was the emergency replacement for MiMo, then needed replacing by Luna a day later) — accepted as the cost of keeping production on a working model at each step rather than waiting for a "final" answer.

## Evidence

[[My Work at Pear Protocol#2026-08-05 – 08-13 — Model rate-limit whack-a-mole: Gemini → Luna → Flash Lite, and MiMo → Gemini → Luna]]
[[Resume Evidence#Live-verified every emergency model swap before merging]]

## Later Evidence

Flash Lite bakeoff 1/1; Luna 8/8 concurrent burst with no 429s; Gemini advanced-mode smoke test ~5.4s with streamed reasoning — each cited at the time of its own swap.

2026-09-25: the composer fallback was moved from a same-key Cerebras fallback to a cross-provider one (OpenRouter), because a same-key fallback shares the rate-limit bucket that rejected the primary. It was live-verified with injected 429s before being considered done (PR #923, still open) — see [[25th Sep 2026]]. *(Correction: #923 had already merged to `development` on 2026-09-24, merge `cda91b87`; not yet in `main`.)*

2026-09-30: a different provider failure mode — not a 429 or 502, but a **successful empty response**. OpenRouter kept routing `deepseek-v4-pro` to Relace (cheapest endpoint), which returned HTTP 200 `finish_reason=stop` with 0-1 tokens. Found by logging the raw response per call, not from the SDK error ("empty prose after 3 attempts"). Fixed at the routing layer (`provider.ignore: ["Relace"]`) rather than only by swapping models; the synth also moved to `deepseek-v4.1-flash` in the same PR (user's call). Live-verified on the previously failing assets, 8/8. PR [#1060](https://github.com/pear-protocol/agent-pear-v2/pull/1060), open. See [[Sentiment Data Ingestion — Improvement Plan]].
