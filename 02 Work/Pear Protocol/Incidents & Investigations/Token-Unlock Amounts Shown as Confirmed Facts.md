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

None noted — verified live against real Tokenomist data after the fix. *(superseded, see Update 2026-10-17 below)*

## Related Work

[[My Work at Pear Protocol#2026-08-07 — Projected token-unlock amounts were being shown to users as confirmed facts]]
[[7th Aug 2026]]

## Update 2026-10-17

- The 7 Aug fix over-corrected. [#1122](https://github.com/pear-protocol/agent-pear-v2/pull/1122) (merged 17 Oct 20:42 UTC) says `getEffectiveUnlockMetrics` nulled the amount, value and impact for every event with no `committedClaim`, and the live Tokenomist v5 API only sets `committedClaim` after an unlock date has passed. So every future unlock was hidden, not just unconfirmed ones. Prod symptom (Langfuse `355251280a0d4b45bcfcfe8d1b814f5a`): the agent said $XPL and $KAITO had no upcoming unlock pressure, while XPL has about 227.8M tokens (about 5% of market cap) scheduled for 25 Oct.
- My reading, not stated in the PR: the earlier "None noted" and the claim that 49 assets were "100% null `committedClaim`" are consistent with this: null is the normal state for a future event, which the 7 Aug reading treated as a sign of an unconfirmed amount only.
- New rule kept from the original: an unconfirmed amount is never presented as announced. Now it is shown as `SCHEDULED (vesting schedule, not announced ...)`, and an unknown size no longer prints as 0% or "manageable".
- Verified per the PR: specs, build, tsc, lint, and a node replay of XPL and KAITO. Not verified: the awareness eval (does not run this node). Open in the PR: allocation list capped at 3, `linearUnlocks` ignored, `tokenomist.txt` names a `get_unlock_schedule` tool that doesn't exist.
- Lesson candidate (not added to Engineering Lessons, one instance): a gate validated on one asset (HYPE) and on the null rate at one moment still hid the normal case. See [[17th Oct 2026]].
