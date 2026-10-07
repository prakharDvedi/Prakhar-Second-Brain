# Asset Sentiment Playground — Research & Experiments

- Status: **research done, experiments not started.** Tracked in GitHub issue [#1043](https://github.com/pear-protocol/agent-pear-v2/issues/1043), which is the full, self-contained brief. This note keeps the findings and reasoning; the issue has the experiment matrix, task checklist and permalinks.
- Date: 2026-09-29
- Project: Agent Pear, `apps/context` sentiment crons (feed basket generation)
- Related: [[Better Trading]] · [[Harness Integration/Step 2 — Claude Basket Generation on Hetzner|Step 2 (Claude baskets, $2.18/basket)]] · [[Harness Integration/Step 7 — Claude CLI Feasibility Research for interest-distiller|Step 7 (Claude CLI spike method, web arm lost)]] · [[29th Sep 2026]] · [[00 Home/Current Focus]]

## How it started

Question: "the context DB has no sentiment rows from 09-08 to 09-26, is this true, did a Railway flag change?"

**Answer (verified):** true, but it isn't a gap and nothing broke.
- Railway `context-app` (production) deploy logs: the daily asset run fired at 12:00 UTC **every day 09-08 → 09-28**. Category ran Wed/Sun, weekly refresh Sun. No `*_SENTIMENT_CRON_DISABLED` in that window. The 09-08 17:39 IST deploy changed nothing.
- Every run logs `updated=N created=0`. `ingestAssetRun` / `ingestCategoryRun` (`packages/shared/src/services/sentiment.service.ts`) overwrite by `assetId` / `category`, creating only when no row exists. **The table holds latest-only, and has since the pipeline was created** (`3659a42a`, Prudhvi, 2025-12-27; `git log -S updateMany` finds only that commit).
- DB on 09-29: 214 asset rows / 213 assets; 15 category rows. All `runAt` from 09-27 onward except one stale asset (05-31) and one stale category (06-14).
- Hetzner journals start 09-28 (sentiment units added `6f4d989b`). Their times match the DB `runAt` exactly, so the agent `.env` context DB = the one `.env.context-production` writes.
- It's the only copy: the context app has no Langfuse, Railway logs only have counts, no exports. Railway `context-db` backups *might* have old snapshots (not checked).

**Design mismatch (inference, not documented anywhere):** the schema is shaped for history (no unique key, `runAt` indexes) and the readers (added later by OleanjiKingCode in `51ea1155`, 2026-06-11) take the latest per key as if history existed. But the writer overwrites. Both give the same "latest" answer, so nobody noticed. Ask Prudhvi if the intent matters.

## Findings on input quality

- Per asset today: gemini-2.5-flash summarises 6 Exa results → deepseek-v4-pro synthesises → gpt-oss-20b extracts. That's 3 LLM calls/asset: ~51 daily (17 assets), ~32 per category run (16 categories, no new research), ~550 on the Sunday weekly refresh (~185 stale assets).
- **Tweet pool is tiny:** 159 tweets / 25 authors in 24h from 42 watched accounts. On 09-29, **5 of 16 assets (all stocks) had 0 tweets**. The prompt still forces `twitterNotes` ("never say insufficient data").
- **No price / funding / OI / positioning data** reaches the model at all.
- `models.ts` comment: depth is capped by the prompt, not the model (every model scored 2.0–2.67 in the #229 sweep).
- **Elfa is not used by the daily cron.** Only the chat agent's `asset-sentiment` node uses it (when `ELFA_API_KEY` is set).
- I wrongly first said `72d4fc2a` (gpt-6-luna) changed the daily sentiment. It only touched `apps/agent` nodes. Corrected in-session.

**Already in the repo but unused by sentiment** *(partly wrong: Tokenomist and Polymarket ARE used, see Correction 2026-09-30 below)*: twitterapi.io advanced search (all of X), Coinglass (funding, OI, top-trader L/S, liquidations, taker, HL wallet distribution), Hyperliquid market overview + TA (covers HIP-3 stocks, free), CoinGecko Pro, Elfa, Tokenomist upcoming unlocks, Kalshi + Polymarket, Firecrawl. Biggest missing piece, which needs a new vendor: stock earnings/analyst data.

## The senior's hypothesis and my read

Senior: *"more data + stronger model + looser prompt → better analysis?"*

- More data: very likely helps. The inputs are the bottleneck.
- Stronger model / looser prompt on the **same** inputs: mostly buys nicer prose, not more correct calls. Every past A/B scored prose with an LLM judge, never accuracy vs forward returns.
- Claude CLI fits sentiment **better than it fit the interest-distiller**. The distiller restated stored data (web arm lost: tie/loss at +45% cost, +70% latency). Sentiment's problem is missing inputs, which tools (WebSearch + MCP) address.
- Estimated Claude CLI cost (*extrapolated, not measured*): ~$0.20–0.40/asset → daily ~$3.5–7; keep the ~185-asset weekly refresh off Claude (same Max-20x login as baskets at 14:00, headroom unreadable, CLI auto-updates unpinned).

## Decisions / plan (as of 2026-09-29)

1. **H1: store history.** Additive `createMany` + 3 reader fixes (distinct in `getAssetIdsByCategory`, freshness cutoff in `getLatestAssetSentimentsByAssetIds`, latest-then-filter for reclassified assets). ~330 rows/week. It's the baseline for everything else. Not yet approved/shipped.
2. **Score against forward returns** (+1/3/7d), vs naive baselines (always-bullish, 7d momentum, funding sign). Track the mixed rate.
3. **Playground** `apps/context/test/sentiment-playground/`: axes D0–D3 data × M0–M3 model (OpenRouter deepseek / stronger OpenRouter / Claude CLI no tools / Claude CLI + MCP tools) × P0/P1 prompt. Cached identical inputs; outputs never to prod tables.
4. Staged runs, ~2 weeks on 20 assets first. Full matrix in #1043.

## Communication

- Drafted a message to the senior asking whether latest-only was intentional, with permalinks (@`d9feca1`) and the benefits of history. Framed as a question ("was that intentional?") with the concrete benefit being **measuring whether sentiment predicts price**, not vague "better analysis".

## Open questions

- Was latest-only intentional? (Prudhvi)
- Positioning source for experiments: live Coinglass vs daily `AssetOi` rows?
- Hit threshold: fixed % vs volatility-scaled?
- Is H1 approved to ship to prod on its own?

## Correction (2026-09-30)

The "already in the repo but unused" list above was wrong about two sources. Verified against `sentiment-cron.service.ts` on `fix/twitter-labeling-dead-image-retry-dev`:
- **Tokenomist is used** by asset sentiment (unlocks, burns, buybacks, fundraising from the `TokenomistData` table). Added in `32021bdf` (Prudhvi, 2026-01-08).
- **Polymarket is used** for crypto assets only (skipped for HIP-3). Added in `f1ef5698` (Prudhvi, 2026-01-17).
- Tweets are not a live search: `TwitterContextService.searchForSentiment` reads the stored-tweet DB, last 24h, up to 10.
- Category sentiment does no research of its own: it summarises the latest asset sentiment rows (`runCategory` → `invokeCategorySentiment`).

Corrected table (daily asset sentiment):

| Data | Used? | Notes |
|---|---|---|
| News articles (Exa web search) | ✅ | 6 results, digested by gemini-2.5-flash |
| Tweets from watched accounts | ✅ | Up to 10 from last 24h. Stocks often get none |
| Token unlocks (Tokenomist) | ✅ | Crypto only in practice |
| Betting odds (Polymarket) | ✅ | Crypto only |
| Price / trend (Hyperliquid) | ❌ | In repo, free |
| Funding / OI / long-short (Coinglass) | ❌ | In repo |
| Full X search (twitterapi.io) | ❌ | In repo (apps/agent x-digest) |
| Elfa, CoinGecko, Kalshi | ❌ | In repo |
| Stock earnings / analysts | ❌ | Needs a new vendor |

Decision history found in code: `c90dd4d5` (#229, 2026-07-02) is the only recorded model/prompt decision; its A/B scored writing (depth/decisiveness), not accuracy. `apps/context/src/config/models.ts:21-32` says depth is capped by the prompt, not the model. No commit records a decision *not* to use price/positioning data; it was never considered.

Next: [[Sentiment Data Ingestion — Improvement Plan]].

## Update (2026-09-30, later)

Scope decision: **additive history (H1), the forward-return scorer and the experiment matrix are on hold.** Current work is only adding data sources to sentiment (price, positioning, social, stock catalysts). The "Decisions / plan" list above is kept as the plan for when history comes back. See [[Sentiment Data Ingestion — Improvement Plan]].
