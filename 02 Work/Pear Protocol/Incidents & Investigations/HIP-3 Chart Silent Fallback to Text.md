---
type: investigation
area: pear
status: resolved
created: 2026-07-17
---

# HIP-3 Chart Silent Fallback to Text

## Symptom

A request like "show me long eth short spcx chart" returned prose only, never a chart — even on an explicit chart request, with no error shown.

## Context

Applied to any `pc`/`bc`/`perfc`/`ac` request naming an `xyz:`/HIP-3 asset.

## Initial Hypotheses

Chart generation itself failing silently for HIP-3 assets.

## Evidence

`generateOne` was returning `null` with no reason captured anywhere in the response.

## Investigation

Traced the chart plan from composer prose through to `ChartService.generateOne`. The composer's prose-extraction step (`composer.extract`) drops the deployer prefix (`xyz:SPCX` → `SPCX`). `normalizeToHlTicker` keeps the bare ticker, the unlisted-asset guard checks only the *core* order book (where the bare ticker isn't listed), and the plan silently drops.

## Finding / Root Cause

Deployer-prefix loss during prose-to-plan extraction, compounded by a guard that only checks the core order book rather than the full asset registry.

## Fix / Decision

`ChartService.normalizeSymbol` recovers the canonical dex-prefixed name from the asset registry, core-first (so a symbol collision can't repoint a real core chart to a HIP-3 venue). `generateOne` now returns a discriminated result (image or a reason) instead of `null`, so a failed chart appends an explicit reason instead of silently falling back to text.

## What Fooled Us

The failure was a silent `null` with no distinguishing signal from "nothing was requested" — nothing in the response shape said a chart had been attempted and failed.

## Remaining Unknowns

The upstream data issue (`composer.extract` dropping the deployer prefix even when the user typed it explicitly) was deliberately left unfixed here — a separate, bigger data-flow change.

## Related Work

[[My Work at Pear Protocol#2026-07-17 — Charts silently failed to text on HIP-3 assets; the composer promised retries it could never keep]]
[[17th Jul 2026]]
[[Ambiguous HIP-3 Symbol Crashed the Whole Reply, Repeatedly]]
