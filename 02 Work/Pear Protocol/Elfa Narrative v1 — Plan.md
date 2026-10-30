---
type: plan
area: pear
status: proposed
created: 2026-10-30
---

# Elfa Narrative v1 — Plan

- Status: design only, nothing built. Proposed 30 Oct 2026, not yet agreed with the team.
- Scope: the news/narrative track of the Elfa replacement (story #1183, PR #1192 on `elfa/narrative`), and most of the twitter track (#1187, PR #1193).
- Sources: code on `epic/elfa-replacement`, the notes below, and one read-only query on the prod agent DB.
- Related: [[Elfa Triggers, Three Classes (Step 1)]] · [[Elfa Replacement — What Agent Pear Actually Needs]] · [[Elfa Replacement — One Switch, Not Math-First]] · [[Harness Integration/Step 7 — Claude CLI Feasibility Research for interest-distiller]] · [[30th Oct 2026]]

## The capability

A user describes something happening in the world in plain words, and Pear notices when it actually happens and alerts them.

## What Elfa did for us (from code and DB)

- **What we sent:** one leaf, `{source: "news" | "tweet", method: "semantic", args: {text, minConfidence: 75, username?}}`, from a few templates in `signal-book.config.ts`: `macro-headline-fed`, interest catalysts (`buildInterestSignalEntry`), personal statement watches (`buildStatementSignalEntry`, text is `"<phrase>: <statement>"`), handle watches (`buildHandleSignalEntry`), and an event leg inside mixed Builder Chat rules. 132 of 146 common and 488 of 547 personal `SignalTrigger` rows are semantic. 65 were armed at the pause.
- **What Elfa did:** all of it. It collected posts from its own stream, checked every post against every standing query with a model, and sent a webhook on a match. This was most of the ~10.4k credits a day.
- **What came back:** a thin webhook (`executionId`, `triggerTime`, `conditionsMet`, `topic=` title). No text, no source.
- **What Pear did with it** (`signal.processor.ts`): find the trigger by topic, pick a pair (`resolveCandidateLegs`, which asks **Elfa** for `narrative:` topics), then fetch evidence (`fetchEventSummary`, **Elfa** event summary, 5 credits). No evidence means a drop as `missing_textual_sources`.
- **What the code assumes:** a fire is already a confirmed match, Elfa does first reading, de-duplication and cooldown, evidence can be found afterwards by keyword, and Elfa picks the pair for narrative topics.
- **What was valuable:** matching free text against a fast news stream, plus source links. Small volume (71 delivered in 30 days) but the best tap rate (21%).

### Key data point

All 317 distinct textual events with source links cite an X post, from 9 accounts (prod agent DB, `SignalNotification.suggestion.evidence.sourceLinks`, read only):

| account | events |
|---|---:|
| @wublockchain | 161 |
| @deitaone | 42 |
| @bulltheoryio | 35 |
| @watcherguru | 31 |
| @lookonchain | 18 |
| @onchainlens | 14 |
| @newlistingsfeed | 13 |
| @treenewsfeed | 2 |
| @phoenixnewsio | 1 |

Caveat: the links came from Elfa's own keyword lookup, which may favour some accounts. A strong lead, not a full picture of Elfa's sources. *(Corrected — see Update 2026-10-30 below.)*

## Why not one research call per rule

The first idea was a scheduled job per rule that searches the web with Claude/Harness and asks whether the watch came true. It breaks on three points:

- Cost grows with rules × cadence, not with news. 65 watches every 15 min is about 6,200 calls a day. The Claude CLI measured about $0.05 and 20 s per call (interest-distiller spike), so roughly 35 hours of serial CLI time a day, on a subscription shared with baskets, Hetzner only.
- Web search lags (index delay) and keeps returning the same old articles, so de-duplication is needed anyway.
- Recall depends on search phrasing, while the news Elfa matched was X posts from a handful of accounts.

What stays from the idea: a model as the judge of meaning, scheduled jobs, deterministic code for everything else.

## Proposed architecture

```
every minute (BullMQ job, narrative poller)
  twitterapi: new posts from allowlist + watched handles since the cursor
  -> drop posts older than the cutoff (createdAt)
  -> no new posts: stop
  -> armed SignalTrigger rows with an event leg
  -> one model call: posts x watches -> matches
  -> check: ids, confidence, author, cooldown, personal cap, last alert
  -> SIGNAL_QUEUE job, source "pear", with the matched tweet id and URL
  -> advance the cursor
signal.processor: trigger -> pair -> evidence from the tweet id (no Elfa) -> compose -> dispatch
```

| stage | owner |
|---|---|
| collect posts | existing `TwitterApiClient` (`x-digest/twitterapi.client.ts`), BullMQ job every minute |
| normalise (id, author, time, text, URL) | deterministic, URL built from id + author |
| drop old and already-seen posts | deterministic, Redis cursor + 2h cutoff |
| judge posts against watches | one model call per tick (Haiku via OpenRouter, same pattern as `rule-builder.agent.ts`) |
| validate, cooldown, cap, de-duplicate | deterministic, same keys and logic as the math poller |
| fire | existing `SIGNAL_QUEUE` -> `signal.processor` |
| arm, expire, rebuild | existing trigger reconcile (no Elfa calls under `PEAR_SIGNALS_LIVE`) |

- No per-rule jobs and no topic grouping: about 70 watches at about 40 tokens each fits one prompt.
- Cadence: collect every minute, call the model only when there are new posts.
- No webhooks in v1. The twitterapi.io stream (about $149–299/month) is the upgrade if polling cost or latency calls for it.
- A handle watch is the same collection with an author filter, so this covers most of the twitter track too. Merge the two tracks.

## Model role

Judge only. No searching, no browsing, no evidence writing. It picks from posts we fetched, so it cannot invent a source.

Input:

```
watches: [{id, text, lastAlert?: {summary, at}}]   // lastAlert only within 48h
posts:   [{id, author, createdAt, text}]
```

Output (zod):

```
matches: [{
  watchId,
  postIds: string[],          // from the input; one entry per event
  confidence: 0-100,
  sameEventAsLastAlert: boolean,
  reason: string              // one sentence, logged
}]
```

Prompt rules: match only when the post reports the event **happened** (not a rumour, forecast, opinion or "could"). "Major" or "serious" means market-moving. A follow-up on the same story sets `sameEventAsLastAlert`.

Deterministic code fires only when all post ids are in the batch, confidence >= 75 (the old `minConfidence`), not the same event as the last alert, the author matches for tweet legs, the post is inside the cutoff, the cooldown key is free and the personal cap is not used. The event time is the post's own `createdAt`.

Claude CLI / web research stays out of the critical path. Possible later use: a fallback for watches the account list doesn't cover (for example equity earnings), only if the eval shows misses.

## State

- `narrative:cursor` — last processed post.
- `narrative:last:<topic>` — summary and time of the last alert, 48h.
- Cooldown and count keys shared with the math poller. Today math's key prefix is math-only, so one shared prefix is needed to keep "one cooldown gates all legs" for mixed rules.
- No new DB fields.

## Reliability

- **First reading:** each post is judged once, in order. A rule armed at T only sees posts that arrive after T. A lost cursor restarts from now and never replays.
- **Same event, many posts:** grouped by the model in one tick; the topic cooldown (2–4h) and `sameEventAsLastAlert` across ticks.
- **Story update vs new event:** `lastAlert` in the prompt.
- **Condition true for days:** not a problem, a news event is a post, not a state.
- **Hallucination:** ids only, evidence is the real post.
- **Stale results, late discovery:** 2h `createdAt` cutoff.
- **Transient failures:** a failed read or model call leaves the cursor where it is, so the next tick retries. Repeated failures log at error level (Discord). A failed queue add resets cooldown and count, as in math.
- **Source trust:** allowlist only.
- **Expiry, `maxTriggers`, edited or re-armed rules:** armed rows only, same cooldown and count keys, edits picked up on the next tick.
- **Mixed OR:** math legs fire through the math poller, event legs through this one, shared cooldown.
- **AND with an event leg:** rejected by the rule builder in v1. None armed today, 4 mixed rows saved.
- **Vague watches:** "sentiment around ETH gets ugly" is not an event, map it to math legs (funding, a drop, fear & greed) or reject it. "A major exchange gets hacked" and "serious regulatory action against stablecoins" are events the judge handles. "Trump says something bullish about crypto": collect @realDonaldTrump (already watched in the book) plus @deitaone and @watcherguru, who quote him.

## Failure behaviour

- twitterapi or the model fails: retry the same posts next tick up to the cutoff, then log the drop.
- Redis down: the tick fails and nothing fires (fails closed).
- Tweet fetch fails at delivery: existing evidence retry and terminal failure path.

## Cost (rough, not measured)

- Posts from these accounts are probably a few hundred to about a thousand a day (estimate). At about 5k input tokens per call and a few hundred calls a day, that is a few million Haiku tokens a day. Current Haiku pricing not checked.
- Collection is about 1,440 calls a day if one search can cover many accounts.

## MVP (what lets the flag go on for narrative)

- The poller and the model call.
- Evidence passed in the job.
- No Elfa calls on the narrative delivery path (pair lookup and event summary).
- A shadow run in log-only mode before going live.

## Not building in v1

Vector DB or embeddings, crawler or RSS, web search in the hot path, per-rule jobs, topic grouping, multi-agent chains, AND with event legs, a "sentiment" class, a twitterapi webhook, new DB tables.

Revisit when watches pass about 300 (add a ticker prefilter), posts pass about 5k a day, or the eval shows misses outside the account list.

## Implementation plan

1. **Replay eval first (read only).** Fetch about 2 weeks of posts from the 9 accounts, run the model call with the watches armed then, and compare with Elfa's past news fires: recall on the 317 sourced events, precision on a hand-labelled sample. Decides the account list and the prompt before any code.
2. Collection method in `x-digest/twitterapi.client.ts`: a search across many accounts at once, or `last_tweets` per account if search isn't available.
3. `ai/background/narrative-judge.agent.ts` (shape of `rule-builder.agent.ts`) and `LLM.narrativeJudge` in `config/models.ts`.
4. `ai/background/narrative-poller.processor.ts`, modelled on `math-shadow.processor.ts`: shadow flag, `PEAR_SIGNALS_LIVE` for live, shared cooldown and count helpers with one shared key prefix.
5. `signal.processor.ts`: Pear fires carry the tweet id and URL in `raw`, evidence comes from that tweet and not `fetchEventSummary`; pair for `narrative:` topics without Elfa (depends on #1184).
6. `rule-builder.service.ts`: reject AND rules with an event leg.
7. Shadow run on prod data (read only), compare with the eval, then dev rehearsal.

## Open questions (blocking)

1. twitterapi.io: does our plan and key allow one search across many accounts, and is the key set on `ap-worker`? It appears in the agent's config, not checked on Railway.
2. #1184: for `narrative:` topics, deliver with no trade pair, or have Pear pick one without Elfa? 387 of 412 dropped news fires were these topics.
3. Account allowlist sign-off, for example whether @bulltheoryio is a trusted source.

## Update 2026-10-30 (eval prep)

- **Correction to the key data point caveat:** the 317 links are the tweets Elfa itself matched (`evidence.matchedConditions` with tweet id and confidence 75–100, median 90, `matchedSourceConfirmed` true), not the event-summary keyword lookup. The 9 accounts are Elfa's real matched sources for delivered news alerts. Still not a full picture of what Elfa watched, since fires that never reached a notification row carry no tweet.
- **Implementation step 5 is mostly there:** `signal.processor` already fetches the tweet in `raw.trigger.matchedConditions[].match.url` and skips `fetchEventSummary` when found. The poller writes that shape. The `narrative:` pair lookup still calls Elfa.
- **Step 6 is a real change:** `rule-builder.service.ts` `check()` allows AND with one event leg today.
- **Open question 1, partly answered:** twitterapi's `advanced_search` documents `from:a OR from:b` with `since_time`/`until_time`, so one call can cover all accounts. Untested live. The agent's `TWITTERAPI_IO_KEY` is unset locally, and the context service's key returns 402 (out of credits).
- **Step 1 status:** eval script written (`apps/agent/test/narrative-replay-eval.ts`, local only), ground truth pulled (52 events in 9–23 Oct). Blocked on twitterapi.io credits. See [[30th Oct 2026#Narrative: design review and replay eval prep (later)]].
