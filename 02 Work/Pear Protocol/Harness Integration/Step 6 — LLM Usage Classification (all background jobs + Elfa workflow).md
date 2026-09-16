# Step 6 — LLM Usage Classification (all background jobs + Elfa workflow)

- Status: Reference report — classification only, no migration work in this note.
- Date: 2026-09-15
- Project: Agent Pear harness integration
- Related: [[Harness Integration/Step 1 — AP Background Processes and Hetzner VPS Inventory]] · [[Harness Integration/Step 5 — Ambient-Agent Job Migration Plan (apps agent)]]

## Purpose

Requested report classifying every background job (old `apps/context` batch + new `apps/agent` 12-job batch) by LLM usage, plus a separate classification of the Elfa-specific workflow (webhook → signal processor → tools), which spans both cron jobs already covered elsewhere and non-cron paths (a user-triggered tool, a post-composition step) that aren't migration candidates at all.

## Part A — Full job classification, by LLM tier

### Tier 1 — Claude (Opus, via the CLI harness with live web research)

- **`made-baskets`** — generates AI-researched pair-trading baskets by combining sentiment, live web research, and stats. The only job in the entire landscape that uses Claude, and the only one where live research changes the output.

### Tier 2 — LLM used, but pure classification/synthesis (not Claude — Cerebras or OpenRouter budget models)

- **Twitter-ingestion** — labels/classifies incoming tweets for context ingestion (OpenRouter `gpt-5-mini`).
- **asset sentiment** — synthesizes per-asset sentiment from already-fetched web/Twitter/tokenomics data (OpenRouter `deepseek-v4-pro`).
- **category sentiment** — aggregates existing asset sentiments into a per-category score, Wed/Sun (OpenRouter `deepseek-v4-pro`).
- **weekly sentiment refresh** — re-runs sentiment synthesis on stale assets weekly (OpenRouter `deepseek-v4-pro`).
- **`interest-distiller`** — distills each active user's stored memory/journal/persona into interest tags nightly (Cerebras `gpt-oss-120b`).
- **`watch-scheduler`** — classifies a stored ghost statement as dated-vs-open while scheduling a user's watch entries (Cerebras).
- **`ghost-synthesis`** — merges ghost triggers into shared cross-user signal context via judge + author calls (Cerebras, plus the one OpenRouter touchpoint in the whole batch — embeddings only, not generation).
- **`x-digest`** — classifies freshly-fetched X posts against each user's interests (Cerebras).
- **`touch-brief`** — synthesizes a personalized notification brief from headlines `global-feedpull` already fetched (Cerebras).
- **signal processor** (`signal.processor.ts` → `SignalDispatchService`) — deterministically matches Elfa-event evidence against user triggers, then calls an LLM (`LLM.signal`, Cerebras) to compose the notification text before dispatch. Not previously covered in the 12-job batch — surfaced via the Elfa-workflow question below.

### Tier 3 — No LLM at all (deterministic compute/IO)

- **Tokenomist refresh** — syncs tokenomics data for every tracked asset from the Tokenomist API.
- **OI refresh** — syncs open-interest data from Hyperliquid via Coinglass.
- **asset/coin/Tokenomist index** — rebuilds the Redis-backed lookup indexes other jobs read.
- **watched-account fetch** — polls X accounts the system watches, hourly.
- **`image-retention`** — deletes object-storage images older than 60 days.
- **`ghost-report`** — builds the monthly ghost-analytics report, read-only.
- **`user-valuescore`** — computes each user's daily value/segmentation score.
- **`signal-funnelrollup`** — aggregates weekly signal/engagement analytics plus Elfa credit-spend usage.
- **`touch-priors`** — computes engagement-state priors that feed notification targeting.
- **`ghost-writer`** — generates/updates ghost triggers that seed the signal system.
- **`global-feedpull`** — pulls Elfa trending/narrative data and writes the global feed every user sees.
- **`ghost-shadow`** — evaluates trigger conditions and records live signal fires every 15 minutes.
- **Elfa webhook** (`/elfa/events`) — receives Elfa alerts, dedupes them in Redis, enqueues to the signal queue.

**Count: 1 job uses Claude, 10 use a cheap LLM for classification/synthesis, 13 use no LLM at all.**

## Part B — Elfa-specific workflow (separate from the migration batch — mixed cron/event/user-triggered)

Verified 3 of these 6 against the actual code this session (marked ✓); the other 3 are taken from the user-supplied classification as given, not independently re-verified.

