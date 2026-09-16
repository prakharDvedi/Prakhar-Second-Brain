---
type: investigation
area: pear
status: resolved
created: 2026-09-02
---

# One Service URL Fed Two Different Services, Nulling Basket Stats

## Symptom

Production `MadeBaskets` cron (in `apps/context`) began emitting a misleading `No baskets generated` alert and retrying. Baskets were in fact still being generated and persisted — but their basket-stat columns came back null. Reported in the incident writeup (PR #677): the Sep 2 batch held 3 Hyperliquid + 2 Lighter + 2 Symmio baskets, all with null stat fields; the last batch with stats stored for all three platforms was Aug 27.

## Context

The Aug 27 config change (commit `11a29155`) repointed `context`'s `STATS_BASE_URL` default to the TypeScript categories service (`pear-pair-signals`), on the assumption that `GET /categories` was its only consumer.

In the running `context` app the same `STATS_BASE_URL` fed **two** consumers on **two different services**:
- `AssetUniverseProvider` → `GET /categories` — served by the TS categories service (`pear-pair-signals`).
- `QuantService.getBasketStats()` → `POST /stats/basket` — served by the Rust stats service (`pear-stat-rs`).

## Initial Hypotheses

The `No baskets generated` alert reads as "generation failed." The alert text pointed away from the real cause: generation was fine; only the stats-enrichment call was failing.

## Evidence

Confirmed this session:
- Code: `STATS_BASE_URL` genuinely had two consumers on different routes — `asset-universe.provider.ts` (`/categories`) and `quant.service.ts` `getBasketStats()` (`/stats/basket`). `indexing` also hit `/categories` via `tokenomist-index`.
- Live route probes: `GET /categories` → 200 on `pear-pair-signals`, 404 on `pear-stat-rs`; `POST /stats/basket` → 404 on `pear-pair-signals`, 401/201 on `pear-stat-rs` (201 with a valid multi-leg body + `STATS_API_KEY`). So the two routes provably live on different services.
- `context-app` carried an **explicit** `STATS_BASE_URL` Railway override (value not read, per handling rule) — the live 404s on `POST /stats/basket` are consistent with it pointing at `pear-pair-signals`. The explicit override, not the code default, is what actually governed prod, so the code default change alone did not fully explain prod behaviour until the override was accounted for.

Reported in the incident writeup, not independently re-queried this session: the `made_baskets` null columns / batch composition, the 5× retry, and that the Aug 31 failure was a separate OpenRouter monthly-limit incident.

## Finding / Root Cause

A single config value (`STATS_BASE_URL`) silently fanned out to two consumers whose endpoints live on different services. Collapsing it onto the categories host made `POST /stats/basket` 404 (`Cannot POST /stats/basket` on `pear-pair-signals`). The cron treated the missing stats as a failed basket attempt, retried, and raised `No baskets generated` even though rows were being written — just with null stats.

This is distinct from the by-design null stats on directional baskets — see [[Directional Baskets — Skip Pair-Stat Enrichment]]. There, one-sided baskets legitimately have no spread. Here, two-sided baskets that *should* have stats were nulled by a misconfiguration.

## Fix / Decision

Split the one variable into two, so each route names its own service:
- `CATEGORIES_BASE_URL` → `GET /categories` (`pear-pair-signals`). `context` `AssetUniverseProvider` and `indexing` `tokenomist-index` both read this.
- `STATS_BASE_URL` → `POST /stats/basket` (`pear-stat-rs`, restored). `QuantService.getBasketStats()` keeps this.

`indexing` only ever calls `/categories`, so its var was renamed `STATS_BASE_URL` → `CATEGORIES_BASE_URL` rather than carrying a dead second var. Both URL vars moved from `z.string()` to `z.url()` so a malformed override fails at boot, not at request time.

Deployed as an emergency hotfix straight to `main` (PR #677, merged 2026-09-02 ~06:25 UTC); backported to `development` via PR #678 (cherry-pick, open in review) to re-level the branches. Prod Railway vars updated by the owner and both `context-app` + `indexing-prod` redeployed SUCCESS; verified services healthy and the new vars present (names only). Dev env needed no var changes — it had no explicit override, so its consumers fall back to the correct code defaults.

## What Fooled Us

The failing config value was named for only one of its two consumers (`STATS_BASE_URL` — stats), so the reasonable-looking "it's only used by categories" assumption behind commit `11a29155` was invisible: nothing at the call sites signalled that the same var also routed `/stats/basket`. And the alert (`No baskets generated`) described a failure mode that wasn't happening.

## Remaining Unknowns

The Aug 27–Sep 2 baskets already persisted with null stats are not retroactively backfilled — the fix is forward-only. Whether a backfill of those rows is wanted is open.

## Related Work

[[My Work at Pear Protocol#2026-09-02 — One service URL fed two different services, nulling two-sided basket stats]]
[[2nd Sep 2026]]
[[Directional Baskets — Skip Pair-Stat Enrichment]]

## Lessons

[[Engineering Lessons#One config value shared by two consumers on different services is a hidden fan-out]]

## Later correction — 2026-09-03

- The earlier branch-status wording above is historical and became stale: PR #678 was subsequently merged into `development` as a clean cherry-pick of the same implementation already merged into `main` by PR #677. The two branches were re-levelled; #677 was not merely a placeholder.
- A current probe on 2026-09-03 confirmed the Rust service is live: `/health` returned 200, `GET /stats/basket` returned 405 (method not allowed), and unauthenticated `POST /stats/basket` returned 401 with `API_KEY_MISSING`. This confirms deployment, route registration, and authentication gating independently; the matching client/server key for the Hetzner harness remains unconfirmed.
