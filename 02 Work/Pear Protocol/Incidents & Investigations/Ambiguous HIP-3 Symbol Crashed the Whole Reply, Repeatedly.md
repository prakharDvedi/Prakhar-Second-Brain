---
type: investigation
area: pear
status: resolved
created: 2026-08-24
---

# Ambiguous HIP-3 Symbol Crashed the Whole Reply, Repeatedly

## Symptom

A bare ticker listed under more than one HIP-3 deployer (e.g. `AVGO` → `para:AVGO` / `xyz:AVGO`) crashed the entire chat reply into a generic "Sorry — I hit an error completing that. Please try again."

## Context

First observed on a basket-building chart request (production, twice ~4 minutes apart, trace `1848c72d…`). After that call site was fixed, the identical message recurred 13 times in the 3 days that followed, through unrelated query types.

## Initial Hypotheses

That the bug was specific to chart generation.

## Evidence

- `AssetCacheStore.findInstrumentBySymbol` intentionally throws a caller-facing `SymbolResolutionError` naming the valid prefixed choices when a bare ticker is ambiguous.
- A live Discord alert traced to `0e25d734e9eb599b94a8b4c30ee51249` showed the same generic fallback firing on a plain z-score/correlation query — not a chart request at all.

## Investigation

Traced the throw's propagation: `composer.extract` drops the deployer prefix from prose, the bare ticker hits the ambiguity guard, and the error propagates uncaught to whichever outer catch was in the call stack. First fixed only in `ChartService.generateOne` (the call site originally observed). Confirmed via the recurrence count that `DagService`'s two other outer catches (`run`'s sync path, `runAsyncInner`'s streaming path) never special-cased this error the way the chart path now did.

## Finding / Root Cause

The same underlying `SymbolResolutionError` is reachable from at least three structurally different call sites (chart generation, pair-stats analysis, general compose), and only one was fixed initially.

## Fix / Decision

Caught `SymbolResolutionError` specifically — not a blanket catch-all — in both remaining `DagService` entry points, using the error's own already-actionable message instead of the generic fallback.

## What Fooled Us

The first fix looked complete because it addressed the exact call site from the first observed trace. The recurrence count (13 times in 3 days) was the signal that the same root cause was reachable elsewhere.

## Remaining Unknowns

The upstream cause (`composer.extract` dropping a deployer prefix the user may have typed explicitly) remains unfixed by design — a separate, bigger data-flow change.

## Related Work

[[My Work at Pear Protocol#2026-08-13 – 08-14 — Chart edge cases: paging on bad input, an ambiguous ticker crashing the whole reply, and wrong display tickers]]
[[My Work at Pear Protocol#2026-08-24 — A confirm-time margin gate, a recurring ambiguous-symbol crash, and a hidden 2x-budget confirm]]
[[13th Aug 2026]]
[[24th Aug 2026]]
[[HIP-3 Chart Silent Fallback to Text]]

## Lessons

[[Engineering Lessons#A narrow fix should be checked against sibling call sites of the same root cause]]
