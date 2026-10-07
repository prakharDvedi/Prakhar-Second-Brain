# Familiar-asset basket/pair ideas: repo analysis and plan

> Data follow-up (2026-09-29): [[Pear Trader Asset Familiarity — Data Findings]]. It answers §6.10: the OI picks overlap 9/10 with Pear's top-10, and the counting unit should be OPEN executions, not fills.

**Bottom line:** most of what this needs already exists. Two pieces are missing:

1. A **global** "what Pear traders trade" aggregate. The authoritative data for it is already reachable, but only from `apps/agent`.
2. A hook that feeds that aggregate into the basket run's candidate universe.

Personal affinity is already computed and already injected into the chat pair/basket nodes. No new ingestion pipeline is needed.

codectx ran fine (semantic mode, no lexical fallback). Everything below was checked against the source.

---

## 1. Current system map

### Basket generation (the daily `made_baskets` set)

|Piece|Location|What it does|
|---|---|---|
|Scheduler (2 paths)|`apps/context/deploy/hetzner/context-baskets.timer` (14:00 UTC → `node dist/main --baskets-full`); in-process `@Cron("0 3 * * *")` at [made-baskets-cron.service.ts:76](https://claude.ai/epitaxy/apps/context/src/sentiment/services/made-baskets-cron.service.ts:76)|Both call `refreshDaily`. The in-process one is disabled only by the env flag `BASKET_CRON_DISABLED` (:87). The table has no unique constraint, so a double run double-writes.|
|Run orchestration|`MadeBasketsCronService.refreshDaily` / `refreshPlatform` (:77–341)|For each platform (HL, SYMM, Lighter): a general pass (6 baskets), then an equity (HIP-3) pass, then forced long-only and short-only passes. It loops until nothing new comes out. Dedup guards: Jaccard overlap, composition overlap, `MAX_ASSET_APPEARANCES=3` (asset frequency cap), opposite-side check, name similarity (:343–483).|
|**Single funnel**|`MadeBasketsService.generateBasket` [made-baskets.service.ts:323](https://claude.ai/epitaxy/apps/context/src/sentiment/services/made-baskets.service.ts:323)|Builds one `input` string (category analysis, avoid-list, platform/structure/exclusion rules) and passes it to `basketAgent.makeBasket(input, platform, opts)` (:385).|
|Narrative input|`getAnalysisOnBasketCategories` (:249)|Reads the latest `CategorySentiment` per category and asks the sentiment agent for the bullish and bearish categories. The result is cached for the run.|
|Agent (OpenRouter)|`apps/context/src/sentiment/agents/basket/agent.ts`|Tools: `assets_by_category` and `quant_basket_stats`.|
|Agent (Claude, Hetzner)|`claude-basket.provider.ts` → `runTwoStageBasket` / `buildCandidateUniverse`; selected by `BASKET_LLM_PROVIDER` at [basket-runner.module.ts:26](https://claude.ai/epitaxy/apps/context/src/basket-runner.module.ts:26)|Research stage (web search) produces survivors; a selection stage builds the basket from survivors only. The universe comes from the same source as the OpenRouter path.|
|**Candidate universe**|`SentimentService.getAssetRowsByCategory` [sentiment.service.ts:107](https://claude.ai/epitaxy/packages/shared/src/services/sentiment.service.ts:107) + `createGetAssetsByCategoryTool` [assets-by-category.tool.ts:20](https://claude.ai/epitaxy/packages/shared/src/tools/assets-by-category.tool.ts:20)|The 20 most-recent `AssetSentiment` rows per category, within 14 days, shuffled, then filtered by `getListedAssetIds(platform)`. **Where candidates come from today.**|
|What gets fresh sentiment|`SentimentCronService.pickDailyAssets` [sentiment-cron.service.ts:428](https://claude.ai/epitaxy/apps/context/src/sentiment/services/sentiment-cron.service.ts:428)|Top 10 by **Coinglass venue-wide open interest** (`AssetOi`, from `stats/services/oi.cron.ts`), plus 7 hard-coded HIP-3 tickers. The whole universe is refreshed only weekly (`updateAssetsWeekly`, :311).|
|Stats validation|`generateBasketWithStats` (:420)|Directional baskets: listed-check only. 1x1: `getPairStats` + `hasHedgedDirection` (:518). Multi-leg: `/stats/basket` API, rejects `listed=false` and `hedge_ratio<0` (:594, :607). Leverage is derived from spread volatility.|
|Chat consumer|`quant_recent_made_baskets` [recent-made-baskets.tool.ts](https://claude.ai/epitaxy/apps/agent/src/ai/tools/quant/recent-made-baskets.tool.ts)|Latest run per platform, ordered by cointegration then correlation. **No per-user ranking.**|

**Misleading abstraction:** `BASKET_AGENT` is two unrelated things. In `apps/context` it is the daily generator; in `apps/agent/src/ai/agents/quant/basket` it is the chat node that _reads_ `made_baskets`.

### Pair trades

- `quant_pair_ideas` → `QuantService.getPairIdeas`: either the signals feed (`signalsv3`/`viable_signals`) or `getPairIdeasFromStats` [quant.service.ts:384](https://claude.ai/epitaxy/packages/shared/src/services/quant.service.ts:384). The latter takes tiered hurst/half-life/correlation/z cuts over the latest `pair_stats_{1h,4h,1d}` for active assets (`days_old>=7`).
- `pair_stats_*` already covers roughly all active pairs, so **statistical validation for any familiar×familiar pair already exists.**

### Trade history, journal, personalization

|Source|Location|Scope|
|---|---|---|
|**DTP** `UserTradeProfile`|[trade-profile.service.ts](https://claude.ai/epitaxy/apps/agent/src/ai/services/user/trade-profile.service.ts), `computeTradeProfile`|Per **wallet**, last 25 closed V3 engine positions across all venues. Fields include `topLonged`, `topShorted`, `topEarners`, mix, `medianNotionalUsd`, `lastTradeAt`. Recomputed after agent executions and lazily after 24h.|
|DTP injection|[node-executor.service.ts:105-125](https://claude.ai/epitaxy/apps/agent/src/ai/dag/executor/node-executor.service.ts:105)|**Already** passed into pair/basket nodes, together with `ExposureHintService` (current open positions, "never pitch a new position in a held asset").|
|`JournalEntry`|[journal.processor.ts:113](https://claude.ai/epitaxy/apps/agent/src/ai/background/journal.processor.ts:113), `buildPayload` :431|One row per closed Pear engine position, **linked users only** (unlinked are skipped). Payload: `longAssets`, `shortAssets`, `notional`, `closedAt`, `clientId`, `connector`.|
|`UserInterest`|interest-distiller|LLM-distilled `asset:<T>` weights; `source:"trade"` is ranked highest. Person-keyed.|
|`UserValueScore`|[user-value-score.processor.ts](https://claude.ai/epitaxy/apps/agent/src/ai/background/user-value-score.processor.ts)|Nightly per-person score. Groups WalletLinks into people and sums journal `notional` over 90 days. The closest existing precedent for a "person-grouped aggregation job".|
|Identity|`WalletLink` (wallet ↔ `pearUserId` ↔ `tgUserId`), `canonical-id.ts`|JournalEntry, UserInterest and signals are keyed by person. DTP is keyed by wallet.|
|Realtime engine events|[v3-realtime-subscriber.service.ts](https://claude.ai/epitaxy/apps/agent/src/ws/v3-realtime-subscriber.service.ts)|Carries `{topic,userId,entityId}` for **all** Pear users, but events for unlinked users are dropped (hydration needs that user's key).|

### Platform-wide, Pear-only sources (the important find)

|Source|Location|Notes|
|---|---|---|
|**Pear BE Postgres (read-only)**|[best-trade.reader.ts](https://claude.ai/epitaxy/apps/agent/src/best-trade/best-trade.reader.ts) `POSITIONS_SQL` / `FILLS_SQL`, env `BE_READONLY_DATABASE_URL`|Already queried weekly. It exposes `positions` (`creation_source='ENGINE'`, `origin<>'EXTERNAL'`, vault users excluded) and `fills` (`usd`, `is_external`, `synthetic`, `status<>'RECONCILED'`, `reduce_only`, `client_id`). It also carries a bot/test-account heuristic (`best-trade.ts:15`: median hold under 5 min _and_ median notional under $50). **This is authoritative and Pear-only for all users, not just agent users.**|
|Engine `markets.basketPositions`|[list-baskets.usecase.ts](https://claude.ai/epitaxy/packages/execution/src/domain/market/list-baskets.usecase.ts), `BasketPopularity` in `domain/market/types.ts`|`opened` and `traders` per pair/basket key. Explicitly Pear-only. But it is **cumulative and never decremented** (no recency) and keyed by composition, not asset. Used only by apps/mcp.|
|Engine leaderboard|[leaderboard-cache.service.ts](https://claude.ai/epitaxy/apps/agent/src/ai/services/leaderboard/leaderboard-cache.service.ts)|Has an `asset` filter and a date window, but rows are per account, capped at 200, and each call takes 14–20s (:30–37). Not viable for per-asset scans.|
|Hyperliquid wallet history|—|No HL info-API fill ingestion found (`userFills`/`clearinghouseState` grep only hit the TG `/portfolio` command). So there is no risk of mixing in non-Pear activity, and no pipeline to reuse.|

---

## 2. Sources for "familiar assets", compared

|Source|Pear-only?|Coverage|Recency|Notional|Distinct traders|Verdict|
|---|---|---|---|---|---|---|
|**BE `fills`+`positions`**|✅ (engine, non-external, non-synthetic)|all Pear users|any window|✅ `fills.usd`|✅ `positions.user_id`|**Authoritative for global.**|
|`JournalEntry`|✅|linked (agent) users only, closes only|✅|✅|✅ (person)|Biased sample: agent users are not the Pear base.|
|DTP|✅|per wallet, last 25 closes|24h stale|—|n/a|**Personal only**; already injected.|
|`basketPositions`|✅|all|❌ cumulative|❌|✅|V2 signal for which _compositions_ are popular.|
|`UserInterest`|mixed (LLM)|active agent users|daily|❌|—|Soft personal signal; too noisy to rank globally.|
|`AssetOi` / Coinglass|❌ venue-wide|—|daily|OI|—|Market popularity, not Pear familiarity. It is what drives `pickDailyAssets` today.|

**Can we query these?**

- **Globally most-traded:** yes, via the BE DB.
- **Per user:** yes. DTP for agent users; BE DB `user_id` for everyone.
- **Cohorts:** only by join (for example DTP `mix`). There is no existing cohort notion.
- **Recent preferences:** yes, from BE fill timestamps or journal `closedAt`.

**Things that distort rankings, and existing mitigations:**

- **Whales:** ranking by notional lets them dominate. Use **distinct traders** as the primary signal.
- **Bots and test wallets:** reuse the best-trade heuristic.
- **Duplicate and failed orders:** the fills `status<>'RECONCILED'` filter handles these. Failed orders never produce fills.
- **Tiny trades:** apply a minimum `usd` per fill.
- **External and liquidation closes:** filter on `is_external` and `synthetic`.
- **Vault accounts:** exclude `users.role='vault'`.
- **Old history:** use a 30-day window.
- **Self-reinforcement:** the agent's own suggestions produce trades that then look familiar. `client_id='AGENTPEAR'` identifies these (it is already used in `POSITIONS_SQL`).

**No canonical "active Pear trader" definition exists.** There are three that disagree:

- Interest distiller ([interest-distiller.processor.ts:184](https://claude.ai/epitaxy/apps/agent/src/ai/background/interest-distiller.processor.ts:184)): recall memory or a journal entry within 30 days.
- Value score: every WalletLink with a `tgUserId`, recency half-life 14 days, volume window 90 days.
- Best-trade: agent-first-fill positions in the week.

All three are agent-user definitions. None covers the Pear base.

---

## 3. Personal vs global

- **Global:** missing. This is the gap.
- **Cohort:** not justified yet. The user count per cohort is unknown and likely too thin. Defer.
- **Personal:** **exists and is already injected.** DTP `Most longed/Most shorted` plus the exposure block go into the pair/basket node `stateHint`.

**Key structural constraint:** `made_baskets` is **one global daily set shared by all users**. Personal affinity therefore cannot shape _generation_. It can only shape _ranking at read time_ (the chat tool or proactive touches). This splits the design:

- global familiarity feeds generation;
- personal familiarity feeds ranking.

Journals and memory are not needed for V1. DTP already gives personal affinity for free.

---

## 4. Where familiarity plugs into basket generation

- **Where candidates come from:** `getAssetRowsByCategory` (sentiment rows), which is ultimately decided by `pickDailyAssets` (top Coinglass OI plus 7 hard-coded equities). A familiar asset outside that set only gets weekly sentiment. That means stale notes, and it can fall out of the 20-row cut per category.
- **What the generator receives:** category sentiment analysis, recent basket names (avoid-list), platform/structure/exclusion rules, and tool results (`assets_by_category` rows with sentiment and notes, `quant_basket_stats`).
- **Injection points, both in `apps/context`:**
    1. **`pickDailyAssets`**: add the familiar assets so they get fresh narrative analysis (`AssetSentiment`). Without this, a boost only surfaces stale notes.
    2. **`generateBasket` `input`** (the single funnel for both providers): add one data block listing familiar assets with their trader counts. Optionally annotate `getAssetRowsByCategory` rows with `pearTraders`, and sort familiar rows ahead of the shuffle so the take-20 cut doesn't drop them.
- **Constrain, rank or boost?** **Boost, don't constrain.**
    - Constraining starves thin pools (SYMM, Lighter, HIP-3) and overrides narrative conviction.
    - The existing `MAX_ASSET_APPEARANCES=3` and overlap guards already stop popular assets from monopolising a run.
    - Measure the share of baskets with at least one familiar leg; don't enforce it.
- **Narrative analysis** happens in `SentimentCronService` (asset + category) and `getAnalysisOnBasketCategories`.
- **Statistical validation** already exists: `/stats/basket` and `pair_stats` checks, plus negative-hedge and no-direction rejection.
- **Reuse for pairs:** yes, if the familiarity list is published as a shared artifact rather than computed inside the basket code (see §6).

---

## 5. Hidden constraints and failure modes

|Concern|Finding|
|---|---|
|Little or no history|Global list covers cold start. The DTP block is already null when there are no trades.|
|Popularity ≈ OI|Familiar Pear assets are probably mostly BTC/ETH/SOL/HYPE, which top-OI already picks. **This could make V1 a no-op.** Validate first (§6.10).|
|Repetitive suggestions|Within a run: guarded. **Across days: only `getLatestBaskets(10)` is in the avoid-list.** A fixed familiar list will re-propose the same themes daily.|
|Global drowning personal|Avoided by the split: global feeds generation, personal feeds ranking.|
|Exposure|**Tension:** `ExposureHintService` says never pitch assets the user already holds. Familiarity should mean _recently traded_, not _currently held_.|
|Unsupported or delisted assets|Already handled (listed filter, `assertLegsListed`, `UnlistedBasketError`, `validate` in the chat tool).|
|ID mapping|BE symbols must be converted to quant `<T>USDT` IDs via `AssetIndexReader.resolveQuantAssetId`. It returns **null for `HYPE`** (reader.spec.ts:760), so some familiar assets cannot enter the quant universe at all. *(Correction 2026-09-29: that was a test fixture; production maps HYPE → `HYPEUSDT`. The Pear top 20 all map. See [[Pear Trader Asset Familiarity — Data Findings]] §12.)*|
|Venue mapping|SYMM corresponds to the binance/bybit/okx connectors. Count per connector group so HL familiarity isn't pushed into a SYMM run.|
|Negative beta|Already rejected: `hedge_ratio<0`, and `hasHedgedDirection` for 1x1.|
|Multi-account users|BE `positions.user_id` is the Pear user, so counting distinct `user_id` dedupes multi-wallet users. DTP stays wallet-scoped.|
|Scan cost|One `GROUP BY` over 30 days of fills per day. No per-address scanning.|
|Claude path parsing|`selectedCategories(input)` falls back to regex-matching category names in `input`. The new block must not contain category words such as "AI" or "Meme".|
|Credential spread|The context app only has `CONTEXT_DATABASE_URL`, `QUANT_DATABASE_URL` and `AGENT_REDIS_URL`. `pg` and `BE_READONLY_DATABASE_URL` exist only in `apps/agent`.|

**Other cleanup to mention (not fixing):**

- DTP `uiShare` and `pearNativeRatio` are always null.
- The `UserTradeProfile` column is named `userId` but holds a wallet.
- `XDigestRun` has no readers.
- There are duplicate basket schedulers.

---

## 6. Smallest sensible architecture

1. **Current state:** generation is driven by narrative (category sentiment → OI-picked asset sentiment → LLM). Personal trade affinity exists only at chat time. Pear-wide trade data is read only by best-trade.
    
2. **Reusable pieces:**
    
    - BE read-only access (`pg` client plus `BE_READONLY_DATABASE_URL`) and the SQL filters in `best-trade.reader.ts`
    - The bot heuristic in `best-trade.ts`
    - `AssetIndexReader.resolveQuantAssetId`
    - The agent Redis, which the context app already reads
    - DTP plus exposure injection
    - `pair_stats_*`
    - Run-scoped memo pattern (`categoryAnalysisCache`)
    - Hetzner one-shot pattern (`main-best-trade.ts`, `main-user-value-score.ts`)
3. **Missing:** a global familiar-asset aggregate, and a way to get it from apps/agent to apps/context.
    
4. **Recommended data source:** BE `fills` joined to `positions` (Pear engine, non-external, non-synthetic, non-reconciled, non-vault, opening fills). Not journals, which cover agent users only.
    
5. **V1 ranking (per asset, per connector group, 30-day window):**
    
    - `traders` = distinct `user_id` (primary); `notional` = Σ `usd` (tiebreak, as log).
    - Keep assets with `traders ≥ 3` (tune once real counts are in).
    - Exclude bot/test users and fills under ~$10.
    - Take the top ~30.
    - No blended weights in V1.
6. **Integration point:**
    
    - A `familiar` flag in `pickDailyAssets` (for fresh narrative);
    - one data block appended to `input` in `MadeBasketsService.generateBasket`, intersected with the platform-listed universe;
    - optionally, familiar-first ordering inside `getAssetRowsByCategory`.
7. **Pairs later:** the same Redis artifact feeds:
    
    - a `getPairIdeasFromStats` variant with `asset1_id, asset2_id ∈ familiar`;
    - the pair node's `stateHint`, one line next to the DTP block.
    
    Nothing in it is basket-specific.
    
8. **Storage and jobs:** one nightly one-shot in `apps/agent`, run before the 12:00 UTC sentiment job. It writes a Redis key such as `familiar-assets:v1` (JSON, TTL about 48h). No Prisma table, no new ingestion.
    
    - Alternative: give `apps/context` `pg` and the BE URL, and query once per run. Fewer moving parts, but it spreads the BE credential and isn't reusable by the chat side.
9. **Files likely to change:**
    
    - new `apps/agent/src/familiar-assets/{familiar-assets.reader.ts,.job.ts}` and `main-familiar-assets.ts`
    - a Hetzner unit
    - [sentiment-cron.service.ts](https://claude.ai/epitaxy/apps/context/src/sentiment/services/sentiment-cron.service.ts) (`pickDailyAssets`)
    - [made-baskets.service.ts](https://claude.ai/epitaxy/apps/context/src/sentiment/services/made-baskets.service.ts) (`generateBasket`)
    - optionally [sentiment.service.ts](https://claude.ai/epitaxy/packages/shared/src/services/sentiment.service.ts)
    - a small shared Redis-key reader in `packages/shared`
10. **Risks and unknowns to validate:**
    
    - **Run the aggregate SQL once by hand and diff it against `pickDailyAssets`.** If it's ≥80% the same majors, V1 changes little. In that case, reframe toward long-tail familiar assets (rank by traders with majors excluded) or skip V1.
    - Confirm the form of `fills.symbol` and the connector values. The best-trade SQL proves the columns exist; the aggregate's joins are unverified.
    - Mapping gaps (HYPE and others). *(Update 2026-09-29: HYPE is fine; top 20 all map. PONS-style category gaps remain.)*
    - Feedback loop: consider excluding `AGENTPEAR`-originated positions.
    - **No success metric exists yet.** `basketPositions` `opened`/`traders` for made-basket compositions could provide one.

### V2

- **Personal rerank** of `quant_recent_made_baskets` using DTP `topLonged/topShorted` and `UserInterest` asset topics.
- **Proactive:** emit `TouchCandidate`s when a new basket overlaps a user's affinity. The governor and budget already exist.
- **Popular compositions:** use `basketPositions` to seed variations of pairs/baskets people already trade.
- **Recency blend:** mix 7-day and 30-day windows, add repeat-trader weight, add cross-day novelty (extend the avoid-list beyond 10 rows, or decay recently used familiar assets).
- **Cohorts:** by DTP mix, once user counts support it.

---

## 7. Proposed flow

```
BE Postgres fills⋈positions (Pear engine, non-external/synthetic/reconciled, non-vault, last 30d)
  → drop bot/test users + dust fills  (best-trade heuristic)
  → per (connector-group, asset): distinct traders, Σ usd
  → map symbol → quant assetId (AssetIndexReader), keep traders ≥ N, top ~30
  → Redis familiar-assets:v1   [nightly one-shot in apps/agent, before 12:00 UTC]
      ├─ context: pickDailyAssets ∪ familiar → fresh AssetSentiment (narrative)
      ├─ context: generateBasket input += "Pear traders actively trade: …" (boost, not constraint)
      │     → assets_by_category (listed-filtered) → LLM/Claude compose
      │     → /stats/basket | pair_stats validation (neg-hedge / no-direction rejects)
      │     → run guards (freq cap 3, overlap, opposite-side) → made_baskets
      └─ agent (V2): pair node familiar×familiar pair_stats filter
  → chat read: quant_recent_made_baskets  +  DTP (personal) + exposure block → rerank (V2)
```

Next step before writing any code: run the aggregate query read-only against the BE and compare it with the current OI picks. That result decides whether V1 is worth building as scoped.