# Elfa Replacement — What Agent Pear Actually Needs

- Status: starting point for the Elfa replacement work. Investigation only, no code changed.
- Date: 2026-10-24
- Full write-up (Notion, private draft): [Elfa: what Agent Pear actually needs](https://app.notion.com/p/4034641ea90c81a7b010d9fed692909f)
- Related: [[Elfa Endpoints and Plan Gating]] · [[Decisions/Elfa Webhook Trust Model]] · [[Harness Integration/Step 6 — LLM Usage Classification (all background jobs + Elfa workflow)]] · [[Pear sentiment basket and pairs plan/Sentiment Data Ingestion — Improvement Plan]]
- Related Notion: [Elfa API: what we use in prod](https://app.notion.com/p/4014641ea90c814091e5c5902fee7a82) · [Elfa notifications today](https://app.notion.com/p/4024641ea90c81ce83e8d7fec6b48445) · [Elfa triggers: why alerts expire](https://app.notion.com/p/4024641ea90c819c816dda32c0ca1a7e) · [Elfa v3 and what it means for us](https://app.notion.com/p/3fb4641ea90c81d1b81fd3ad22752e43) · [Elfa AI 1-pager](https://app.notion.com/p/3a44641ea90c814ebb05ea6c859feaae)

## Why this exists

Business paused Elfa on 23 Oct until a call, and asked to split "things we can replicate" from "things on users' click rate". There was also talk that Elfa is "95% marketing, 5% product" and that Fortuna might come next. This note records what Elfa actually does for us, what users respond to, and what we can own ourselves.

## Main finding

Elfa is mostly a hosted condition checker for us, not a data source.

Observed, prod agent DB, 30 days to 24 Oct, Elfa trigger alerts only:

| condition class | delivered | tapped | tap rate | users who tapped | who else can check it |
|---|---:|---:|---:|---:|---|
| market data (funding, liquidations, price shocks, Bollinger, fear & greed) | 446 | 36 | 8.1% | 17 | Pear already (`SignalMeasurementService`) |
| Kalshi repricing | 20 | 4 | 20% | 4 | Pear already (`measureKalshi`) |
| SEC 8-K (Strategy) | 78 | 10 | 12.8% | 6 | public EDGAR |
| news.semantic | 71 | 15 | 21.1% | 9 | only truly Elfa data |
| tweet.semantic on named accounts | 0 | 0 | — | 0 | — |
| personal triggers | 0 | 0 | — | 0 | — |

- 50 of 65 taps (77%) need no social data.
- news.semantic is small but has the best tap rate, so the social part is not worthless.
- Credits go the other way: most of ~10.4k/day went to the 60 standing news.semantic queries.
- 5 users give 67% of taps. Feed pings (s2): 691 delivered, 48 taps, 21 users, 80% BTC/ETH/HYPE.
- Class is assigned by hand from `signal-book.config.ts`. Small counts, treat as a lead.

## Five Elfa surfaces (verified in code)

1. Signal alerts: trigger-manager arms Auto queries → webhook `/elfa/events` → `signal.processor.ts` → dispatch → touch governor → Telegram buttons.
2. Global feed: `global-feed-pull.processor.ts` (Hetzner, 2x daily) → `GlobalFeedItem` → s2 feed, s4 follow-ups (token_news only), s5 brief, watch-scheduler.
3. Chat tool `get_social_pulse` (research, asset-sentiment, twitter findings, advanced). Returns links + engagement counts only, no tweet text, except `event_summary`. Key-gated, so the pause removed it cleanly.
4. Twitter app scheduled posts: trending narratives in the context.
5. Signal evidence: 5-credit event-summary per alert.

No Elfa link to Ask / Pick / Grow found in this repo. `apps/context` has no Elfa code; grep hits there are `ModelFactory` false positives.

## How the pause was done (verified 24 Oct)

- Railway: `ap-api` and `ap-worker-prod` redeployed 23 Oct 17:41 and 17:51 UTC on the same commit `d36a85ae` (a variable change).
- Both services now have `ELFA_API_KEY_PAUSED` and no `ELFA_API_KEY` (names only checked).
- ap-api logs hourly: `ELFA_API_KEY is unset — cannot arm Elfa queries; skipping reconcile`. Nothing else errors.
- Elfa `/v2/key-status`: `ap-prod` still `active`, enterprise, billing arrears, usage today 0, month 233,461.
- DB: last webhook 23 Oct 10:48 UTC, last s1 alert 09:01, last feed pull 00:50. After: 0 s1, s2 down from ~20/day to 1, brief starves, s3 position check-ins unaffected. *(times superseded — see Correction below)*

## Correction — 2026-10-24 (Hetzner checked, DB times were shifted 5:30)

- The DB times above were read through a local client in IST from `timestamp without time zone` columns, so they are 5:30 early. Corrected: last webhook 16:18 UTC, last s1 alert 14:31 UTC, last feed pull 06:20 UTC (the box journal confirms the 06:20 run). The day buckets were UTC days, not IST. The "7 hour gap before the rename" was an artefact of the shift.
- Hetzner (`prakh@46.224.111.239`, box in UTC): `/home/prakh/pear/.env.agent-global-feed-pull` was edited 23 Oct 17:42:58 UTC. `ELFA_API_KEY` is now empty and `ELFA_API_KEY_PAUSED` is filled (checked empty or not only, no values read). `agent-global-feed-pull.timer` is disabled and inactive, so the 18:20 UTC run never happened. No other Hetzner env file has an Elfa key.
- So the pause was all done 23 Oct 17:41–17:43 UTC, on Railway and Hetzner together.
- Still open: no webhook arrived after the rename, though we never cancelled the Elfa queries (reconcile only skips). Either they stopped firing or Elfa's side was paused too.

## Update — 2026-10-24 (who, and what the pause stopped)

- Who: Railway API shows the 17:40 and 17:51 UTC redeploys on ap-api were created by a team member's Railway account (deploy `creator`), `reason: redeploy`. No commit, PR or Actions run. Hetzner edit was a non-interactive SSH session (no history entry); keys on the box are `prakh@BetterHalf` plus two `github-actions-hetzner-deploy` keys, no Actions run in the window; sshd/wtmp not readable as prakh.
- Stopped (24h before → after): s1 alerts 90 → 0, feed pull timer disabled (0 live feed rows), s2 26 → 1, s5 18 → 1, chat social tool gone, trigger reconcile skipped hourly.
- Drifting: 90 triggers still `armed` in DB, 12 Elfa queries expire by 31 Oct; watch scheduler expired 126 entries; 495 new follow-up hooks, none fired from feed; distiller/ghost-writer/ghost-synthesis/x-digest still spend LLM calls for nothing.
- Not affected: s3 position check-ins, taps on old alerts (last 20:59 UTC), twitter-agent prod has no Elfa vars at all (narratives were already off). Nothing crashed; all Hetzner jobs exit 0.
- Resume checklist: refill `ELFA_API_KEY` on Railway + Hetzner (emptied, not removed), re-enable `agent-global-feed-pull.timer`, watch the first reconcile (stale state).

## Recommendation (first pass)

- BUILD: market data, Kalshi and SEC triggers in house. Start from the parked `ghost-shadow.processor.ts` (already uses `SignalMeasurementService` + `evaluateDeterministicPredicate`), keep crossing state + cooldowns in Redis, enqueue the same job shape the webhook enqueues so everything downstream stays unchanged.
- EXPERIMENT FIRST: news.semantic replacement. Options: Elfa v3 entity-filtered `events` (0-credit filter), twitterapi.io stream of 30–80 curated news accounts + cheap LLM, or a headlines API. Prefilter by entity before the LLM check.
- USE EXISTING PEAR DATA: feed token-news and the chat tool (internal twitter tools over 42 accounts + web research).
- REMOVE: personal triggers, named-account tweet watches (0 delivered), likely trending tokens/narratives.
- KEEP EXTERNAL only if a feature needs it: smart-account weighting, caller track records, mindshare, history. We never wired these in, and they are Elfa's real moat.

## Senior's twitterapi.io proposal (checked)

- Prices hold: twitterapi.io stream $299/month for 150 accounts, $149 for 50, WebSocket or webhook. Elfa Grow's 100 active plan cap is real.
- But the cap only bites because market data/Kalshi/SEC conditions use plan slots. In house, we need ~10–20 semantic queries. We already pool (one common query fans out to all users).
- A 150-account stream replaces the named-account class (0 alerts in 30 days), and news.semantic only if the accounts are curated news sources. It does not give smart accounts, caller records or mindshare.
- Verdict: worth doing as step 2, scoped to news accounts.

## Measurement gaps (fix before deciding on value)

1. Tap → ticket → trade is unmeasurable since `c65e3fde` (24 Sep). Stamp the notification id on the ticket and the trade.
2. No holdout. A random ~10% per alert class is needed to measure retention lift.
3. Chat tool usage unknown: Langfuse keys from the skill were rejected on 24 Oct.
4. 111 s1 delivery failures on 18 users, cause unknown.

## Open questions

- Why webhooks stopped at 10:48 UTC, ~7h before the env rename (Elfa-side pause? not checked). *(resolved: time was shifted, see Correction)*
- Whether the Hetzner global-feed-pull env still has the key (the 12:50 UTC run on 23 Oct wrote no rows). *(resolved: emptied and timer disabled, see Correction)*
- Contract cost per month, co-marketing obligations, and which metric decides survival. These are business questions.

## Update — 2026-10-28 (Step 1 done: trigger mechanics, see [[Elfa Triggers, Three Classes (Step 1)]])

- Resolves the open question above about why no webhook arrived after the rename: probed Elfa with one read-only GET per armed query id. 88 cancelled, 1 expired, 1 triggered, none live. We never call cancel, so the "90 still armed" drift note is DB state only. Who cancelled and when is still unknown.
- New for the BUILD side: 202 of 595 math fires (34%) never passed Pear's own re-measure (eth-bollinger 71%, btc-liquidation 57%, eth-funding 56%), and only 25% of math fires reached a user. Reason is only in logs. Check this before copying the math triggers as they are.
- New for the twitter row: the 0 delivered is a Pear drop (`dropped_no_legs` on all 64 fires, `handle:` topics have no legs), so "REMOVE named-account tweet watches" above rests on a bug, not on value. Same drop hits 412 of 1,379 news fires.
- Credit note: most of the ~10.4k/day is Elfa checking every post against the standing news queries, the 5-credit event summary is small next to it.
