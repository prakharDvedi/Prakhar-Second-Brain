# Elfa Triggers, Three Classes (Step 1)

- Status: Step 1 of the Elfa replacement done. Investigation only, no code changed.
- Date: 2026-10-28
- Full write-up with every trigger listed (Notion, private draft): [Elfa triggers: the three classes](https://app.notion.com/p/4064641ea90c81ac8f42dcbaa3c21f08)
- Parent: [[Elfa Replacement — What Agent Pear Actually Needs]] (click rates, what the pause stopped). This note is the mechanics: what we sent, what came back.
- Related: [[Elfa Endpoints and Plan Gating]] · [[Decisions/Elfa Webhook Trust Model]]
- Window: 17 Aug to 23 Oct 2026 17:41 UTC (pause). Read-only queries on the agent DB, one read-only call per query id to Elfa, code on `development`.

## What was asked

Take every trigger, see what was armed, and reverse engineer how each fires from the query we send and the webhook we get back. Group into three classes: math/deterministic, narrative news semantic, twitter.

## Findings (confirmed unless marked)

- 693 `SignalTrigger` rows saved, 90 armed in the DB. By class: 14 math, 65 news, 7 twitter, 4 `events` (personal, don't fit). Mixed (4) and cron (1) are saved but not armed.
- **Elfa side: none are live.** One GET per armed query id: 88 cancelled, 1 expired, 1 triggered. We never call cancel (reconcile only skips), so the DB armed state is stale. Who cancelled and when: unknown (no cancelled-at field). 61 were last evaluated 23 Oct 16:00 to 16:59 UTC. This answers the open question in the parent note about why no webhook arrived after the rename.
- **Webhook is thin.** Body has id, type, title, body, queryId, executionId, triggerTime, conditionsMet. No value, no tweet text. Everything richer is fetched by Pear afterwards. Raw bodies are only in BullMQ (last 1000 complete and 1000 failed), not in a table.
- Query contract: AND/OR tree of leaves `{source, method, operator, value, args}`, `expiresIn`, `repeat {cooldown, maxTriggers}`, `webhookAuthVersion: signed-v1`. Title carries `topic=<topic>;` so the webhook maps back to a trigger.
- 421 distinct condition documents, 45 structural signatures (30 math, 2 news, 1 twitter, 4 mixed, 7 events, 1 cron).

### Math (14 armed)

- Elfa says "true", Pear re-measures in `SignalMeasurementService` and checks the predicate. If the fresh reading does not back it up, the job throws, retries 4 times, logs `enrichment_failed` and no notification exists.
- 202 of 595 math fires (34%) failed that way. Passed re-measure: 393 (66%). Worst: eth-bollinger-breakout 35/49 (71%), btc-liquidation-cascade 95/167 (57%), eth-funding-flip 53/94 (56%). Weekly failure rate 17% to 48%, no improving trend.
- Where all 595 ended: 202 enrichment_failed, 12 no legs, 138 nothing saved (likely empty audience, not verified), 73 notification but none dispatched, 23 dispatched not delivered, 147 delivered. So only 25% of math fires reached anyone.
- Why they failed: **unknown**. Reason is only in logs. Hypotheses, not checked: crossings move back before the job runs, Coinglass bucket differs from Elfa's for liquidations.
- Measurer limits (code): price change 24h only, bollinger period 20 / 2 sd, SEC 48h freshness with no once-per-accession state, crossings trust Elfa and only verify the landing side, standalone RSI/SMA not covered.
- Kalshi tickers armed are the September contracts: rollover is needed.

### News semantic (65 armed: 44 common, 21 personal)

- One `news.semantic == true` leaf, minConfidence 75. Only 5 text templates. 720h 4h/3 except macro-headline-fed (2h/5) and three 168h common ones (XYZ:GOLD, memecoins, ai-coins).
- After Elfa matches, Pear calls event summary (5 credits) for summary, tweet ids, source links. 3,290 of 4,320 textual notifications have a confirmed source.
- 509 fires on armed ones, BTC + ETH + the Fed query are 55%. Only 29 of 44 common and 13 of 21 personal ever fired.
- 1,379 fires, 412 dropped `dropped_no_legs` (30%), 387 of all drops on personal `narrative:` topics. Real enrichment failures for news are rare (26).
- Credits: most of the ~10.4k/day went to Elfa checking every news post against every standing `news.semantic` query (from the cost docs, not re-measured). The 5-credit event summary is small next to it (my estimate 1 to 4%).

### Twitter (7 armed: 4 common, 3 personal)

- `tweet.semantic` on one username, 720h. 64 fires, **all 64 dropped by Pear with `dropped_no_legs`** before audience selection, 0 notifications. `resolveCandidateLegs` only gives legs for book entries with legs or `asset:`/`narrative:` topics, so `handle:` topics return null.
- So the 0 delivered number measures our drop, not whether account watches are useful. 14 fires from common (vitalik 9, hyperliquid 4, tier10k 1), 50 from personal handles.
- 38 failed records: cause not stored.

### Other

- `events` source: 8 saved, 4 armed, all personal OR trees with listing/upgrade/unlock/exploit leaves. 15 fires, 4 dropped no legs, 2 failed re-measure. The evidence classifier does not treat `events` as a supported leaf.

## Corrections made along the way

- "83% agreed" was wrong. It counted cap-suppressed placeholder rows (all 2,018 `predicateSatisfied: false` notifications are empty evidence, never dispatched). Real figure above.
- "Costliest task" was pinned on the event summary. The standing news queries are the costliest part.
- Personal news triggers are 720h, not 168h.

## Next steps

1. Why math triggers failed: Railway logs for the terminal reason, and/or Elfa execution records (`GET /v2/auto/executions/{id}`) to compare Elfa's number with Pear's re-read. Not checked whether Elfa still keeps old executions.
2. Twitter pipeline: handle topics need legs or a different path, and the same fix covers the 412 dropped news fires. Then judge what the 64 fires would have been.
3. Replicate the news layer: spec is the 5 templates plus the evidence shape; need a data source (Elfa v3 events with entity filter, twitterapi.io stream, headlines) and a scoring method against the 3,290 grounded notifications. Prefilter by entity before any LLM check.
4. Open: who cancelled the Elfa queries and when; where the 138 silent math exits go; share the Notion doc (it is a private draft). A team summary message was drafted 2026-10-28, not confirmed sent.

## Update 2026-10-28: why math re-measure fails (step 2, first check)

Source: Railway `ap-worker-prod` logs, `Signal evidence unavailable` lines, across 103 deployments from 10 Aug to 25 Oct. Logs only go back to 28 Sep, so this covers 28 Sep to 23 Oct, not the full window. Langfuse only traces `signal.compose` (after re-measure passes), so it can't show failures.

- 96 terminal failures found. 83 carry inline metadata, all `predicate_condition_not_met`: btc-liquidation-cascade 37, eth-funding-flip 30, eth-bollinger-breakout 13, one each fed / crude / a personal ETC. 13 are `logger.error` lines where the metadata object isn't printed, so the reason is lost (observability gap).
- Every final measurement runs about 106 to 123 s after Elfa's trigger (attempt 4/4).
- **funding (confirmed on 12 of 30):** Pear's value matches Hyperliquid settled ETH funding to 2 decimals (3.365 vs 3.363, 3.868 vs 3.867). Elfa's value was negative while HL was positive in the hour before and the hour of the fire. Elfa is not reading HL ETH funding, so these 30 are false fires that Pear correctly rejected.
- **bollinger (strong evidence, 13 of 13):** Elfa's price is above the band computed from the last 20 CLOSED 4h candles in every case, by $0.03 to $1.50. Pear builds the band with the live partial candle included, which moves it up. Margins are tiny, so a definition mismatch plus 2 min of drift flips them.
- **liquidation (hypothesis, not verified):** Pear reads 0 in 22 of 37, and fires cluster at minute 00 to 03 of the hour. Pear takes the latest Coinglass 1h bucket (`limit: 1`), which is the just-opened hour. Elfa's 0.11 to 1.64% likely uses a trailing or closed hour. Not checked against Coinglass (no key access in the session).
- Scratch scripts and raw logs: session scratchpad, not kept.

### Update 2026-10-28 (later): liquidation checked against Coinglass

Coinglass `liquidation/history` BTC-USD Hyperliquid 1h, for all 37 logged btc-liquidation-cascade failures. % uses today's BTC OI ($3.47B) for every row, so it is rough.

- Pear reads the bucket for the hour the fire lands in. At fire time that bucket is partial or just opened, so Pear sees 0 in 22 of 37. Later, the same buckets fill to $0.6M to $16.9M.
- The previous closed hour is above 0.1% in 19 of 37, so reading the closed hour would have passed about half.
- For top-of-hour fires (minute 00 to 05, 20 fires), Elfa's % is within 30% of the previous closed hour in 11 (e.g. 0.151 vs 0.152, 0.321 vs 0.294, 0.236 vs 0.230).
- Mid-hour fires (17) don't match either bucket. That fits a trailing 60 min window spanning two buckets (assumed, not checked with 5m data).
- So the earlier hypothesis holds in part: the failures are a window mismatch, not Elfa noise. The replacement should use a trailing 60 min sum (5m buckets), not `limit: 1` on 1h.

## Update 2026-10-29: measurer fix committed, stage 1 started

- Commit `e7a82057` on local branch `investigate/elfa-step2` (not pushed, no PR). Bollinger is built from closed candles only; a 1h liquidation rule sums the trailing twelve 5m buckets. Funding untouched. Spec 19/19.
- Bollinger check on the 13 failures with the new definition: Pear's later price is above the closed band in 7 of 13. In the other 6 the price really fell back during the ~111 s re-measure delay, which an own poller removes (fire and measurement are the same moment).
- Liquidation not verified: Coinglass returns `{"code":"401","msg":"Upgrade plan"}` on every endpoint since 2026-10-28 16:27 UTC (last clean prod call 16:12 UTC). Cause unknown. Fallback if Startup has no 5m: 30m buckets (current + 2 previous).
- Webhook latency is not the problem: Elfa's webhook median is 2.7 s after fire. Pear's final measurement median is 111 s (attempt 4/4).
- Next: stage 1, own math poller from `ghost-shadow.processor.ts` in shadow mode next to Elfa, compare fires for 1 to 2 weeks before cutover. See [[29th Oct 2026]].

### Update 2026-10-29 (later): backtest of past Elfa math fires vs Hyperliquid history

Elfa is paused (queries cancelled), so there is no live Elfa to shadow against. Instead: 598 `received` events for math triggers from the agent DB (17 Aug to 23 Oct), checked against Hyperliquid public history (1h/15m/4h candles, settled hourly funding). Liquidation skipped, Coinglass is down.

Was the condition really true when Elfa fired (Pear's definitions, closed-candle Bollinger):

| trigger | condition true on HL | note |
|---|---|---|
| crude-oil-shock | 58/58 | within 1h before fire |
| gold-breakout | 16/17 | |
| nasdaq-daily-shock | 6/6 | |
| eth-bollinger-breakout | 31/31 | 15m high above the closed-candle band, from 6 Sep (15m history limit) |
| eth-funding-flip | 22/94 | settled funding < 0; confirms Elfa's funding fires are mostly false |

- Elfa's price-change, funding and liquidation fires land at minute 00 to 04 of the hour, so Elfa looks like it checks those hourly. Bollinger fires at any minute, so it is live.
- Simulated poller (false→true edge plus the book cooldown) caught 81% of crude, 70% of gold, 100% of nasdaq fires. The misses are real conditions inside our 12h cooldown. Elfa fired again 1 to 4 h later, so its effective cooldown is shorter (probably re-arms reset it, not checked).
- Bollinger at 15m close polling caught 64%. Intrabar pokes are short, so the poller runs every minute.
- Pear-side DB gotcha: node-pg reads `timestamp` columns as local time, so in IST the minutes shift by 30. Run with `TZ=UTC`.

Built `MathShadowProcessor` (uncommitted, `investigate/elfa-step2`): every minute, curated book math triggers, edge plus cooldown, log `math shadow fire topic=…` only. Liquidation leaves every 5th minute. Off unless `MATH_SHADOW_ENABLED=true`. Spec 5/5; not run live yet.

### Update 2026-10-29 (end of day): what is replicated, and where it lives

- Replicated: detection for the math class (curated book only), as code and a backtest. Not live, not delivering, liquidation unverified, personal price triggers not covered. News and twitter not started.
- Decided to keep the Elfa replacement in agent-pear-v2, not a new repo. It reuses the measurer, book, evidence evaluator and signal queue. Give it its own deploy target inside the monorepo when it needs isolating from `ap-worker`.
- Next steps list: [[29th Oct 2026#Next steps (tomorrow)]].
