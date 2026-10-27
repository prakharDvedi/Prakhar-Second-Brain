---
type: investigation
area: pear
status: diagnosed
created: 2026-10-21
---

# Generated Baskets Lost on Net-Long Semis Beta, Not Asset Selection

## Symptom

A first pass over `made_baskets` since 2026-09-22 (793 rows, 775 priced) showed the generated ideas net negative: mean −1.7%, 36% positive, long-only −5.5%, Semiconductors tag −8.2%, INTC in about 115 baskets.

## Context

- Generator: `apps/context/src/sentiment` (two-stage Claude research → selection when `BASKET_LLM_PROVIDER=claude`). The cron forces one long-only and one short-only pass per platform per run, plus an equity (HIP_3) pass. `MAX_ASSET_APPEARANCES = 3` is per platform per run only.
- Provider by date (from [[2nd Oct 2026]] and [[12th Oct 2026]]): Claude 09-22 → 10-01, **OpenRouter 10-02 → 10-11**, Claude from 10-12. So "Claude-generated" covers 466 of the 793 rows; `made_baskets` has no provider column.
- No PnL column; reconstructed from `asset_prices_1h` closes. Read-only, quant DB. Scripts in the session scratchpad (`lib.js`, `a1.js`–`a9.js`), not committed.

## Initial Hypotheses

1. Bad asset selection (stale or priced-in theses).
2. Early runs (Sep 23 – Oct 6) worse because of a generator/prompt problem later fixed.
3. Momentum chasing on longs.

## Evidence

- **Reconstruction checks.** Bar `time` is the open time (INTC 13:00 bar close 125.62 = 14:00 bar open 125.67), and runs fire at 14:00:16, so "bar at run_at" looks 1h ahead. Last closed bar: −1.65%. Entry timing within +4h: −1.57% to −1.65%. Hand-checked 7 baskets, one against raw SQL. XAU/XAG proxy: 35 baskets, excluding them −1.47%. Unpriced: SCR 8, US100 5, MAVIA 2, TST 2, CHILLGUY 1.
- **Weights matter.** `asset1_weight_pct` is the long leg (`BASKET_LONG` in `quant.service.ts`), mean 0.63. Two-sided: stat weights −1.04%, 50/50 +0.49%. Two-sided baskets are about 26% net long at stat weights.
- **Factor vs selection** (each leg swapped for its sector benchmark: crypto → BTC, semis → equal-weight 12-name semis index, other equities → XYZ100): all −1.65% = factor −2.30% + selection **+0.65%** (day-bootstrap CI +0.01..+1.31). Semis tag: −8.08% = −8.48% factor + 0.40% selection. Long-only selection −0.67% (CI −1.62..+0.24).
- **Random benchmark.** Same structure and weights, random legs from the same platform universe: random −2.08%, actual at the 98th percentile; sector-matched random −2.52%, actual at the 100th.
- **Concentration.** INTC/MU/AMD/NVDA long contributions sum to −1404 pp, more than the whole net −1279 pp. INTC long in 108 of 117 legs, signed −17.4%, up to ~10 legs a day across platforms.
- **Fixed horizons.** 4h −0.09%, 24h −0.08%, 72h −0.22%, 7d −0.77%; selection ≈ 0 at every horizon. Only short-only is positive (7d +2.86%, n 67, wide CI).
- **By provider era, fixed horizons (stat weights):** Claude 09-22..10-01 72h −0.19% / 7d −0.13%; OpenRouter 10-02..10-11 72h −0.19% / 7d −1.25%; Claude 10-12..10-20 72h −0.37% (7d n=50 only). No era shows an edge.
- **Theses** (36 read, stratified; plus regex buckets): unlock/delist/exploit shorts selection +5.3% (CI +1.4..+8.6, n 44); buyback/burn/fee-switch longs −2.15% (CI −4.0..−0.4, n 65). More specific theses did not do better (digits <10: +1.54% selection; ≥25: −0.15%). 65 asset-days had the same asset long in one basket and short in another on the same run.
- **Stats fields.** `roll_zscore` < −1 (n 56) 7d +0.6–0.9% vs about −0.4..−0.9% otherwise; corr −0.08. 60% of two-sided baskets entered at z ≥ 0. Only 6 rows with hedge < 0 (pear-stat-rs#150 barely touches this sample).
- **Leverage.** Two-sided only, mean 4.47x, 36 at 10x. Levered to-now −6.27%. With 1h highs/lows (all legs' worst in the same hour, an upper bound), 45% lost ≥20% of margin, 4.4% ≥80%.
- **signalsv3** (different universe, half unpriced): hold-to-now +0.49%, 7d −0.19%.

## Investigation

Rebuilt the PnL with entry/weight/proxy sensitivities, then decomposed by sector factor, fixed horizons, run week, provider era, prior momentum (7d and 21d universe percentile), thesis buckets, stats buckets, and drawdown. Backtested drop-only rules (cooldowns, caps). CIs bootstrap over run days (29 days); positions overlap heavily.

## Finding / Root Cause

- **Strong evidence:** the loss is mostly factor exposure, not selection. The set runs about 31% net long of gross (equity 22%, semis 12.7%) through a window where semis fell 14.4%. The sources are structural: the forced long-only pass, long-skewed stat weights on two-sided baskets, a forced equity pass, and no cross-run/cross-platform repeat limit (INTC daily).
- **Strong evidence:** "early runs worse" is a holding-period artifact. At a fixed 7d horizon, W1 (09-22) is +0.13% and W3 (10-06..12) is the worst at −2.02%; early baskets just sat through the Oct 6–20 semis selloff longer.
- **Strong evidence:** no edge at any fixed horizon, in any provider era. Selection is roughly random-plus, not alpha.
- **Hypothesis:** mild 3-week momentum tilt on longs (41% of long crypto legs in the top 21d quartile vs 25% neutral; INTC +38% Sep 1 → Sep 22).
- **Hypothesis:** `roll_zscore` ≤ −1 helps two-sided entries (small n).

## Fix / Decision

None yet. Read-only investigation. Staged plan: [[Basket Quality — Staged Improvement Plan]].

## What Fooled Us

- "To now" PnL with different holding periods made the early runs look broken; fixed horizons removed that.
- Calling the whole sample "Claude-generated": 10 of the 29 days ran on OpenRouter.
- Equal-weight vs stat-weight two-sided legs flips the two-sided sign (−1.04% vs +0.49%).
- Reading the INTC/semis loss as bad picks: random baskets from the same universe lost more.

## Remaining Unknowns

- Whether the Pear app executes baskets at `asset1_weight_pct` or another weighting.
- Whether generation saw stale prices during the 7–11 Oct price-fetch stall (1h series now have no gaps; backfilled or never hit this table).
- Fees, funding and slippage are not modelled. Single regime, 29 days.
- Factor benchmarks are a choice: the semis index includes INTC/MU; crypto is BTC-only, so part of short "selection" is alt beta vs BTC.

## Related Work

[[2nd Oct 2026#Generated-idea PnL: Claude (Hetzner) vs OpenRouter (Railway) — read-only, quant DB]] (earlier read, 50/50 weights, 6 run days)
[[Familiar-Asset Ranking — Final Plan]] (#1069 post-release read)
[[21st Oct 2026]]
