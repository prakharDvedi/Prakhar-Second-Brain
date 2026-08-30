---
type: knowledge
domain: Web3 & Trading
confidence: deep
last_reviewed: 2026-08-30
---

# Trading Execution Safety

## In Simple Terms

Money-moving software should assume that state can change between planning and execution, that external venues can reject individual legs, and that the system must never silently turn an explicit user instruction into a different action.

## What I Understand

Pear is the strongest evidence base. I worked on confirm-time margin checks, leverage validation, HIP-3 collateral rules, reduce-only verification, TWAP close-all behavior, partial-fill messaging, account/venue binding, and trigger-condition validation. The recurring pattern is to place safety checks immediately before writes, use fresh state where needed, fail closed when the risk is material, and disclose changes that affect user exposure.

## Why It Matters

A technically successful request can still violate user intent or leave an unhedged position. In trading, incorrect defaults, stale state, silent fallbacks, and misleading explanations are safety failures, not merely UX issues.

## How I Learned This

- [[02 Work/Pear Protocol/Decisions/Confirm-Time Margin Gate]]
- [[02 Work/Pear Protocol/Decisions/TWAP Close-All — Explicit Fallback Required]]
- [[02 Work/Pear Protocol/Decisions/Reduce-Only Verification Source]]
- [[02 Work/Pear Protocol/Incidents & Investigations/Partial Fill Alert Blamed the Wrong Cause]]
- [[02 Work/Pear Protocol/Incidents & Investigations/Reduce-Only Rejection Didn't Mean the Trade Failed]]
- [[02 Work/Pear Protocol/Incidents & Investigations/HIP-3 Margin Backfill Passed Orders It Could Never Fund]]
- [[02 Work/Pear Protocol/Incidents & Investigations/Silent TP-SL Default on the Streaming Chat Path]]
- [[02 Work/Pear Protocol/Incidents & Investigations/TRIGGER-LADDER JSON Crash Silently Substituted the User's Trade]]

## Real Failure Modes I've Seen

- A streaming context bug replaced an explicit 5% profit instruction with a 100% default.
- A leverage cap changed a $1,000 stated margin into $1,999.95 while the ticket failed to disclose the delta.
- HIP-3 collateral rules allowed an unfundable order through pre-check and failed only at execution.
- A reduce-only rejection was treated as a failed user request even when another action had already flattened the position.
- A partial fill left the user unhedged while the alert asserted a disproven notional-floor cause.
- An account-wide close-all could silently downgrade TWAP to market.

## Engineering Tradeoffs

- Fail-closed gates add friction during stale/unavailable reads, but protect the money-moving path.
- Fresh reads add latency and can still be imperfect; downgrade-only decisions prevent uncertainty becoming false success.
- Sequencing individual TWAP closes is slower than batching, but preserves execution-style intent.
- Keeping a backfill for some account populations is safer than deleting it globally.

## Patterns / Rules I Currently Believe

**Strong evidence:** Validate and disclose immediately before the side effect. Preserve explicit execution mode and user intent unless the user chooses a fallback.

**Current belief:** The system should report what the venue proves, not a convenient explanation inferred from a generic error.

**Open question:** The vault does not document a complete compensation/rollback strategy for non-atomic multi-leg execution.

## Examples From My Work

- Confirm-time margin gate: fresh `availableToTrade`, 100bps buffer, reduce-only exempted.
- Reduce-only verification: forced live-position reconcile, downgrade-only, 15s timeout fallback.
- TWAP close-all: market-only by default; alternate execution requires explicit choice.

## Things I Still Don't Fully Understand

- Atomicity, compensation, and recovery for multi-leg orders across venues.
- Venue-specific liquidation, margin, and settlement behavior beyond the documented cases.

## Related

- [[Multi-Adapter & Multi-Venue Consistency]]
- [[Retry Semantics & Failure Recovery]]
- [[Structured Outputs & Validation]]
- [[Authentication, Authorization & Trust Boundaries]]
- [[Wallets & Account Binding]]