| Item | Schedule | What it does | LLM? | Migration note |
|---|---|---|---|---|
| Elfa webhook ✓ | event-driven | Receives Elfa alerts at `/elfa/events`, dedupes in Redis (`SETNX`), enqueues to the signal queue — confirmed no LLM, pure ingress | No | User-facing signal path — keep on Railway. |
| signal processor ✓ | event-driven (queue) | Processes queued Elfa events: deterministic evidence-matching, then an LLM call to compose the notification text before dispatch — confirmed via `SignalDispatchService`'s `resolveNodeModel(...LLM.signal)` | Yes (Cerebras) | Close to user notifications — keep on Railway. |
| `watch-scheduler` | daily 05:30 | Uses Elfa token news while scheduling user watch entries — already covered in Part A/Tier 2 | Yes (Cerebras) | Already in the 12-job batch; conditional — affects user workflows directly. |
| `global-feedpull` | 2x daily | Uses Elfa trending data to build the global feed — already covered in Part A/Tier 3 | No | Already in the 12-job batch; indirectly user-facing. |
| `signal-funnelrollup` | weekly (Mon) | Reads signal engagement + Elfa credit usage for analytics — already covered in Part A/Tier 3 | No | **Correction to the "possible later candidate" framing: this one is already shipped — [PR #789](https://github.com/pear-protocol/agent-pear-v2/pull/789), code-complete and Hetzner-validated (see [[Harness Integration/Step 5 — Ambient-Agent Job Migration Plan (apps agent)]]).** |
| scheduled X post context | when posts are prepared | Fetches Elfa trending narratives to help compose scheduled posts — not independently verified this session | Unconfirmed | Taken as given: keep with the X-posting flow on Railway. |
| social pulse tool ✓ | user-triggered | `get-social-pulse.tool.ts` — an AI-SDK tool the live chat agent calls to query Elfa for mentions/token news/event summaries; confirmed no standalone scheduled LLM call of its own, it just fetches data for whichever agent is running | No (own call) — feeds whatever model the live chat session uses | Confirmed not a cron job — keep with the user-facing agent, not a migration candidate at all. |

**Net finding for Part B:** of the 6 items, only 3 are actual cron/background jobs at all (`watch-scheduler`, `global-feedpull`, `signal-funnelrollup`) and all 3 are already accounted for in Part A/Step 5 — one already shipped, two still held back under Epic #717's exclusion. The other 3 (webhook, signal processor, social-pulse tool) are event-driven or user-triggered, not scheduled jobs, so they were never migration candidates under this epic's "cron job" framing to begin with — they're correctly excluded, not merely deprioritized.

## Part C — Cost analysis attempt: Langfuse has zero visibility, OpenRouter dashboard confirms 2 of 10

**Langfuse: checked directly (2026-09-15), no data exists.** Queried the metrics API (`observations` view, grouped by `name`, summed `totalCost`, both `production`/`development` environments, full 30-day window) and searched the complete result set for every job identifier in this report. Zero matches on any of them. Everything Langfuse has traced is the interactive chat DAG (`composer.*`, `planner.invoke`, `tradeExecution.invoke`, tool calls) — **none of these background jobs are instrumented with Langfuse tracing at all.** This is a real observability gap, not a query mistake.

**Root cause found for why OpenRouter's own dashboard can't attribute cost per-job either:** grepped the whole `apps/agent` codebase for `HTTP-Referer`/`X-Title` (the two headers OpenRouter uses to populate its "App" column) — zero matches. No OpenRouter call anywhere in this codebase, background job or interactive chat, tags itself. Every row in the OpenRouter Logs UI looks identical regardless of source. The only usable signal today is **model name**, which happens to map cleanly onto job groups since each job uses a distinct model:

| Job(s) | Model | Confirmed cost (user-provided, from OpenRouter dashboard, 2026-09-15) |
|---|---|---|
| Twitter-ingestion | `gpt-5-mini` | **$16 / August** ≈ $0.52/day. Cross-checks against Story #773's own measured volume (~280–310 calls/day) — consistent with "cheap, high-frequency." |
| asset + category + weekly sentiment refresh (combined, can't be split further without per-job tagging) | `deepseek-v4-pro` | **$4 / whole year** ≈ $0.011/day. Confirms Step 1's original call to keep these off Opus was correct — cost pressure to revisit that is effectively zero. |

**KB-build + `ghost-synthesis` embeddings, confirmed (2026-09-15):** `openai/text-embedding-3-large` via OpenRouter — **$0.63 / last month**. Small, as expected, and now a real figure rather than an assumption.

**Cerebras (`gpt-oss-120b`) — attempted, no dashboard access.** This is the model behind 6 of the 7 still-unmeasured jobs: `interest-distiller`, `watch-scheduler`, `ghost-synthesis`'s judge/author calls, `x-digest`, `touch-brief`, signal processor (`LLM.signal`). User checked and could not see a Cerebras usage/billing dashboard — **not confirmed to cost nothing, confirmed to be unmeasurable with current access.** This is now the one real remaining gap in this report; closing it needs either Cerebras account/dashboard access sorted out, or per-job request tagging (if Cerebras' API supports an equivalent to OpenRouter's `X-Title`) plus wiring that into whatever billing visibility does exist.

**`made-baskets` — deliberately not chased further.** Runs on the Claude Max subscription (5-hour rolling windows), not pay-per-token billing, so there's no dashboard $ figure to go find — user confirmed this doesn't need checking. The existing ~$2.18/basket figure (from a local `basket-eval.jsonl` file on the Hetzner box, not Langfuse or OpenRouter — see [[Harness Integration/Step 2 — Claude Basket Generation on Hetzner]]) remains the only cost data for this job, and it's a per-basket unit cost, not a dashboard-billed monthly figure, so it isn't directly comparable to the OpenRouter numbers above.

### Final state of the cost picture (2026-09-15)

| Coverage | Jobs | Status |
|---|---|---|
| Measured, OpenRouter | Twitter-ingestion, asset+category+weekly sentiment, KB-build + `ghost-synthesis` embeddings | 3 of 10 — all confirmed cheap ($0.52/day, $0.011/day, $0.63/month respectively) |
| Measured, separately | `made-baskets` | Claude subscription, ~$2.18/basket, not a dashboard $ figure |
| Unmeasured — access gap, not a zero | `interest-distiller`, `watch-scheduler`, `ghost-synthesis` (judge/author), `x-digest`, `touch-brief`, signal processor | 6 jobs, all Cerebras `gpt-oss-120b`, no dashboard access currently — genuinely unknown, not assumed cheap |

**Recommended fix, applies to both gaps at once:** add per-job `HTTP-Referer`/`X-Title` tagging to every OpenRouter call, and wire Langfuse tracing into the background processors the same way the interactive DAG already has it. Both are the same root cause — nothing in this codebase gives background-job LLM calls their own identity, on either provider — and both are small, mechanical changes, not a redesign.
