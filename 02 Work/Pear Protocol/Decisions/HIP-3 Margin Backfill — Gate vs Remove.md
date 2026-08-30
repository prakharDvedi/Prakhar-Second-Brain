---
type: decision
area: pear
status: active
created: 2026-07-22
---

# Decision: HIP-3 Margin Backfill — Gate vs Remove

## Context

A margin-backfill branch (credit core USDC when a leg reports `availableToTrade = 0`) was letting unfundable No-Dex-Abstraction HIP-3 orders pass validation.

## Problem

The backfill was originally added to correctly handle unified/dex-abstraction accounts. Simply removing it to fix the No-Dex-Abstraction case would reintroduce a false-reject for the population it was built for.

## Options / Alternatives

- **Remove the backfill branch entirely:** fixes the No-Dex-Abstraction over-permissiveness, but breaks the unified/dex-abstraction accounts that legitimately need the backfill.
- **Gate the backfill on the account's actual abstraction mode (chosen):** reads `webData3.userState.abstraction` and skips the backfill only for the specific population that can't use it.

## Decision

Gate, don't remove. Added a cached (30s TTL, per-wallet) mode probe and narrowed the existing branch to skip only when a HIP-3 USDC leg is present *and* the account is confirmed No-Dex-Abstraction.

## Why

The behavioral delta needed to be confined to exactly one population (No-Dex-Abstraction + empty HIP-3 pool); every other population — core perps, unified/portfolio-margin/dex-abstraction accounts, and funded HIP-3 pools — needed to stay byte-identical.

## Tradeoffs

Fails open on a probe error (assumes backed), so a transient RPC blip can't harden a legitimate trade into a rejection — Hyperliquid remains the backstop either way. No extra RPC on the pure-core-order hot path; HIP-3 orders make one cached, deduped call.

## Evidence

[[My Work at Pear Protocol#2026-07-22 — HIP-3 orders were passing margin validation they could never actually fund]]
[[HIP-3 Margin Backfill Passed Orders It Could Never Fund]]

## Later Evidence

36/36 sibling `open-position.*`/ticket-gate specs stayed green after the change, confirming no regression to the other populations.
