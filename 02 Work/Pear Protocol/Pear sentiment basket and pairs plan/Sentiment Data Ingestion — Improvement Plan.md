# Sentiment Data Ingestion — Improvement Plan

- Status: **plan, nothing built.** Draft 2026-09-30, rescoped same day to **adding data sources only** (history + scoring on hold, see Scope decision). Extends issue [#1043](https://github.com/pear-protocol/agent-pear-v2/issues/1043). **Tracked as story [#1052](https://github.com/pear-protocol/agent-pear-v2/issues/1052)** (sub-issue of #1043, opened 2026-09-30), which is the team-facing version of this plan.
- Related: [[Asset Sentiment Playground — Research & Experiments]] · [[Better Trading]] · [[00 Home/Current Focus]]
- Code read on `fix/twitter-labeling-dead-image-retry-dev` (2026-09-30): `apps/context/src/sentiment/services/sentiment-cron.service.ts`, `agents/sentiment/{agent,prompt}.ts`, `services/exa-web.service.ts`, `packages/shared/src/services/{twitter.context,polymarket,tokenomist,sentiment}.service.ts`, `made-baskets.service.ts`, `assets-by-category.tool.ts`.

## Handoff (current state)

*Snapshot 2026-09-30. Read this first; the sections below are the history of how we got here (later "Update" sections override earlier ones).* *(Handoff partly superseded: phase 1 is built and merged into the parent. Current to-do list: see "Next steps (as of 2026-09-30 late)" at the bottom.)*

- **Tracking:** story [#1052](https://github.com/pear-protocol/agent-pear-v2/issues/1052) (sub-issue of #1043) is the team-facing plan: checklist per phase, decisions, consumers table, CoinGecko test, branch flow. This note has the same plus reasoning.
- **Scope:** (1) more data for **asset** sentiment, (2) agentic sentiment with a stronger model + own web research (phase F, after phase 1). Output schema `{sentiment, webNotes, twitterNotes}` stays fixed (4 consumers). Category sentiment changes wait. Prod history (H1) + forward-return scorer on hold.
- **Branch flow:** every change = PR into `sentiment-enhancement` (cut from `development` @ `5676208`). Test combined branch locally. One PR `sentiment-enhancement` → `development` at the end. Merge `development` in periodically. Windows pre-push hook is broken → push with `--no-verify` only after the user OKs it.
- **Done (merged into `sentiment-enhancement`):** #1054 dropped Polymarket (+ weekly mislabel). #1055 `runAt` on `assets_by_category` rows. Verified: shared build, context + agent `tsc`, 114/114 context claude+sentiment specs. Not run live.
- **Next = phase 1** (behind `SENTIMENT_MARKET_DATA_ENABLED`):
  1. small Hyperliquid candle fetch in `apps/context` (don't move the 1,245-line `apps/agent` `HyperliquidMarketService`)
  2. 1d/7d/30d return, vs BTC (crypto) / `xyz:XYZ100` (stocks) / none (commodities), 30d range position, 30d vol, each with a plain reading computed in code
  3. `MARKET DATA` section in the asset prompt, before news/tweets
  4. CoinGecko `/news` for crypto, with guards: last 7 days, dedupe by story, name/ticker in title. Needs `COINGECKO_API_KEY` in the context env (Railway + Hetzner)
  5. Exa prompt: dated events + sources only (drop price action / market sentiment / unlocks)
  6. "missing inputs" line; source failure never skips an asset; daily + weekly runs
  7. validate: old vs new side by side, daily 17 × 3 days, senior reviews before flag on
- **Open question for the user:** include playground call-logging (store every call, score vs HL price at +1/3/7d relative to benchmark) in phase 1? Recommended yes.
- **Then:** phase 2 positioning (HL funding/OI all assets; Coinglass L/S + liquidations daily crypto), phase 3 tweet matching, phase F agentic (`gpt-6-luna` / `grok-4.3` vs deepseek; Claude CLI as alternative), phase 4 stock vendor.
- **Key code:** `apps/context/src/sentiment/services/sentiment-cron.service.ts` (daily/weekly runs), `agents/sentiment/{agent,prompt}.ts`, `services/exa-web.service.ts`, `packages/shared/src/services/{sentiment,twitter.context,coinglass}.service.ts`. Remember: rebuild `@agent-pear/shared` dist before typechecking apps.

## What sentiment is actually for

Asset sentiment decides **LONG vs SHORT** for basket legs, over a horizon of days. Category sentiment picks **which sectors** a basket draws from. So the question for every input is: *does it help call the direction of this asset over the next 1–7 days?* Anything that doesn't is cost at best and noise at worst.

## How data flows today

```
per asset (daily 17 / weekly ~185)
  Exa (6 results) ──► gemini-2.5-flash digest ──┐
  tweet DB (24h, ≤10, substring match) ─────────┤
  Tokenomist table (unlocks/burns/buybacks/raise)├─► deepseek-v4-pro prose ──► gpt-oss-20b extract ──► AssetSentiment
  Polymarket search (crypto, price markets) ────┘      {webNotes ≤500, twitterNotes ≤300, sentiment}

category (Wed/Sun): latest asset rows ≤14d, first 15, notes cut to 150/100 chars ──► CategorySentiment
baskets: CategorySentiment ──► "which categories bullish/bearish" LLM call ──► assets_by_category (rows ≤14d, no runAt) ──► basket agent
```

## Noise that gets in unintentionally (verified in code)

Each point says what happens and why it hurts. Whether each one *measurably* hurts accuracy is a hypothesis: ablations are on hold with history.

1. **Tweet ticker collisions.** `searchForSentiment` matches `text contains <ticker>`, case-insensitive (`twitter.context.service.ts` ~L370). Short or common-word tickers pull unrelated tweets: `HYPE` matches "hype", `META` matches "metaverse"/"MetaMask", `OP` matches "top"/"opinion", `SUI` matches "suite", `ENA` matches "arena", `AI`, `S`, `W`, `IP`… Those tweets are then presented as "community voice" for that asset.
2. **Tweet labels are per tweet, not per asset.** Each tweet carries a pre-computed `sentiment` from the labeling model, printed as `Sentiment: BULLISH`. A tweet bullish on BTC that calls ETH a laggard is still "BULLISH" when shown for ETH. The synthesizer anchors on these labels.
3. **Tweet sample is small and one-sided in source.** 42 watched accounts, 24h window, ≤10 tweets. The prompt says "focus on the last 7 days" but tweets only cover 24h. *(Bias of those accounts toward bullish CT takes = hypothesis, not measured.)*
4. **The web digest is an LLM's opinion passed off as facts.** gemini-flash is asked for "market sentiment" and "community/institutional sentiment", not just events. Then `fetchWeb` **drops the source URLs** (`sources` returned but discarded), so the synthesizer can't see dates or judge recency/credibility. The digest also reports "price action" second-hand from articles, which may be days old.
5. **Duplicate, possibly conflicting unlock info.** The crypto web prompt asks for "token unlocks" and Tokenomist also supplies unlocks. Two versions of the same fact can disagree.
6. **Polymarket is interpreted wrong.** The search keeps only price markets (`PRICE_KEYWORDS`: "above", "below", "drop", "fall", "hit"…), which are mostly strike ladders ("Will BTC be above $X on date?"). The prompt rule "high Yes (>65%) = bullish" is wrong for these: a 95% "above $60k" is just a deep in-the-money strike, and "Yes" on a "drop below" market is bearish. Without spot price and strike next to each other, the odds say little about direction.
7. **Weekly run mislabels Polymarket as tokenomics.** `updateAssetsWeekly` merges Polymarket into `tokenomicsContext` and passes no `polymarketContext` (`sentiment-cron.service.ts` ~L383–392). The prompt then shows prediction-market odds under "TOKENOMICS DATA" and "No prediction markets data available" under the Polymarket header. Straight bug.
8. **Tokenomist fields with no near-term meaning.** "Fundraising (bullish backing)" is historical and says nothing about the next week. Burns/buybacks are only useful if recent and sized against supply.
9. **The prompt forces confidence on thin data.** "Never say insufficient data", "extrapolate from what's available", "be DECISIVE, avoid neutral". A stock with 0 tweets and a thin digest still gets a confident call, and the output has **no confidence/coverage field**, so downstream can't tell a strong call from a guess.
10. **Freshness is invisible downstream.** Basket rows (`getAssetRowsByCategory`) drop `runAt`. Only 17 assets are fresh daily; everything else is from the weekly run (up to 14d via the freshness cutoff). The basket agent can't tell today's call from one made last week.
11. **Category step throws away information at random.** It takes the **first 15** rows in `runAt desc` order (not the most important ones), truncates notes to 150/100 chars, and tallies fresh and week-old calls equally ("60%+ lean one way → be decisive").
12. **Basket category prompt invites made-up research.** `CATEGORY_SENTIMENT_AGENT_SYSTEM_PROMPT` says "research on the web for the latest news, tweets" but the call has no tools or web plugin. The model can only invent "research".
13. **Four LLM hops on the path** (flash digest → v4-pro → extractor → category → basket-category). Each hop paraphrases and can drift. Deterministic numbers (prices, funding) should never pass through an LLM summary before the decision step.

Dead code seen (mention only, not noise): the `webResults` score>0.5 path in `agent.ts` (Tavily era; `webResults` is always `[]`), and the stale comment "equities ride Tavily scored snippets".

## What's probably not needed (for baskets)

| Item | Why it's low value | Action |
|---|---|---|
| Polymarket as used today | Strike-ladder odds without spot price, wrong interpretation rule (#6) | Remove from the prompt; revisit only as a computed "implied move vs spot" number |
| Tokenomist fundraising | Historical, no 1–7d signal | Drop |
| Web "market sentiment" / "community sentiment" | An LLM's vibe read, not an event | Ask the digest for dated events only |
| Web "price action" | Second-hand and stale once real price is added | Replace with real price data |
| Kalshi per asset | Mostly macro/event markets | At most once per run as macro context for categories |
| CoinGecko | Overlaps Hyperliquid price/volume | Skip |
| Elfa **and** twitterapi.io | Both are "social" | Pick one; don't stack both |
| Many TA indicators (RSI, MACD, BB, …) | Collinear with returns; invite pattern stories | Use a few plain price features (below) |
| Long free-text notes as the main basket input | The basket agent needs direction, strength, key catalyst, key risk | Add structured fields; keep notes short |

## What to add (highest signal-to-noise first)

All numbers are computed in code and shown as numbers **plus a plain reading against the asset's own recent history** (e.g. "funding 0.03%/8h, 95th percentile of its 30d range"), so the LLM doesn't have to know what's normal.

1. **Price context (Hyperliquid, free, covers HIP-3):** 1d / 7d / 30d return; return relative to BTC (crypto) or a stock benchmark (HIP-3); distance from 30d high/low; 30d realized volatility (lets us say whether a move is big for this asset). Note: `HyperliquidMarketService` lives in `apps/agent/src/ai/dag/services/`, so it needs moving to `packages/shared` or a small candle fetch in `apps/context`.
2. **Positioning (Coinglass, already in `packages/shared/src/services/coinglass.service.ts`):** funding now + 7d average (`frOhlcHistory`), OI change 1d/7d (`oiOhlcHistory`), top-trader L/S position ratio (`topLongShortPositionRatio`), 24h long vs short liquidations (`liquidationHistory`). Main use is **crowding**: price up + OI up + extreme funding = crowded long (fragile), price up + OI down = short squeeze. *Unknown: whether Coinglass covers HIP-3 tickers; if not, use HL's own funding/OI for those.*
3. **Better tweet matching (bug fix, no new vendor):** match `$TICKER` cashtags, the asset field (`assets has`), or the full name as a whole word. Never a bare substring for tickers ≤4 chars. Report tweet count and unique authors as coverage.
4. **Wider social (only if coverage is still thin after better matching):** twitterapi.io `$TICKER` search with an engagement floor, daily set only (cost). Pass raw text, and have the synthesizer judge the tweet's stance **on this asset**, instead of the per-tweet label.
5. **Stock catalysts (new vendor, needs a decision):** next/last earnings date, last surprise, recent analyst rating/target changes. Until then keep the Exa stock prompt, which already asks for these.
6. **Event calendar per asset:** unlocks within 14 days as % of circulating supply (Tokenomist, already there), earnings date (#5). Macro dates (FOMC/CPI) once per run at category level.

## Output changes (small, kept with the new data)

- ~~Add a **coverage** DB field~~ → replaced by a code-generated "missing inputs" line in the prompt (round 4, no schema change).
- Pass `runAt` (age) through `getAssetRowsByCategory` to the basket agent. **Ships now** with the bug PRs (round 5).
- Deferred with history: input snapshots, confidence field, structured `keyCatalyst`/`keyRisk` fields.

## Scope decision (2026-09-30)

**This plan = adding data sources only.** Additive sentiment history (H1), the forward-return scorer and source ablations are **on hold**. Consequences, stated so they aren't forgotten:
- We can't *prove* the new inputs make calls more accurate. Validation is a side-by-side read on the same day (old vs new pipeline output) plus spot checks against what the price did.
- The noise findings above stay as findings. The ones that are plain bugs are listed as separate follow-ups, not part of this plan.
- When history comes back, the measurement phases below slot in before any prompt/model change.

- **Decisions (grilling round 2, 2026-09-30):**
  1. Asset sentiment only; category changes wait.
  2. Ship now as separate small PRs: the weekly Polymarket-under-tokenomics mislabel, and "research the web" in the basket-category prompt.
  3. Polymarket: remove from the prompt now. Possible return after Phase 1 as a computed "implied move vs spot" number.
  4. Spend: only already-paid APIs (Hyperliquid, Coinglass, OpenRouter); new data on the daily 17 assets only; stock-vendor decision deferred to Phase 4.
  5. Before Phase 1 reaches the prod cron, the senior reviews a few days of side-by-side output. Same gate for later phases.

- **Decisions (grilling round 3, 2026-09-30):**
  1. HL data in `apps/context` via a small local fetch (daily candles + current funding/OI), not by moving the 1,245-line `HyperliquidMarketService`.
  2. Benchmarks: crypto vs BTC (BTC itself: absolute only); HIP-3 stocks vs `xyz:XYZ100` (Nasdaq-100; priority stocks are all tech); commodities absolute only. `xyz:XYZ100` and `xyz:SP500` verified listed on HL (live `meta`, dex `xyz`).
  3. Positioning: funding + OI from HL for all assets incl. HIP-3 (free). Coinglass only for top-trader L/S + liquidations, daily crypto set. This sidesteps the Coinglass HIP-3 coverage question.
  4. HL price/funding/OI also on the weekly ~185-asset run (free). Coinglass daily set only.
  5. Presentation: numbers + a plain reading computed in code ("7d −12%, ~2× its usual weekly move"; "funding at 95th pct of its 30d range"). The model interprets direction, code decides what's unusual.

- **Decisions (grilling round 4, 2026-09-30):**
  1. One env flag `SENTIMENT_MARKET_DATA_ENABLED` gates the whole new market-data block (HL price/funding/OI + Coinglass). Off = no deploy needed on Hetzner.
  2. A failing source doesn't skip the asset: it runs without that section and the prompt says the data is unavailable (same as Exa today).
  3. Senior gate: old vs new side by side in the playground, daily 17 assets × 3 trading days (~50 comparisons). Per-asset table: old call, new call, one-line reason for any change, price move over the next 1–3d. Presented as a sanity check, not accuracy proof.
  4. Tweet matching: `assets` field contains ticker, OR `$TICKER` cashtag, OR full name as a whole word (stocks — the labeler's `assets` schema says "Crypto assets mentioned", so stocks may be missing there), OR plain ticker as a whole word only for tickers ≥5 chars.
  5. Thin data: keep the prompt rule; add a code-generated "missing inputs" line (e.g. "No tweets found in the last 24h"). **The `coverage` DB field is dropped** (it would need a schema change).

- **Decisions (grilling round 5, 2026-09-30):**
  1. Phase 1 (price) and Phase 2 (positioning) roll out separately, each with its own 3-day side-by-side + senior review, so a changed call can be attributed.
  2. The Exa digest trim (dated events + source domains only; no "price action"/"market sentiment") ships in the same change as Phase 1.
  3. Pass `runAt` to basket rows now, alongside the two round-2 bug PRs.
  4. twitterapi.io: not in Phase 3 for now. Ship the matching fix, watch per-asset tweet counts for a few days, then decide (candidate trigger: <3 matched tweets on a daily asset).

## Phases

**Phase 1 — price context (Hyperliquid, free, all assets incl. HIP-3)**
- Compute in code: 1d/7d/30d return, return vs BTC (crypto) or a stock benchmark (HIP-3), distance from 30d high/low, 30d realized vol.
- Add a `MARKET DATA` section to the asset prompt, placed before web/tweets so the model anchors on real numbers.
- Stop asking the Exa digest for "price action" and "market sentiment"; ask for dated events only (removes the duplicate, second-hand price).
- Prereq: make an HL candle fetch available in `apps/context` (move `HyperliquidMarketService` to `packages/shared`, or a small local fetch).

**Phase 2 — positioning (Coinglass, already paid for, crypto)**
- Funding now + 7d avg, OI change 1d/7d, top-trader L/S position ratio, 24h long vs short liquidations.
- Each shown as a number plus where it sits in its own 30d range, with one-line crowding guidance in the prompt (price↑ + OI↑ + extreme funding = crowded long).
- HIP-3: use HL's own funding/OI if Coinglass doesn't cover them (unknown).
- Daily 17 assets first; weekly ~185 only after checking Coinglass rate limits.

**Phase 3 — social coverage**
- Tweet matching by `$TICKER` / asset field / whole-word name instead of substring (also fixes collisions).
- twitterapi.io `$TICKER` search (engagement floor, daily set only) **deferred**: decide after a few days of per-asset tweet counts on the new matching. Pick it **or** Elfa, not both.

**Phase 4 — stock catalysts (new vendor, needs a decision)**
- Earnings date and surprise, analyst rating/target changes. Until then the Exa stock prompt keeps covering these.

**Validation per phase:** run old and new pipelines on the same assets for a few days in `apps/context/test/sentiment-playground/` (outputs never to prod tables). Read side by side: did the call change, and is the new reasoning grounded in the new numbers? Note where the price actually went afterwards. Anecdotal, not a score.

## Bug follow-ups (separate small PRs, not this plan)

- Weekly run shows Polymarket under "TOKENOMICS" (#7).
- Basket-category prompt says "research the web" with no tools (#12).
- Polymarket "high Yes = bullish" rule is wrong for strike/drop markets (#6) → **decided: remove Polymarket from the prompt.**
- Dead Tavily `webResults` path + stale comment in `agent.ts`.

## Risks

- **More data can still add noise:** a big block of numbers the model doesn't understand invites storytelling. Keep each section short, with plain readings, not raw dumps.
- **No proof of accuracy** while history is on hold (see scope decision).
- **API limits/cost** on the weekly run (4 Coinglass calls × ~185 assets); unknown.
- **Coupling:** moving `HyperliquidMarketService` into shared touches `apps/agent` too (typecheck both).
- **Prompt size/latency:** synth p99 is ~40s today; extra sections add tokens. Watch timeouts (120s).

## Open questions

- Coinglass coverage of HIP-3 tickers, and plan rate limits.
- Which stock-data vendor (not researched).
- Stock benchmark for HIP-3 relative returns (is there an index on HL?).
- When does history (H1) come back, and who decides?

## Update (2026-09-30, after opening #1052)

- **Dropped the basket-category "research the web" fix (noise #12, round-2 decision 2).** Prod baskets run on Hetzner via `ClaudeSentimentAgentProvider` (`apps/context/src/basket-runner.module.ts:62`), which gives Claude WebSearch/WebFetch and appends matching instructions (or a "do not browse" override when web is off). The line is only misleading on the OpenRouter fallback path, and the prompt is shared, so it stays. Pointed out by me; verified in code.
- `runAt` to basket rows matters more than first thought: `assets_by_category` is the tool the Claude research stage calls to shortlist candidates.
- Ship-now list is therefore 3 PRs: weekly Polymarket mislabel, remove Polymarket from the prompt, `runAt` to basket rows.

## Update (2026-09-30, scope widened: agentic sentiment)

Scope of #1052 widened on my call: not only more data, also **a stronger model with a looser prompt and its own web research**. The "data sources only" scope decision above is superseded for this point; history in prod (H1) is still on hold. Issue retitled "sentiment: more data + stronger model with its own research for asset sentiment" and a phase F added.

Why it's worth it (reasoning, not measured): #229 showed a bigger model on the *same* thin inputs mostly buys nicer prose. An agent that fetches what's missing (why something moved, catalysts, stock earnings) addresses the actual gap, the thin inputs. It might also cover phase 4 without a new vendor.

Decisions:
1. Numbers (price, funding, OI) still come from code as a fact sheet. The model's freedom is for news/catalysts/reasons.
2. Output schema unchanged (`sentiment` + notes) so baskets don't change. Loosen the reasoning: it picks what to research, can say "no real signal", not forced decisive. Track the mixed/neutral rate.
3. Daily 17 only; weekly ~185 stays on the cheap pipeline.
4. Built after phase 1, own flag.
5. Models to test: `openai/gpt-6-luna` and `x-ai/grok-4.3` vs today's `deepseek/deepseek-v4-pro` (OpenRouter, tool loop with web search + fetch, capped tool calls).
6. Playground stores every call (asset, time, call, model) outside prod, scored later against HL price at +1/3/7d. This is the guard against the #229 trap (judging models by prose).
7. **Potential alternative: Claude CLI**, same pattern as the interest-distiller (`apps/agent/src/ai/background/claude-cli.ts`, fallback after repeated failures) or the basket runner (`apps/context/src/claude/claude-code.client.ts`). Caveats: shared Max login with baskets on Hetzner, unpinned CLI auto-updates, distiller web arm tied/lost at +45% cost / +70% latency (different task), estimate ~$0.20–0.40/asset. See [[Harness Integration/Step 7 — Claude CLI Feasibility Research for interest-distiller|Step 7]].

## Update (2026-09-30): who consumes asset / category sentiment

Verified in code (`development` @ `5676208`), also added to [#1052](https://github.com/pear-protocol/agent-pear-v2/issues/1052):

| consumer | where | weight |
|---|---|---|
| feed baskets (Hetzner Claude runner + OpenRouter fallback) | `apps/context` made-baskets / claude-basket.provider / pear-tools | a lot: categories, candidates, long/short lean |
| chat sentiment answers (TG + web, advanced mode) | `apps/agent/src/ai/agents/asset-sentiment/agent.ts`, `advanced/direct-tools.ts:379` | a lot: it is the answer; notes shown to users |
| chat basket suggestions | `apps/agent/src/ai/agents/quant/basket` via `asset_sentiment_query` | medium: prompt uses sentiment as per-leg reason and allows sentiment-based leg weights |
| chat pair suggestions | `apps/agent/src/ai/agents/quant/pair` via `asset_sentiment_query` | a little: pairs come from quant stats (`getPairIdeas` → `signalsv3` + `viable_signals`, or `pair_stats_1h`); sentiment is optional support |

- There is no "pair sentiment" concept. Pair ideas come from the stat-arb backend's tables, not a cron in this repo.
- Tweet labels (`TweetContext`) are a separate pipeline and an input to asset sentiment.
- **Correction to Claude's first answer in chat that day:** it first said the pair agents don't use sentiment (it grepped the wrong folder, `agents/pair` instead of `agents/quant/pair`). They do, via `asset_sentiment_query`.
- Implication: keep the output schema fixed for all four consumers, and check a few chat replies in each side-by-side, not only stored calls.

## Update (2026-09-30): branch flow

All #1052 work goes through a parent branch **`sentiment-enhancement`** (cut from `development` @ `5676208`, created on GitHub via the refs API since the Windows pre-push hook fails). Each change = its own PR into `sentiment-enhancement`; test the combined branch locally and iterate; only then one PR `sentiment-enhancement` → `development`. Reason: `development` auto-deploys to Railway dev, and the changes should be checked together first. #1054 and #1055 retargeted to it. Also written into #1052 ("branch flow" section).

## Update (2026-09-30): CoinGecko `/news` added to phase 1 (crypto only)

Senior suggested scraping CoinGecko coin pages (e.g. the Lighter page) for token news. Checked instead of scraping:
- **Our CoinGecko plan is Lite** (verified via `/key`, ~570k of 2M monthly credits used). `/news` needs Analyst or above → works with our key (HTTP 200). Docs: https://docs.coingecko.com/reference/news. `per_page` and `page` both cap at 20.
- API returns **headlines only**: title, url, source, author, `posted_at`, `type`, `related_coin_ids`. No article body, no LLM summary, no Elfa items. The page's "why it's moving" summary and Elfa AI cards are not in the API.
- The summarised part comes from **Elfa**, which we already pay for (Grow plan, `apps/agent/src/signals/elfa-data.service.ts`): `event_summary` = AI summaries; `token_news` = news-account posts with no text (X ToS). No `ELFA_API_KEY` in local `apps/agent/.env`, so not tested live. Elfa is not wired into `apps/context`.
- Recommended against scraping: we pay for both sources, layouts change, and scraping is probably not allowed (ToS not read).

Live test, 16 coins + 7 priority stocks (script + output in the session scratchpad, `cgnews-eval.mjs` / `cgnews-stocks.mjs`):
- Top ~50 coins: 20 to 40+ items in 7 days, newest usually <2h old, 8 to 21 outlets, real catalysts.
- Lighter (#82): 13 items. Pendle (#126): 2. Pear (#1078): 0 in 14 months.
- **Stocks: no coverage.** Only tokenized-stock ids exist (xStock/Ondo/bStocks/Robinhood), with 0 to 2 items ever, about the token not the company.

Caveats → guards in phase 1:
- loose tagging (20 to 40% of big-coin items off-topic, `related_coin_ids` noisy) → title must contain name/ticker (whole word or `$TICKER`)
- duplicates of the same story under different titles → dedupe by story, not title prefix
- stale for small caps → last 7 days only, else "no recent news"
- filler (predictions, "top 5 to buy") → prompt weighs catalysts over predictions
- no stocks → Exa stays for HIP-3 and as small-cap backup

Implementation note: needs a small fetch in `apps/context` (existing CoinGecko service is in `apps/agent`) and `COINGECKO_API_KEY` added to the context env (Railway + Hetzner). Added to #1052 phase 1 + a "coingecko news test" section.

## Update (2026-09-30): phase 1 built, local commit, not pushed

- **Decided:** playground call-logging is in phase 1. Store = local JSONL (gitignored), not a DB table.
- Branch `feat/sentiment-market-data` off `sentiment-enhancement` @ `5fe5843`, commit `87284905`. **Pushed, PR [#1058](https://github.com/pear-protocol/agent-pear-v2/pull/1058) into `sentiment-enhancement`** (open). Local `apps/context/.env` filled from `apps/agent/.env` (AGENT_REDIS_URL = agent REDIS_URL, discord bot vars placeholders).
- What's in it (all behind `SENTIMENT_MARKET_DATA_ENABLED`, flag off = asset prompt byte-identical to before):
  - `market-data.service.ts` in `apps/context`: HL daily candles → 1d/7d/30d return with "Nx its usual move" vs own 30d vol, vs BTC / `xyz:XYZ100` / none, 30d range position, 30d vol. CoinGecko `/news` for crypto with the guards (7 days, title names asset, word-overlap dedupe, max 8).
  - HL coin + CoinGecko id come from `AssetIndexReader` (asset index in agent Redis). Already in `SentimentModule`; added to `SentimentRunnerModule` (Hetzner one-shot) + redis quit in `main.ts`.
  - Exa events-only prompt (dated bullets + source domain; no price action/sentiment/unlocks).
  - MISSING INPUTS line. Source failure never skips the asset.
  - Daily + weekly now share `analyzeAsset`.
  - Playground: `apps/context/test/sentiment-playground/run.ts` (old vs new per daily asset → `out/calls.jsonl`), `score.ts` (+1/3/7d relative return, hit rate per variant).
  - `COINGECKO_API_KEY` optional in context env.
- Verified: context `tsc` clean, 118/118 claude+sentiment specs (4 new). Live free check on HYPE / `xyz:TSLA` / `xyz:GOLD` candles + HYPE news: output looks right.
- **Not verified:** full playground run. Local `apps/context/.env` is empty, so the app can't boot here.
- Seen in the live check: word-overlap dedupe misses paraphrased duplicates (2 pairs in 8 HYPE headlines). Price-move headlines ("HYPE falls to $86") also get through. Left as is for now, prompt tells the model to weigh catalysts over those.

## Update (2026-09-30): first smoke runs (ETH, 1 asset × old/new, twice)

- Local `apps/context/.env` filled from `apps/agent/.env`. Agent's `DISCORD_WEBHOOK_STATS_URL` was empty, so the user added one; run.ts never posts to it.
- Both runs: old and new both **bullish** on ETH. New had no missing inputs.
- Fix commit `cf532828` on #1058: a 0.0pp benchmark gap now reads "in line" (was "-0.0pp (underperforming)"); near-high/low bands 85/15% → 80/20%. 119/119 specs.
- Seen (anecdotal, 1 asset): the old Exa digest contradicted itself across runs on ETH ETF flows ("cooling $689M → $17M", then "record $689.8M weekly inflows"). The new events-only digest said outflows ($15.3M) both times. Fits noise #4 (digest = LLM opinion), not proof.
- Still open: webNotes cut mid-sentence at 500 chars (`cleanSentimentOutput`), both variants. Pre-existing, affects all consumers; raise separately.

## Update (2026-09-30): first full side-by-side (17 assets, run `2026-09-30T15:05Z`)

- Logged 29/34 calls. **New failed 4/17** (NVDA, GOOGL, ZEC: deepseek-v4-pro empty prose ×3 attempts; TSLA: timeout). **Old failed 1/17** (SOL: timeout).
- Flips on the 12 assets where both succeeded: META bearish → bullish, BTC neutral → bullish. The other 10 were the same call.
- META: new leans on the 30d +27.8% (+24pp vs XYZ100) and JPM target hike, and calls the -1.7% day noise (0.6x usual). Old leans on "Q2 missed, weak guidance" + valuation. Unverified which of the web facts is right.
- BTC: old says neutral (ETF streak stalling, spot demand down, yields). New says bullish on the BITCOIN Act reserve + 9-day ETF streak, with market data flat (0.1-0.6x usual moves).
- **Open, not diagnosed:** the empty-prose rate on the new prompt. Known deepseek behaviour (~1-8%, worse on HIP-3 equity prompts), but 3/17 new vs 0/17 old is a gap. Hypotheses: new prompt shape/length triggers it; the concurrency of 34 at once. Not tested.

## Update (2026-09-30): why the new variant failed (empty prose)

Reran NVDA / GOOGL / ZEC new variant one at a time, logging deepseek's raw OpenRouter response (script `out/diag-empty.ts`, gitignored):
- NVDA → provider **Relace**: 1st call 200 "stop" with 1 completion token and no content; retries 2-3 also Relace, **0 tokens, $0, <1s**. Failed again.
- GOOGL → DeepInfra, ZEC → Novita: normal output, both bullish.
- **Cause (strong evidence, n=1 Relace hit):** OpenRouter routes deepseek-v4-pro to Relace, which returns an empty 200. Relace has the cheapest input price of 16 endpoints ($0.38/M vs $0.78+), so price-weighted routing likely favours it. The 3-attempt retry doesn't help because retries stay on Relace.
- Not the new prompt: the same prompt shape worked on the other providers. The 3/17 vs 0/17 gap in the full run is most likely routing luck (not proven, providers weren't logged in that run).
- Probably also the source of the "~1-8% empty completion" rate noted in the agent code, and it hits prod today (unverified).
- Fix idea: `provider: { ignore: ["Relace"] }` on the sentiment synth (factory already supports `ignore`). Affects old pipeline too → separate PR.
- Timeouts (TSLA new, SOL old): not diagnosed. Likely synth/extractor under 34 concurrent calls.

## Update (2026-09-30): synth model → deepseek-v4.1-flash, Relace ignored

- User call: move the synth off `deepseek-v4-pro` to `deepseek/deepseek-v4.1-flash` ($0.02/$0.40 vs $0.78/$1.57 per 1M) and ignore Relace (it also serves flash). PR [#1060](https://github.com/pear-protocol/agent-pear-v2/pull/1060) into `sentiment-enhancement` (`fix/sentiment-synth-model`). `LlmSpec` gains `provider`. Blast radius: the synth also serves category sentiment + `invokeCategorySentimentForBaskets`.
- Added `ASSETS=` filter to run.ts (#1058 branch, commit `fee45633`, **local, not pushed**).
- Reran the 4 failed assets on flash (local branch `local/combined` = #1058 + #1060, not pushed): **8/8 ok, 0 empty.** NVDA/GOOGL/ZEC bullish both variants, TSLA bearish both. Notes read about as good as v4-pro on a glance; new TSLA grounds on market data (near 30d low, -8.8pp vs XYZ100).
- Today's side-by-side is now mixed-model: 13 assets on v4-pro, 4 on flash (the `model` field in calls.jsonl separates them). "Flash is stronger" is not measured.

## Update (2026-09-30): full 17 on flash + timeouts + retry/cache finding

- `fee45633` (ASSETS filter) pushed to #1058. Full run `2026-09-30T15:24Z` on `deepseek-v4.1-flash` (from `local/combined`): **0 empty. New timed out on 4/17** (MSTR, HYPE, BTC, NEAR). Old 0 failures. On the 13 where both worked: **0 flips** (12 bullish, LIT + TSLA bearish on both).
- Rerunning the 4 (new only) with every request timed: all succeeded. Synth (flash) 4.6-21.8s, extractor (gpt-oss-20b) 2.6-11.6s, HL/CG <0.6s. The script meant to run them in parallel ran them one at a time (my edit didn't apply), so load wasn't reproduced. **Which step timed out in the full run is still unknown.** Hypothesis: synth or extractor slow with 34 calls in flight, and new is longer + flash reasons more (up to ~1.3k reasoning tokens). Not tested.
- **Finding (strong evidence):** OpenRouter's response cache is always on (`CACHE_TTL_SECONDS = 300`, exact-request match, hits report 0 usage). So the synth's empty-prose retries resend the identical request and get the **cached empty response back**. That's what the "0 tokens, $0, <1s" retries 2-3 on NVDA/Relace were. The 3-attempt retry can't recover from an empty completion within 5 min. Pre-existing, affects prod. Fix idea: bypass the cache on retry.
- False alarm checked: the Exa digest showing 0 tokens / $0 in the diagnostic was also cache hits (same prompts minutes earlier). `web digest unavailable` never appears in any logged run's missingInputs.

## Update (2026-09-30): web digest model → gpt-6-luna

- User call: Exa digest model `google/gemini-2.5-flash` → `openai/gpt-6-luna` ($0.10/$0.50 vs $0.30/$2.50 per 1M). Commit `f2860a65` on `fix/sentiment-synth-model` (#1060), **local, not pushed**. Only `ExaWebService` uses `LLM.exaWeb`.
- Live check (ETH events-only, TSLA events-only, ETH old prompt), both models:
  - both follow the events-only format (date, event, source link), 6 sources each
  - luna is more careful and less padded: hedges where the source hedges (SEC FAQ "did not classify cbETH or stETH by name", Kalshi "not under investigation"); old prompt 2.4k vs 3.6k chars, no "Token unlocks: none mentioned" filler
  - **luna is slower:** 12-19s vs 7-10s. Timeout is 45s. Not tested under load.
  - the two models gave different dates for the same events (SEC FAQ 09-25 vs 09-27, Kalshi 09-28 vs 09-30). Not checked which is right.
- Gotcha: `git merge` into `local/combined` fails with "stash failed" the first time (the EOL-dirty files + OneDrive). Retry works. The first luna test accidentally ran on gemini because of it.

## Update (2026-09-30): day 1 of the 3-day side-by-side (run `2026-09-30T15:35:58Z`)

- Model set, fixed from here: synth `deepseek-v4.1-flash` (Relace ignored), extractor `gpt-oss-20b`, digest `gpt-6-luna`. Ran from local `local/combined` (#1058 + #1060). `f2860a65` pushed; #1060 retitled to include luna.
- **New timed out on 3/17** (ETH, HYPE, ENA). Old 0. Running total today: new 8 timeouts across 3 full runs, old 1. The failing step is still unknown (synth 120s or extractor 60s; every other source catches its own errors).
- Flips on the 14 both-ok assets: **PLTR, BTC, ZEC old bearish → new bullish.** Calls: old 12 bullish / 5 bearish, new 12 bullish / 2 bearish.
- **Old isn't stable run to run:** those same 3 assets were old-bullish in the 15:24 run, 11 min earlier. Between the runs only the digest model changed (gemini → luna) and the search results refreshed. Old's bearish notes lean on digest price/flow claims (BTC "cooling from $87.4k", ZEC "-12% over 7d, long liquidations").
- Open question: new is almost all bullish. Possible bias (e.g. under-weighting recent drawdowns: new ZEC leads with "+68%/30d" and ETF inflows). Not checked against its market data yet.
- Day 2/3 should run near the same time (~15:30 UTC) with the same model set.

## Update (2026-09-30, late): timeout instrumentation, scorer pairing

- User: no diagnostic rerun tonight; tomorrow's day-2 run is the diagnosis.
- Commit `5b484597` on #1058 (**local, not pushed**), merged into `local/combined`:
  - run.ts logs every OpenRouter request to `out/requests.jsonl` (model, asset, ms, provider, finish, tokens, real `usage.cost`) and every failed call to `out/errors.jsonl` (elapsed ms, error). Prints the run's real OpenRouter cost.
  - score.ts: `FROM=` run filter, and a "paired" summary (only assets where both variants made a call in the same run).
- Timeout fix itself not done: cause unknown until tomorrow's logs.
- Expectation for tomorrow's +1d score on day 1 (paired, 14 assets): 11 calls agree, so they score the same for both. The difference comes only from PLTR / BTC / ZEC (old bearish, new bullish), so the paired hit-rate gap will be 0-3 of 14 calls and decided by where those 3 moved. A direction, not a verdict.

**Update 2026-09-30 (late):** `5b484597` pushed; [#1058](https://github.com/pear-protocol/agent-pear-v2/pull/1058) merged into `sentiment-enhancement` (`b0abeda0`), then [#1060](https://github.com/pear-protocol/agent-pear-v2/pull/1060) (`7b62243c`). Still nothing in `development`/prod. Local checkout now on `sentiment-enhancement` @ `7b62243c` (same `apps/context` tree as the old `local/combined`); day-2 runs go from there. #1060 isn't flag-gated: flash + luna also hit category sentiment + basket-category fallback once the parent reaches `development`, so check those paths before that PR.

## Next steps (as of 2026-09-30 late)

State: #1058 + #1060 merged into `sentiment-enhancement` @ `7b62243c`. Nothing in `development`/prod. Local checkout on `sentiment-enhancement`. Day 1 of the 3-day side-by-side = run `2026-09-30T15:35:58Z`.

**Tomorrow (2026-10-01)**
- [ ] ~15:35 UTC: day-2 run from `apps/context`: `npx ts-node -r tsconfig-paths/register --transpile-only test/sentiment-playground/run.ts` (~$0.8; prints real OpenRouter cost)
- [ ] read `out/errors.jsonl` + `out/requests.jsonl` → which step times out on the new variant (digest / synth 120s / extractor 60s)
- [ ] fix the timeout cause (raise the limit, cap flash reasoning, or lower concurrency, depending on what the logs show), as a PR into `sentiment-enhancement`, before day 3
- [ ] after 15:36 UTC: `FROM=2026-09-30T15:35 npx ts-node … score.ts`. Read the **paired** rows. Expect only PLTR/BTC/ZEC to differ, so a direction, not a verdict

**This week**
- [ ] day-3 run, same time, same models. Then +3d/+7d scores over the following days
- [ ] build the per-asset side-by-side read (old call + notes vs new call + notes + market data) for the senior's review. Offered, not built
- [ ] fix the synth retry vs OpenRouter response cache (bypass cache on retry). Pre-existing, affects prod
- [ ] decide: day-2/3 runs manual vs a scheduled task + separate worktree
- [ ] decide: phase 2 as a third arm (`old` / `+P1` / `+P1+P2`) vs after phase 1's window. The third arm would change grilling round-5 decision 1, so it's my call
- [ ] check whether new's bullish lean (12/2 on day 1) is bias: compare its notes against its own market data on the flipped assets

**Before `sentiment-enhancement` → `development`**
- [ ] senior review of #1060 (not flag-gated: flash + luna also run category sentiment + basket-category fallback) and a check of those two paths
- [ ] `COINGECKO_API_KEY` in the context env (Railway + Hetzner)
- [ ] senior reviews the 3-day side-by-side before `SENTIMENT_MARKET_DATA_ENABLED` goes on
- [ ] merge `development` into `sentiment-enhancement`

**Separate / housekeeping**
- [ ] webNotes cut mid-sentence at 500 chars (`cleanSentimentOutput`), pre-existing, all consumers. Own issue/PR
- [ ] news dedupe misses reworded duplicates. Upgrade only if it matters in reads
- [ ] local cleanup: `git branch -D local/combined`; restore the ~23 CRLF→LF-only modified files in `apps/context`

## Update (2026-10-01): +1d score on day 1 (run `2026-09-30T15:35:58Z`, scored 15:37 UTC)

- Paired (14 assets): **new 4/14 (29%), old 3/14 (21%)**. Avg signed relative return: new -1.40pp, old -0.91pp. Unpaired: new 4/14, old 5/17.
- As expected, only the 3 flips differ: BTC +0.3% (new hit), PLTR +0.0% (new hit, effectively flat), ZEC -3.7% (old hit). New gets the edge on hit count; old does better on magnitude because ZEC was the big move.
- Low hit rates on both sides: both lean bullish, and most assets fell vs benchmark over the day (NEAR -8.0, ENA -5.5, PUMP -3.8). One day, one regime, so this is a direction, not a verdict.
- Still not done as of 15:37 UTC: day-2 run, timeout diagnosis from `errors.jsonl`.

## Update (2026-10-01): day 2 of the 3-day side-by-side (run `2026-10-01T15:47:34Z`)

- Ran 15:47:30–15:49:56 UTC from `sentiment-enhancement` @ `7b62243c`, same model set. Real OpenRouter cost **$0.29** (estimate was ~$0.8).
- Timeouts: **new 2/17 (NVDA, TSLA), old 2/17 (MSTR, META)**. Not new-only today.
- Cause from `requests.jsonl` (aborted rows have no provider):
  - 3 of 4 = **extractor `gpt-oss-20b` hangs to its 60s limit** (status 200, no body). Shared by both variants. Extractor rows don't log `asset`, so the mapping to NVDA/MSTR/META is from `errors.jsonl` elapsed times (74–87s = synth + 60s), inferred not logged.
  - 1 = **synth `deepseek-v4.1-flash` hit 120s** (TSLA, new). Flash on provider OpenInference is slow even when it finishes (75–97s vs p50 ~5–10s elsewhere).
- Possible fixes (not done): shorter extractor timeout + one retry; ignore OpenInference for flash; add `asset` to extractor request logs.
- Calls that differ (both-ok): PLTR new bearish / old bullish; ZEC new bullish / old bearish; HYPE new bullish / old mixed; LIT new bearish / old mixed. New is again mostly bullish (13 bullish / 2 bearish).

## Update (2026-10-02): baselines + prompt-enhancement arm (v2), first run `2026-10-02T07:11:42Z`

- **Why v2:** the day-1/2 calls showed new = 25 bullish / 4 bearish / 0 neutral-mixed, bullish on 9 of 13 day-2 calls with a negative 7d return (ZEC -10.7pp vs BTC, AMZN near its 30d low). In the prompt: no scored target (horizon/benchmark undefined, "structural trend" invited), 4 separate "be decisive / mixed is last resort" pushes, a bullish example that says "dip ... noise against the flows", and contradictory length rules (500 vs 400 chars; 2–4 vs 2–3 bullets). 57/61 new webNotes were cut mid-sentence at 500 chars, which drops the "decisive risk" bullet (written last). Diagnosis = mine from logs + prompt read; the causal link (example/decisiveness → bullish lean) is a hypothesis.
- **Branch:** `feat/sentiment-prompt-enhancement` off `sentiment-enhancement` @ `7b62243c`, commit `e6a69a17`, **local, not pushed**. Worktree `../agent-pear-v2-sentiment-prompt` (own `apps/context/.env`; `out/*.jsonl` copied there, so future runs append in the worktree, not the main checkout).
  - v2 = new data layer + `SENTIMENT_AGENT_SYSTEM_PROMPT_V2`: call = next-7d return vs benchmark; neutral is legitimate; start from MARKET DATA, a call against a big relative move must name what turns it in 7d; tweet labels are per tweet; notes price → risk → driver, 400 chars; no examples. Switched on by `SentimentContext.promptV2` (playground only), prod prompts byte-identical.
  - run.ts runs `old` / `new` / `v2` in one run (51 calls). score.ts adds `base:momentum` (sign of 7d vs benchmark from logged marketData, no look-ahead) and `base:always-bull`; "paired" = every variant of that run made a call.
- **Baselines on day 1 (+1d, paired 14):** new 29% / -1.40pp, always-bull 29% / -1.19pp, old 21% / -0.91pp, momentum 21% / -1.31pp. New doesn't beat always-bull on day 1 (n=14, one day).
- **v2 run `2026-10-02T07:11:42Z`** (07:11 UTC, off the ~15:35 slot, so an extra run, not day 3). $0.44.
  - Calls: old 11 bull / 2 bear (4 timeouts); new 12 / 2 / 1 mixed (2 timeouts); **v2 9 / 3 / 4 neutral** (1 timeout).
  - Works as intended on: ZEC v2 bearish leading with "-11.4% over 7d, underperforming BTC by 13.6pp"; AMZN/ETH/META/GOOGL neutral ("nothing in the tape is forcing a direction"); risk bullet now 2nd, survives the cut.
  - Not fixed: 15/16 v2 webNotes still hit the 500-char cut (400 instruction ignored; now the driver bullet is cut). Still bullish against negative 7d relative on ENA (-10pp), XRP (-5), SOL (-2.7), NEAR (-1.6). ETH note has garbled numbers ("USD 55372.37m-equivalent daily" ETF outflows, "300d-range equivalent month band"); one case, nothing checks numbers.
  - Timeouts 7/51 (14%): 4 extractor `gpt-oss-20b` at 60s, 3 synth flash at 120s (inferred from `errors.jsonl` elapsed). Same causes as day 2, now at 51 concurrent.
- **Not known yet:** whether v2 is more accurate. +1d score due after ~07:12 UTC 2026-10-03 (`FROM=2026-10-02T07:11`).

### Next steps (as of 2026-10-02)
- [ ] fix timeouts before the day-3 run (extractor shorter timeout + 1 retry, or lower concurrency)
- [ ] length: cut notes at a sentence boundary in `cleanSentimentOutput` (code, all variants), the prompt can't enforce it
- [ ] day-3 run ~15:35 UTC with all three arms; day-2 +1d score after 15:47 UTC
- [ ] +1d score on the v2 run after 07:12 UTC tomorrow; judge v2 only on paired rows vs `base:momentum` / `base:always-bull`
- [ ] not done: conviction field (schema change through the DB insert), reject garbage extractor output (SOL `"..."` on day 1)

## Update (2026-10-04): scorer re-run over all runs (old / new / v2), read-only from the v2 worktree

- Ran `score.ts` in `../agent-pear-v2-sentiment-prompt` (branch `feat/sentiment-prompt-enhancement`, `e6a69a17`) against its `out/calls.jsonl`. Free (Hyperliquid only), nothing written to prod.
- **All runs, +3d** (`all` scope): new 30% / -1.48pp (n=46), old 36% / -1.42pp (n=55), `base:momentum` 39% / -0.56pp, `base:always-bull` 30% / -2.32pp. **+1d**: new 33% / -0.90pp, old 38% / -0.97pp, momentum 38% / -0.52pp, v2 25% / -0.97pp (n=12, one run, 4 neutral not scored).
- Neither old nor new beats `base:momentum` at +1d or +3d. v2 has only the 2026-10-02 07:11 run at +1d; no +3d yet (due after 2026-10-05 07:11 UTC). Small n, one regime: a direction, not a verdict.
- Per-asset old/new/v2 calls for the 09-30, 10-01 and 10-02 runs were read from the same output; the 10-01 run's +3d is still pending.

## Update (2026-10-05): scorer re-run, v2 +3d now available (read-only, v2 worktree `e6a69a17`)

- Same command as the 10-04 update (`score.ts` from `../agent-pear-v2-sentiment-prompt/apps/context`), free. `calls.jsonl` has runs 09-30 (several, last `15:35:58Z`), `2026-10-01T15:47:34Z`, `2026-10-02T07:11:42Z`. **No day-3 run and no run after 10-02 exists in the log.**
- **All runs, +3d:** base:momentum 47% / +0.09pp (n=77), v2 50% / +0.61pp (n=12), old 41% / -0.78pp (n=81), new 37% / -0.62pp (n=75), base:always-bull 35% / -1.40pp. **+1d:** momentum 38% / -0.52pp, old 38% / -0.97pp, new 33% / -0.90pp, v2 25% / -0.97pp.
- v2 +3d (paired 7, one run): 57% / +1.03pp vs momentum 45% / +0.54pp, new 50% / +0.63pp, old 55% / +0.28pp on the same run. n=7 to 12, one run: not a verdict. The 10-01 run's +3d is only partly matured and the 10-02 run's +7d is pending.

## Update (2026-10-05, later): second v2 sample, run `2026-10-05T09:01:02Z`

- Three-arm run from the v2 worktree (`e6a69a17`), $0.4505, 153 OpenRouter requests, 2 errors (both timeouts: ENA, HYPE). Calls: old 13 bull / 4 bear; new 14 bull / 2 bear; **v2 5 bull / 2 bear / 8 neutral / 1 mixed**. Not scored yet: +1d due after 07:00 UTC 10-06 (`FROM=2026-10-05T09:01`), +3d 10-08, +7d 10-12.

## Plan (2026-10-05): enhancement catalog and run order, Step 0 = measurement fixes

- Decision: v2 (new data + `SENTIMENT_AGENT_SYSTEM_PROMPT_V2`) is the direction to iterate on. Evidence is thin (v2: 12 scored calls on one run; second v2 run `2026-10-05T09:01Z` unscored), so each enhancement is tried as its own arm and kept only if it helps. Catalog added to story [#1052](https://github.com/pear-protocol/agent-pear-v2/issues/1052) (section "iteration plan: enhancement catalog", items A1–A6 inputs, B1–B5 output, C1–C3 reasoning, D1–D5 measurement).
- Verified facts behind it: tweet intake is identical in old/new/v2 (24h, ≤10, `twitter.context.service.ts` has no diff vs `5676208`); `old` already runs on the new synth (`deepseek-v4.1-flash`) and digest (`gpt-6-luna`) models because #1060's model change isn't flag-gated, so old-vs-new isolates data + prompt, not prod.
- Run order (my recommendation, accepted for work): **Step 0** measurement fixes with no change to calls: D2 timeouts (~14% on 10-02), D3 synth retry vs OpenRouter cache, B4 `webNotes` cut at 500 chars, D5 `FROM` upper bound in `score.ts`, then re-baseline old/new/v2 and run one arm twice to measure run-to-run noise. **Step 1** A2 tweet matching (+A4 per-asset tweet counts), then A3 wider window. **Step 2** A1a HL funding/OI, then A1b Coinglass L/S + liquidations. **Step 3** B1 confidence field (playground-only first), B3 number check. **Step 4** C1 agentic vs the best fixed-input arm; A5/A6 vendor decision after; C3 later.
- Reading rules: stack on current best (best vs best+X), same assets and same run, ≥3 runs before keep/drop, judge on paired hit rate + avg relative return vs `base:momentum`, track the neutral rate, keep one frozen control arm.
- Branch for Step 0: `feat/sentiment-measurement`, off `feat/sentiment-prompt-enhancement` (`e6a69a17`, local only), own worktree `../agent-pear-v2-sentiment-measure`. Status: starting.

## Update (2026-10-05, evening): Step 0 code done on `feat/sentiment-measurement` (local commit `15c674e3`, not pushed) *(superseded: pushed and merged as #1080, see "night" update below)*

- Worktree `../agent-pear-v2-sentiment-measure`, off `feat/sentiment-prompt-enhancement` (`e6a69a17`). Verified: 107/107 context sentiment specs (8 new), `tsc --noEmit` clean in `apps/context`.
- D2: synth and extractor timeouts now per attempt (fresh `AbortSignal`), a timeout is retried (synth 3, extractor 2). Before, one signal spanned the SDK's `maxRetries`. D3: retries send `X-OpenRouter-Cache: false`. **Unverified live:** whether OpenRouter honours it and per-call headers override the provider-level one; check that a retry shows non-zero cost in `requests.jsonl`. B4: `truncateNotes` drops trailing bullets, then sentence, then hard cut. D5: `score.ts` `TO` bound.
- Not done: re-baseline old/new/v2 and the same-arm-twice noise check (needs `apps/context/.env` in the new worktree and OpenRouter spend).

## Update (2026-10-05, evening): baseline run on the fixed pipeline, `2026-10-05T09:56:45Z`

- Run from `feat/sentiment-measurement` (`15c674e3`). $0.4391, 158 requests (all HTTP 200), **0 failed calls** (earlier runs: 2/51 on 10-05 morning, 7/51 on 10-02). 5 extractor timeouts hit (attempt 1/2) and all recovered on the retry; none of the retries came back fast at $0, so the cache bypass appears to work (indirect evidence).
- Calls (17 assets/arm): old 12 bull / 4 bear / 1 mixed; new 14 / 3; **v2 5 bull / 3 bear / 9 neutral**. v2 stays heavily neutral (9/17 here, 8/16 on the morning run, 4/16 on 10-02). Unscored; +1d due after 07:00 UTC 10-06 (`FROM=2026-10-05T09:56`), note the morning run `09:01` is the other 10-05 sample.

## Update (2026-10-05, night): state after the day's work (read this first)

- **Merged into `sentiment-enhancement`:** #1080 (v2 arm + Step 0 fixes, `934a8441`) and #1083 (A2 strict tweets, `b4ba1122`; verified `4438db58` is an ancestor). The "Step 0 ... local commit, not pushed" and "starting" lines above are superseded. #1081 was merged into the intermediate branch `feat/sentiment-measurement` and never reached `sentiment-enhancement`; #1083 replaced it. Nothing is in `development`.
- **Arms in the playground** (`ARMS`, default `old,new,v2,v2s`): `old` = flag off (already on the #1060 models); `new` = phase 1 market data + CoinGecko news + events-only digest; `v2` = `new` + `SENTIMENT_AGENT_SYSTEM_PROMPT_V2`; `v2s` = `v2` + strict tweets (`SENTIMENT_STRICT_TWEETS`).
- **What the data says so far:** v2's prompt is the only change that moved calls (most bullish calls became neutral). `old`→`new` changed about 3 calls. `v2`→`v2s` showed no visible effect (4 of 6 flips had identical tweets). v2's neutral share varies a lot on the same code (4/16, 8/16, 9/17, 13/17, 11/17), so single-run differences are noise-sized. Neither old nor new beats `base:momentum` (+1d, +3d).
- **Tweets:** unchanged window (24h, fetch limit 10, the model sees the first 8 at 200 chars each). A3 (wider window) is deprioritised: stocks have 0 tweets regardless, and tweets barely move calls. A2's matcher went through two versions; the dropped-tweet read is the evidence for the final rule (see [[5th Oct 2026]]).
- **D1 finding:** 3 back-to-back repeat runs cost about $0.69 and only one is independent: runs 2 and 3 replayed OpenRouter's 5 min cache. Do not score runs `2026-10-05T10:29` and `10:31`. Need `X-OpenRouter-Cache: false` on all playground requests before repeating (about $1.8 for 3 real repeats).
- **A1 prep:** HL has no OI history (funding history yes); A1a = funding now + 7d avg, premium, OI notional, OI/volume; OI change needs Coinglass (HIP-3 coverage unknown) or our own snapshots.
- **Next, in order:** cache-bypassed repeat runs (D1) → A1a → A1b → B1 confidence field (playground-only) → B3 number check → C1 agentic vs best fixed-input arm; A5/A6 vendor decision after C1. The catalog with checkboxes lives in #1052.

## Update (2026-10-06): cache bypass verified, repeats running, arm decision

- **Arms from here:** `v2` dropped. `v2s` = phase-1 data + V2 prompt + strict tweets (the only difference from `v2` is strict tweets, which showed no visible effect). Running `old`, `new`, `v2s` so data layer and prompt stay separable. Future arms stack on `v2s` (`v2s+A1a`, then `+A1b`). `old` stays the frozen control (today's pipeline on the #1060 models). "Better prompt" is a direction picked on 10-05, not a measured win: v2's +3d is 50% / +0.61pp on n=12 vs momentum 47% / +0.09pp, one run.
- **Cache bypass (D1/D3):** `X-OpenRouter-Cache: false` now goes on every request from the playground fetch wrapper in `run.ts` (before it was only on synth/extractor retries in `agent.ts`). Local edit in `../agent-pear-v2-sentiment-measure`, unstaged, no PR. Verified with two 2-asset four-arm runs (`2026-10-06T04:17`, `04:18`, $0.069 / $0.071): 0 of 24 requests zero-cost in each, calls differed between the two. Cost-based check only. Exclude those two runs when scoring.
- **Valid 10-05 runs:** `09:01`, `09:56`, `10:21` (v2/v2s only), `10:27`. `10:29` and `10:31` are cache replays (69 of 203 zero-cost each). Score with `FROM=2026-10-05T09:01 TO=2026-10-05T10:29`. All of these share one day's price path, so more runs add LLM noise samples, not independent accuracy samples.
- **Repeats started 04:27 UTC:** 3 runs of `old,new,v2s`, 10 min apart, ledger in `out/repeat-ledger.log`. Score each runId separately, never pool as n x3. Results not in when this was written.
- **Code facts found while checking the next steps:** `market-data.service.ts` only uses HL `candleSnapshot`, so A1a (funding now + 7d avg, premium, OI size, OI vs volume) is all new code. Coinglass is already in `apps/context` (`oi.cron.ts` uses `oiOhlcHistory` at 1h, batches of 10 with a 10s gap), so 1d/7d OI change for crypto belongs in A1b even though HL has no OI history. B1 (confidence) shares `SentimentAgentResponseSchema` with prod, so it must be playground-gated. With about 17 calls per run, splitting by confidence leaves small buckets. C1 needs a stated rule for "best fixed-input arm" first.
- **Reminders:** one-time scheduled tasks run the scorer read-only and save `out/score-<label>.txt`. 10-05 runs: +1d 6 Oct 1:00 PM, +3d 8 Oct 4:00 PM, +7d 12 Oct 4:00 PM (local). Repeats (`FROM=2026-10-06T04:20 TO=2026-10-06T06:00`): +1d 7 Oct 12:30 PM, +3d 9 Oct 12:30 PM, +7d 13 Oct 12:30 PM.
- **GitHub:** #1052 catalog ticked for A2, B4, D2, D3, D5 and a status comment posted ([comment](https://github.com/pear-protocol/agent-pear-v2/issues/1052#issuecomment-6009358319)). See [[6th Oct 2026]].
- **Next, in order (unchanged):** noise read from the repeats, score 10-05 runs, A1a, A1b, B1, B3, C1.

## Update (2026-10-06, later): noise read on 4 cache-bypassed repeats

- Runs `04:23` (an unplanned extra, the stopped task's process kept going and finished), `04:27`, `04:41`, `04:53` UTC, arms `old,new,v2s`, daily 17, 0 errors. $0.44 per run, $1.78 total. Full numbers in [[6th Oct 2026]] and in the #1052 [comment](https://github.com/pear-protocol/agent-pear-v2/issues/1052#issuecomment-6010171251).
- **Noise (same inputs, 4 runs):** `old` 76% pairwise agreement, 7 of 17 assets flipped bull<->bear. `new` 92%, 2 flipped, but bullish on 15 to 17 of 17 every run. `v2s` 91%, 0 direction flips, neutral count 9 to 12 of 17.
- **Reading:** V2 prompt is a real effect on calls (new vs v2s differ on 10 to 12 of 17). Data layer alone: `new` differs from `old` on 2 to 5 of 17, inside old's own noise, but `new` is bearish on 2 calls over 4 runs against old's 14, so it leans bullish. This revises the earlier "no clear effect" reading. Cause untested. Noise band to judge later arms against: `v2s` neutral count 9 to 12 of 17, and no flips on assets stable here.
- **Not shown:** accuracy. One day's inputs and price path. The 4 runs are scored by the scheduled tasks (`FROM=2026-10-06T04:20 TO=2026-10-06T06:00`), per runId, never pooled.
- **Catalog ticks in #1052:** A2, B4, D2, D3, D5 (morning), A4 (logging only, twitterapi.io decision still open). D1 stays open, repeats on different days still needed.
- **Ideas beyond data ingestion (from #1043, planning only):** structured single call (drop the extractor), sample N + majority, momentum-anchored variant, un-hold H1, within-category long/short spread in the scorer, M1 model arm, category as code aggregate, check whether baskets follow sentiment, agentic last. Order and reasoning in [[6th Oct 2026]]. Nothing built.

## Update (2026-10-06, later): v2s vs old in plain words

Same runs only (`10-05T10:27` + the four 10-06 repeats, post Step 0, 85 calls per arm). 4 of the 5 runs share one day's inputs, so it's not 85 independent samples. Counted from `calls.jsonl` with the scorer's own 7d-vs-benchmark number.

**what got better (confirmed from the logs)**
- it stops fighting the trend. when a coin was down vs its benchmark over 7d (49 cases), old still said bullish on 33 of them (67%). v2s did on 5 (10%). new (data only, old prompt) was worst, 44 of 49 (90%)
- when it makes a call it mostly follows the tape. v2s agrees with the 7d move on 28 of 33 direction calls (85%). old 48 of 84 (57%)
- steadier. same inputs over 4 runs, old changed direction on 7 of 17 assets, v2s on 0
- it holds back when there's nothing. v2s neutral/mixed on 52 of 85 (61%), old 1 of 85

**what didn't change**
- notes cut at the cap: 2 vs 3 of 85, same. that was the Step 0 fix and applies to every arm
- accuracy is not shown. v2s isn't scored yet (+1d after 07:00 UTC 10-06). agreeing with the trend is by design (the prompt says start from the market data), so it's not proof the calls are right. nothing has beaten `base:momentum` yet

**risk**
- v2s gives a direction on only 33 of 85 (39%), old on 84. baskets read these as long/short lean, so less direction to work with. not checked how the basket agent treats neutral
- it's close to a momentum follower now. if momentum is the whole answer, code can do that for free (idea 3 in [[6th Oct 2026]])

**where i land (opinion):** v2s is the more cautious and more stable version. fine to use it as the base for the next arms, imo. i would not say it's more accurate until the +1d/+3d/+7d scores are in. "move forward with v2s" is a leaning, not a decision, so not recorded as one.

## Update (2026-10-09): +3d scores in, replay logging PR raised, A1 branch cut

- **+3d on the 10-06 repeats and 10-05 runs:** `v2s` is last at +3d on both price paths (repeats 11% / -3.93pp on 27 calls; 10-05 runs 36% / -0.06pp on 14). Momentum stays the best arm (47%). On the 27 assets `v2s` called, every one underperformed its benchmark (always-bull 0 of 27), and `old` scored 4 hits / -3.18pp on the same set, so this is not shown to be worse than `old`, and not shown to be better. The earlier "build on `v2s`" leaning is not supported at +3d; still a leaning, not a decision. Numbers and caveats (4 runs share one price path; the per-run table at the bottom of the score file disagrees with the pooled one, cause not checked): [[9th Oct 2026]].
- **Replay logging, PR [#1095](https://github.com/pear-protocol/agent-pear-v2/pull/1095) into `sentiment-enhancement` (open, not merged):** branch `feat/sentiment-call-logging`, commit `194bba51`, pushed with `--no-verify` on 9 Oct (the Windows pre-push hook issue). `run.ts` now writes every OpenRouter request body to `out/prompts.jsonl` (the digest request carries the raw Exa results), adds an `inputs` field (digest, tokenomics, full tweets) to `calls.jsonl`, tags `requests.jsonl` rows with arm and asset, and commits the cache bypass that had been a local edit. `tsc` clean; **not run live yet.** Runs before this have no digest or prompt text and can't be backfilled, so replay only works from the next run on.
- **A1a / A1b branch:** `feat/sentiment-positioning`, cut from `feat/sentiment-call-logging` (`194bba51`) in the `agent-pear-v2-sentiment-measure` worktree, so the arm can be replayed on stored inputs. Local only, no commits yet. Stacked on #1095: rebase onto `sentiment-enhancement` once that merges.
  - A1a: HL funding now + 7d avg, premium, OI size, OI vs volume, as an arm on `v2s`. HL funding history is point-in-time; premium and OI have no history, so those two can't be replayed for past days.
  - A1b: Coinglass L/S ratio + liquidations (crypto, daily set) and 1d/7d OI change. Coinglass HIP-3 coverage is still unknown.
- **Next:** a different-day repeat run with the new logging on, a small replay script reading `inputs`, then A1a. +7d scores: 10-05 runs 12 Oct, repeats 13 Oct.
