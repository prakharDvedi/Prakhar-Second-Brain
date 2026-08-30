---
type: decision
area: pear
status: active
created: 2026-07-28
---

# Decision: Directional Baskets — Skip Pair-Stat Enrichment

## Context

Basket generation was structurally forced to produce a two-sided (long + short) basket even when conviction was genuinely one-sided, because pair-stat enrichment assumes a spread between two sides.

## Problem

A directional (long-only or short-only) basket has no spread, so correlation, z-score, hedge ratio, and spread volatility are undefined concepts, not missing numbers.

## Options / Alternatives

- **Compute placeholder/best-effort stats for one-sided baskets anyway:** would let the pipeline stay uniform, but manufactures numbers for concepts that don't apply.
- **Bypass pair-stat enrichment entirely for directional baskets (chosen):** `getBasketStats` requires both sides and throws for a one-sided basket; the directional path skips it rather than calling into an API that structurally can't serve it.

## Decision

Directional baskets bypass pair-stat enrichment entirely. Stat fields persist as null, and `suggested_leverage` is null since it's derived from measured spread volatility.

## Why

Inventing numbers for an undefined concept (spread volatility on a basket with no spread) is worse than honestly showing null — the shared `QuantService.normalizeBasketInput` guard was deliberately left untouched rather than loosened, since the chat basket node uses the same service and loosening it there would let one-sided requests reach an API that cannot serve them.

## Tradeoffs

The directional path calls `getListedAssetIds` directly for its own listed-check (since the normal listed-check rides on the stats response), keeping the cron's exclude-then-retry loop identical on both paths.

## Evidence

[[My Work at Pear Protocol#2026-07-28 – 08-01 — Basket generation could only produce two-sided baskets, and had no way to tag them]]
[[Features Owned#Basket generation]]

## Later Evidence

`tsc --noEmit` clean; 25/25 tests passing (up from 13) at time of merge.
