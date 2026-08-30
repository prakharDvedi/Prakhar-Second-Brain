---
type: investigation
area: pear
status: resolved
created: 2026-08-07
---

# Token-Unlock Amounts Shown as Confirmed Facts

## Symptom

The agent stated things like "HYPE has a 9.9M unlock on Sept 6" as if it were a confirmed figure.

## Context

`getEffectiveUnlockMetrics` returned the raw `cliffAmount` — a scheduled/projected vesting ceiling — whenever an unlock event existed, regardless of whether any allocation in that event had a `committedClaim`.

## Initial Hypotheses

That this was specific to HYPE's unlock data being wrong.

## Evidence

Checked directly against production data: HYPE was the *only* asset with any committed claim at all.

## Investigation

Cross-checked all assets with unlock schedules against their `committedClaim` field.

## Finding / Root Cause

The other 49 assets with unlock schedules were 100% null `committedClaim` — meaning all of them were being presented with the same unconfirmed-as-confirmed overstatement, not just HYPE. The raw `cliffAmount` is a projected ceiling, not a project-confirmed number.

## Fix / Decision

Suppress amount/value/market-cap-impact fields entirely when no committed claim exists, rather than caveat the number in prose. `get_upcoming_unlocks` drops uncommitted events from ranking instead of sorting on an unconfirmed ceiling.

## What Fooled Us

The bug looked asset-specific until the committed-claim field was checked across the whole dataset — a single bad-looking example (HYPE) was actually evidence of a systemic gap.

## Remaining Unknowns

None noted — verified live against real Tokenomist data after the fix.

## Related Work

[[My Work at Pear Protocol#2026-08-07 — Projected token-unlock amounts were being shown to users as confirmed facts]]
[[7th Aug 2026]]
