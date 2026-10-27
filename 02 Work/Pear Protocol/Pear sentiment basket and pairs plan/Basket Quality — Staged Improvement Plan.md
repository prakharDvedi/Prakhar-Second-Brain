---
type: plan
area: pear
status: proposed
created: 2026-10-21
---

# Basket Quality — Staged Improvement Plan

Evidence: [[Generated Baskets Lost on Net-Long Semis Beta, Not Asset Selection]]. Nothing here is implemented or approved. Tracked in [#1162](https://github.com/pear-protocol/agent-pear-v2/issues/1162).

## Goal

Generated baskets that are not a hidden net-long equity bet, don't repeat or contradict themselves, and show positive **selection** (return after removing sector factor) at a fixed 72h/7d horizon. A raw "to now" average is not the target: it mostly measures the market.

## Rules for every stage

- **One change per stage.** Ship it, read it, then decide on the next. Two changes in one window can't be told apart.
- **Two kinds of check.**
  - *Mechanical checks* (net exposure, repeats, contradictions, leverage spread) come straight from the stored rows. They pass or fail on the first run day.
  - *PnL checks* need time. About 25 baskets a day (HL + Lighter) gives ~175 in 7 run days. With day-clustered CIs of roughly ±0.7pp, only effects of about 1pp or more show up. PnL is a guardrail ("didn't get worse"), not proof, for anything smaller.
- **Backtest first, forward second.** Each stage is replayed on the 09-22 → now sample before it ships, then read forward for at least 7 run days.
- **Readout:** the same report every time (Stage 0 script): mean, median and hit rate at 24h/72h/7d, factor vs selection split, net long/equity/semis exposure, top-asset share, same-day contradictions, levered drawdown. Day-bootstrap CIs.
- **Advance rule:** the mechanical target is met, and 72h selection is not worse than the baseline beyond the CI. Otherwise roll back the one change.
- **Split by provider.** Runs on different LLM providers are separate samples.

## Stage 0 — Measurement baseline (no generator change)

Why first: today there is no repeatable read, no provider tag on rows, and an unknown execution weighting. Every later stage depends on these.

1. **Repeatable eval.** Port the investigation scripts (session scratchpad `lib.js`, `a3.js`) next to the existing box scripts `~/pear/pnl/`. Use the last closed 1h bar as entry (bar `time` is the open time). Output the readout above.
2. **Record provider/model per run.** `made_baskets` has no such column. Options: a quant schema column (owned by whoever owns the quant DB), or a run manifest in the context DB/Redis keyed by `run_at`. Needs a decision.
3. **Find which weights the app executes.** Does the Pear app/FE size a basket from `asset1_weight_pct`, 50/50, or user input? Stage 1 depends on this.
4. **Freeze a baseline** on Claude-era runs 10-12 → 10-25 (stat weights and 50/50 both reported).

Done when: one command produces the report, and the baseline is written down here.

## Stage 1 — Make two-sided baskets dollar-neutral

- **Evidence:** stat weights put the long leg at 0.63 on average, so two-sided baskets are about 26% net long. Two-sided: −1.04% at stat weights vs +0.49% at 50/50 (*Confirmed* on this sample, one regime).
- **Change:** depends on Stage 0.3. If the app uses `asset1_weight_pct`, store/display 50/50 (or a beta-neutral split) for two-sided baskets. If it already uses 50/50, this stage is a no-op and the earlier "−1.04%" was never the real user outcome.
- **Where:** where the stats weights land on the row (`made-baskets-cron.service.ts` ~L293, `main.ts` ~L241), or the consumer.
- **Mechanical target:** mean two-sided net notional ≈ 0.
- **Risk:** legs with very different volatility give a noisier spread at 50/50. Watch pair volatility.

## Stage 2 — Stop forcing a long-only basket every run

- **Evidence:** the cron forces one long-only and one short-only pass per platform per run (`made-baskets-cron.service.ts` ~L148–162). Long-only: −5.36% to now, selection −0.67% (CI −1.62..+0.24). Dropping long-only: −1.65% → −0.23% (n 560). Short-only is the only consistently positive group (selection +3.51%, CI +1.09..+5.96).
- **Change:** remove the forced `long_only` pass; keep `short_only`. The model may still choose long-only unforced.
- **Needs a product decision** (Jeyd): are long-only ideas a product requirement? Alternative: keep it but pair it with an index hedge leg (XYZ100 / BTC), which makes it two-sided.
- **Mechanical target:** net long notional across the run below about 10% of gross (now 31%).

## Stage 3 — Repeat limits across runs and platforms

- **Evidence:** INTC/MU/AMD/NVDA longs alone exceed the whole net loss; INTC was long almost daily, up to ~10 legs a day. 65 asset-days had the same asset long in one basket and short in another. The current cap (`MAX_ASSET_APPEARANCES = 3`) is per platform per run.
- **Change:** in `made-baskets.service.ts` (`legCounts` / `formatRunAssetRules` / ranked-pool rotation), read recent `made_baskets` rows and enforce:
  - one direction per asset per day across platforms;
  - a cooldown per ticker+side (start at 3 days);
  - a cap per asset per day across platforms.
- **Backtest:** 7d cooldown −1.65% → +0.48% (n 66, CI −1.7..+2.8). Not significant. The case is concentration and contradiction, not proven PnL.
- **Risk:** the ranked pool is only ~20 assets. A cooldown can drain it; `MIN_FRESH_POOL` top-up and pool size need checking in a replay first.
- **Mechanical target:** 0 same-day contradictions; no asset in more than about 5% of a week's baskets.

## Stage 4 — Sector-neutral two-sided baskets

- **Evidence:** after removing the sector factor, selection is +0.65% (CI +0.01..+1.31); two-sided +0.68% (CI +0.07..+1.27). This is the only part that looks like skill.
- **Change:** a deterministic post-generation check (not a prompt-only rule) on net exposure per sector bucket (semis, other equities, crypto, commodities) at or below a set share of gross. Reject and retry like the existing listed-check. Add a short prompt line asking to pair within a sector.
- **Needs:** a sector map. Start from `asset_platforms.hl_dex` plus a semis list and the existing basket tags.
- **Mechanical target:** sector net exposure within the limit on every stored two-sided basket.
- **Risk:** fewer valid baskets; more retries and cost.

## Stage 5 — Research-stage "priced-in" guard

- **Evidence (Hypothesis, moderate):** 41% of long crypto legs and 39% of long semis legs were in the top quartile of 21-day return at entry (25% would be neutral). Longs whose thesis cites a recent gain: selection −0.63% vs +0.60%. Buyback/burn/fee-switch longs −2.15%. Unlock/delist/exploit shorts +5.3%. More numbers and dated events in a thesis did not help.
- **Change:**
  - Feed each candidate's prior 7d/21d return and universe percentile into the research input, as data rather than model memory.
  - Rule: a top-quartile 21d long needs a catalyst dated after the move, or it is `priced_in`.
  - Buyback/burn/fee-switch alone is not a long reason.
  - Dated supply events (unlocks, delistings, exploits) are preferred short reasons.
- **Validate offline first:** replay logged research inputs (as in the #1161 replay) before prod.
- **Mechanical target:** top-quartile 21d share on long legs near 25%.

## Stage 6 — Spread entry gate, shadow first

- **Evidence (Hypothesis, n 56):** `roll_zscore` ≤ −1 gave 7d +0.6–0.9% vs about −0.4..−0.9% otherwise; 60% of two-sided baskets entered at z ≥ 0.
- **First:** confirm the z sign for negative hedge ratios (pear-stat-rs#150).
- **Change:** log "would be gated" for 2 weeks, compare, then gate only if it holds.

## Stage 7 — Leverage

- **Evidence:** mean 4.47x, 36 baskets at 10x. 45% of levered baskets lost ≥20% of margin at some point; 4.4% ≥80% (upper bound).
- **Change:** cap suggested leverage (start at 5x) and size it from measured 1h drawdown, not only the vol scalar (`leverage.ts`, scalar 65).
- **Mechanical target:** share of baskets with ≥20% levered margin drawdown, read at 7d.

## Stage 8 — Later, once the structure is fixed

- Revisit the forced daily equity (HIP_3) pass.
- Compare providers (Claude vs cheaper) on the selection metric only. With the structural bets removed, that comparison becomes meaningful.

## Not doing

- "No semis longs": +0.19% on this sample, but that is hindsight on one selloff.
- Inverting direction: it only shorts the factor.

## Open decisions

1. Provider tag storage (Stage 0.2).
2. Executed weights (Stage 0.3).
3. Whether long-only ideas are a product requirement (Stage 2).
4. Cooldown length and caps against the ~20-asset ranked pool (Stage 3).

## Update (2026-10-22) Stage 0 progress

- **0.1 done:** `apps/context/src/eval/basket-eval.ts`, run with `pnpm --filter context eval:baskets` (commit `35ec9280`, branch `feat/basket-eval-script`, no PR yet). Lives in the repo, not in `~/pear/pnl/` as planned above. Reproduces the investigation's mechanical numbers exactly. PnL "to now" depends on the price cutoff, so it takes `--asof`. Details: [[21st Oct 2026#Update (late 21 Oct) #1162 Stage 0.1 eval script committed]].
- **0.2 open decision, new constraint:** a column on `made_baskets` would expose the model id to the agent LLM (`recent-made-baskets` tool), the public `/baskets/latest` API (`SELECT *`) and the twitter raw-SQL tool. Recommended: the context DB run manifest option above. See [[22nd Oct 2026]].
- **0.3, 0.4:** not started.


## Update (2026-10-22, later) Stage 0.2 decided: date map, no DB change

- Decision (mine): no DB writes or schema change for the provider tag. Neither option A (context DB run table) nor B (`made_baskets` columns).
- Instead `basket-eval.ts` has a hand-kept `PROVIDER_ERAS` list (claude from 09-22, openrouter from 10-02, claude from 10-12, earlier = "unknown") and the returns table adds an `all <provider>` row per era. Works because `BASKET_LLM_PROVIDER` is fixed per process with no fallback.
- Ceiling: someone has to add a line when the provider or `CLAUDE_MODEL` changes on the box. Revisit a per-run record if switches get frequent or for Stage 8.
- Uncommitted on `feat/basket-eval-script`. Spec 8/8, context `tsc` and biome clean.

## Update (2026-10-22, evening) Stage 0.3 partial, 0.4 provisional, BTC mismatch explained

- **0.3 (partial, from code):** stored `asset1_weight_pct` is the **4h** basket-stats weight at run time (`made-baskets.service.ts` 4h → 1d → 1h fallback). The agent (TG/web chat) does not trade the stored row: its basket prompt refetches `quant_basket_stats` and sizes off the live 4h row. So agent trades use stat weights, refreshed, and the stat-weight rows of the eval are the closer model for them. **Still unknown:** the external FE that reads `/baskets/latest`. That is the question for the trading app owner.
- **0.4 provisional** (`--since=2026-10-12 --until=2026-10-22 --asof=2026-10-22T08:00:00Z`, all Claude, 196 baskets, 10 run days):
  - 72h (n 135): all −0.45% (CI −1.25..+0.33), selection −0.07% (CI −0.40..+0.29). Two-sided stat −0.17%, 50/50 +0.21%. Long-only −2.00%, short-only +0.46%.
  - 7d (n 70, runs through ~10-15 only): all −1.26%, selection −0.51% (CI −0.94..+0.21). Long-only 7d selection −1.00% (CI −2.27..−0.07).
  - Exposure: net long 12.65% of gross (was 31% over 09-22 → 10-21). Semis now net **short** (−35% of semis gross). Two-sided mean long weight 60.5%, net notional +21%.
  - Repeats: ETH short 28 legs over 10 days; MSTR short and AMZN long up to 6 legs a day. 13 same-day contradictions.
  - Leverage: mean 4.31x, 6 at 10x, 14.3% lost ≥20% margin (upper bound).
  - Not the frozen baseline. Runs after 10-22 need 72h of prices (full 72h for runs to 10-25 by ~10-29, 7d by ~11-02).
  - Reading: unlike the earlier window, this one is not a semis-long story, and selection is about zero, not positive. Earlier +0.65% selection does not carry over to the Claude-only 10-12 window so far.
- **BTC mismatch explained (strong evidence):** same 09-22 → 10-21 sample at different `--asof` on 10-21: BTC "to now" 00:00Z −1.01%, 04:00Z −1.37%, 08:00Z −1.39%, 14:00Z −0.53%, 20:00Z +0.53%, while the basket mean stayed −1.43..−1.76%. The investigation's −0.45% fits a cutoff around 14:00Z. A price-cutoff difference, not a script bug. Exact cutoff of the investigation not recorded.

## Update (2026-10-22, night) The month before flips the long-only and short-only results

- Ran the eval on 08-22 → 09-22 (971 baskets, 926 priced, provider before 09-22 not in the date map, mostly pre-Claude). Forced long-only + short-only pass has existed since #354 (2026-07-29), so the same structure applied.
- That month the market rose (BTC +2.0% at 7d). At 7d: long-only **+2.73%** (CI +0.43..+5.10, 58% positive, selection +0.70% CI −0.38..+1.90); short-only **−6.82%** (selection −4.72%, CI −7.17..−2.35); two-sided at stat weights +1.45% vs 50/50 −0.08%; all baskets selection −0.15%.
- What-ifs that month at 7d: no long_only +0.81% → +0.20% (selection −0.15% → −0.42%), 50/50 selection −0.54%, "all fixes" −1.66%. The 09-22 → 10-22 what-if gains reverse.
- Reading (strong evidence across two months): long-only vs short-only results follow market direction. Selection stays near zero overall and changes sign between months. "Short-only works" and "50/50 beats stat weights" do not hold in the earlier month.
- Consequence: Stage 2 (`1947c597`, local, not pushed) is a risk choice (make the set near market-neutral, 25.6% → 3.3% net long), not a proven return gain. Its commit message overstates the case. Stage 1 evidence is weaker than written above. The real gap is pick quality (selection ≈ 0), which is Stage 5.
- Eval script: fixed a stack overflow (`Math.max(...bars)`) on long windows. Unpriced in that window: 1000-prefixed memes quoted without `k` (SHIB, PEPE, BONK, FLOKI) and a few others, so 45 baskets drop.

## Update (2026-10-22, late night) Stage 2 on hold, first per-pick read for Stage 5

- Stage 2 held by my call: `1947c597` stays local on `feat/basket-no-forced-long-only`, not pushed.
- Added `--picks` to the eval script (uncommitted on `feat/basket-eval-script`): each leg's return minus its sector at 72h/7d, signed by side, grouped by sector, platform, prior 21d/7d return vs sector (pooled quartiles), picked in the last 3 days, thesis bucket (regex). Each half of the window shown apart. Ran on 08-22 → 10-22 (4451 long legs, 3438 short legs at 7d).
- **Crypto "selection" is mostly alts vs BTC.** Long crypto +4.78% then −1.24%, short crypto −4.57% then +2.46% between halves. The BTC benchmark can't judge crypto picks. Needs an equal-weight crypto index before reading crypto pick quality.
- *(superseded, BTC-benchmark artifact, see Correction below)* **Only long group positive in both halves:** laggards (bottom quartile of prior 21d vs sector): 7d +2.20% (CI +0.81..+3.48), 59% hit, +3.11% / +0.89% by half, 72h +1.44%. Top-quartile run-up longs flip (+4.61% / −1.80%). Supports the Stage 5 "don't chase, prefer laggards" direction. Hypothesis: ~40 groups tested, one could be luck.
- Also same sign both halves: stock shorts vs XYZ100 +1.04% (+0.34% / +1.70%, 59% hit); semis longs vs semis index −0.37% (CI −0.81..−0.06); earnings-thesis longs slightly negative. No consistent difference for repeat vs fresh picks, platform or most thesis buckets.

## Correction (2026-10-22, late night) Equal-weight crypto benchmark

- Switched the eval's crypto benchmark from BTC to an equal-weight index of the crypto universe (uncommitted on `feat/basket-eval-script`). All factor/selection numbers before this point used BTC for crypto and are not comparable with runs after it.
- Re-ran `--picks` on 08-22 → 10-22. Crypto selection no longer flips: long crypto +0.95% / +0.52% by half, short crypto +0.13% / +0.88%. All longs +0.21% (+0.20% / +0.23%), all shorts +0.44% (+0.16% / +0.98%). Small, same sign in both halves, CIs cross zero.
- **Correction:** the "21d laggard longs" signal does not survive (+0.30%, −0.36% / +1.06%). It was alts vs BTC.
- *(does not survive a median index, see Correction (median) below)* **Strongest same-sign result now:** shorts on the top quartile of prior 7d return vs sector: 7d +2.52% (CI +1.21..+3.83), 68% hit, +2.08% / +3.52% by half. Longs in that quartile slightly negative in both halves (−0.23% / −0.58%, 72h −0.73%). Reading (hypothesis): last week's sector outperformers mean-revert, so chasing them as longs loses and shorting them works.
- Still same sign: semis longs −0.37%, stock shorts +1.04%, earnings-thesis longs −0.36% (CI −0.76..+0.07). Overall 7d selection over both months +0.03%. Two-sided 50/50 selection +0.52% (CI +0.06..+0.98).
- Caveats: about 50 groups tested. The EW index is a mean, so a few big movers pull it up and hit rates are biased (longs below 50%, shorts above). A median-based index would check that.

## Correction (2026-10-22, median) Crypto pick results depend on the yardstick

- Switched the crypto index to the median coin of the crypto universe (uncommitted). Re-ran `--picks` on 08-22 → 10-22.
- Shorts on the top 7d run-up quartile: +2.52% (EW mean) → **+0.53%** (median, CI −0.66..+1.72, 62% hit, +0.58% / +0.43% by half). The signal mostly came from a few spiking coins pulling the mean up. Not confirmed.
- With the median, crypto longs +2.48% (CI +1.33..+3.51) and crypto shorts −1.42% (CI −2.17..−0.70), same sign both halves. Picked coins beat the median coin on both sides. Hypothesis: the AssetSentiment crypto universe includes many small coins not tradable on the platform that drifted down, so the picks look good as longs and bad as shorts for reasons unrelated to skill.
- Reading (strong evidence): BTC, EW mean and median give different signs for crypto pick quality. No crypto pick signal holds across all three. Results that do not depend on the crypto yardstick and held in both halves: semis longs −0.37% vs semis index, stock shorts +1.04% vs XYZ100.
- Next: a yardstick matched to what was tradable on the platform that day, or a yardstick-free check (long-leg minus short-leg spread inside two-sided baskets, or random picks from the same platform universe as in the 10-21 investigation).

## Update (2026-10-22, platform rank) Picks vs what was tradable on the same platform

- `--picks` now ranks each leg against same-sector assets listed on the same platform at run time (`asset_platforms`, `added_at` ≤ run). 50% = a random tradable pick. No index involved. Caveats: no SYMMIO rows in `asset_platforms`, so Symmio legs get no rank; no delisting dates, so assets inactive now still count; sector from AssetSentiment categories. Also batched the price load (the single query hit the shared DB statement timeout twice). All local, uncommitted.
- 08-22 → 10-22, 7d (HL + Lighter):
  - **Longs beat 54.2% of tradable peers** (CI 51.8..56.8), 54.3% / 54.2% by half. Crypto longs 55.7%. Small, consistent edge.
  - **Shorts 47.4%** (CI 44.7..50.4), 45.2% / 52.0%. No edge, slightly worse than random.
- Same pattern on both sides, mostly both halves (hypothesis, ~50 groups tested):
  - Contrarian picks beat random: longs on 21d laggards Q1 57.5% (57.6% / 57.4%), Q2 56.5%; shorts on last week's top run-ups (7d Q4) 54.4% (52.6% / 58.0%).
  - Chasing picks don't: longs on last week's top run-ups 51.2%; shorts on last week's biggest losers (7d Q1) **43.4%** (41.8% / 46.3%, CI below 50); shorts on 21d laggards ~45%.
  - Thesis: earnings-reason shorts 43.0% (both halves below 50); launch/upgrade longs 58.3%.
- Stage 5 candidates from this: give the model prior 7d/21d return vs platform peers as data; don't short what just fell; buys of last week's top movers need a fresh catalyst; prefer laggards for longs. Validate by replaying logged research inputs before prod.

## Update (2026-10-22, rule backtest) Code rules on last week's peer rank

- `--picks` now also backtests code rules (uncommitted): a basket with any blocked leg is dropped. Rule input: where the asset's prior 7d return ranked among same-platform same-sector listed peers. HL + Lighter only (1243 baskets). 7d, stored weights.
- As generated: mean +0.22%, selection +0.22%, buy rank 54.2%, sell rank 47.4%, halves +1.05% / −0.82%.
- No sell after a 1w drop (bottom 25%): drops 31% of baskets. Sell rank → 49.4% (to random), mean unchanged +0.22%, halves +1.46% / −1.21%. Removes bad sells, no return gain.
- **No buy of a 1w top gainer (top 25%): drops 47% of baskets.** Buy rank → 58.2%, mean +0.39%, 52% positive, halves +1.32% / −0.53% (better than baseline in both). Most promising, but too aggressive at this threshold.
- Both: drops 65%, mean +0.62% but halves +2.55% / −1.17%.
- Every return CI overlaps the baseline. Rank changes are the clearer signal. Dropping baskets ignores what the generator would pick instead.
- Context DB login failed for a while during this session (pooler `server_login_retry`, then SASL on the local URL). Fixed by updating `apps/context/.env`.
- Thresholds for "no buy of a 1w top gainer" (same run): top 25% drops 47% of baskets, mean +0.39%, buy rank 58.2%. **Top 15% drops 34%, mean +0.38%, buy rank 57.3%, halves +1.31% / −0.62%** (both better than baseline). Top 10% drops 25%, mean +0.26%, buy rank 56.3%. Top 15% keeps nearly all of the gain. In production this would be a candidate-pool filter (the asset is not offered as a buy), so the drop rate overstates the disruption. Still not significant on returns.

## Update (2026-10-22, five months) Rule backtest 05-27 → 10-22

- 1h prices start 2026-05-05, so 05-27 is the earliest start with 21d of history. 3354 baskets, 145 run days. Rule table is HL + Lighter (2164 baskets). Before July, `asset_platforms` rows from its first day (a backfill) count as listed for every run. Lighter only from late Aug. Earlier months ran a different pipeline/provider (not tagged).
- Picks overall: buy rank 53.5% (CI 51.5..55.7), sell rank 49.2%. Buy rank by month: Jun 53.6%, **Jul 48.1%**, Aug 54.3%, Sep 55.5%, Oct 53.0%.
- **"No buy of a 1w top-15% gainer" raises buy rank in every month:** Jun 53.6 → 61.1, Jul 48.1 → 54.2, Aug 54.3 → 57.0, Sep 55.5 → 58.7, Oct 53.0 → 56.3. Strong evidence the rule removes worse-than-average buys.
- Returns barely move: all-period 7d −0.07% → +0.09%. By month: Jun −1.32 → −1.40, Jul −1.42 → −1.22, Aug +0.67 → +0.76, Sep +1.12 → +1.15, Oct −1.35 → −0.83. Better in 4 of 5. Basket return is dominated by the market. The rule improves pick quality, not the headline number much.
- "No sell after a 1w drop": no consistent effect over five months (sell rank 49.2 → 50.3, returns worse in Jul/Aug/Oct). Drop it.
- Fixed another stack overflow in the eval (`push(...batch)` on the long window). Uncommitted.

## Correction (2026-10-23) Peer groups polluted by uncategorised listings

- Found: ~44% of active HL + Lighter `asset_platforms` rows (236 of 537) have no `AssetSentiment` category, because the sentiment universe is the hard-coded `TOKEN_CATEGORIES` list in pear-stat-arbs (unchanged since 2026-01-05). The eval's `sectorFromCategory` sent them all to "crypto", so oil, FX, stocks and ETFs sat in the crypto peer group. Every platform-rank number above used those polluted peers.
- Re-ran the five-month window (05-27 → 10-22, asof 10-23 08:00Z) with peers restricted to categorised assets (301 of 537 rows kept). Same baskets, same returns, only the ranks change.
  - Buys beat **52.0%** of peers (CI 50.1..53.8), was 53.6%. Sells 50.7%, was 49.1%. The small buy edge mostly came from the polluted crypto group (long crypto 54.8% → 51.9%).
  - Chasing shape survives: longs on last week's top quartile 48.1% (worst long group), shorts on last week's bottom quartile 48.8% (worst short group), shorts on last week's top quartile 53.0%.
  - **"No buy of a 1w top-15% gainer" still lifts buy rank: 51.96% → 55.75% overall** (was 53.57% → 57.72%). By month: Jun 53.9 → 61.4, Jul 50.2 → 55.9, Aug **50.6 → 50.9 (flat)**, Sep 51.7 → 55.4, Oct 53.8 → 57.3. "Every month" no longer holds, 4 of 5.
  - Returns: base −0.07% vs rule −0.01% at 7d (was +0.08%). No return gain.
  - "No sell after drop": still nothing.
- The live rule on `feat/basket-no-top-gainer-buy` already ranks categorised assets only, so it matches the clean run. Semis (4 categorised) and commodities (2) never reach the 10-peer floor, so the rule can't touch them.

## Update (2026-10-23) Where each stage stands

- Stage 0: eval done (`feat/basket-eval-script`, draft #1166), provider date map done, clean-peer fix made 10-23 (uncommitted at time of writing). 0.3 web-app weights still unknown. Baseline freeze ~10-29 (72h) / ~11-02 (7d).
- Stage 1: on hold (50/50 vs stored weights flips by month).
- Stage 2: `1947c597` local on `feat/basket-no-forced-long-only`, on hold, needs product call.
- Stage 5: became the code rule `398517c5` on `feat/basket-no-top-gainer-buy` (not pushed, off by default). Weekly scheduled check from 10-23, verdict ~11-13. Then Lighter-only switch-on vs HL.
- Stages 3, 4, 6, 7: not started. Stage 3 may partly come from fixing [#1167](https://github.com/pear-protocol/agent-pear-v2/issues/1167) (chip menu is 4 names).
- #1162 story body rewritten to this state (old body in GitHub edit history).
