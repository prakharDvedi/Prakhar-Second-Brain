---
type: decision
area: pear
status: active
created: 2026-08-30
---

# Decision: Rebalance Weight Validation — Reject vs Rescale

## Context

Rebalance target weights are shares of one whole. The trade-execution LLM sizes 13-15 leg baskets to 4 decimal places, which cannot reliably land inside a 1e-4 sum-to-1 tolerance.

## Problem

The existing hard reject on sum != 1 was measured against real traces: 4 traces, 21 rejected `rebalance_position` attempts, drift up to 15% — one trace showed 10 back-to-back self-correction attempts before landing in tolerance.

## Options / Alternatives

- **Reject on sum != 1 (existing behavior):** simple, but the measured data showed it just burns retries against an LLM that structurally can't hit exact sums.
- **Rescale each weight proportionally (`w / sum`) before the leg mapping (chosen):** treats the relative shares as the caller's real intent and the sum as arithmetic noise.

## Decision

Rescale weights outside the existing tolerance, rather than reject. Applied only outside tolerance, so a clean sum stays bit-exact — no behavior change for currently-passing inputs.

## Why

The relative shares are the caller's intent; the absolute sum is the arithmetic the model got wrong. Rescaling is strictly more permissive and backward-compatible for both adapters (agent + MCP share `planRebalanceUsecase`), with no wire/SDK surface change.

## Tradeoffs

Per-leg weights are already `zod .positive().max(1)`, so the sum is provably > 0 before division (rescale never divides by zero) and an absurd single leg is still rejected upstream. `normalizeLegWeights` (open/trigger-open) shares the same tolerance constant but showed no drift in traces yet — deliberately left as-is rather than changed pre-emptively.

## Evidence

[[My Work at Pear Protocol#2026-08-30 — Rebalance weights were rejected instead of rescaled (open, current branch)]]
[[Resume Evidence#Quantified and fixed retry-storming on rebalance weight validation]]

## Later Evidence

Not yet merged as of 2026-08-30 — this decision is on the branch this vault entry was generated from.
