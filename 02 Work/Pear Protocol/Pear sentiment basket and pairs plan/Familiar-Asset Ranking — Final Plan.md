# Familiar-Asset Ranking — Final Plan

Status: **decided, not implemented.** Finalised 2026-09-30 after a grilling session over the Notion status page.
Evidence: [[Pear Trader Asset Familiarity — Data Findings]] (§11 decision, §12 validation). Earlier repo analysis: [[Better Trading]].
Notion (Jeyd/Huf): [Familiar-Asset Pair/Basket Ideas](https://app.notion.com/p/3ea4641ea90c81af8bebe1578a9c28e0)

This note lists only the decisions and what to build. The numbers behind each decision are in the Data Findings note.

---

## 0. Where this started, and how it changed

| Stage | What we believed | What changed it |
|---|---|---|
| Vision | Generate pair/basket ideas around assets Pear users already trade, to raise volume. | — |
| First design ([[Better Trading]]) | 30-day distinct traders per asset from BE fills, then into `pickDailyAssets` and `generateBasket`. | Data Findings §2: about 75% of fills are imported off-Pear activity, and about 70% of Pear buy fills are scheduled ADJUST. The counting unit changed to **completed OPEN executions** and **Pear-engine positions**. |
| Composition archetypes (§9–§10) | Top compositions are mostly influencer or copy-trade bursts; only BTC/ETH is durable. | Jeyd (§11): all Pear activity is valid signal. Archetypes stay as diagnostics only, with no trending layer and no classifier. |
| 65/35 decision (§11) | One asset score = 0.65 current + 0.35 historical, top ~20, feeding the existing narrative + quant pipeline. | Validation (§12): there are no data blockers. But the 20-per-category cutoff drops some ranked assets, the generator makes 0.4% 1x1 pairs, and generated ideas are opened ≈2% of the time. |
| Plan as of 2026-09-30 AM | The decisions below; V1 improves baskets, a pair pass later (§6). | Jeyd (2026-09-30): don't pre-decide pair vs basket — explore both and pick whichever has the stronger evidence-supported opportunity. Fixing the 0.4% pair bias is part of this project. See §11. |
| **This plan** | §1–§5, §7–§9 unchanged; §6 and §10 superseded by §11. | — |

---

## 1. Ranking signal (settled)

- **Event sources (Pear BE, read-only via `BE_READONLY_DATABASE_URL`)**
  - **Current:** distinct `user_id` holding a position where:
    - `positions.status IN ('OPEN','OPENING')`, `creation_source='ENGINE'`, `origin<>'EXTERNAL'`;
    - the asset is a leg in `metadata.initialExposureUSD`.
  - **Historical:** distinct `user_id` with an `executions.intent='OPEN' AND status='COMPLETED'` in the last **90 days**, touching the asset through Pear-placed fills (`fills.execution_id` set, `is_external=false`).
- **Normalization:** `current_pct = holders(asset) / all current holders`; `historical_pct = openers(asset) / all 90d openers`.
- **Score:** `final = 0.65 × current_pct + 0.35 × historical_pct`.
- **Tickers:** strip the dex prefix (`xyz:`, `hyna:`, `km:`, `flx:`, `mkts:`, `io:`, `para:`) and the scale prefix (`k`, `1000`), and keep a HIP-3 flag. Watch for collisions (e.g. `GAS` vs `flx:GAS`).
- **Exclusions:**
  - vault users (`users.role='vault'`);
  - bot/test users by the best-trade rule over the last 30 days of closed Pear positions: median hold < 5 min **and** median notional < $50 (`testAccounts` in `apps/agent/src/best-trade/best-trade.ts`; export it rather than copy it).
- **Not excluded:** agent-originated (`AGENTPEAR`) opens. This follows Jeyd's "Pear activity is valid whatever caused it". The agent share per asset is logged as a diagnostic.
- **No rank smoothing** (hysteresis) in V1. Churn is about 1.9 top-20 swaps a day, mostly at ranks 15–25.
- **To confirm with Jeyd (build in parallel):** *(confirmed 2026-09-30, see §11.6)*
  > "For implementation I'm taking current = distinct Pear users with OPEN/OPENING positions, historical = distinct users with completed OPENs over 90d, both normalized before the 65/35 merge. I'll proceed with that unless you want a different window/definition."

## 2. Usable vs diagnostic assets (settled)

An asset is **usable** only if all of these hold:
1. It resolves to a quant ID via `AssetIndexReader.resolveQuantAssetId` (Redis `asset_index:full`).
2. It has an `AssetSentiment` row, which gives its category. apps/agent already has `ContextPrismaService`.
3. It is listed on at least one of HL / Lighter.

The job walks the ranked list and emits **exactly 20 usable** assets. Ranked assets that fail a check go into `diagnostics` with a reason:
`missing_quant_id` | `missing_sentiment_category` | `not_listed`.
Example: PONS is `missing_sentiment_category`. So "top 20" always means 20 usable assets.

## 3. Transport and schedule (settled)

- **New one-shot in `apps/agent`**, following the best-trade / user-value-score pattern:
  - `src/pear-asset-ranking/` (reader SQL + compute + runner module)
  - `src/main-pear-asset-ranking.ts`
  - `deploy/hetzner/agent-pear-asset-ranking.{service,timer,env.example}`
- **Timer:** daily **10:00 UTC** (`OnCalendar=*-*-* 10:00:00 UTC`, `Persistent=true`). That is 2h before the 12:00 sentiment run and leaves room to rerun by hand. Hetzner only; no Railway/BullMQ cron.
- **Output:** agent Redis key `pear-asset-ranking:v1`, JSON, **TTL 48h**:

```json
{
  "version": 1,
  "computedAt": "2026-10-01T10:00:12Z",
  "base": { "currentHolders": 153, "historicalOpeners": 480, "windowDays": 90, "excludedUsers": 8 },
  "usable": [
    { "rank": 1, "asset": "BTC", "quantId": "BTCUSDT", "category": "Layer1",
      "currentHolders": 49, "currentPct": 32.5, "openers90d": 253, "historicalPct": 53.6,
      "score": 39.9, "longUsers": 104, "shortUsers": 81, "agentSharePct": 37 }
  ],
  "diagnostics": [
    { "rank": 21, "asset": "PONS", "score": 8.9, "reason": "missing_sentiment_category" }
  ]
}
```

- **Shared reader** in `packages/shared`: returns `usable[]`, or `null` when the key is missing, unparsable, or `computedAt` is **older than 36h**. The caller logs a warning when it gets `null`.
- **Credential stays in apps/agent only**. apps/context does not get `BE_READONLY_DATABASE_URL`.

## 4. PR 1: ranking job + `pickDailyAssets` union

- The ranking job, Redis key and Hetzner unit from §3.
- `apps/context/src/sentiment/services/sentiment-cron.service.ts` `pickDailyAssets`:
  - returns **OI top 10 ∪ 7 hard-coded HIP-3 ∪ ranking `usable` (20)**, deduped by `assetId`, about 27 assets a day;
  - categories come from the existing sources (OI row / asset universe) and fall back to the ranking's `category`.
- **Reader `null` → today's behaviour** (OI + HIP-3 only) plus a `warn`.
- **Do not touch the shared `take: 20`** in `SentimentService.getAssetRowsByCategory`. The union makes ranked assets the newest `runAt` each day, so they survive the cut deterministically (≤10 per category; Layer1 holds 10).
- `apps/context/src/sentiment/sentiment-runner.module.ts`: bind the agent-Redis client. Copy the `ASSET_INDEX_REDIS` provider from `basket-runner.module.ts:51`, which reads `AGENT_REDIS_URL`. Checked 2026-09-30: the sentiment runner currently imports only `RedisModule`, while the basket runner already binds agent Redis.
- **Nightly metric logging** (in the ranking job): *current* performance only, not a rolling baseline:
  - generated-idea open rate for yesterday's and last 7 days' `made_baskets` (exact composition, 7-day window);
  - distinct openers of any generated idea;
  - HL + Lighter only.
- Expected cost: sentiment calls go from about 17 to about 27 assets a day.

## 5. PR 2: generator context (after PR 1 has run a few days)

- **Pre-check:** read the live Hetzner env for `BASKET_LLM_PROVIDER` and `CLAUDE_BASKET_RESEARCH_STAGE`. `context.env.example` says `claude` / `true`, which shows intent but doesn't prove what production runs. If research is on, it shortlists 4 per category and 12 assets in total.
- `MadeBasketsService.generateBasket` (`apps/context/src/sentiment/services/made-baskets.service.ts`) gets one appended block, per platform, intersected with that platform's listed universe:
  - Pear rank, current holders, 90d openers, long/short user skew.
  - **Informational only**: no forced inclusion and no shortlist quota ("boost, don't constrain").
  - Must not contain category words ("AI", "Meme", …): the Claude path regex-matches category names in `input`.
- The basket runner already has agent Redis, so no module change is needed there.
- Kept separate from PR 1 so their effects can be told apart: PR 1 changes which assets have fresh sentiment; PR 2 changes what the LLM is told.

## 6. Pairs (deferred, flagged to Jeyd) — (superseded 2026-09-30, see §11)

- The generator makes **0.4% 1x1 pairs** (3 of 797). V1 improves baskets only.
- Message to Jeyd:
  > "One caveat: current generator outputs almost no 1x1 pairs (~0.4%). I'm planning the ranking/basket integration first and a dedicated pair pass separately unless you want pairs included in V1."
- A pair pass would be a separate PR touching `made-baskets-cron.service.ts` and the basket prompts. The quant gate for pairs already exists (`getPairStats` + `hasHedgedDirection`).

## 7. Equities, SYMM (settled for V1)

- **HIP-3:** keep the forced equity pass and the 7 hard-coded tickers unchanged. Follow-up option: rank HIP-3 assets by 65/35 separately to replace the hard-coded 7 (27% of traders touched HIP-3 in 30 days).
- **SYMM:** keep generating for it. The success metric covers **HL + Lighter only**, because the BE shows no Pear-engine OPENs on SYMM. Ask Jeyd where SYMM trades are recorded.

## 8. Success metric (settled)

- **Primary:** distinct traders who open any generated idea (exact long/short composition after normalization) within 7 days of generation.
- **Secondary:** exact-composition open rate (share of generated compositions opened within 7 days).
- **Frozen baseline:** measured once before PR 1 ships over the previous 21 days, and recorded in this note and on Notion. Last pre-plan read (§12.5): 778 compositions → 17 opened by 10 users ≈ 2%. Retake it just before shipping; never recompute it later.
- **Target:** set after 2 weeks of PR 1 data.

### 8.1 Frozen baseline (measured 2026-09-30; do not recompute)

**Method** (read-only: BE role is read-only; quant DB queried inside `BEGIN TRANSACTION READ ONLY … ROLLBACK`):
- **Generated set:** `made_baskets` rows, `platform IN ('HYPERLIQUID','LIGHTER')`, `run_at ∈ [2026-09-02, 2026-09-23)` UTC. That is the 21 days ending 7 days before measurement, so every idea has a complete 7-day follow-up.
- **Composition key:** `(platform, sorted long set, sorted short set)`. Legs are normalized: strip dex prefix (`xyz:` …), `USDT` suffix, `1000` / lowercase-`k` prefix. The same composition generated on several days counts once.
- **Opens:** BE `executions` with `intent='OPEN'`, `status='COMPLETED'`, created in `[2026-09-02, 2026-09-30)`. Legs come from Pear-placed fills (`is_external=false`, `synthetic=false`, `status<>'RECONCILED'`), BUY = long / SELL = short, with the base from `instruments`. Connector `hyperliquid` → HL; `lighter`/`lighter_rhc` → Lighter.
- **Excluded:** vault users. Bots: best-trade rule (median hold < 5 min AND median notional < $50) over Pear-engine positions closed in `[09-02, 09-30)`, which excluded 8 users.
- **Match:** exact key equality, with the open's first fill in `(run_at, run_at + 7d]` of any generation of that composition.

**Result**

| | Rows | Unique compositions | Compositions opened | Matched opens | Distinct users |
|---|---|---|---|---|---|
| **Overall (HL + Lighter)** | 557 | 551 | **16 (2.9%)** | 18 | **8** |
| Hyperliquid | 304 | 302 | 15 (5.0%) | 16 | 8 |
| Lighter | 253 | 249 | 1 (0.4%) | 2 | 1 |

By generation week (a composition that spans weeks is counted in each week; an open is credited to the latest generation it followed):

| Week | Unique compositions | Opened | Distinct users |
|---|---|---|---|
| W1 (09-02 → 09-09) | 134 | 5 | 3 |
| W2 (09-09 → 09-16) | 187 | 7 | 5 |
| W3 (09-16 → 09-23) | 234 | 4 | 3 |

Context:
- **Primary: 8 distinct users out of 278 non-bot HL/Lighter Pear openers in the period (≈2.9%).**
- Matched opens by client: PEARUI 17, AGENTPEAR 1.
- Generated shapes: 126 directional (one side empty), **0 1x1 pairs**.
- Every opened composition was opened by a single user, except `L GRASS/RENDER/TAO · S OP/POL/STRK` (2 users).
- Differs from the §12.5 pre-plan reading (17 opens / 10 users): that one used an incomplete follow-up window and no bot exclusion.
- Week-to-week noise is ±2 users on a base of 3–5. A post-ship lift is only meaningful if it clearly exceeds that.

Script: session scratchpad `baseline.cjs` (not committed). The definitions above are the spec for re-measuring post-ship with identical logic.

- **Attribution is exposure-based** (causal attribution doesn't exist). Later: an optional `madeBasketId` on agent tickets, following the `signalNotifId` pattern (`trade-ticket.types.ts`, `execute-ticket.usecase.ts`). Web-UI attribution needs frontend work outside this repo.

## 9. Not in scope

- trending product / discovery surface;
- social or campaign attribution;
- behavioural classifier;
- scoring engine beyond 65/35;
- cohort or per-user generation (`made_baskets` is one global set);
- composition-level recommender;
- agent chat "assets Pear traders are active in" context line (not in the Notion plan; revisit only if Jeyd asks).

## 10. Order of work — (superseded 2026-09-30, see §11.4)

1. Send Jeyd the two messages (§1 definitions, §6 pairs).
2. ~~Freeze the baseline (read-only queries). Record it here and on Notion.~~ Done 2026-09-30, §8.1.
3. PR 1: ranking job + Hetzner unit + shared reader + `pickDailyAssets` union + sentiment-runner Redis binding + nightly metric logs. Verify: one manual run writes the key; the 12:00 sentiment log shows ~27 picks; a missing key falls back with a warning.
4. Watch PR 1 for a few days: check the metric logs and that ranked assets survive the category cutoff at the 14:00 run.
5. Check the live Hetzner basket env, then PR 2 (generator context block).
6. After Jeyd replies: pair pass; optional HIP-3 65/35 ranking.

## Risks

- **Top-10 overlap:** ranks 1–10 are 9/10 the same as the OI picks. Any lift has to come from the current-signal reordering and ranks 11–20. If the metric doesn't move after PR 1 + PR 2, the premise is wrong for baskets; don't tune weights to compensate.
- **Research cap:** if the Claude research stage is on, ranked assets still compete for 12 research slots. PR 2's context block is the only lever, and it is informational only by decision.
- **XMR** is categorised `HIP_3_crypto`, so it only reaches baskets when a HIP-3 category is chosen. This is a known quirk and not a blocker.
- **Job failure** means silent fallback to today's behaviour. Hetzner one-shots have no retries, so watch the `warn` line.

---

## 11. Update 2026-09-30 — structure-neutral generation (Jeyd's direction)

Supersedes §6 (pairs deferred) and §10 (order of work). §1–§5, §7–§9 and every settled decision still hold.

### 11.1 The decision

After the 65/35 ranking and narrative analysis, **don't decide ahead of time whether the output is a pair or a basket.** For the selected universe, explore plausible pairs *and* baskets and keep whichever shows the stronger opportunity.

How "stronger return opportunity" is read:
- **No invented returns.** The LLM never states or ranks by projected ROI or future %.
- "Opportunity" = narrative conviction from the research stage **plus** the historical/statistical evidence we already have: correlation, cointegration, roll z-score, hedge ratio / β validity, and spread volatility (`pair_stats_*` for 1x1, `/stats/basket` for multi-leg).
- Prefer the risk-adjusted, better-evidenced structure over a bigger but unsupported story.
- Pair and basket are equally valid. Correcting the basket bias (0.4% pairs; 0 in the baseline window) is **in scope**.
- The existing quant gate (`hasHedgedDirection`, `/stats/basket` negative-hedge rejection) stays the final filter.
- Not a scoring engine: the comparison is a written rubric the LLM applies and cites. No new composite score.

### 11.2 Updated flow

```
clean Pear activity (vault/bot/test removed, tickers normalized)
→ current    = distinct traders holding the asset now (Pear-engine OPEN/OPENING)
→ historical = distinct traders with completed OPENs, 90d
→ normalize each → score = 0.65 × current + 0.35 × historical
→ top 20 USABLE assets → agent Redis (10:00 UTC, TTL 48h, ignored if >36h)
→ pickDailyAssets = OI top 10 ∪ 7 HIP-3 ∪ ranked 20  → fresh narrative (12:00)
→ research the selected assets (survivors)
→ build ≥1 candidate pair AND ≥1 candidate basket from survivors
→ check each: pair_stats (1x1) / quant_basket_stats (multi-leg)
→ pick the structure with the stronger narrative + stats evidence (no pair/basket preference)
→ existing quant gate → made_baskets (14:00)
→ measure: distinct traders opening generated ideas within 7d, split by shape
```

### 11.3 Where the basket bias comes from (verified in code 2026-09-30)

- **Prompt defaults:** `agents/basket/prompt.ts` "Collect 2-3 assets from each category"; `research/selection-prompt.ts` "Pick 2-5 assets per side"; all name/leg examples are 3-asset. Research `prompt.ts` calls "a lone pair" a *thin* outcome to avoid.
- **Evidence asymmetry:** the LLM's only in-loop quant tool is `quant_basket_stats`, which rejects 1x1. A pair "earns its place on conviction alone", so a multi-leg basket is always the better-evidenced choice.
- **Pair stats are HL-keyed:** `getPairStats` uses `resolveHyperliquidAssetId`, so Lighter/SYMM pairs get null stats. Pair evidence is only authoritative on HL.
- There's already a `structure` hint (`long_only` / `short_only`) and a guaranteed directional pass in `made-baskets-cron.service.ts`. Leave it unchanged.

### 11.4 Implementation phases

Each phase is one reviewable PR (Phase 0 is not a PR) with its own check, and ships only after the previous one has had time to show up in the metric.

| Phase | What | Verify / measure | Depends on |
|---|---|---|---|
| **0. Prep** | Send Jeyd §1 definitions confirmation (drop the §6 pairs question; it's answered). ~~Freeze baseline~~ done (§8.1). Add **shape mix** to the baseline record: 126 directional, 0 pairs, rest multi-leg. Read live Hetzner `BASKET_LLM_PROVIDER` / `CLAUDE_BASKET_RESEARCH_STAGE`. | Baseline + shape mix recorded here and on Notion. Live env known. | — |
| **1. Ranking + universe** (PR 1, §4 unchanged) | Ranking job in apps/agent + Hetzner 10:00 timer + Redis key + shared reader + `pickDailyAssets` union + sentiment-runner Redis binding + nightly metric log. **Extend the log with the shape split** (pair / multi-leg / directional) of generated and opened ideas. | One manual run writes the key; 12:00 log shows ~27 picks; missing key → fallback + warn; ranked assets survive the 14:00 cut. | 0 |
| **2. Generator context** (PR 2, §5 unchanged) | Informational Pear-activity block per platform in `generateBasket`. No forced inclusion, no category words. | Block appears in prompts; no change to gate rejections; metric trend. | 1 running a few days |
| **3. Structure-neutral generation** (PR 3, new) | (a) Give the selection/agent LLM a **`quant_pair_stats` tool** that wraps the existing `getPairStats` + `hasHedgedDirection` (reuse, don't copy), so pairs get evidence like baskets. (b) Rewrite the structure rules in `prompt.ts` / `selection-prompt.ts`: 1–5 legs per side, 1x1 is first-class; build ≥1 pair and ≥1 basket candidate, check each with its stats tool, pick on the evidence rubric in §11.1; never state projected returns. Adjust the "lone pair = thin" wording in research `prompt.ts` (research survivor target unchanged). (c) Log the rejected alternative and the reason in `eval-log.ts`. Tool budget: pair + basket checks ≤ 3 in total. | Playground/eval run on a fixed day's universe: pair share of outputs rises from ~0; quant-gate pass rate for pairs ≥ baskets; no projected-return text in theses. In prod: shape mix in the nightly log. | 2 (so its effect can be told apart) |
| **4. Measure + decide** | 2 weeks after Phase 3. Compare against the frozen baseline: primary (distinct openers), secondary (open rate), **per shape**. Set the target (§8). | Lift only counts if it clearly exceeds ±2 users/week noise. | 3 |
| **5. Follow-ups (only if data asks)** | If the pair share is still ≈0 after Phase 3: add a guaranteed pair candidate pass, using the existing `structure` hint the same way as the directional pass. Optional HIP-3 65/35 ranking in place of the hard-coded 7. Optional `madeBasketId` on agent tickets. SYMM answer from Jeyd. | Each is its own PR with the same metric. | 4 |

Why Phase 3 is after Phase 2 and not bundled: PR 1 changes *which* assets have fresh narrative, PR 2 changes *what the LLM is told*, PR 3 changes *what shapes it may produce*. Shipping them together makes any lift unattributable.

### 11.5 Risks added by this change

- **Lighter pairs have no pair stats** (HL-keyed). On Lighter a pair can't be compared on evidence, so it loses to a basket by construction. Accept for V1 (baseline: Lighter = 1 of 16 opened), or map pair stats per platform later.
- **Selection budget:** checking both a pair and a basket costs more quant calls per generation. Keep the total cap small (≤3) so the turn limit and cost stay flat.
- **Over-correction:** a prompt that says "consider pairs" can swing to all-pairs. Phase 3's check is the shape mix, not just "pairs > 0".
- **No projected ROI:** the evidence rubric must stop the LLM from writing "+X% expected". Add a validation or eval check on thesis text.

### 11.6 Progress (2026-09-30, late)

- **Signal definitions locked for V1 by Jeyd:** OPEN/OPENING Pear-engine holders; completed OPENs over 90d; 65/35; vault/bot/test excluded; agent-placed trades counted. §1 needs no change.
- **Issue tree:** epic [#1064](https://github.com/pear-protocol/agent-pear-v2/issues/1064); stories #1065 ranking job, #1066 union, #1067 structure-neutral, #1068 generator context, #1069 measure + decide.
- **Releases** (replaces the order in §11.4): Release 1 = #1065 + #1066 stacked on `feat/familiar-asset-pair-basket`; Release 2 = #1067; Release 3 = #1068. Structure-neutral goes before generator context.
- **Prod path (verified):** Hetzner is the only prod runner and deploys only from `main` (`deploy-hetzner.yml`, reviewer-gated). A release reaches traders on `development → main`. Merging to `development` = dev env only.
- **Correction to §5 / §11 assumptions:** prod research cap is `CLAUDE_BASKET_MAX_RESEARCH=18`, not 12. The research stage is on, so #1067's main edits go in the research + selection prompts.
- **Correction to §3:** the Hetzner units belong in `apps/agent/deploy/hetzner/`, not repo-root `deploy/hetzner/`.
- Details: [[30th Sep 2026#Familiar-asset pair/basket: structure-neutral plan, issue tree, Phase 0 (late)]].

### 11.7 Update 2026-10-01 — #1065/#1066 built, gate correction, replay findings, #1067 plan

- **Built:** #1065 → PR [#1070](https://github.com/pear-protocol/agent-pear-v2/pull/1070) (ranking job, shared reader, nightly metric, `--metric`/`--backtest`, Hetzner units). #1066 → PR [#1071](https://github.com/pear-protocol/agent-pear-v2/pull/1071) (union in `pickDailyAssets`, fallback + warn), stacked. Neither merged.
- **Metric verified:** `--metric` over 09-02→09-23 reproduces §8.1 exactly (8 users, 16/551, 278 openers, 126 directional / 0 pair).
- **Correction to the Release 1 gate:** the coverage backtest (44% → 75% of next-week openers covered) is near-circular and skips the input→output chain, so it is not a gate ([#1069 correction](https://github.com/pear-protocol/agent-pear-v2/issues/1069#issuecomment-5925215680)). Gate = offline generator replay A vs B; live metric judges adoption.
- **Replay findings (Hetzner, read-only):** 16/20 ranked assets are already in prod's candidate lists; #1066 adds only ADA, BNB, SUI, ETHFI (Layer1/Defi). The generator fixes one category set per run; Layer1/Defi appear in ~31% of attempts. Per attempt: ~25 candidates → ~6 survivors. So #1066's effect is small by construction; the paid #1066 replay was skipped by decision.
- **Order change (pending Jeyd):** hold Release 1 until #1067's replay shows a positive result, then release together. Supersedes "one release at a time" in §11.4/§11.6 if Jeyd agrees.
- **#1067 plan:** see [[1st Oct 2026#1067 plan (not coded) and why it's needed]]. Matches §11.4 Phase 3, with three refinements: reuse the agent's `quant_pair_stats` by moving it to shared; ≤3 checks and selection turns 10 → ~12; mechanical projected-return check in `finalizeBasket`.

### 11.8 Update 2026-10-01 — #1067 pre-check (pair evidence exists)

- Prod's last-week survivors, read-only: on HL, 23% of attempts already contain a strong pair (hedge-valid, stats direction agrees with the research lean, |rollZ| ≥ 1), but only 4% produced a pair. Lighter, using HL stats as a proxy: 49% vs 0%.
- Cointegration is rare (1.7% of HL candidate pairs), so the §11.1 evidence rubric should treat it as a bonus, not a requirement.
- §11.5's Lighter risk ("a pair loses to a basket by construction") is fixable: HL stats as a labelled proxy cover every Lighter survivor pair checked (100% have a row). Proposed for V1, pending the user.
- §11.4 Phase 3's check bar is now concrete: expect ~10–25% pairs on HL, not all-pairs.
- Details: [[1st Oct 2026#1067 pre-check: pair evidence in prod's survivors (read-only)]]; [#1067 comment](https://github.com/pear-protocol/agent-pear-v2/issues/1067#issuecomment-5925739698).

### 11.9 Update 2026-10-01 — end goal is familiar assets in ideas; #1068 re-scoped

- **End goal (user, 2026-10-01):** more familiar (ranked top-20) assets in generated basket/pair suggestions. Research/category prompts may be loosened for familiar assets to get there.
- **Prod funnel** (220 HL+Lighter runs since 09-24, `basket-eval.jsonl`): pool 17% familiar (53% of runs have any) → survivors 14% (35%) → ideas 14% (33%). Lost at the **category pick** (47% of runs have no familiar asset in the pool) and in **research** (BTC 17/82, ETH 14/82, SOL 10/82 survive when in the pool; familiar survive 16% vs 20% overall). Selection is neutral; #1067 leaves familiarity unchanged (14% → 14% on the same 40 runs).
- **Supersedes §11.4/§11.6's view of Phase 4 (generator context) as the weakest lever.** #1068 now: research diagnostic (`considered[]` → eval log) → research prioritises ranked assets within its cap and accepts trend/flow cases for them (loosen the bar if the diagnostic shows priced_in/weak drops) → category step prefers categories holding ranked assets. Gate = the funnel, tracked per release in #1069. Forced "Pear favourites" pass = epic follow-up only if this doesn't move it.
- #1066 gains a check: SUI and LIT never reached the pool in 220 runs.
- Issues updated (bodies, dated): [#1064](https://github.com/pear-protocol/agent-pear-v2/issues/1064), [#1066](https://github.com/pear-protocol/agent-pear-v2/issues/1066), [#1068](https://github.com/pear-protocol/agent-pear-v2/issues/1068) (title + body, old scope kept collapsed), [#1069](https://github.com/pear-protocol/agent-pear-v2/issues/1069). Notion §6.4/§9 not yet updated.

### 11.10 Decision 2026-10-01 — replace the HL/Lighter candidate pool with the ranked assets

- Re-read Jeyd's original brief: rank by 65/35, then "generate baskets for those most traded assets … not from random assets which they never traded. simple", pair or basket equally. The "union, not replace" (§4) and "informational only, boost don't constrain" (§5) rules were **our** design choices — no record of Jeyd approving them — and they cap familiarity at ~14% of legs.
- **Decision (user):** replace. On HL and Lighter the generator's candidate pool = ranked `usable` assets listed on the venue; research narrates them (bar loosened for trend/flow on majors), selection picks pair vs basket (#1067). Ranking null → today's pool + warn. SYMM and the forced HIP-3 pass unchanged (ranked HIP-3 stays an epic follow-up). Supersedes §11.9's "research favours ranked assets" and §5; the "Pear favourites pass" follow-up is obsolete.
- Risks recorded on #1068: repetition (same ~20 assets daily vs distinctness / per-asset caps), quiet majors failing the catalyst bar, Lighter listing gaps. Gate: replay → ≈100% familiar legs on HL/Lighter main passes, stored ideas/day, gate pass rate, cost.
- Updated: [#1068](https://github.com/pear-protocol/agent-pear-v2/issues/1068) (title + body), [#1064](https://github.com/pear-protocol/agent-pear-v2/issues/1064) (goal, flow, settled decisions, update section, Release 3, follow-up struck). Notion §6.4/§9 still not updated.

### 11.11 Progress 2026-10-01 — #1068 built and replayed

- Release 1 (#1070, #1071) merged into `feat/familiar-asset-pair-basket`. #1068 built per §11.10 on top of it (local commit `69fce053`, not pushed); directional passes also use the ranked pool, ranked HIP-3 assets stay in the main pool.
- Replay gate (prod box, general pass, HL + Lighter, ≤8 attempts/venue): familiar share of stored legs 25% → 100% (HL), 32% → 100% (Lighter); research cost/attempt flat; survivors/attempt up (4→7, 5→8). Stored ideas HL 6 → 5, **Lighter 6 → 3** — the fixed pool repeats SOL/SUI/PUMP and the run rules drop near-copies (inferred; skip reasons not logged). `considered[]` still shows ~40% of researched ranked assets dropped as weak.
- Gate status: familiarity + cost pass; stored-ideas criterion fails on Lighter. Next: exclude assets at the per-run cap from later attempts' pool (the "raise variety, not the pool" risk in §11.10), re-run arm C. Details: [[1st Oct 2026#1068 built + replay gate (ranked assets replace the HL/Lighter pool)]].

### 11.12 Update 2026-10-01 (evening) — #1077, repetition root cause, open discovery question

- #1068 is draft PR #1077 into `feat/familiar-asset-pair-basket`. The Lighter shortfall in §11.11 came from the cron's composition-overlap rule, not the per-asset cap (the earlier "inference" there is corrected by this). Fix `eb6353fd`: unused ranked assets first on each attempt; not re-replayed.
- Stats are a soft veto, not the reason an idea is stored: no cointegration, |roll z| < 1 and some near-outright spreads (k < 0.1) in both arms. A real stats bar would be a separate change (ties to the "return %" sign-off).
- **Open (for Jeyd):** a fully ranked pool removes whole-market discovery on HL/Lighter, and the ranking can reinforce itself. Alternative: ≥1 ranked asset per idea, other legs from the category pool. Supersedes nothing yet; §11.10 stands until decided.

### 11.13 Update 2026-10-02 — #1077 merged into the Release branch

- PR [#1077](https://github.com/pear-protocol/agent-pear-v2/pull/1077) (#1068 + fix `eb6353fd`) merged into `feat/familiar-asset-pair-basket` 2026-10-01 18:04 UTC. Integration branch has #1070, #1071, #1077; no PR to `development` yet. #1076 (#1067) targets `development` directly and is still open, unreviewed.
- Fix still not re-replayed; the Lighter stored-ideas gate (§11.11) is unproven.
- Notion replay page updated with the overlap-rule root cause, the fix, and the merge status.
- **Priority change (user, 2026-10-02):** familiar assets in ideas ship first, pairs after. #1076 closed (branch kept, reopen + rebase later). Release branch → `development` as PR [#1078](https://github.com/pear-protocol/agent-pear-v2/pull/1078) (#1065 + #1066 + #1068; mergeable, no conflicts).

### 11.14 Update 2026-10-05 — Release 1 + #1068 on prod

- **Shipped (GitHub, verified 2026-10-05):** [#1078](https://github.com/pear-protocol/agent-pear-v2/pull/1078) (#1065 + #1066 + #1068 incl. `eb6353fd`) merged into `development` 10:23 UTC; release [#1082](https://github.com/pear-protocol/agent-pear-v2/pull/1082) to `main` merged 11:16 UTC. Stories #1065, #1066, #1068 closed as completed. Open: epic #1064, #1067 (pairs, #1076 closed), #1069 (measure + decide).
- **Not verified:** the Hetzner ranking timer. A look at `~/pear/agent-pear-v2` on the box (checkout at `fb7985e0`, no `dist/main-pear-asset-ranking.js`, no ranking timer in `systemctl list-timers`) shows nothing installed there, but that may not be the deploy dir. If the key is missing, the generator falls back to the category pool with a warn (§4), so check the key before reading any metric as "the ranking is live". Also unverified: whether #1082 shipped the "at least one ranked anchor, other legs from the category pool" design (`rankedAnchors` in `made-baskets.service.ts`) or the §11.10 full-replace pool; the anchor code was uncommitted in the working tree on `feat/fapb-generator-hardening` at the time of writing. This is the §11.12 open question.
- **Research drops, checked on the box (read-only):** the #1068 replay's `considered[]` stores verdicts only (no reason text, no transcripts), so why ZEC / HYPE / NEAR come back `weak` is still unread. Prod history (249 HL + Lighter runs since 09-24, old category pool): survived / in pool: HYPE 23/82, ENA 11/49, BTC 17/82, PUMP 11/82, ZEC 10/82, SOL 10/82, NEAR 4/46, WLD 1/46. So prod research does not drop HYPE or ZEC systematically; the replay's 1/16 and 2/16 look like a property of that setup (one day, 8 attempts, fixed pool of 20). Pool membership is not the same as being researched, so these are lower bounds. `weak` means social-only or unfetched evidence by design (`research/schema.ts:55`).
- **Replay arms A vs C, paper return** (legs from the Notion replay page, HL prices from 2026-10-01 16:00 UTC, about 4 days, long minus short, equal weight, no fees; Lighter priced on HL): A 12 ideas mean -3.9% (5/12 up), C 8 stored ideas +0.4% (5/8 up); BTC +2.3%. Driven by one A idea (-31%). Not evidence for either arm; the replay wrote nothing, so no traders saw these ideas and there is no open-rate comparison.
- **Baseline unchanged:** 16 of 551 compositions opened (2.9%), 8 of 278 non-bot openers (§8.1); the 10-02 ranking run logged 5/209 (2.4%) for 09-18 → 09-25. The 87% multi-leg share (30 days, 298 users) is from the Data Findings note.
- **A/B idea (not decided):** `made_baskets` is one global set and nothing links an open to an idea, so the only workable split is per idea within a day (an `arm` column, exact-composition open rate per arm). At a 2.9% baseline, detecting about 6% needs roughly 700 compositions per arm, which is about two months at about 12 ideas a day. Not started.

### 11.15 Update 2026-10-05 (afternoon) — ranking timer installed on the box

- **Checked on the box (read-only):** the checkout `~/pear/agent-pear-v2` was at `60c88c4f` (#1082 merge) and `apps/agent/dist/main-pear-asset-ranking.js` existed, so the deploy workflow had already built it. The ranking units were not installed. This resolves the "not verified" item in §11.14 (nothing was missing in the checkout; the units and env file were).
- **Env file:** `~/pear/.env.agent-pear-asset-ranking` (mode 600) created by the user from the existing prod env files. An automated attempt to create it was blocked by the permission system (secret-store write), so the user ran the copy. All 8 keys set; the file's `REDIS_URL` has the same hash as `AGENT_REDIS_URL` in `.env.context-production`, so the job writes to the Redis the generator reads.
- **Install:** units copied to `/etc/systemd/system`, `daemon-reload`, one manual `systemctl start` at 14:47 UTC (exit 0, 8 s): 163 current holders, 508 90d openers, 20 usable, no diagnostics; key `pear-asset-ranking:v1` written with TTL about 48 h. Then `systemctl enable --now agent-pear-asset-ranking.timer`; next fire 2026-10-06 10:00 UTC.
- **Adoption metric posted by that run (generated 09-28 → 10-05 window as the job computes it):** overall 7 users, 9/160 compositions opened (5.6%); HL 7/80 (8.8%); Lighter 2/80 (2.5%); 0 pairs. Not comparable to the §8.1 baseline (2.9%): different window and run date.
- **Prod generator config (read from the box):** `BASKET_LLM_PROVIDER=openrouter`. `CLAUDE_BASKET_RESEARCH_STAGE` and `CLAUDE_BASKET_MAX_RESEARCH=18` are set but only apply to the Claude provider, so the research stage and its 12/18 cap are NOT active in prod. This supersedes the §11.6 note that "the research stage is on". The #1068 replay (Claude + research) therefore does not describe prod's path; on OpenRouter the in-process agent gets the ranked pool through its category tool.
- **Deployed generator design:** the built `made-baskets.service.js` has `rankedPool` + `freshFirstPool` (unused ranked assets first, ranking pool for HL and Lighter) and no `rankedAnchors`. So prod runs the §11.10 full-replace pool, not the anchor design from the working tree. Settles the open point in §11.14.
- **Today's 14:00 run** (before the key existed) logged "Pear asset ranking missing or stale: ... use the category pool". A manual rerun of `context-baskets.service` was started at 14:50 UTC and logged `Ranked pool [hyperliquid|lighter]: 20/20 listed`. Result: see the next update.

### 11.16 Update 2026-10-05 (afternoon) — first prod run with the ranked pool

Rerun of `context-baskets.service` at 14:50 UTC (OpenRouter provider, ranking key present, `Ranked pool [hyperliquid|lighter]: 20/20 listed`), compared with the 14:00 UTC run the same day (category pool, ranking key absent). Stored legs read from quant `made_baskets` (read-only), "ranked" = base ticker in the key's 20 usable assets.

| | 14:00 run (category pool) | 14:50 rerun (ranked pool) |
|---|---|---|
| HL general pass | 6 in 6 attempts | 6 in 10 attempts |
| Lighter general pass | 6 in 6 attempts | 6 in 7 attempts |
| HL stored ideas / legs / ranked | 9 / 46 / 3 (7%) | 9 / 31 / 22 (71%) |
| Lighter stored ideas / legs / ranked | 9 / 47 / 4 (9%) | 8 / 30 / 24 (80%) |

- Ideas stored per venue held (general pass hit the 6 cap in both runs); the Lighter drop to 3 seen in the Claude replay did not appear on OpenRouter. More attempts were needed (HL 6 → 10, Lighter 6 → 7), i.e. more near-copy skips.
- Ranked share is 71–80%, not 100%, because the equity (HIP-3) and directional passes are outside the ranked pool by design; the general pass alone was not split out.
- The rerun's rows are the newest `run_at`; the 14:00 set stays in history. One sample, one day. Open-rate effect not measurable yet.

### 11.17 Update 2026-10-07 — first scheduled runs with the ranking live (read-only check on the box over SSH)

- **Ranking timer:** `agent-pear-asset-ranking` fired on schedule 10-06 10:00 UTC, exit 0, 11 s, 20 usable (161 holders, 512 openers at 90d, 7 bots excluded). Next fire 10-07 10:00 UTC. Order today: BTC, ETH, ZEC, HYPE, SOL, NEAR, LIT, UNI, XRP, PUMP, SUI, ENA, LINK, AERO, ADA, XMR, WLD, AAVE, ONDO, DOGE.
- **First scheduled basket run with the key present:** `context-baskets` 10-06 14:00 UTC, exit 0, 12 min 35 s, logged `Ranked pool [hyperliquid|lighter]: 20/20 listed`. General pass hit the 6 cap on HL, SYMM and Lighter; equity passes ended on "diversity exhausted" (HL 3, Lighter 2, SYMM 2); one short-only each on SYMM and Lighter. Provider in the log is OpenAI (the OpenRouter path), as §11.15 said.
- **Ranked share of stored legs** (my own count from quant `made_baskets`, read-only, base ticker in the 20; includes equity/directional passes, so it is lower than the general pass alone): 10-06 14:00 HL 15/27 (56%), Lighter 26/34 (76%); SYMM 9/46 (20%, not part of the change). 10-05 14:00 (key absent): HL 9%, Lighter 13%, SYMM 11%. 10-05 14:50 rerun: HL 68%, Lighter 77%. 10-04: HL 11%, Lighter 21%, SYMM 15%. Ideas with at least one ranked leg on 10-06: HL 6/9, Lighter 7/9.
- **Still not measured:** whether traders open more of them. The job's own metric (generated 09-22 → 09-29, 7d follow-up): 7/137 opened (5.1%), HL 5/70, Lighter 2/67, 0 pairs. That window predates the change, so it is not a post-release read. #1069 (measure + decide) is still open and not started; the first post-release window needs about 7 days of generation + 7 days of follow-up from 10-05, so a first read is not due before about 10-19.
- **Box state at the same check:** 31 timers loaded and enabled, every service's last result `success`, `systemctl --failed` empty, disk 51% (19 of 38 GB), RAM 2.8 GB available, load ~0. No stored basket rows appear at 03:00 UTC (only 14:00 runs in the last 3 days), consistent with the Railway cron being off.

### 11.18 Update 2026-10-09 — are traders opening our suggestions, or just the common assets? (read-only, first partial read)

Read-only: BE role plus quant `made_baskets` inside `READ ONLY` transactions; ranking key read from the box's agent Redis over SSH. The 2026-10-08 ranking is applied to both periods (ranking churn is about 1.9 swaps a day, §12.4). Non-bot, non-vault HL + Lighter Pear OPENs only. Generated 09-28 → 10-05 14:30 UTC = category pool (PRE); from the 10-05 14:50 rerun = ranked pool (POST).

- **Generation did what it was meant to.** Familiar (top-20) share of generated legs: PRE 14.9%, POST 66.5–72%. By day: 09-28 → 10-04 between 9% and 20%; 10-06 59%, 10-07 68%, 10-08 68%. Ideas with at least one familiar leg: 43% → 74–82%.
- **Traders open common assets, not our exact ideas.** Of 745–900 opens per period, exact suggested compositions were 0.3% (3 of 900) PRE and 0.3–1.7% POST (1 of 358 at a 3-day follow-up, 6 of 358 at 1 day). All-legs-familiar opens were 51–55% in both periods, at least one familiar leg 78–80%.
- **Exact-composition adoption** (opened / unique compositions): PRE 3/159 (1.9%, 3-day) or 2/159 (1.3%, 1-day); POST 1/17 (3-day) or 4/54 (7.4%, 1-day, 4 users). POST is 17–54 compositions and 1–4 users, so it is not a lift. Compare against the ±2-user weekly noise in §8.1.
- **Familiar share of what traders open did not move:** 59% → 61% of open legs. Our ideas moved toward what traders already trade; what traders trade did not move toward our ideas.
- **Caveats.** The POST follow-up is 1–4 days, not 7. The suggested-asset universe overlap (60% / 53–59% of opens fully inside it) is mostly the same BTC/ETH/SOL/ZEC set, so it says little by itself. Nothing links an open to an idea (§8, exposure-based only). Script: session scratchpad `familiar-check.cjs` (not committed). The full 7-day read for 10-05 → 10-08 generation is due about 10-15 → 10-16; the post-release window from #1069 is still about 10-19.
