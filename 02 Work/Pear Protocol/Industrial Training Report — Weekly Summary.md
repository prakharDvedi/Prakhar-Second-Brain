# Industrial Training Report — Weekly Summary

*Pear Protocol · repo `pear-protocol/agent-pear-v2` · 7 Jul 2026 → 20 Oct 2026 (15 full weeks, plus 19–20 Oct). Built from [[My Work at Pear Protocol]], [[Daily Work Logs]], [[Features Owned]] and the incident notes. Draft for planning the report; the artifact comes after this is approved.*

## The one-paragraph version

Pear Protocol runs an AI trading agent (Telegram, web, MCP) that places leveraged pair and basket trades on Hyperliquid and Lighter. My work fell into four streams: (1) making the agent safe and honest on the money path, (2) making failures diagnosable, (3) building features (Lighter venue, basket generation, sentiment, familiar-asset ranking), and (4) moving background jobs off Railway onto a Hetzner server.

## Stats to quote (all from the vault)

- Jul 7 → Aug 30: **62** self-opened merged PRs, **130** authored commits (the later weeks are catch-up entries, so no clean count after Aug 30).
- Roughly 100 PRs touched in total across the 15 weeks (from the PR numbers #241 → #1122). *Exact count to be confirmed from GitHub before it goes in the report.*
- 12 background jobs studied for migration, 9 cut over live on 28 Sep.
- 11 more PRs merged 19–20 Oct (#1139 to #1153), checked against the repo.

## Week-by-week

| Wk | Dates | Theme | What I did |
|---|---|---|---|
| 1 | 6–12 Jul | Onboarding + first fixes | Downgraded noisy sentiment-cron errors so on-call stopped being paged (#241). Fixed autonomous scheduled tasks being routed back into the scheduler (#246). Traced a Dutch user's trace to find that the streaming chat path dropped the user's 5% take-profit instruction and used a silent 100% default (#248). Fixed a false "truncated" alarm in telemetry (#247). Gave the composer a real UTC clock after it invented the time (#253), and rescued a finished PnL card thrown away by an unrelated timeout (#254). |
| 2 | 13–19 Jul | Planner reliability + LLM error visibility | Added one recovery attempt when the planner emitted a non-schema output and silently downgraded real requests to generalChat (#250). Built a shared `formatLlmError()` so every model failure logs status, body, provider and request id (#258). Removed a silent 5x leverage default from the trade prompt (#260). Fixed HIP-3 charts silently falling back to text, and stopped the composer promising retries it cannot perform (#289, #290). |
| 3 | 20–26 Jul | Trade-execution correctness | Chased a recurring Gemini "corrupted thought signature" failure by extending the error pipeline (#300) and instrumenting the basket cron (#298). Rejected fractional leverage up front (#305). Corrected a partial-fill alert that blamed the wrong cause (#309). Guarded a `/linkwallet` poll that paged on-call for an expected Telegram 403 (#310). Gated HIP-3 margin backfill so unfundable orders fail before Confirm (#324). Fixed `/aipear` being advertised in DMs where it did nothing (#327). |
| 4 | 27 Jul–2 Aug | Basket generation features | A reduce-only rejection no longer counts as a failed trade when the position is already flat, using a downgrade-only verification (#337). Added Twitter-labeling diagnostics (#340). Added closed-vocabulary semantic tags and long-only/short-only baskets, skipping fake pair stats for one-sided baskets (#351, #352, #367). Fixed group trade pitches showing buttons that only worked in DM (#362). |
| 5 | 3–9 Aug | Agent context + model instability | Built `runtimeContext` so the agent can review the Basket Builder draft a user is editing, treated as untrusted input with an injection test suite (#379). Added Telegram slash-command usage counters (#380, #390). Swapped models three times as providers rate-limited (#384–#387, #399). Stopped showing projected token unlocks as confirmed facts (#391, #392). |
| 6 | 10–16 Aug | Chart edge cases + more model swaps | Stopped chart-send errors paging Discord for bad input (#469). Caught the ambiguous HIP-3 ticker (`AVGO`) crash in the chart path (#431). Fixed wrong display tickers on persisted baskets (#481). Finished the model swaps (#436, #437, #478, #479). |
| 7 | 17–23 Aug | Lighter integration (2nd venue) | Brought Lighter online next to Hyperliquid: venue-aware price tool, invocation-scoped account pinning, margin-mode check (227/227 instruments verified live), Telegram account chooser, KB corrections (#556–#564, plus commits inside #542–#548). Fixed leaderboard routing and rebuilt the KB embeddings (#520). Hardened Elfa signal validation (#553) and TWAP close-all (#491). Updated the SDK and alias resolution (#493, #472). |
| 8 | 24–30 Aug | Safety gates + Lighter hardening | Confirm-time margin gate for basket opens (#569). Found that malformed TRIGGER/LADDER JSON crashed and the model silently swapped the user's trade for a different one (#584). Closed the ambiguous-symbol crash at every remaining entry point (#514). Disclosed a hidden ~2x margin change on leverage-clamped trades (#593). Retuned the internal-leak guard (#639), made signal failures self-diagnosing (#640), added delivery-ordering observability (#641), and rescaled rebalance weights instead of rejecting them (#642). |
| 9 | 31 Aug–6 Sep | Hotfix + Claude basket research | Hotfixed basket stats going null because one env var fed two services (#677, #678). Groomed the Hetzner migration issues. Built and evaluated a Claude (Opus + web) basket-research harness against the existing OpenRouter path over 20 baskets, dry-run only. Hardened Twitter fetch retries. |
| 10 | 7–13 Sep | Execution hardening + paper trading + Hetzner canaries | Fixed trade-exec issues: no silent new position for an unmergeable leg, TWAP result trimming (a request was ~3.44M tokens), account-scoped balances (#681, #693, #697). Built the read-only `paper-shadow.ts` tracker. Investigated an SL/TP hallucination and a Kalshi routing gap (read-only). Started the background-jobs-to-Hetzner migration with canary jobs. |
| 11 | 14–20 Sep | Hetzner migration, first production install | Resolved the merge conflict on the basket-migration PR (#786), cut the daily basket run from ~3h to ~1h with `Promise.all` (#796), and did the first real systemd install on the Hetzner box. Ran an interest-distiller spike (Claude CLI vs Cerebras). Found a live production collision in a dry-run, and investigated BNB/AVAX pair-orientation loss. Fixed basket research diversity (#811). |
| 12 | 21–27 Sep | Provider fallback + review + tooling | Moved the composer fallback to a different provider and fixed a dead fallback in the streaming path (#923). Shipped the Elfa background jobs (#946, #988). Reviewed the release PR #1010 and several teammate PRs. Built `codectx`, a persistent codebase-retrieval tool (code graph + embeddings). Fixed a stock-pick routing bug (#986). |
| 13 | 28 Sep–4 Oct | Production cutover + two research plans | Merged the remaining agent and Twitter jobs (#1027) and cut **9 of 12 jobs** over from Railway to Hetzner. Found and fixed a Prisma transaction timeout that only appeared over the public proxy (#1034). Researched which assets Pear users actually trade (65% current holders / 35% 90-day openers), wrote the final plan and an epic with five stories (#1064–#1069). Started sentiment-ingestion improvements: price context, CoinGecko headlines, and a playground that scores each call against real price moves (#1052, #1058, #1060). Built the familiar-asset ranking job and ranked asset pool, and an evidence-based pair-vs-basket selector, replayed on production data with nothing written (#1070, #1071, #1077). Results in the project section below. |
| 14 | 5–11 Oct | Familiar assets go live, sentiment evaluation, small fixes | Shipped familiar-asset ideas to production: release to `main` on 5 Oct (#1078, #1082), ranking timer installed on the Hetzner box, first scheduled runs checked 6–7 Oct. Ran a repeated old-vs-new sentiment comparison scored against real price moves. Retired the Coinglass wallet-distribution tool (#1090). Fixed the chart/chip extractor picking up an earlier turn's asset (#1091). Added retries on transient DB errors (#1092, #1093). Fixed a one-shot shutdown race (#1096). Filed an upstream price-feed outage. |
| 15 | 12–17 Oct | Alert noise, Prime Agent trial, unlock fix | Measured whether traders opened the familiar-asset ideas (read-only, 14 Oct). Moved basket generation back to Claude and retired SYMM from the daily run (#1111, #1112). Found why Elfa returned 400 (title over the 120-char limit) and fixed it (#1107). Skipped sub-1KB audio that triggered paid transcription calls (#1103). Tested Prime Agent against advanced mode by replaying a real session; decided to drop it for now. Stopped confirm-drift refusals paging Discord (#1121). Fixed token-unlock amounts hidden for every upcoming unlock (#1122), reversing part of the 7 Aug fix. |
| 16 | 19–20 Oct (partial) | Adjust sizing, honesty fixes, alert floods | Added margin sizing to `adjust_position` after traces showed "add 500 margin" ordering $500 notional (#1139); signed uPnL stops on ticket cards (#1141); total-margin line only when every leg is margin-sized (#1146). Stopped the agent inventing refusals for unlinked web users (#1147), reporting failed or empty data sources as "nothing found" (#1148), and refusing chart asks (#1149); no blank reply when an advanced run fails (#1153). Fallback when the advanced stream opens then fails (#1143), auth Link-tap race (#1144), touch-priors pool timeout (#1145), poll-failure and Discord alert floods capped (#1152). Release #1151 put #1139–#1149 on `main` on 20 Oct. |

## Project: baskets built from assets Pear users know

Basket and pair ideas were built mostly from open-interest picks, and about 2% of generated ideas were ever opened by a trader. The goal was to build ideas from the assets Pear traders actually hold and open (ranked 65% by current holders, 35% by 90-day openers), then check whether that changed anything.

- **Built (30 Sep – 5 Oct):** daily ranking job, ranked asset pool for Hyperliquid and Lighter generation, adoption metric that reproduced a frozen baseline exactly. Replayed on the production box with nothing written; shipped to `main` 5 Oct.
- **Changed course on evidence:** a funnel measured from the production log showed familiar assets were lost at category pick and research, not selection. The plan moved from "boost" to "replace the candidate pool", and the pair-vs-basket work (replayed pairs 5% to 33%) shipped after, not first.
- **Results:**
  - Familiar share of generated legs: 15% to 64–72%.
  - Ideas with at least one familiar leg: 43% to 74–82%.
  - Ideas opened by a trader: 1.9% (3 of 159) to 6.1% (8 of 131) in the first full week after, inside the pre-release weekly range of 0.5% to 5.6% on 1–7 users.
  - All Pear opens per day: 86 after, 74–100 before. No rise.
  - Same-asset opens (any direction): 4% to 8%, cause unknown.
- **Read:** generation did what it was meant to. Adoption lift is not shown, and nothing links an open to an idea, so causation is not shown. The full 7-day read (due about 19 Oct) is not in the vault yet.

## Project: asset sentiment, measured against price

- Built a playground that runs old and new sentiment prompts per asset and scores each call at +1, +3 and +7 days. The new prompt cut "bullish while falling" calls from 67% to 10% and flipped direction less often on identical inputs, but gave a direction on only 39% of calls.
- On later 7-day scores nothing beat plain momentum, and the new prompt had the worst return in every repeat run. Both weeks tested were down weeks with 17 assets moving together, so the sample is small. The decision was not to switch on this evidence.

## My contribution and what it led to

Evidence tags: **Measured** = a number from production or a live check. **Tested** = a failing case reproduced and fixed, with specs. **Not yet seen** = shipped, effect not observed. Few fixes have a measured before-and-after from production.

**Traders got correct, honest answers**
- A stated 5% take-profit was silently replaced by a 100% default on the streaming chat path; traced from a Dutch-language session (#248). *Tested.*
- Malformed TRIGGER/LADDER conditions crashed and the model quietly retried with a different trade; now they fail with a named error (#584). *Tested.*
- Projected token unlocks were shown as confirmed: 49 of 50 assets had no confirmed amount (#391). The first fix then hid every upcoming unlock (XPL, about 227.8M tokens, reported as none), which I found and corrected (#1122). *Measured.*
- Stopped invented refusals for unlinked web users, "nothing found" for failed data sources, and refusals of chart asks (#1147, #1148, #1149). Blank replies, invented retry promises and a made-up clock time were replaced by real messages (#1153, #290, #253). *Tested.*

**Money-path safety**
- "Add 500 margin" was ordered as $500 notional, a tenth of the margin at 10x; adjust can now be sized in margin (#1139). *Measured.*
- Confirm-time margin gate; hidden margin change on leverage-clamped trades now stated ($1,000 asked, $1,999.95 resulting) (#569, #593). *Measured.*
- Unfundable HIP-3 orders fail before Confirm; close-all can no longer turn a TWAP into market orders (#324, #491). *Tested.*
- Rebalance weights with up to 15% drift are rescaled, not rejected; 4 traces showed 21 rejected attempts (#642). *Measured.*

**Less noise, fewer wasted calls**
- Poll-failure logging cut from about 12,000 lines an hour in the 14 Oct outage to one line on failure, one every 5 minutes, one on recovery; Discord alerts capped at 20 a minute (#1152). *Tested.*
- Stopped paging for expected events: Telegram 403s, bad chart input, confirm-drift refusals, rejected webhooks, sentiment batch failures (#310, #469, #1121, #241). *Tested.*
- Audio under 1 KB (all prod alerts were 5–27 bytes) no longer triggers a paid transcription call and a 500 (#1103). A TWAP result that made one request about 3.44M tokens was trimmed (#697). *Measured.*
- Found why Elfa returned 400 on every reconcile (title over 120 characters) (#1107). Traced Elfa credit use to about 10.4k a day; cause strongly evidenced, not proven, no code change. *Measured.*

**New capability**
- Lighter as a second venue next to Hyperliquid, checked live on 227 of 227 instruments (#556–#564). *Measured.*
- Long-only and short-only baskets with tags; the agent can review the basket draft a user is editing (#351, #379). *Not yet seen.*
- Basket ideas built from familiar assets: familiar legs 15% to 64–72% (#1078, #1082). Trader adoption not shown to have moved. *Measured.*
- A read-only paper tracker for generated baskets and a sentiment playground that scores each call against real price moves. *Tested.*

**Operations and decisions**
- Moved 9 of 12 background jobs from Railway to Hetzner by running each by hand and comparing with Railway first; that caught a transaction timeout that only appeared over the public proxy (#1027, #1034). First scheduled runs exited 0. *Measured.*
- Daily basket run expected to drop from about 3 h to about 1 h by running platforms in parallel (#796). Not observed on a real run. *Not yet seen.*
- Fixed a model fallback that never fired in the streaming path (#923, #1143). Injected 429s recovered in 6 of 6 trials; a real one has not been seen. *Not yet seen.*
- Tested Prime Agent on a replayed production session (about 2x slower, same quality); the team dropped it for now. Reviewed release #1010 and teammates' PRs. *Measured.*

## By the numbers

The vault has no revenue or volume figures, so none are shown. "Projected" means calculated, not yet observed on a real run.

| What | Before | After | Status |
|---|---|---|---|
| True cost of one Claude basket | $1.55 estimated (missed the $0.62 category call) | $2.18 and about 340 s | Measured, 11 Sep dry run |
| Category-call memo, per full run | about 60 calls, about $37 | about 2 calls, about $1.24; about $36 (28%) saved | Projected |
| Daily basket run | about 3 h | about 1 h expected | Projected |
| Input tokens, replayed advanced session | 5.7M | 1.87M (production used 1.97M) | Measured, Prime Agent dropped |
| Size of one TWAP-orders request | about 3.44M tokens | per-slice fills dropped (size after not recorded) | Measured |
| Poll-failure log lines, 14 Oct outage | about 12,000 an hour | 1 on failure, 1 every 5 min, 1 on recovery | Tested |
| Discord error alerts | unbounded | capped at 20 a minute | Tested |
| Wasted paid transcription calls | every prod alert was a 5–27 byte payload | skipped under 1 KB | Measured cause |
| Composer fallback on a provider 429 | 88 s to a generic error | 6 of 6 injected 429s recovered, worst 29.8 s | Not yet seen on a real 429 |
| Rebalance weight rejections | 21 in 4 traces, drift up to 15% | rescaled, none rejected | Tested |
| Ambiguous-ticker crash | 13 times in 3 days | closed at every entry point | Tested |
| Margin shown before Confirm | $1,000 asked, $1,999.95 resulting, undisclosed | change stated first | Tested |
| Background jobs on Hetzner | 0 of 12 | 9 of 12, first runs exit 0 | Measured |
| Unlock amounts shown as facts | 49 of 50 assets had no confirmed amount | withheld 7 Aug; shown again as SCHEDULED 17 Oct | Measured |
| Familiar share of generated legs | 15% | 64–72% | Measured |
| Elfa credit use | about 10.4k a day (165,696 this month) | cause traced, not proven, no code change | Not yet seen |

## Cross-cutting themes (for a "learning outcomes" section)

- **Fail closed on the money path:** margin gate, TWAP close-all, TRIGGER/LADDER validation, downgrade-only verification.
- **Debug from evidence:** Langfuse traces, replay harnesses, write-free A/B on the prod box. Retracted my own near-circular backtest when shown.
- **Make failures name themselves:** shared error formatter, signal-evidence diagnostics, delivery observer.
- **Don't let the agent assert what it can't know:** fabricated time, fake retry promises, projected unlocks as facts.
- **Operations:** GitFlow (`development` → `main`), systemd timers, kill-switches, manual run-and-compare before every cutover.

## Gaps and things to confirm before this becomes the report

1. **Internship title, mentor, and exact start/end dates.** The vault has no source for these. 7 Jul is only the earliest log; I extended the window to 20 Oct.
2. **PR counts after 30 Aug.** The vault counts stop there; later entries were backfilled from GitHub. Weekdays with no daily log: 16 Jul, 24 Jul, 10–11 Aug, 25–27 Aug, 4 Sep, 14–15 Sep, 23 Sep.
3. **Reasons behind some choices** (why three models rate-limited in one week, how the Lighter work was split with a teammate) are not recorded anywhere.
4. **Confidentiality.** This repo is a company's private codebase; check with the company what can go into a college report (trace IDs, ticket IDs, user session IDs, internal service names). I'd keep those out.
5. **Week 1 detail** is thin. I only have PR titles for 7–10 Jul.
6. **Familiar-asset 7-day read** (due about 19 Oct) is not recorded yet; the results above are the earlier partial reads.
