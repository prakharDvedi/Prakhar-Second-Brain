# My Work at Pear Protocol

*Engineering record — author `prakharDvedi`, repo `pear-protocol/agent-pear-v2`. Reconstructed from merged PRs, commit history, PR descriptions, diffs, and tests. Not a list of titles.*

**Range:** 2026-07-07 → 2026-08-30 · **62** self-opened merged PRs · **130** authored commits · **4** open PRs as of 2026-08-30.

**Confidence key** — `Confirmed`: stated with concrete evidence (trace ID, test output, quoted numbers). `Likely`: plausible from the diff/PR claim, not independently verified beyond the repo. `Unknown`: repo has no evidence either way.

---

## Timeline

Grouped chronologically by underlying problem, not by PR title. Iterative PRs on the same root cause are collapsed into one entry, with every relevant PR listed under Evidence.

### 2026-07-07 — Sentiment cron stopped paging on routine failures
**PRs:** #241 · `Likely`

- **Reason:** Unknown from repo evidence — the PR body is empty. The diff is the only source.
- **Decision:** Category, asset, and weekly sentiment-cron failures logged at `error` (which fans out to the Discord alert channel) were downgraded to `warn`. No retry, catch, or scheduling logic changed.
- **What changed:** Three `this.logger.error(...)` → `this.logger.warn(...)` calls in `sentiment-cron.service.ts` — a 3-line, 6-change diff.
- **Result:** Individual settled-promise rejections inside a batch no longer alert on-call; still visible in application logs.
- **Evidence:** `apps/context/src/sentiment/services/sentiment-cron.service.ts`

### 2026-07-08 — Autonomous scheduled tasks were being re-routed back to the scheduler
**PRs:** #246 · `Confirmed`

- **Reason:** A recurring scheduled task's own trigger language ("monitor", "hourly", "alert when…") looked like a brand-new scheduling request to the planner, so an already-running autonomous job could be reinterpreted and re-routed to the scheduler node instead of executing.
- **Finding:** A real production fixture (`CRON_7e9c5f57`) mixed live-state checks with monitoring language and mis-routed under the old planner context.
- **Decision:** Give the planner explicit context rather than pattern-matching scheduling language away — a `PLANNER_AUTONOMOUS_DIRECTIVE` injected only when `ctx.trade.autonomous === true`, so interactive planning is untouched.
- **What changed:** `PlannerAdapterService` gained the conditional directive injection; the planner E2E harness extended to mirror production message construction for autonomous fixtures.
- **Result:** Autonomous runs keep executing their existing task instead of looping back into scheduler creation. Scope confirmed contained to planner context.
- **Evidence:** New adapter test + regression fixture for `CRON_7e9c5f57`

### 2026-07-09 — DAG context bugs: a fake telemetry alarm, and a real silent TP/SL default
**PRs:** #247, #248 · `Confirmed`

- **Reason:** Two independent problems from Langfuse trace review the same day. (1) Trace/span output capped `finalText` at 500 chars under a name implying it was the full value, so any consumer reading it misreported long-but-complete answers as truncated. (2) On the streaming chat entry point (`dag.runAsync`), `DagContext` was built with hard-coded empty `userInput`/`history`.
- **Finding:** (1) Confirmed false positive: a raw Composer generation of 753 chars, `finishReason=stop`, full text streamed and persisted — only the trace preview was cut, at exactly char 500. (2) Confirmed in production, cross-lingual trace `TG_DM859134181`: a Dutch user's turn N said *"ik wil 5% winst maken waarom dan tp op 100% zetten?"* (why is TP set to 100% if I want 5% profit?); turn N+1 staged a ticket carrying the documented 100%-PnL silent default because the streaming path never received the user's actual 5% instruction. The synchronous `run()` entry point was unaffected.
- **Decision:** (1) Don't touch behavior — add `finalTextLength`/`finalTextTruncatedForTelemetry` alongside the existing capped preview. (2) Pass the real `userInput`/`history` into `buildDagContext` on the streaming path, mirroring the sync path — a one-line-pair fix.
- **What changed:** A shared `finalTextTelemetryPreview` helper across all six DAG telemetry mappers; the two missing arguments in `runAsync`'s `buildDagContext` call. New `dag.service.context-propagation.spec.ts`.
- **Result:** Monitoring can't mistake a capped preview for a truncation; a user's explicit prior-turn TP/SL instruction now actually reaches the trade-execution agent on the streaming path. 12 suites / 66 tests green.
- **Evidence:** Langfuse trace `dbd5ee49bef3dee86f74c41430fa51c2`-adjacent session `TG_DM859134181`; spec fails pre-fix, passes post-fix (verified by revert)

### 2026-07-10 — The composer had no clock, and a finished PnL card was being thrown away on timeout
**PRs:** #253, #254 · `Confirmed`

- **Reason:** (1) The composer's system prompt carried a calendar date but no time-of-day. (2) `share_pnl_card` could finish and produce images in one step, then a later step's 60s LLM abort would fire and reject the whole call — tool results were only lifted after `generateText` resolved, so the already-built card was discarded.
- **Finding:** (1) Langfuse trace (session `TG_DM615280068`) shows the composer fabricating "02:30 UTC" against a real time of ~04:46 UTC, then admitting after correction it had no live clock. (2) Production cases where the card was built 22–52s before the abort fired, and the turn still degraded to a bare "timed out."
- **Decision:** (1) Add a minute-precision UTC clock as its own message positioned *after* the cacheable base prompt, mirroring the planner-adapter's volatile-after-cacheable pattern. (2) Capture completed steps via `onStepFinish`; on a confirmed abort/timeout where `share_pnl_card` ran and no money-moving tool was involved, lift the recovered images instead of rethrowing.
- **What changed:** `ai/utils/current-datetime.ts` (new); composer system-message ordering; `isAbortOrTimeout` + `MONEY_PATH_WRITE_TOOLS` gate reusing the existing `liftToolResults` helper.
- **Result:** Time/date answers are UTC-labelled and computed, not guessed. A PnL card that finished before an unrelated timeout now reaches the user.
- **Evidence:** Langfuse trace `dbd5ee49bef3dee86f74c41430fa51c2`, session `TG_DM615280068`; regression suite for recovery/non-timeout rethrow/mixed write+share/dedup

