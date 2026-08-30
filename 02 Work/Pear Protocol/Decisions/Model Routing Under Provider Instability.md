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
