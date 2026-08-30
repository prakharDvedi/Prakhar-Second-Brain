---
type: investigation
area: pear
status: resolved
created: 2026-07-22
---

# HIP-3 Margin Backfill Passed Orders It Could Never Fund

## Symptom

Repeated confusing execution-time failures for real users trading `xyz:` stock perps:
```
Failed to execute trade to Hyperliquid: Order 0: Insufficient margin to place order. asset=110062
```

## Context

Under Hyperliquid's "Disabled (No Dex Abstraction)" mode, each HIP-3 DEX holds isolated collateral — core USDC does not back it.

## Initial Hypotheses

A margin-calculation bug in the order-sizing logic.

## Evidence

A HIP-3 leg reports `availableToTrade = 0` in the per-DEX view regardless of actual funding (funds pool in core) — exactly the condition that triggered an existing "USDC availability is 0 → backfill core USDC" pre-check branch.

## Investigation

Traced the pre-check's backfill logic and found it was unconditionally crediting core USDC to any zero-availability leg, including HIP-3 legs under No-Dex-Abstraction where that credit is not actually usable.

## Finding / Root Cause

The backfill branch didn't distinguish "zero because unfunded" from "zero because this DEX can't draw on core at all." For a No-Dex-Abstraction account with an empty HIP-3 pool, the backfill let an unfundable order pass validation and fail only at execution.

## Fix / Decision

Added `HyperliquidMarketService.getNoDexAbstraction(wallet)` (cached 30s) and gated the backfill branch to skip only when a HIP-3 USDC leg is present *and* the account is confirmed No-Dex-Abstraction — preserving the backfill for the unified/dex-abstraction accounts it was built for. Fails open (assumes backed) if the mode probe itself errors.

## What Fooled Us

The failure surfaced as a hard execution-time error in the (untraced) worker, not a clean pre-Confirm rejection — making it look like a random venue failure rather than a predictable validation gap.

## Remaining Unknowns

None noted — verified against 3 explicit cases (No-Dex-Abstraction + empty pool rejected; dex-abstraction + empty pool still backfills; core-only leg never consults the mode at all).

## Related Work

[[My Work at Pear Protocol#2026-07-22 — HIP-3 orders were passing margin validation they could never actually fund]]
[[22nd Jul 2026]]

## Lessons

[[Engineering Lessons#Fail-closed only where the account's actual state says so — not a blanket rule]]