### 2026-07-13 — Planner phantom tool-calls were silently downgrading real requests to generalChat
**PRs:** #250 · `Confirmed`

- **Reason:** When the planner model understood a query but emitted a non-schema output (a phantom `suggest_plan({subQueries})` call instead of the required decision object), both tiers threw `planner_no_output`, and the DAG substituted `createDefaultPlan()` — `generalChat: true` — silently running zero analysis nodes.
- **Finding:** Production trace `7e0824d9`: user asked "Out of these 5 which has the highest likely success rate," got routed to generalChat, which admitted it couldn't rank anything without fresh stats.
- **Decision:** Add exactly one schema-focused recovery attempt, gated on the fallback failing with `planner_no_output` specifically. Any other fallback failure still rethrows with no wasted third call.
- **What changed:** One recovery re-plan on the fallback model with an added directive forbidding tool-call/`suggest_plan`/`subQueries`/prose output.
- **Result:** A query the model actually understood no longer silently degrades to "I can't help with that."
- **Evidence:** Planner agent recovery specs — double no-output, exactly-one-attempt, fallback-success skips recovery, non-structured-output error skips recovery, genuine generalChat preserved

### 2026-07-15 — LLM failures went dark past "the model errored"; the trade-execution prompt contradicted itself on leverage
**PRs:** [[Features Owned#Observability / agent infrastructure|#258]], #260 · `Confirmed`

- **Reason:** Every `generateText`/`streamText` failure logged only `err.message` — `statusCode`, `responseBody`, `url`, `isRetryable`, request id were dropped, and some failures logged at `warn` (no Discord metadata channel). Separately, the trade-execution prompt still carried a silent 5x leverage default contradicting the confirm-before-write gate, plus contradictory parameter-selection guidance.
- **Decision:** One shared, best-effort `formatLlmError()` extracting the `APICallError` surface, threaded through every failure site — deliberately never throws itself. On leverage: remove the silent default rather than patch around it; 5x survives only as a suggested starting point, never an execution default.
- **What changed:** `ai/utils/llm-error.ts` (new); wired into the shared node-executor catch, twitter findings, composer service + extractor, market-scanner. Also: scheduler re-routing tripwire, planner log-only direction check + schema tripwire, an `ac` chart-plan recovery fix, a shared trading-only scope rule in `COMMON_RULES_BLOCK`.
- **Result:** Provider failures became debuggable from the log line alone; two previously-invisible failure classes (twitter swallow-to-empty, market-scanner no-data degrade) started surfacing. No behavior changed — only log content and level.
- **Evidence:** `llm-error.spec.ts` 6/6 incl. a circular-body case proving the formatter never throws; regression across twitter/findings, market-scanner, composer.service all green

### 2026-07-17 — Charts silently failed to text on HIP-3 assets; the composer promised retries it could never keep
**PRs:** #289, #290 · `Confirmed`

- **Reason:** A chart request for a HIP-3/Trade.xyz asset returned prose only, never a chart. A reduce-only close rejection rendered as generic "please try again" even though the position was already closed elsewhere. After a `tradeExecution` timeout, the composer told a user "I'll keep attempting it… I'll let you know as soon as it's confirmed" — a capability that does not exist.
- **Finding:** Chart bug: the composer's prose-extraction step drops the deployer prefix (`xyz:SPCX` → `SPCX`), and the unlisted-asset guard checks only the core order book. Retry bug: production trace `591ab86366970ed7360cf21350f64400`, session `TG_DM7962497189` — no retry loop, scheduler, or notification hook exists behind a failed step; the user waited ~50 minutes for a confirmation that could never arrive.
- **Decision:** Chart: recover the canonical dex-prefixed name from the asset registry, core-first; when nothing renders, say why. Reduce-only: propagate the thrown reason and point the user at `/resync`. Retry promises: extend the composer's failed-steps rule to forbid promising future/background action, including acknowledgment-phrased standing-rule compliance.
- **What changed:** `ChartService.normalizeSymbol`/`generateOne` returns a discriminated result; receipt mapping reads `parseCoreError`/`out.error`; composer prompt gained the no-future-promises rule plus `composer-failed-retry-eval.ts`.
- **Result:** HIP-3 charts render or fail with a stated reason. The composer can no longer manufacture a retry capability it doesn't have — first eval draft failed 5/5 on a standing-rule query, final wording passed 15/15 across 3 variants.
- **Evidence:** `chart.service.spec.ts`, `receipt.spec.ts`, `execute-ticket.usecase.spec.ts`; `composer-failed-retry-eval.ts` 15/15

### 2026-07-20 — Chasing a recurring Gemini failure meant enriching the error pipeline twice
**PRs:** #300, [[Features Owned#Observability / agent infrastructure|#298]] · `Confirmed`

- **Reason:** Validating `formatLlmError` against a live dev failure — `google/gemini-3.1-flash-lite` throwing "Corrupted thought signature." — showed OpenRouter reports body-level errors as HTTP 200 with an error object, surfaced via `APICallError.data`, not `responseBody` — the most diagnostic field was silently dropped. A loose request-id header match also recorded the literal "MISS" as a fake id. Independently, the basket-generation cron had no telemetry and no Langfuse trace at all.
- **Decision:** Fall back to `APICallError.data` when `responseBody` is absent; tighten request-id matching to two known exact headers. Add the same class of instrumentation to the basket cron plus a read-only replay harness for isolating failure rates.
- **What changed:** `llm-error.ts` `data` fallback + strict header matching; `apps/context` local `formatLlmError`, `onStepFinish` capture, `experimental_telemetry`, `test/basket-repro-harness.ts`.
- **Result:** The next "Corrupted thought signature." occurrence (recurring ~every 1–2 days in dev, 27× in the sampled prod window) now logs the upstream provider name directly — later used to diagnose the Vertex↔AI-Studio reroute behind the Gemini pinning work.
- **Evidence:** `llm-error.spec.ts` 9/9; trace `70701db1b2473dfe5a08af3059e8af63`

### 2026-07-21 — A trade-execution trio: fractional leverage, a wrong root-cause string, and an unhandled Telegram rejection
**PRs:** #305, #309, #310 · `Confirmed`

- **Reason:** (1) `open_position`'s schema bounded leverage 1–100 but not integrality, so a derived value like 5.47x passed validation and only failed at the backend post-confirm. (2) A real partial-fill incident (ticket `dcfb9d9c…`) left a user unhedged, and both the alert and receipt blamed a hardcoded "$11 minimum" guess the actual numbers (23× above the floor) proved wrong — the real cause was a venue-side rejection. (3) An unguarded `setInterval` polling `/linkwallet` let a grammY 403 escape as an unhandled rejection, paging Discord.
- **Decision:** (1) Fail fast at schema validation. (2) Stop asserting a specific cause the code can't know — report only what the fills prove. (3) Guard the timer body; treat 403 as an expected lifecycle event, not a system fault.
- **What changed:** `.int()` on `open_position`'s `leverage`; a 15-line rewrite of the PARTIAL FILL message in `execute-core.ts`; timer body extracted to `tick()`, invoked as `void tick().catch(...)`.
- **Result:** Fractional-leverage requests retry pre-confirm instead of failing post-confirm. Partial-fill alerts stop misdirecting toward "increase margin/notional." A blocked/deleted bot no longer pages on-call.
- **Evidence:** `execute-ticket.usecase.spec.ts` 24/24, `receipt.spec.ts` 44/44; ticket `dcfb9d9c-967b-42e0-8958-a24fde7218ab`

### 2026-07-22 — HIP-3 orders were passing margin validation they could never actually fund
**PRs:** #324 · `Confirmed`

- **Reason:** Under Hyperliquid's "Disabled (No Dex Abstraction)" mode, a HIP-3 DEX holds isolated collateral core USDC cannot back — so a HIP-3 leg always reports `availableToTrade = 0`, exactly the signal that triggered a "backfill from core USDC" pass. For a No-Dex-Abstraction account with an empty HIP-3 pool, that let an unfundable order pass validation and fail at execution with "Insufficient margin to place order" — repeatedly, for real `xyz:` stock-perp traders.
- **Decision:** Gate the backfill on the account's actual abstraction mode rather than removing it — must not break dex-abstraction/unified accounts. Fail open if the mode probe errors.
- **What changed:** New `HyperliquidMarketService.getNoDexAbstraction(wallet)` (cached 30s); the backfill branch in `open-position.tool.ts` now skips only when a HIP-3 USDC leg is present *and* the account is No-Dex-Abstraction.
- **Result:** That population now gets a clean pre-Confirm rejection with actionable guidance instead of a confusing execution-time failure. Every other population is byte-identical.
- **Evidence:** `open-position.hip3-margin.spec.ts` 3 cases; 36/36 sibling specs green

### 2026-07-23 — /aipear was advertised in DMs where it silently did nothing
**PRs:** #327 · `Confirmed`

- **Reason:** `/aipear` was registered in Telegram's private-chat command menu, but grammY's `bot.command("aipear")` consumed the update and returned early for private chats — the `onText` fallback that would route it to the agent never ran.
- **Decision:** Stop advertising it where a bare prompt already works, but still handle it gracefully if typed.
- **What changed:** `/aipear` scoped to groups only in the menu; typed `/aipear …` in DM now routes through onboarding→wallet-gate→agent; `buildTelegramCommands()` extracted into an import-free module; `scripts/check-tg-commands.ts` added.
- **Result:** Verified live against a throwaway bot: DM menu count dropped 22→21; group menu unchanged.
- **Evidence:** Live Telegram verification — `PASS`

### 2026-07-27 – 07-29 — A "would increase position" rejection didn't mean the trade failed; Twitter labeling failures were unreadable
**PRs:** #337, #340 · `Confirmed`

- **Reason:** A reduce-only ticket freezes an absolute per-leg target; if a sibling close or direct-exchange trade flattened that leg first, Hyperliquid rejects the stale order — and the executor treated every such rejection as a hard FAILED, even when the user's actual requested end-state (flat) had already been achieved. Separately, `AI_NoObjectGeneratedError` recurred across many tweets but the catch logged only message + stack.
- **Decision:** Verify before assuming — re-read the freshest live position and downgrade FAILED to benign only when every targeted leg is provably at target; asymmetric by design, can only ever downgrade, never upgrade uncertainty into success. Twitter: log the SDK's own diagnostic fields so the failure mode names itself.
- **What changed:** A guard + read-only helper in `ExecuteTicketUsecase` using `getPositions(syncPosition=true)`; `TwitterLabelingAgent.analyze()`'s catch logs raw-output shape, finish reason, token usage, model-served-vs-configured.
- **Result:** One previously-FAILED outcome class now settles benign, only when a fresh live read positively confirms it — fully backward compatible. Six explicit follow-ups documented and deliberately deferred.
- **Evidence:** New position-sync verification path in `ExecuteTicketUsecase`; diagnostic logging in `analyze()`

### 2026-07-28 – 08-01 — Basket generation could only produce two-sided baskets, and had no way to tag them
**PRs:** [[Features Owned#Basket generation|#351]], [[Features Owned#Basket generation|#352]], [[Features Owned#Basket generation|#367]] · `Likely`

- **Reason:** Baskets had no semantic classification for discovery, and the pipeline structurally forced every basket to have both a long and a short side — even when the upstream category analysis had genuinely one-sided conviction, because its own prompt always asked for "2–3 bullish **and** 2–3 bearish categories."
- **Decision:** Enforce the tag vocabulary structurally (a closed Zod enum), clamp rather than throw on a bad tag. For one-sided baskets, bypass pair-stat enrichment entirely rather than inventing meaningless spread/correlation numbers. Explicitly exclude state labels (Trending/New/Hot) since the model can't know them at generation time.
- **What changed:** `agents/basket/tags.ts` (18-value two-tier taxonomy), `normalizeBasketTags`, a directional branch skipping `getBasketStats` for one-sided output; a follow-up (#367) collapsed the tag model to a single tag after real usage.
- **Result:** Baskets can now be two-sided, long-only, or short-only, and carry a queryable tag. Tests grew from 13 to 25 (later simplified with the single-tag change).
- **Evidence:** Explicitly blocked merge/deploy on an external DB migration (`pear-stat-arbs#93`) landing first, since a client aware of the new column against a database without it breaks every read of `made_baskets`

### 2026-07-30 – 07-31 — Group trade pitches showed execution buttons that only worked in DM
**PRs:** [[Features Owned#Telegram / product surface|#362]] · `Likely`

- **Reason:** Telegram group trade pitches already redirected users to DM to execute (via a `REDIRECT_TO_DM` sentinel), but the metadata-extraction phase still emitted Execute/Adjust/Skip buttons underneath regardless — a group message told users to go to DM and offered execution buttons that couldn't execute from there.
- **Decision:** Use the sentinel already emitted as a deterministic discriminator rather than adding new plumbing — an explicit ordered decision tree, stopping at the first match.
- **What changed:** The `EXTRACT_INSTRUCTION` prompt only; also folded in same-window Redis atomic-consume work for suggested-action callbacks (`getDel`).
- **Result:** Group trade pitches now surface conversational follow-ups; DM execution behavior unchanged.
- **Evidence:** Regression coverage for group/DM/analysis/greeting paths and pending-ticket atomicity

### 2026-08-03 – 08-06 — The agent couldn't see the Basket Builder draft a user was actively editing
**PRs:** [[Features Owned#Basket generation|#379]], [[Features Owned#Telegram / product surface|#380]], [[Features Owned#Telegram / product surface|#390]] · `Likely`

- **Reason:** Asking the agent to "review my basket" had no way to reach the in-progress Basket Builder draft — the agent could only invent one. Separately, no visibility existed into which Telegram slash commands were actually being used.
- **Decision:** Thread an optional, request-scoped, never-persisted `runtimeContext` the way `imageKeys` already flows; treat the client-supplied draft as untrusted data — fenced, bounded, neutralized against injection — never as system content. Count command intent (before the wallet gate), not just successful execution.
- **What changed:** New `RuntimeContext`/`BasketContext`/`BasketLeg` schemas in `@pear-protocol/agent-sdk` (additive, minor changeset); two injection seams (planner-adapter hint, basket-node-only prompt prepend); a grammY middleware incrementing `tg:cmd:count:<command>` in Redis (21-day TTL).
- **Result:** Non-basket turns byte-identical; only the basket node ever sees the draft. 22 unit tests plus a dedicated prompt-injection suite — SDK 56/56, contract 29/29.
- **Evidence:** Agent-SDK dual-maintenance rule followed: schemas, client methods, drift-guard, changeset, README all updated together

### 2026-08-05 – 08-13 — Model rate-limit whack-a-mole: Gemini → Luna → Flash Lite, and MiMo → Gemini → Luna
**PRs:** #384, [[Features Owned#Model routing|#386]], [[Features Owned#Model routing|#387]], #385, #436, #437, #399, #478, #479 · `Confirmed`

- **Reason:** Over roughly a week, three different upstream models hit provider-side rate limits or capacity walls in production: `google/gemini-3.6-flash` (tradeExecution) under burst load, then its replacement `gpt-5.6-luna` across ten node configs, then Luna itself, then the advanced-mode orchestrator's direct MiMo integration (no cross-model failover by design) exhausting retries on HTTP 429.
- **Decision:** Each swap explicitly distinguished rate-limiting (retriable, swap-worthy) from upstream deprecation (a flat 502, different problem), and validated the replacement live before merging.
- **What changed:** Model-string swaps only in `apps/agent/src/config/models.ts` / `apps/context/src/config/models.ts`. Also: a GPT-OSS output cap (`maxOutputTokens: 4096`) keyed on model name rather than tier, after the tier-based version let an A/B override diverge from production; a bakeoff fixture bug (#385) where a stub position with no `takeProfit` made the TP-preservation contract impossible to exercise.
- **Result:** Production stayed on a working model at each step, at the cost of several same-week reversals — the strongest repo-visible evidence of upstream model instability in this window.
- **Evidence:** Flash Lite bakeoff 1/1; Luna 8/8 concurrent burst, no 429s; Gemini advanced-mode smoke test ~5.4s with streamed reasoning

### 2026-08-07 — Projected token-unlock amounts were being shown to users as confirmed facts
**PRs:** #391, #392 · `Confirmed`

- **Reason:** `getEffectiveUnlockMetrics` presented the raw `cliffAmount` — a scheduled/projected vesting ceiling — as if it were project-confirmed, e.g. "HYPE has a 9.9M unlock on Sept 6," even with no `committedClaim` on any allocation in that event.
- **Finding:** Checked in production data: HYPE was the *only* asset with any committed claim at all — the other 49 assets with unlock schedules were 100% null `committedClaim`, so all were exposed to the same overstatement.
- **Decision:** Suppress the amount/value/market-cap-impact fields entirely when no committed claim exists, rather than caveat the number.
- **What changed:** `tokenomist.service.ts`; `asset_tokenomics` now states "No confirmed unlock amount is available"; `get_upcoming_unlocks` drops uncommitted events from ranking.
- **Result:** Verified live against real Tokenomist data — "next HYPE unlock" now returns date + allocation with no confirmed amount.
- **Evidence:** `tokenomist.service.spec.ts` 5/5, `upcoming-unlocks.tool.spec.ts` 2/2

### 2026-08-13 – 08-14 — Chart edge cases: paging on bad input, an ambiguous ticker crashing the whole reply, and wrong display tickers
**PRs:** #469, #431, #481 · `Confirmed`

- **Reason:** Slash-command chart sends logged every failure at `error`, paging Discord twice on a Telegram `400 IMAGE_PROCESS_FAILED` — a bad-render input, not a system fault. A basket chat turn crashed into a fully generic error whenever a bare ticker was listed under more than one HIP-3 deployer (production, `AVGO` → `para:AVGO`/`xyz:AVGO`, twice ~4 minutes apart, trace `1848c72d…`) — the caller-facing `SymbolResolutionError` was never caught in the chart path. Basket generation persisted quant-database IDs (e.g. `1000PEPEUSDT`) as display tickers instead of the Hyperliquid market name (`kPEPE`).
- **Decision:** Scope the Discord page to systemic failures only (401/429), downgrade client-render edges (400/403) to warn. Catch `SymbolResolutionError` specifically — not a blanket catch-all — surfacing both valid prefixed options. Map quant IDs to Hyperliquid tickers only at the persistence boundary, failing closed when the asset index is cold.
- **What changed:** `chart.service.ts`'s `sendChartResponse` catch; a narrow try/catch in `ChartService.generateOne`; a persistence-boundary mapping step with regression cases for two-sided, directional, cold-index, listing-ranking scenarios.
- **Result:** 24/24 chart-service tests pass (22 existing + 2 new); the money path (`packages/execution`) untouched throughout.
- **Evidence:** Trace `1848c72d…` — ambiguous AVGO, twice in ~4 minutes

### 2026-08-17 — Stale close-all SDK contract, unsafe alias resolution, and the development→main promotion review
**PRs:** #493, #472, #515 · `Likely`

- **Reason:** The V3 close-all endpoint started returning terminal executions, but the installed core-sdk version still carried the older response contract, risking a crash on `.flatMap` after a close had already succeeded. Models could relay a user's human-readable account alias instead of the UUID shown in the choice list, and the prior UUID-only lookup rejected that retry outright.
- **Decision:** Bump the SDK, default a missing executions array instead of crashing. Resolve only unambiguous, case-insensitive alias matches evaluated against the full account list before the supported-venue gate — a duplicate alias fails closed instead of guessing.
- **What changed:** `@pear-protocol/core-sdk` 1.12.1 → 1.16.0; alias resolution added to both agent and MCP adapters, persisting the resolved UUID. #515 was a promotion-readiness draft enumerating the `development`/`main` divergence (103 commits ahead / 12 behind, 278 files) — not itself a code change.
- **Result:** A missing executions field no longer crashes response handling. Unique aliases resolve; duplicates fail closed.
- **Evidence:** Focused account-resolution suite 18/18 (agent), 7/7 (MCP)

### 2026-08-17 – 08-20 — The agent claimed a live product feature "doesn't exist"
**PRs:** #520 · `Confirmed`

- **Reason:** Leaderboard/Hypear/X-handle questions weren't routed anywhere useful by the planner, and the KB's own docs described the live leaderboard in terms implying it wasn't real.
- **Decision:** Fix routing and the source doc together, and — per this repo's KB dual-maintenance rule — rebuild the committed embedding artifact in the same change (the agent searches that artifact in-process; a doc edit without a rebuild is inert at runtime).
- **What changed:** Planner routes leaderboard/Hypear questions to generalChat; KB doc restated as a canonical "Hypear is…" definition; bare "X" disambiguated to "X (Twitter)".
- **Result:** Eval showed no regression vs. main: routing top-1 75% (24/32), retrieval @1/@3/@5 97%/100%/100%, MRR 0.984, surface filter PASS.
- **Evidence:** `kb:build` + `kb:eval` output quoted directly in the PR

### 2026-08-18 — Elfa signal triggers accepted invalid conditions, and close-all could silently downgrade a TWAP to market
**PRs:** #553, #491 · `Confirmed`

- **Reason:** Elfa's validation endpoint returns HTTP 200 with a body-level `valid: false` for an invalid condition, but the client treated any 2xx as success — so an invalid condition armed a query that then failed with a 422 and retried hourly, indefinitely. An account-wide close-all could silently downgrade a user's explicit TWAP instruction to market.
- **Decision:** Normalize on the field the provider actually uses; fail closed unless it explicitly says `valid: true`. Keep close-all market-only and require an explicit fallback choice before changing execution mode; sequence individual TWAP closes one ticket at a time.
- **What changed:** `ElfaClient.validateQuery` normalization + regression coverage for the HTTP-200/`valid:false` shape; close-all tool/prompt/regression suite for TWAP sequencing.
- **Result:** An invalid X-account condition is now rejected up front instead of arming a query doomed to fail hourly. A TWAP-instructed close-all can no longer be quietly executed as market orders.
- **Evidence:** Focused Elfa client + TriggerManager suites passing

### 2026-08-18 – 08-20 — Bringing up Lighter as a second execution venue
**PRs:** [[Features Owned#Multi-venue execution: Lighter integration|#556]], [[Features Owned#Multi-venue execution: Lighter integration|#557]], [[Features Owned#Multi-venue execution: Lighter integration|#558]], [[Features Owned#Multi-venue execution: Lighter integration|#559]], #560, [[Features Owned#Multi-venue execution: Lighter integration|#564]] (own) + authored commits inside teammate-opened #542, #545–548, #615 · `Confirmed`

- **Reason:** Adding Lighter as a second venue (Epic #521, Story #524) touched almost every place the system assumed Hyperliquid was the only venue: the only current-price tool was HL-only, so an agent-derived conditional level on a Lighter account was anchored to the wrong exchange's price. The invocation-scoped account resolver memoized explicit vs. omitted `tradeAccountId` calls under different keys, so an explicit price read could pin account A while a later omitted write reused a stale cached default B. The asset-cache price store merged each `/prices` response into the previous map, so an omitted symbol could keep a stale price while its freshness timestamp kept advancing. The KB still said triggers were "enforced by HL" — actively wrong once a position can be on Lighter.
- **Finding:** Verified live against the gateway (2026-08-18): all 227/227 Lighter instruments report `marginMode="any"` — confirming the existing cross-margin default was safe today, while flagging the isolated-margin predicate as needing to become data-driven the moment that changes.
- **Decision:** Add a venue-aware price-read tool keyed strictly off the resolved trade account's connector — never model-provided venue text. Pin the resolved account within one agent invocation. Make a successful (including empty) price snapshot fully replace the prior one. Correct the KB to name "Pear Execution Engine, per the position's connector." Harden the one integration test meant to prove a Lighter trigger arms, which had two blind spots that would pass even if nothing armed.
- **What changed:** New `execution_venue_price` read tool; an invocation-scoped account-resolver pin inside `TradeExecutionAgent.invoke()`; replace-not-merge semantics in `AssetCacheStore`; KB doc correction + rebuilt embedding artifact; Telegram multi-account chooser with read-only selection that never mutates the stored execution default.
- **Result:** 7 unit tests for the venue-price tool; 9+27+26 passing across invocation-resolver, surrounding account/venue, and V3 default-account suites; KB eval held at retrieval @1/@3/@5 97/100/100, MRR 0.986, all new Lighter eval queries ranking 1st.
- **Evidence:** Live gateway check 227/227 Lighter instruments, `marginMode="any"`; commits landed both as own opened PRs and as authored commits inside teammate GPSxtreme's umbrella PRs on the shared `lighter-integration` branch

### 2026-08-21 — Surface copy still said "Hyperliquid" regardless of the account's actual venue
**PRs:** [[Features Owned#Multi-venue execution: Lighter integration|#574]] · `Likely`

- **Reason:** Surface copy across flex/touch sweeps and signal CTAs was hardcoded to Hyperliquid language, and one bad account's read failure could sink an entire batch sweep.
- **Decision:** Name the venue the account is actually on; isolate per-account failures so one account's error can't take down the batch.
- **What changed:** Venue-honest copy in surface rendering; per-account isolation in flex-watcher and touch-position-sweep; account chooser for signal-CTA tickets on multi-account users.
- **Result:** A stale `/flex` assertion for account-scoped `getOpenPositions` was updated to match.
- **Evidence:** New unit specs for flex-watcher and touch-position-sweep per-account isolation

### 2026-08-24 — A single day of hardening the Lighter surface: wiring gaps, a doc that contradicted itself, and a crash that silently substituted the user's request
**PRs:** #587, [[Features Owned#Basket generation|#588]], #589, #584 · `Confirmed`

- **Reason:** `validate_assets` only widened to allow Lighter when `forcePlatform` was explicitly set — but on Lighter-capable surfaces (tg_dm/web) that flag is exactly the value that's never set, so the widening never fired. The basket-stats tool had the same gap one layer deeper, defaulting an omitted platform to Hyperliquid and contradicting what `validate_assets` had just told the model. The KB doc said triggers are "enforced by HL" in one place and "enforced by the selected venue" in another — both wrong. Most seriously: `open_position`'s TRIGGER/LADDER `condition` argument was a JSON string spread `as never` into the plan usecase with no schema check — found while triaging five days of Langfuse ERROR-level traces — so a malformed shape crashed with a raw `TypeError`, the model silently retried with a different (wrong) condition type, and the user's actual weighted-basket trigger was replaced with a single-symbol price trigger with no error shown.
- **Finding:** Trace `b9164502960cd3af356c603488ee5657`: the model sent a `weighted_ratio` condition with a `symbol` field instead of the required `basket` array, crashed on `undefined.map`, then quietly retried with a `price`-type condition and succeeded — silently changing what the user asked for. The MCP adapter already validated this shape correctly via a Zod discriminated union; only the agent/Telegram adapter took an unvalidated JSON-string shortcut.
- **Decision:** Rebuild the two Lighter-surface tools to unconditionally rebuild their enum per invocation. Reuse the exact schema the MCP adapter already trusts to validate the parsed condition before it reaches the usecase — a bad shape now fails as a named `PlanningError` instead of crashing and silently substituting a different trade. Correct the KB wording; harden the integration test that would pass even if nothing armed.
- **What changed:** `agents/quant/pair/agent.ts` and `basket/agent.ts` rebuild `validate_assets`/`quant_basket_stats` unconditionally with `allowLighter: !input.forcePlatform`; `open-position.tool.ts` validates parsed TRIGGER/LADDER JSON against shared Zod schemas; `capabilities-and-limits.txt` corrected in both spots, artifact rebuilt in the same commit; `d.tpsl.spec.ts` D6 hardened to require a positive read-back.
- **Result:** A malformed TRIGGER/LADDER condition now fails with an actionable message instead of crashing and silently swapping the user's request. KB eval held (routing 75%, recall@1 94%, MRR 0.972).
- **Evidence:** New `open-position.tool.spec.ts` — malformed shape rejected pre-usecase; 8/8 new basket-stats enum tests

### 2026-08-24 — A confirm-time margin gate, a recurring ambiguous-symbol crash, and a hidden 2x-budget confirm
**PRs:** [[Features Owned#Trade execution|#569]], #514, [[Features Owned#Multi-venue execution: Lighter integration|#594]], #593 · `Confirmed`

- **Reason:** `open_basket` had no check that a fresh margin read still covered the plan at confirm time. The earlier ambiguous-HIP-3-symbol fix (#431) only covered chart-rendering — every other `DagService` entry point (pair-stats analysis, `open_position`, general compose) still fell through a generic catch, discarding the actionable disambiguation text the error already carried. Confirmed live: the exact "AVGO is listed by more than one deployer" reply recurred 13 times in the 3 days before this fix. And when a leg's leverage cap forced a downgrade, the engine's notional-preservation behavior re-derives margin — an explicit margin input could silently come back roughly doubled — but the ticket only mentioned the leverage adjustment, not the margin delta.
- **Finding:** Trace `0e25d734e9eb599b94a8b4c30ee51249` — a plain z-score/correlation query, not a chart request — still returned the generic error fallback. Trace `5a3192cee9b685c8a3e6e8201caf58ad` (session `TG_DM1110963912`) — a user asked for "$1,000 margin at 10x leverage," INJ got capped to 5x, resulting margin was $1,999.95 with no standalone callout.
- **Decision:** Fail-closed confirm-time margin gate with a configurable 100bps buffer, exempting reduce-only adjustments. Catch `SymbolResolutionError` specifically in the two remaining `DagService` entry points. Require the composer to state the margin change as its own sentence, before the leverage-cap note — correctly a prompt-level fix, since the preceding prose is LLM-composed, not code-generated. Replace `/linkwallet`'s typed-arg venue selection with inline venue buttons.
- **What changed:** A margin gate reading fresh `availableToTrade` before leverage/order writes, with replayable diagnostics; `SymbolResolutionError` handling added to `DagService.run` and `runAsyncInner`; `trade-execution/prompt.ts` requires the standalone margin-change sentence; `/linkwallet` mints its code only after venue selection.
- **Result:** Insufficient/unavailable/changed-balance cases covered by focused tests. The ambiguous-symbol crash is closed at every entry point — full DAG suite 8/8 files, 60/60 tests green. A leveraged-clamp trade now discloses the margin change before confirmation.
- **Evidence:** `dag.service.symbol-ambiguity.spec.ts` — 3 new tests proving the fix isn't a blanket catch-all; 26/26 `telegram.service.linkwallet.spec.ts` + dispatch tests

### 2026-08-28 — The internal-leak guard blocked legitimate AI-industry reporting; signal failures collapsed two mechanisms into one alert; delivery ordering had no visibility *(open)*
**PRs:** #639, [[Features Owned#Observability / agent infrastructure|#640]], [[Features Owned#Observability / agent infrastructure|#641]] · `Confirmed`

- **Reason:** Model/provider names like `gpt-5`, `gemini-3`, `openrouter` are simultaneously real internal identifiers and ordinary AI-industry vocabulary. Reproduced in production: a Z.AI/Zhipu monitor's news summary saying "outperforming some GPT-5.6 benchmarks" got its whole reply nuked by the unconditional substring deny-list. Terminal `SignalProcessor` evidence failures collapsed two distinct root causes (a fresh re-measurement genuinely missing the condition, vs. a crossing that's met but lacks authenticated Elfa execution proof) into one undifferentiated `predicate_not_satisfied` alert. No way existed to tell whether an older Telegram turn's media landing after a newer turn's text was an ordering bug or client rendering quirk.
- **Decision:** Move model/provider names out of the unconditional deny-list into a sentence-scoped check firing only on an actual first-person disclosure frame — distinctive internal identifiers stay hard-blocked regardless. A same-day follow-up found the first cut had narrowed too far (missing "based on X," "running on X"), so the frame was widened without restoring unconditional blocking. For signals: capture the failure diagnostic inside the real evaluator traversal so it can never drift from the boolean verdict. For delivery: add a pass-through, opt-in observer correlated via `AsyncLocalStorage` — observability only.
- **What changed:** `internal-leak-guard.ts`: new `MODEL_PROVIDER_TOKENS` sentence-scoped check, `findInternalLeak()` returning `{category, matched}` for match observability without replaying the withheld draft. `signal-evidence.ts`: detailed evaluator variants returning a typed `PredicateFailureDetail`. `delivery-observer.ts`: a grammY API transformer logging send_start/complete/error per turn.
- **Result:** Leak-guard: 60/60 tests including adversarial and benign near-miss cases. Signals: 40/40 including an explicit boolean⇔detailed parity test.
- **Evidence:** Not yet merged as of 2026-08-30 — status: open

### 2026-08-30 — Rebalance weights were rejected instead of rescaled *(open, current branch)*
**PRs:** #642 · `Confirmed`

- **Reason:** Rebalance target weights are shares of one whole; a sum that isn't exactly 1 is never a genuine target, only arithmetic drift from an LLM sizing 13–15 leg baskets to 4 decimal places. The hard reject in `plan-rebalance.usecase.ts` just burned retries.
- **Finding:** Traced 4 traces / 21 rejected `rebalance_position` attempts with drift up to 15% — one trace shows 10 back-to-back self-correction attempts (1.1488 → … → 0.9998) before landing inside tolerance.
- **Decision:** Rescale each weight proportionally (`w / sum`) before the leg mapping instead of throwing — the relative shares are the caller's intent, the absolute sum is the arithmetic error. Applied only outside the existing tolerance, so a clean sum stays bit-exact.
- **What changed:** `plan-rebalance.usecase.ts` in `packages/execution` — shared by both agent and MCP adapters. A same-day follow-up disclosed the rescale behavior in user-facing copy and dropped stale sum-to-1 documentation.
- **Result:** Strictly more permissive and backward compatible — the rebalance engine already accepted the 0.9998-sum outputs; a rescaled set lands within 1e-15 of 1.0.
- **Evidence:** Per-leg weights already `zod .positive().max(1)`, so the sum is provably > 0 before division; not yet merged as of 2026-08-30 — the branch this record was requested from

---

## Recurring Engineering Themes

Only themes with independent support across multiple, separately-dated pieces of work.

**Trade-execution safety and disclosure** — A running series of fail-closed gates and honest disclosure fixes on the money-moving path: fractional-leverage rejection, HIP-3 margin backfill, confirm-time margin checks, TWAP close-all protection, TRIGGER/LADDER schema validation, rebalance-weight handling, margin-change disclosure on leverage clamps.
`#305 #324 #337 #491 #569 #584 #593 #642`

**Turning silent failures into diagnosable ones** — Instrumenting rather than guessing: shared LLM-error formatting, basket-cron telemetry, Twitter-labeling diagnostics, self-diagnosing signal-evidence failures — each built to answer "which of N mechanisms fired" from the log line alone.
`#247 #258 #298 #300 #340 #514 #431 #640`

**Multi-venue execution correctness (Lighter)** — Bringing Lighter online surfaced Hyperliquid-only assumptions across pricing, margin mode, account binding, KB wording, Telegram surface copy — spanning both self-opened PRs and authored commits inside teammate-led umbrella PRs.
`#545 #546 #547 #548 #556 #557 #558 #559 #560 #564 #574 #587 #588 #589`

**Model routing under provider instability** — Repeated same-week emergency model swaps in response to real upstream rate limits and capacity walls, each validated live before merge.
`#384 #386 #387 #436 #437 #399 #478 #479`

**Telegram surface reliability** — Production-facing bot hardening: an unhandled-rejection guard, a dead DM command, a broken group→DM funnel, Discord-paging chart sends, a two-phase venue picker.
`#310 #327 #362 #469 #594 #641`

**Constraining LLM behavior to ground truth** — Stopping the agent from asserting things it cannot know or promising capabilities it doesn't have: fabricated retry promises, an unconfirmed leverage default, a buried margin change, an over/under-broad leak-detection frame.
`#290 #260 #593 #639`

**Knowledge-base correctness for agent retrieval** — Enforcing this repo's KB dual-maintenance rule in practice: doc changes, index updates, and the committed embedding artifact rebuilt together, with an eval run each time to confirm no retrieval regression.
`#520 #559 #589`

---

## Strongest Resume / Career Evidence

Ranked by technical difficulty, ownership, safety impact, and how well the repo can substantiate the claim — not by PR size.

1. **Root-caused a silent trade-execution defect from a cross-lingual production trace (#248).** A Dutch user's explicit 5% TP instruction was silently overridden by a documented 100% default because the streaming entry point never threaded real user input into the DAG context. Found via direct trace reading, root-caused to a one-line-pair fix, proven with a regression spec that fails on the pre-fix code.

2. **Designed a confirm-time margin safety gate (#569).** Fail-closed check against fresh `availableToTrade`, configurable buffer, positioned before any leverage or order write, deliberately exempting reduce-only adjustments. A genuine safety feature, not a bug fix.

3. **Found and closed a crash that was silently substituting a user's trade request (#584).** An unvalidated TRIGGER/LADDER JSON payload crashed, the model quietly retried with a different condition type, and the user's request changed with no error shown. Found by manually triaging five days of Langfuse ERROR traces.

4. **Closed a recurring 13-in-3-days production error at its remaining call sites (#431, #514).** An ambiguous HIP-3 ticker crashed chart generation, then kept crashing every other DAG entry point via the same underlying error. Shows the discipline to verify a narrow fix actually covered the whole failure surface.

5. **Disclosed a hidden ~2x margin exposure on leverage-clamped trades (#593).** Traced to one specific user's actual numbers ($1,000 → $1,999.95), fixed at the correct layer (prompt, not code) after confirming the underlying engine behavior was intentional.

6. **Quantified and fixed retry-storming on rebalance weight validation (#642, open).** Measured 4 traces / 21 rejected attempts / up to 15% drift before deciding to rescale rather than reject — a data-backed validation-design decision with a stated, provable safety property.

7. **Removed an unsigned-webhook fallback from the signals security surface (#555).** Every armed production Elfa webhook now requires HMAC verification; existing armed queries detected as drift and re-armed post-deploy.

8. **Built an asymmetric, downgrade-only verification gate for reduce-only failures (#337).** A rejected order gets re-verified against a fresh live position read and only ever downgraded, never upgraded — every ambiguous case preserves the original failure.

9. **Tuned an LLM-output security guard against a reproduced false-positive/negative pair (#639 + follow-up).** Widened a leak-detection guard, then caught its own over-correction via review before it shipped, backed by 60/60 adversarial+benign tests.

10. **Built the shared LLM-error observability layer later credited with diagnosing a Gemini incident (#258, #300).** Infrastructure investment that paid off in a documented later incident, not observability for its own sake.

11. **Breadth across a full new execution venue (Lighter, multiple PRs across two weeks).** The same underlying assumption (Hyperliquid-only) had to be found and fixed in five or six structurally different places across `apps/agent`, `packages/execution`, and the KB pipeline.

12. **Live-verified every emergency model swap before merging (#384–#479 sequence).** Operational discipline under repeated pressure — the same rigor applied on the fifth emergency swap of the week as on the first.

---

## Missing Context Worth Adding Manually

Places where the repository cannot tell the whole story — fill these in from memory before treating this record as complete.

- **Why three different models rate-limited in the same window.** The repo shows the swaps but not the underlying cause — provider capacity, an OpenRouter pooling tier, contract limits, or a traffic spike.
- **Division of labor on the Lighter integration.** Commits show this author's code interleaved with teammate GPSxtreme's inside shared umbrella PRs (#542, #545–548, #615). The repo shows *what* each person wrote, not how the work was planned or divided in discussion.
- **Real-world impact of the incidents cited.** Several entries quote a specific trace and user session (#248, #514, #593, #642) — but the repo has no way to show how many other users hit the same bug, or whether any caused a support ticket, refund, or complaint.
- **Rationale behind specific numeric thresholds.** The 100bps margin buffer (#569), the GPT-OSS 4096-token cap (#399), the Elfa HMAC rollout sequencing were each stated as decisions but not argued from a documented discussion.
- **Post-deploy outcomes.** No entry includes a "since this shipped, error rate X dropped to Y" data point — the repo has no monitoring-dashboard exports, only pre-merge test and eval output.
- **Start date and pre-window history.** 2026-07-07 is the earliest commit visible in this branch's history — whether that's this author's actual start date at Pear, or just the start of retained git history, is unknown from repo evidence alone.
- **Current status of the two open PR clusters.** #639/#640/#641 and #642 were open as of 2026-08-30. Whether they merged as-is, changed under review, or were superseded needs checking against live PR state.

---

*Generated from git history and GitHub PR data in pear-protocol/agent-pear-v2, as of 2026-08-30. Author filter: prakharDvedi.*