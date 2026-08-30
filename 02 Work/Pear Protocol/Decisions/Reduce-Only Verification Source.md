---
type: decision
area: pear
status: active
created: 2026-07-29
---

# Decision: Reduce-Only Verification Source

## Context

A reduce-only rejection ("would increase position") needed re-verification against live state before deciding whether to downgrade it from FAILED, but the verification source itself has different confidence/cost tradeoffs.

## Problem

Which live-state read to trust for the downgrade decision, given the read source can itself be stale or wrong.

## Options / Alternatives

- **Direct Hyperliquid `clearinghouseState` read:** higher-confidence, the exchange's raw truth. Needs cross-module wiring and HL-asset-id → leg-name mapping not currently in place.
- **`getPositions(syncPosition=true)` via the Pear backend (chosen):** forces a BE-mediated reconcile against live HL before returning — materially fresher and non-circular versus the default cached read, and requires no new module dependency or DI wiring.

## Decision

Use `getPositions(syncPosition=true)`, explicitly chosen over direct HL access, to keep the fix minimal and low-blast-radius.

## Why

The direct-HL path would be more accurate but requires wiring effort disproportionate to a first version of the gate; the asymmetric downgrade-only design (see [[Engineering Lessons#A downgrade-only gate is safer than a symmetric recheck]]) already bounds the risk of using a slightly-less-authoritative source.

## Tradeoffs

`sync=true` is BE-mediated: it forces reconciliation but does not return the exchange's raw truth, so residual staleness is possible — mitigated by the downgrade-only gate, a tight epsilon, a flip guard, and a bail on uninterpretable data. Bounded by a 15s timeout that falls through to FAILED on read failure.

## Evidence

[[My Work at Pear Protocol#2026-07-27 – 07-29 — A "would increase position" rejection didn't mean the trade failed; Twitter labeling failures were unreadable]]
[[Reduce-Only Rejection Didn't Mean the Trade Failed]]

## Later Evidence

Direct HL `clearinghouseState` verification was noted as a deliberate follow-up, not yet done as of the source PR.
