---
type: investigation
area: pear
status: resolved
created: 2026-07-29
---

# Reduce-Only Rejection Didn't Mean the Trade Failed

## Symptom

A `close_position`/`adjust_advance_position` ticket hitting Hyperliquid's "Reduce only order would increase position" was always treated as a hard FAILED, even when the user's requested end-state had already been reached.

## Context

An `adjust_advance_position` ticket freezes an absolute per-leg target and replays it on confirm. If a sibling close or a direct-exchange trade flattened that leg first, the replayed order is no longer reducible against live state.

## Initial Hypotheses

That every such rejection represented a genuinely failed trade.

## Evidence

The rejection only proves the *submitted order* wasn't reducible against current state — it doesn't prove the user's requested target was missed, and it isn't proof the target *was* achieved either.

## Investigation

Determined the only safe way to know which case applied was to re-check live state, specifically via a forced live-exchange reconcile (`getPositions(syncPosition=true)`) rather than the cached read that generated the stale order.

## Finding / Root Cause

The executor conflated "this specific order couldn't execute against current state" with "the user's request failed" — the two are not the same when other trades on the same wallet can independently change the position.

## Fix / Decision

A guard downgrades FAILED to benign only when every targeted leg is provably already at target on a fresh read — asymmetric by design, can only downgrade, never upgrade uncertainty into success. See [[Reduce-Only Verification Source]] for the source-of-truth tradeoff behind this.

## What Fooled Us

The rejection message names the *symptom* (order not reducible) rather than the *cause* (position already changed elsewhere) — nothing in the rejection itself hints that the underlying request may have already succeeded.

## Remaining Unknowns

Non-zero-target verification, the `close_position`/`close_all`/`reverse` throw-path, and direct-HL (non-BE-mediated) verification were all explicitly deferred as follow-ups.

## Related Work

[[My Work at Pear Protocol#2026-07-27 – 07-29 — A "would increase position" rejection didn't mean the trade failed; Twitter labeling failures were unreadable]]
[[29th Jul 2026]]

## Lessons

[[Engineering Lessons#A downgrade-only gate is safer than a symmetric recheck]]
