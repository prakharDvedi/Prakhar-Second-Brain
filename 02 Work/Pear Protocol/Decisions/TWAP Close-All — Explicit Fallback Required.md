---
type: decision
area: pear
status: active
created: 2026-08-18
---

# Decision: TWAP Close-All — Explicit Fallback Required

## Context

An account-wide close-all request could silently downgrade a user's explicit TWAP instruction into market orders.

## Problem

Whether execution-mode changes for a close-all should ever happen implicitly, versus requiring the user to choose.

## Options / Alternatives

- **Allow a silent downgrade to market when TWAP isn't feasible account-wide (previous behavior):** simpler, but changes the user's stated execution style without them choosing it.
- **Require an explicit fallback choice before changing execution mode (chosen).**

## Decision

Keep account-wide close-all operations market-only by default, and require an explicit fallback choice before changing the execution mode at all. Individual TWAP closes are sequenced one ticket at a time rather than batched.

## Why

Execution style (market vs. TWAP) is something the user stated on purpose; changing it without an explicit choice removes information the user deliberately provided.

## Tradeoffs

Sequencing individual TWAP closes one at a time is slower than a batched approach, accepted in exchange for not silently reinterpreting the user's execution-style instruction.

## Evidence

[[My Work at Pear Protocol#2026-08-18 — Elfa signal triggers accepted invalid conditions, and close-all could silently downgrade a TWAP to market]]

## Later Evidence

Focused regression coverage for TWAP sequencing confirmed passing at merge time.
