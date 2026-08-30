---
type: investigation
area: pear
status: resolved
created: 2026-07-10
---

# Finished PnL Card Discarded on an Unrelated Timeout

## Symptom

A user's turn degraded to a bare "timed out" message even though `share_pnl_card` had already produced the images 22–52 seconds before the abort fired.

## Context

`share_pnl_card` could finish and produce a completed step, then a *later* step's 60s LLM abort signal would fire and reject the whole call.

## Initial Hypotheses

The card generation itself was slow or failing.

## Evidence

Production cases showed the card built well before the timeout — the images existed, but were never surfaced.

## Investigation

Traced how tool results were lifted from the generation call: only after `generateText` resolved. An abort partway through a later step meant the call never resolved, so already-completed tool results were discarded along with it.

## Finding / Root Cause

Tool results were lifted only after full resolution of `generateText`, so a timeout on a *later* step discarded a card that had already finished on an earlier step.

## Fix / Decision

Capture completed steps via `onStepFinish`. On a confirmed abort/timeout where `share_pnl_card` ran and no money-moving tool was involved, lift the recovered images instead of rethrowing. Every other error, a pre-tool timeout, or any write-involved timeout still rethrows exactly as before.

## What Fooled Us

The bug required a specific ordering (card step completes, then a *different* later step times out) — a single-step trace wouldn't have shown it.

## Remaining Unknowns

None noted.

## Related Work

[[My Work at Pear Protocol#2026-07-10 — The composer had no clock, and a finished PnL card was being thrown away on timeout]]
[[10th Jul 2026]]
