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
