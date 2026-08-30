---
type: investigation
area: pear
status: resolved
created: 2026-07-17
---

# Composer Promised a Retry Capability That Didn't Exist

## Symptom

After a `tradeExecution` timeout, the composer told a user "I'll keep attempting it until it lands or I hit a hard limit. I'll let you know as soon as it's confirmed." The user waited roughly 50 minutes for a confirmation that could never arrive.

## Context

Production trace, session showing a 60s LLM abort on the trade-execution node, followed by the fabricated promise. The user's own standing history instruction ("if it ever times out instantly assume i want to try again") plausibly amplified the fabrication.

## Initial Hypotheses

None recorded — the mechanism was clear once the trace was read: no retry loop, scheduler, or notification hook exists behind a failed step.

## Evidence

No code path corresponding to "keep attempting" or "notify when confirmed" exists in the failed-step handling.

## Investigation

Reviewed the composer's failed-steps prompt rule and confirmed it had no explicit prohibition on promising future or background action.

## Finding / Root Cause

The composer prompt allowed (by omission) claiming a capability the system doesn't have, especially when a user's standing instruction created pressure to "comply."

## Fix / Decision

Extended the composer's failed-steps rule to explicitly forbid promising future/background action, including acknowledgment-phrased compliance ("noted, I'll auto-retry from now on") — the same false promise in different clothing. Deliberately scoped to failed steps only, not a blanket no-future-promises rule, to avoid colliding with the genuine scheduler/monitoring capability.

## What Fooled Us

An acknowledgment-shaped response ("noted, I'll...") reads as compliant and safe, but is functionally identical to the more obvious fabricated promise.

## Remaining Unknowns

None noted — verified via a dedicated real-LLM eval (15/15 clean across 3 query variants on the final prompt; the first draft failed 5/5 on the standing-rule query).

## Related Work

[[My Work at Pear Protocol#2026-07-17 — Charts silently failed to text on HIP-3 assets; the composer promised retries it could never keep]]
[[17th Jul 2026]]

## Lessons

[[Engineering Lessons#A capability that doesn't exist must never be implied to a user]]
