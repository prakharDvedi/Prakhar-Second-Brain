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

### 2026-09-02 — One service URL fed two different services, nulling two-sided basket stats
**PRs:** #677 (→ `main`, merged), #678 (→ `development`, open) · `Confirmed`

- **Reason:** In `apps/context`, one env var `STATS_BASE_URL` fed two consumers whose routes live on different services — `AssetUniverseProvider` → `GET /categories` (TS categories service, `pear-pair-signals`) and `QuantService.getBasketStats()` → `POST /stats/basket` (Rust stats service, `pear-stat-rs`). An Aug 27 change (`11a29155`) collapsed both onto the categories host on the assumption `/categories` was the only consumer.
- **Finding:** `POST /stats/basket` then 404'd on `pear-pair-signals`; baskets still generated and persisted but with null stat columns, and the `MadeBaskets` cron raised a misleading `No baskets generated` alert. Prod `context-app` carried an explicit `STATS_BASE_URL` Railway override (so the code default alone never governed it). Confirmed via live route probes that the two endpoints genuinely live on different services.
- **Decision:** Split the var — `CATEGORIES_BASE_URL` for `/categories` (`pear-pair-signals`), `STATS_BASE_URL` restored to `/stats/basket` (`pear-stat-rs`). `indexing`, a `/categories`-only consumer, had its var renamed rather than carry a dead second one. Both URL vars moved to `z.url()` so a bad override fails at boot.
- **What changed:** `apps/context` config + `asset-universe.provider.ts`; `apps/indexing` config + `tokenomist-index.service.ts` (+ spec); env templates + READMEs. Shipped as an emergency hotfix straight to `main` (#677), then backported to `development` (#678) as a clean cherry-pick to re-level the branches.
- **Result:** `tsc` clean (both apps); tokenomist spec green; verified live — `GET /categories` 200 pair-signals / 404 stat-rs, `POST /stats/basket` 201 (valid multi-leg) stat-rs / 404 pair-signals. Prod vars updated by owner, both services redeployed SUCCESS, 8/8 services Online. Dev env needed no var change (no explicit override there).
- **Evidence:** [[One Service URL Fed Two Different Services, Nulling Basket Stats]] · [[2nd Sep 2026]]

### 2026-09-03 – 09-05 — Evaluated Claude basket research against OpenRouter *(open experiment)*
**Branches:** `feat/live-twitter-basket-context` (stable single-call comparison) · `feat/claude-basket-research-stage` (experimental causal research stage) · source artifact: `C:\Users\prakh\Downloads\evals.pdf`

- **Reason:** Senior's intended basket objective is research-first: current sentiment and strong internet research should drive asset selection, while correlation and related pair statistics act as a booster/validator rather than the sole objective. A controlled dry-run evaluation was needed before changing the production basket path.
- **Inputs held constant:** Development Context DB snapshot, platform-specific asset universe, Pear `assets_by_category` and `quant_basket_stats` tools, basket schema/final validation, and dry-run/no-persistence mode. Claude used Opus with optional WebSearch/WebFetch; OpenRouter used the legacy in-process path. These are application-level parity controls, not byte-identical model executions.
- **Evaluation set:** 20 baskets total from the PDF: Claude + web fetch (4: 2 Hyperliquid, 2 Lighter), Claude without web fetch (6: 3/3), OpenRouter legacy (4: 2/2), and OpenRouter legacy without web fetch (6: 3/3). The PDF's scorecard labels both OpenRouter columns identically; the grouping above follows the section headings and should be confirmed before treating it as a strict web/no-web comparison.
- **Findings from the scorecard:** Claude + web produced the most quantified theses (7.5 hard-number claims per thesis; 5.6 numbers per 100 words), greatest basket diversity (mean pairwise Jaccard 0.13; zero exact duplicates), and the highest equity share of longs (85%), but lower mean long/short correlation (0.39), lower suggested leverage (2.25x), and higher concentration (72% mean maximum leg weight). Claude without web retained quantified theses (8.0 claims; 5.3 per 100 words) but was more repetitive (Jaccard 0.32; one exact duplicate), supporting a model/behaviour difference beyond live-web freshness.
- **Quantitative comparison:** The two OpenRouter groups had higher mean correlation (0.58 and 0.66) and generally more balanced/higher-leverage sizing than Claude, while cointegration was rare across all groups (0% Claude + web, 0% Claude no-web, 25% first OpenRouter group, 0% second). These metrics are descriptive of a small sample, not proof of trading superiority.
- **Interpretation:** Claude was stronger in thesis specificity, current-market framing, and diversity; OpenRouter was stronger on the historical pair-stat profile in this sample. Live research can intentionally select assets whose current catalysts diverge from historical relationships, so lower correlation does not by itself mean the basket is worse. Forward paper-trading performance is required.
- **What changed:** Built and tested the opt-in Claude harness, exposed the same two read-only Pear tools through a private MCP bridge, added tool telemetry and dry-run logging, and prototyped a two-stage Claude research→selection path with survivor validation and failure diagnostics. The latter currently needs further turn-budget validation after live runs hit `error_max_turns` in the research stage.
- **Safety/status:** All evaluation runs were dry-run only; no `made_baskets` rows, trades, cron schedules, or production services were changed. The research-stage branch is not production-ready. Next step is bounded research orchestration followed by forward paper-trading evaluation.
- **Evidence:** `evals.pdf` (24 pages; raw basket outputs and scorecard), `git` commits `45443208`, `3865dfa`, `98329a8d`, `b5e1a163`, `89de0d3b`, and [[5th Sep 2026]].

### 2026-09-05 – 09-10 — Production hardening plus forward basket evaluation
**Status:** Mixed: the product fixes below were merged through the recent development/main merge history; the Claude research-stage and paper-shadow work remains experimental on `feat/claude-basket-research-stage`; the final prompt guard listed below is unmerged WIP.

- **Context ingestion reliability:** hardened Twitter fetch retries against stale keep-alive socket aborts and added retry-cause/abort-timeout diagnostics (`d9f4da12`, `a8999c09`, `0328ac73`; PR #682 line of work). The goal was to make upstream context failures explainable instead of looking like generic fetch failures.
- **Trade-execution hardening:** prevented silently opening a new position when a requested leg could not be merged, grounded multi-account ambiguity in the account list, scoped account-snapshot balance authority to its own account, clarified TWAP `fillCount`, and removed per-slice fill detail from the result that caused context growth (`4663fd83`, `d70bbc2e`, `4446c9ab`, `d77dfd6c`, `7a3cb583`, `f24d634f`). The venue-word grounding work was shipped through the #699 merge line.
- **Composer/model-routing and surface fixes:** retired MiMo beta from advanced mode, carried the advanced model migration and GPT-OSS fallback work through their merge lines, and shipped leaderboard follow-up intent fixes plus the advanced suggested-action ticket guard (#687, #689, #690, #692, #694, #696, #713, #714).
- **Claude basket research iteration:** on the experimental branch, restored research breadth, scaled research timeouts and stage turn caps to workload, hardened the WebFetch prompt against low-quality/injected research, enriched 1x1 pairs from pair stats, and reconciled selected legs to validated survivor IDs (`36150900`, `3d3d4f66`, `b5e1a163`, `89de0d3b`, `65e825db`, `d494d9da`). A later live dry run completed the research→selection flow and produced a stats-enriched basket; all runs remained dry-run only.
- **Forward evaluation tooling:** added a read-only `paper-shadow.ts` tracker and later multi-basket/Lighter support (`9ebc0ac5`, `1176bd88`). It records live public Hyperliquid marks, per-leg/leg-level/gross/net PnL, fees and return, without wallet keys, signed requests, exchange orders, database writes, or cron installation. A user-run overnight snapshot reported 10/10 positive paper results (+0.45% to +6.68%); this is encouraging but preliminary and not evidence of live-trading superiority.
- **Current unmerged WIP:** `ba5db298` on `fix/composer-no-tool-name-echo` adds a prompt rule forbidding literal tool-name echoing in generated user output. It is one commit ahead of `origin/development` and must not be described as merged.

**Evidence:** local/remote git history through 2026-09-10, the Claude research branch, the `paper-shadow.ts` implementation, user-provided dry-run/paper-shadow logs, and the evaluation artifact `C:\Users\prakh\Downloads\evals.pdf`.

---

### 2026-09-18 — Resolved the Hetzner basket-migration merge conflict, root-caused a stale 3-hour bottleneck, did the first real production install *(outcome pending tonight's run)*
**Status:** Code shipped and merged (PRs #786, #796); GitHub issues reconciled; production systemd install done and armed. The one thing NOT yet confirmed: the Hetzner timer's first real fire (14:00 UTC today) and a post-run DB check — this entry documents work done, not a proven-in-production outcome.

- **Resolved PR #786's merge conflict.** #725 (kill-switch) merged into `development` first, which put #786 (`feat/basket-hetzner-migration`, Story #718) into `CONFLICTING` — both branches had edited the same lines of `refreshDaily` independently. Resolved additively (kept both the kill-switch guard and the `--baskets-once` signature change), verified `tsc --noEmit` clean + full context suite (15/15 suites, 98/98 tests) before pushing. PR flipped to `MERGEABLE`/`CLEAN`; merged same day.
- **Root-caused and fixed a stale performance-bottleneck comment.** A real dry-run log gave concrete per-basket timing (category ~116s + research ~247s + selection ~20s ≈ 383s), consistent with the user's flagged ~3-hour full-run estimate. The code ran 3 platforms sequentially, citing a shared-budget comment that predated the concurrency-safe cache it claimed still applied (confirmed via an investigation pass: no shared budget tracker, no lock, the cache is explicitly Promise-memoized for concurrent callers). Changed the 3 sequential loops to `Promise.all` (PR #796), same verification bar, merged same day. Expected ~3h → ~1h wall-clock — not yet observed on a real run.
- **First real production systemd install on the Hetzner box.** With explicit user-granted SSH access, synced the box to the merged `development` HEAD, rebuilt, and confirmed the build was genuine (turbo cache miss, not a replay). The checked-in unit template assumed a generic user/paths that don't match this box — wrote a corrected `context-baskets.service` with the real user, paths, and a live-derived absolute node binary path (systemd's minimal PATH has no nvm shims, so `env node` would have failed silently at 14:00 UTC with nobody watching). Installed and enabled the timer; confirmed via `systemctl list-timers` this is genuinely its first activation, not a re-enable.
- **Found the env file backing this job is already shared infrastructure.** `.env.basket-development` — despite its name — is also the `EnvironmentFile=` for two other already-installed jobs (OI and tokenomist canaries) from an earlier work phase. Flagged the misleading name and a safe rename, deferred pending go-ahead rather than renaming unprompted mid-task.
- **Reconciled the GitHub issue tracker against real evidence**, not memory: closed #731 (fixed back on 2026-09-10, never auto-closed) and #723 (validated in earlier sessions, confirmed by the user after an initial overclaim in a draft closing comment was caught and corrected before sending). Left #724/#718/#717 open, correctly — the Railway kill-switch flag is still unset, so cutover isn't actually done yet.

**Evidence:** PR #786 (merge commit `b4b2714c` on the resolving branch, `gh pr view` state), PR #796 (`9138a07f`/`23a14602`, merged `e46d372a`), live SSH session output from `ap-tg-fallback` (`systemctl list-timers`, `systemctl cat context-baskets.service`), issues #731/#723/#719–722/#724 GitHub state.

---

### 2026-09-23 – 09-25 — Composer fallback moved off the rate-limited provider, plus a dead-fallback bug in the stream path
**Status:** Merged to `development` 2026-09-24 20:47 UTC (PR #923, merge `cda91b87`); live in the dev env only. Not in `main`/prod yet. Recovery from a *real* Cerebras 429 hasn't been observed; only injected 429s were tested.

- **Problem:** a Cerebras tokens-per-minute 429 on the composer surfaced as `Delay was aborted`. The SDK's retry backoff outlasted the 30s abort, then the fallback, which used the same model on the same Cerebras key, hit the same quota. The user got the generic error after 88s (trace `a5b982db36d1029bd46d7e8e71c4db26`).
- **Fix (`df486dce`):** moved `composerFallback` to `openai/gpt-oss-120b` on OpenRouter (a different provider and quota, same model so prompt tuning holds). Primary calls run with `maxRetries: 0`, so a 429 goes straight to the fallback.
- **Found and fixed a dead fallback path (`ae33a54c`):** in `ai@6` `streamText`, an error on a later step (after a tool call) doesn't throw. `textStream` just ends empty, so a 429 on step 2+ never reached the fallback. The stream now falls back whenever nothing was emitted. Also built the fallback through `resolveNodeModel`, so its own timeout/temperature config actually applies.
- **Verification:** `composer/agent.spec.ts` (`0faadd94`), 4 tests on the real SDK with mock models; the step-2 test fails on `df486dce` and passes on the fix. A live trial against real Cerebras + OpenRouter with an injected 429 recovered in all 6 cases (worst 29.8s against a 30s per-tier budget; a ~90k-token prompt took 17.2s). The step-2 case failed 2/2 before the fix.

**Evidence:** PR #923 (description has the trial table), commits `df486dce`, `ae33a54c`, `0faadd94`, merge `cda91b87`; [[25th Sep 2026]]; [[Engineering Lessons#A streaming fallback must trigger on "nothing emitted", not only on a thrown error]].

---

### 2026-09-28 — Remaining daily/weekly agent jobs and three Twitter crons merged to development

**Status:** [PR #1027](https://github.com/pear-protocol/agent-pear-v2/pull/1027) merged to `development` at 12:43 UTC (`d3ad7e43`). Code and CI are complete; the merge is not in `main`, and live validation/cutover for this batch are unverified. [Story #1026](https://github.com/pear-protocol/agent-pear-v2/issues/1026) remains open for its remaining acceptance criteria.

- Shipped the six agent one-shots (touch-priors, touch-brief, ghost-writer, ghost-synthesis, x-digest, best-trade) and Twitter daily-summary, weekly-recap and trusted-sync. The runners retain a single scheduler-owner cutover path and avoid starting unrelated always-on consumers.
- Preserved the cron ownership review fixes, fixed four Biome errors and a stale x-digest claim assertion, and replaced the head commit with the user-requested lease-protected push. Both builds, repository Biome, 68 affected agent tests and 148 Twitter tests passed; all three PR checks are green.
- Remaining work is operational/documentation follow-through, not another migration implementation: agent runbooks/index, full background-suite sweep, isolated Hetzner validation, release/deployment and per-job cutover.

**Evidence:** final commit `25b5a353`; [build/Biome CI](https://github.com/pear-protocol/agent-pear-v2/actions/runs/36423167681); [agent/MCP E2E CI](https://github.com/pear-protocol/agent-pear-v2/actions/runs/36423172675); [[28th Sep 2026]]; [[Harness Integration/Step 5 — Ambient-Agent Job Migration Plan (apps agent)#Update — 2026-09-28 (Story #1026 and PR #1027 merged to development)|Step 5 update]].

---

### 2026-09-28 — Production cutover of 9 background jobs from Railway to Hetzner, plus a latency-exposed transaction bug

**Status:** after release #1010, 9 of 12 jobs cut over live: 3 sentiment, touch-priors, touch-brief, ghost-writer, x-digest, Twitter trusted-sync and daily-summary. Their first scheduled Hetzner runs are still to be confirmed. ghost-synthesis, weekly-recap and best-trade stay on Railway, each for a stated reason.

- Ran each job manually on the box and compared it with Railway's own run that day before flipping anything. Then set the Railway kill-switch, had the service redeployed, verified the scheduler was gone (boot logs plus BullMQ repeat keys in Redis), and only then enabled the systemd timer.
- The manual-run step caught three problems before cutover: the weekly sentiment job failing only from Hetzner, a Telegram handle in `TWITTER_BOT_USERNAME`, and the Twitter one-shots pointed at the agent Redis instead of Twitter's.
- Root-caused the weekly sentiment failure: an interactive Prisma transaction with a per-row round trip exceeded the 5 s default over the ~30 ms public-proxy path. It had never failed on Railway's private network. Shipped the fix in PR #1034, re-ran the job successfully, then cut over.

**Evidence:** [PR #1034](https://github.com/pear-protocol/agent-pear-v2/pull/1034) (`64ae34f7`, merged `2765c164`); [[28th Sep 2026#Cutover — 9 of 12 daily/weekly jobs moved from Railway to Hetzner (13:59–18:35 UTC)|28 Sep cutover log]].

---

### 2026-09-29 — Which assets Pear users actually trade (basket/pair familiarity research) *(open, no code changed)*

**Status:** investigation only; ranking logic and architecture not decided.

- Traced the basket/pair generation path and every trade-data source in the repo (plan in [[Better Trading]]), then queried the Pear BE read-only DB.
- Found ~75% of BE fills are imported off-Pear activity, and ~70% of Pear-placed non-reduce fills are scheduled ADJUST (rebalance/TWAP) from 40 users. The correct unit is a completed OPEN execution.
- Clean 30d ranking by distinct users (298 users): BTC 135, ZEC 129, ETH 127, HYPE 120, SOL 109. 87% of opened legs are in pairs/baskets.
- The OI-based `pickDailyAssets` top-10 overlaps 9/10 with Pear's top-10, so a global top-N boost would be near no-op. The value is in compositions, direction skew and mid-tail assets. Popular "assets" like AERO/UNI and ZEC are largely single campaign-style pairs.

- **Update (same day):** Jeyd decided the approach. Rank assets by `0.65 × current holders + 0.35 × 90-day openers` (distinct traders), take the top ~20, then run narrative → pair/basket generation → quant gate. No trending layer. A read-only preview showed ranks 1–10 are 9/10 the OI picks, so the new value sits in the current-holder reordering and ranks 11–20. Status report drafted in Notion.

- **Update (validation):** read-only end-to-end validation of the 65/35 plan.
  - All top-20 assets pass sentiment → listing → quant ID → pair stats.
  - The weights are stable (~1.9 top-20 swaps/day).
  - Blockers: the 20-per-category candidate cutoff (fix: union into `pickDailyAssets`), BE→context transport, and near-zero pair output (0.4%).
  - Baseline: ~2% of generated compositions opened within 7 days.
  - Corrected two earlier claims ("dev DB", "HYPE has no quant ID").
  - Suggested first slice: ranking job + agent context line.

**Evidence:** [[Pear Trader Asset Familiarity — Data Findings]] (§11 for the decision and preview, §12 for validation).

**Update 2026-09-30:** plan finalised and scoped; still no feature code.
- Final plan written ([[Familiar-Asset Ranking — Final Plan]]); baseline frozen read-only (8 distinct users opened a generated idea within 7d, 16 of 551 compositions, 0 1x1 pairs).
- Jeyd widened the goal: choose pair vs basket per idea on evidence, and fixing the 0.4% pair bias is in scope. Located the bias in prompt defaults and in the missing pair-stats tool for the model.
- Groomed into epic [#1064](https://github.com/pear-protocol/agent-pear-v2/issues/1064) + 5 stories (#1065–#1069), three separately measurable releases, branches pushed.
- Verified live that prod jobs run only on Hetzner from `main` (not `development` as the runbooks said), and fixed the runbooks (`f5cb1a14`).
- Jeyd locked the V1 signal definitions.

**Evidence:** [[30th Sep 2026#Familiar-asset pair/basket: structure-neutral plan, issue tree, Phase 0 (late)|30 Sep log]], #1064–#1069, `f5cb1a14`.

**Update 2026-10-01:** first two stories built as PRs; nothing merged or in prod.
- [#1070](https://github.com/pear-protocol/agent-pear-v2/pull/1070) (#1065): daily 65/35 ranking job + shared Redis reader + idea-adoption metric. The metric reproduced the frozen baseline exactly on a live read-only run.
- [#1071](https://github.com/pear-protocol/agent-pear-v2/pull/1071) (#1066): ranked assets added to the daily sentiment picks, with a safe fallback.
- Called my own coverage backtest a gate, then retracted it publicly when the user showed it was near-circular; replaced it with an offline replay of the real generator.
- Built a no-write replay harness and ran it on the Hetzner box in a separate clone (preload patch reaching the Claude-spawned MCP bridge; prod eval log verified unchanged). It showed #1066 changes only 4 candidate assets, reachable in ~31% of runs, so the paid A/B was skipped and the effort moved to #1067 (pair vs basket on evidence).
- Measured prod's generation funnel: ~25 candidates → ~6 survivors per idea.

**Evidence:** [[1st Oct 2026]], [[Familiar-Asset Ranking — Final Plan]] §11.7, #1070, #1071, #1069 comments.

---

### 2026-09-30 — Sentiment ingestion: plan, story #1052, first two cleanups on a parent branch *(in progress, not in development yet)*

**Status:** 2 PRs merged into the parent branch `sentiment-enhancement`; nothing shipped to `development`/prod. Phase 1 not started. *(superseded — see Update 2026-09-30 evening below)*

- Mapped exactly what asset sentiment consumes (news digest, ≤10 tweets, Tokenomist, Polymarket) and found 9 code-level noise sources; mapped every consumer (feed baskets, chat sentiment node, chat basket/pair agents).
- Grilled and wrote the plan as story [#1052](https://github.com/pear-protocol/agent-pear-v2/issues/1052): more data (HL price, positioning, CoinGecko news for crypto, better tweet matching) and agentic sentiment with a stronger model; prod history on hold.
- #1054 (`3deb6f0d`): removed Polymarket strike-ladder odds that were read backwards, which also removed a weekly-run mislabel. #1055 (`5213957c`): sentiment rows now carry `runAt` to every reader.
- Checked the senior's CoinGecko-scraping idea against the live API instead: plan tier, endpoint limits, coverage on 16 coins + 7 stocks.

**Evidence:** [[Sentiment Data Ingestion — Improvement Plan]], [[30th Sep 2026]], #1052, #1054, #1055.

**Update 2026-09-30 (evening):** still in progress, nothing merged beyond #1054/#1055, nothing in `development`/prod.
- Phase 1 built as [#1058](https://github.com/pear-protocol/agent-pear-v2/pull/1058) (open, into `sentiment-enhancement`): Hyperliquid price context (returns vs own usual move, vs BTC / Nasdaq-100, 30d range, vol), CoinGecko headlines for crypto with relevance/dedupe guards, events-only web digest, missing-inputs line, all behind `SENTIMENT_MARKET_DATA_ENABLED` (off = prompt byte-identical). Plus a playground that runs old vs new per asset, logs every call, and scores calls against HL price at +1/3/7d. Commits `87284905`, `cf532828`, `fee45633` pushed; `5b484597` (request/error logging, paired scoring) local.
- First real side-by-side runs found the new variant failing more. Diagnosed from raw OpenRouter responses: deepseek was being routed to Relace, which returns an empty 200; the retry loop can't escape it (strong evidence it's the always-on response cache replaying the empty answer). [#1060](https://github.com/pear-protocol/agent-pear-v2/pull/1060) (open): synth → `deepseek-v4.1-flash` with Relace ignored, web digest → `gpt-6-luna` (model choice was the user's call). Live rerun of the failed assets: 8/8.
- Day 1 of the 3-day comparison logged (run `2026-09-30T15:35:58Z`). No accuracy result yet. New-variant timeouts (3/17 that run) still undiagnosed.

**Update 2026-09-30 (late):** `5b484597` pushed; [#1058](https://github.com/pear-protocol/agent-pear-v2/pull/1058) merged into `sentiment-enhancement` (`b0abeda0`), then [#1060](https://github.com/pear-protocol/agent-pear-v2/pull/1060) (`7b62243c`). Still nothing in `development`/prod. Local checkout now on `sentiment-enhancement` @ `7b62243c` (same `apps/context` tree as the old `local/combined`); day-2 runs go from there. #1060 isn't flag-gated: flash + luna also hit category sentiment + basket-category fallback once the parent reaches `development`, so check those paths before that PR.

---

### 2026-10-01 — Familiar-asset ideas: ranking + universe + pair-vs-basket built, replayed on prod data, plan re-scoped *(in progress, nothing in development/prod)*

**Status:** 3 PRs open (none merged): #1065 → [#1070](https://github.com/pear-protocol/agent-pear-v2/pull/1070), #1066 → [#1071](https://github.com/pear-protocol/agent-pear-v2/pull/1071) (Release 1), #1067 → [#1076](https://github.com/pear-protocol/agent-pear-v2/pull/1076) (Release 2). #1068 re-scoped, not built. *(superseded — see Update 2026-10-01 evening below)*

- Ranking job (65% current holders / 35% 90d openers, top 20 usable) + nightly adoption metric reproduces the frozen baseline exactly (8 traders, 16/551). A coverage backtest was posted then publicly retracted as circular.
- #1067: diagnosed why the generator almost never made 1x1 pairs (selection's only stats tool rejected 1x1; prompts pushed size; no shape comparison) and built evidence-based pair-vs-basket selection (pair-stats tool moved to shared, rubric, projected-return check, eval-log `structureChoice`).
- Built a write-free replay harness on the prod box (separate clones, Redis writes no-op'd, inserts captured, prod logs md5-verified). Paired selection-only A/B on 40 saved prod research runs: HL pairs 5% → 33%, Lighter 0% → 29%, 12/12 pairs pass the final gate, 0 failures after two fix rounds (directional-first + no-hedge fallback; two regex false positives). Full-pipeline run: research cost flat ($1.23 vs prod $1.20), pairs survive the cron into storage.
- Measured the familiarity funnel from prod's eval log (pool 17% → survivors 14% → ideas 14%): familiar assets are lost at the category pick and in research, not selection — so #1065–#1067 don't move familiarity. Re-read the original brief; "union, boost don't constrain" were our design choices. Decision: ranked assets **replace** the HL/Lighter candidate pool (#1068 rewritten, epic updated).
- Side findings: stored pairs lose z/hedge when the pair_stats row is reverse-oriented (#1075); backtested returns cover 7 of 1,338 candidate pairs (no return-based ranking possible); Claude Code auto-update on the box briefly removed the CLI binary that prod's basket job uses.

**Evidence:** [[1st Oct 2026]], [[Familiar-Asset Ranking — Final Plan]] §11.7–§11.10, #1064–#1069, #1070, #1071, #1075, #1076.

**Update 2026-10-01 (evening):** Release 1 PRs merged into the release branch; #1068 built and replayed; still nothing in `development`/prod.
- [#1070](https://github.com/pear-protocol/agent-pear-v2/pull/1070) and [#1071](https://github.com/pear-protocol/agent-pear-v2/pull/1071) merged into `feat/familiar-asset-pair-basket` (13:52 / 13:53 UTC). Ranking timer not installed on Hetzner.
- #1068 built as draft [#1077](https://github.com/pear-protocol/agent-pear-v2/pull/1077) (`69fce053`): HL/Lighter general + directional passes take their candidate pool from the ranked assets listed on the venue; research gets the pool + stored notes, no category fetch; `considered[]` diagnostic in the eval log; null ranking falls back unchanged.
- Replay on the prod box (main vs #1068, same day/ranking/category pick, nothing written, prod log md5s unchanged): familiar share of stored legs 25–32% → 100%, research cost per attempt flat (~$1.20), survivors per attempt 4–5 → 7–8; Lighter stored ideas 6 → 3. Opus `529 Overloaded` hit arm C's first try; stopped it and reran once Opus answered.
- Traced the Lighter drop to the cron's composition-overlap rule (near-copies sharing SOL/SUI/PUMP), not the per-asset cap I first assumed. Fix `eb6353fd` (unused ranked assets first each attempt, exhausted pool fails instead of falling back) pushed to #1077; not re-replayed.
- `considered[]` showed the top 2 ranked assets rarely survive research (ZEC 1/16, HYPE 2/16). Stored ideas' stats are a soft veto, not a selection reason (no cointegration, |roll z| < 1, some k < 0.1 at 5x) in both arms.
- Open product question raised: a fully ranked pool removes whole-market discovery on HL/Lighter; option of ≥1 ranked asset per idea.
- Wrote the replay up in Notion; updated checklists on #1064–#1069 (nothing closed: live checks pending).

**Evidence:** [[1st Oct 2026#Post-replay: #1077, Lighter root cause + fix, epic checklists]], #1077, `69fce053`, `eb6353fd`, #1068 and #1069 comments, Notion "#1068 replay" page.

### 2026-10-05 → 2026-10-11 — Small production fixes merged to development (catch-up entry from GitHub)

**Status:** merged into `development` unless noted. Sentiment work is tracked separately in [[Sentiment Data Ingestion — Improvement Plan]].

- #1085 reversed 1x1 pair stats keep z-score and hedge ratio; #1087 and #1088 Elfa empty-message and "a system" false-positive fixes; #1084 (Gemini pin) closed unmerged.
- #1090 retired the Coinglass HL wallet distribution tool ([[Coinglass HL Wallet Distribution — Retire, Don't Mute]]).
- #1091 chart/chip extractor now marks the current ask: [[Chart and Chip Extractor Picked Up an Earlier Turn's Asset]] (100-input prod comparison, model switch tried and rejected, one weak point left open).
- #1092 and #1093 retry transient context DB errors on connect and on the tweet-queue writes (P2002 after a retried create counts as queued). Not exercised against a real outage.
- #1096 stopped startup index refreshes racing one-shot shutdown: update in [[Hetzner One-Shot's Full AppModule Boot Fired an Unrelated Cron, Colliding With Railway]]. Prod confirmation pending.
- #1097, #1098, #1099 telemetry-only: real provider error on advanced stream failures, Elfa 400/422 response body in logs (cause of the Elfa 400 still unknown).
- Filed pear-price-fetch#86 (Orbs feed 503 freezes all price runs and signals since 7 Oct; open).

**Evidence:** [[6th Oct 2026]], [[7th Oct 2026]], [[8th Oct 2026]], [[9th Oct 2026]], [[10th Oct 2026]], [[11th Oct 2026]], PR bodies #1085 to #1099.

### 2026-10-11 → 2026-10-13 — Alert-noise and failure-cause fixes, SYMM retired from the daily run (catch-up entry from GitHub)

**Status:** merged into `development`; #1105, #1112 (and #1101, #1099 earlier) also on `main`. As of 14 Oct, #1103, #1107, #1117 and #1119 are on `development` only (checked `main` contents on GitHub, not a release PR).

- **Elfa 400 cause found:** #1105/#1106 put the response body in the alert; it showed `title` over the 120-character limit; #1107 caps the title and keeps the `topic=` marker. Personal narrative topics with long slugs were failing every reconcile. Closes the open question from #1098/#1099. Not run against live Elfa.
- **Transcription:** #1100/#1101 logged the provider body; #1103 skips audio under 1 KB (prod alerts were all 5 to 27 byte payloads, each a paid STT call and a 500) and allow-lists the format. Sender of the payloads unknown.
- **Alert noise:** #1104 untagged rejected Elfa webhooks and made a missing `sessionId` a 400 instead of a Prisma 500.
- **SYMM:** #1111/#1112 dropped it from the daily basket run after pear-price-fetch#87 removed SYMMIO rows; broader retire PRs #1102/#1108 closed unmerged. Details in [[12th Oct 2026]].
- **Chat missing message:** #1113 merged then reverted (#1116) on review findings, reworked as #1117 (400 on missing or non-string message, CTA guard). Behaviour change: empty message with no images is now a 400.
- **X share:** #1119 retries the media upload once on 502/503/504.
- #1109 merge-commit of `main` into `development` to unblock release PR #1094.
- Basket gen and interest distiller moved back to Claude on 12 Oct (env change on the box, no PR): [[12th Oct 2026]].
- #1118 (planner fallback → generalChat) closed 13 Oct with no linked PR; fix not verified: [[Planner Fallback Sent a Trade-Idea Request to generalChat After Auto-Task History]].

**Evidence:** [[11th Oct 2026]], [[12th Oct 2026]], [[13th Oct 2026]], PR bodies #1099 to #1119.

### 2026-10-14 → 2026-10-15: Tested Prime Agent against advanced mode by replaying a real session *(finished, repo untouched)*

**Status:** study only. Nothing committed, nothing merged. Decision: drop Prime Agent as a runtime for now. Full log: [[Prime Agent Harness testing]].

- Coding trial on the #1096 task (Claude Code against Prime Agent, same model): five runs, each touched only the two tokenomist files while the real PR touched five. Diffs not read.
- Replayed turns 2 to 14 of a real production session through Prime Agent with the production model, prompt and recorded tool results. Final setup against production: 2,762 s against 1,403 s, 1.87M input against 1.97M, 136k output against 95k.
- Input tokens went from 5.7M to below production over the runs. Time did not, because Prime writes about 1.4x the output. Quality ended about level with production, weaker on the last two turns.
- Side findings in production, none fixed: mislabeled "% margin" column, advanced cost under recorded in Langfuse, a 25 ETH gap in a closed trade record.

**Evidence:** [[14th Oct 2026]], [[15th Oct 2026]], `C:\scratch\pear-replay\REPORT.md`.


### 2026-10-14 → 2026-10-17 — Confirm-drift alert noise, and the token-unlock fix that hid every upcoming unlock (catch-up entry from GitHub)

**Status:** merged into `development`; not on `main` as of 18 Oct (not rechecked).

- #1121 confirm-drift refusals (mark moved past the limit before Confirm, no order placed) log at `warn` instead of paging Discord; stale-price and no-mark aborts stay `error`.
- #1122 scheduled token unlocks show as `SCHEDULED` instead of being hidden: [[Token-Unlock Amounts Shown as Confirmed Facts]] (Update 2026-10-17). Replayed on XPL and KAITO; awareness eval not run.
- Reviewed and approved GPSxtreme's #1120 with one non-blocking wording comment on cross vs isolated margin mode.

**Evidence:** [[14th Oct 2026]], [[15th Oct 2026]], [[17th Oct 2026]], PR bodies #1120 to #1122.

### 2026-09-02 → 2026-10-05 — Merged PRs missing from earlier entries (catch-up entry from GitHub, added 18 Oct)

**Status:** merged; `main` vs `development` noted per PR in each daily log.

- **Signals:** #676/#679 evidence timing observability ([[3rd Sep 2026]]); #680/#684 `readHlContext` over-gating fix for a `measurement_unavailable` miss ([[7th Sep 2026]]); #792/#793 stop paging on expected fail-closed evidence reasons ([[17th Sep 2026]]); #1020 signal compose falls back to OpenRouter on Cerebras failure ([[28th Sep 2026]]).
- **Trade execution:** #681 guard against silently opening a new position for an unmergeable leg; #683 pre-fetched account list for multi-account users; #693 account-snapshot authority scoped to its own account; #697 dropped per-slice `fills[]` from `get_twap_orders` (about 3.44M-token request from one account); #1019 one ambiguous ticker no longer fails a whole `hyperliquid_price` batch; #812 set-leverage concurrency cap and 429 retry (root cause per the PR is the nginx gateway, not Hyperliquid).
- **Rate limits and alerts:** #790/#814 Elfa client throttled to about 140/min; #1016 OI cron throttled for Coinglass 429s; #817 mentions scan failure downgraded to `warn`.
- **Other:** #1045/#1050 tweet labeling retries text-only on a dead image URL; #1061 hotfix moving `gpt-6-luna` nodes to `deepseek-v4.1-flash`; #1073 chat session rename; #1074 vault answers link to the official site; #716 no literal tool names in output; #821/#1015/#1035 Hetzner docs and a back-merge.
- Closed unmerged with no reason in the PR: #795, #816, #819, #1014. #795 (heartbeat tasks going silent after run 1) may still be an open problem; not checked.

**Evidence:** [[3rd Sep 2026]], [[7th Sep 2026]], [[9th Sep 2026]], [[17th Sep 2026]], [[18th Sep 2026]], [[21st Sep 2026]], [[22nd Sep 2026]], [[28th Sep 2026]], [[30th Sep 2026]], [[5th Oct 2026]], PR bodies.

### 2026-10-05 → 2026-10-14 — Familiar-asset ideas shipped to prod, and what the first reads showed (catch-up entry, added 20 Oct from the plan note)

**Status:** shipped. #1078 merged to `development` and release #1082 to `main` on 2026-10-05; ranking timer installed on the Hetzner box the same day. Whether traders open more of the ideas is **not shown**. Full numbers and caveats: [[Familiar-Asset Ranking — Final Plan]] §11.14–§11.21.

- **What it does:** a daily job ranks assets 65% by current holders and 35% by 90-day openers (top 20 usable); the HL and Lighter generation passes take their candidate pool from that list instead of the category pool (#1065, #1066, #1068). The pair-vs-basket selector (#1067) was held back by the user and #1076 closed.
- **Generation changed as intended:** familiar share of stored legs 15% before to 64–72% after; ideas with at least one familiar leg 43% to 74–82%.
- **Adoption did not clearly move:** unique compositions opened within follow-up 1.9% (3 of 159) before, 6.1% (8 of 131) in the first full week after, inside a pre-release weekly range of 0.5% to 5.6% on 1–7 users. All Pear opens per day 86 after against 74–100 before. Same-assets-either-direction opens rose from 4% to 8%, cause unknown. Nothing links an open to an idea, so causation is not shown.
- **Caveats:** follow-ups were 1–7 days, the provider switched back to Claude on 12 Oct partway through the window, and the full 7-day read (#1069) was due about 19 Oct and is not in the vault yet.
- **Evidence:** [[1st Oct 2026]], [[Familiar-Asset Ranking — Final Plan]], #1070, #1071, #1077, #1078, #1082.

### 2026-10-19 → 2026-10-20 — Adjust sizing, false refusals, failed-source honesty, alert floods (catch-up entry from GitHub, added 20 Oct)

**Status:** all merged to `development`. On `main` via release #1151 (20 Oct 07:12 UTC): #1139, #1141, #1143–#1149. #1152 and #1153 are on `development` only.

- **Trade execution:** #1139 margin-sized `adjust_position` (margin was being ordered as notional); #1141 signed uPnL levels on ticket cards; #1146 total-margin line only when every leg is margin-sized, with a plan-to-execute test. Open: no collateral check on adjust increases; what armed the +$1 stops seen on 14 Oct is unknown.
- **Agent honesty:** #1147 planner no longer routes an unlinked user's trade to the pair node, which invented "two-leg pairs only"; #1148 thrown tool errors and empty feeds are reported as unavailable, not "nothing found"; #1149 composer stops refusing chart asks; #1153 no blank reply when an advanced run fails. Details: [[20th Oct 2026]]; lesson: [[Engineering Lessons#A thrown tool error is not a tool result, so a summary built from results omits failures]].
- **Reliability:** #1143 advanced stream that opens then fails falls back to the second model; #1144 losing Link tap no longer deletes the winner's marker; #1145 touch-priors upserts batched to stay inside the Prisma pool (cause of the 17 and 18 Oct failures unconfirmed); #1152 poll-failure and Discord alert floods capped.
- **Not merged:** `c861b8b6`, Elfa v3 events behind `ELFA_EVENTS_INTEREST` (branch `worktree-feat-elfa-v3-events-eval`, no PR).

**Evidence:** [[19th Oct 2026]], [[20th Oct 2026]], PR bodies #1139 to #1153.

---

### 2026-10-21 — Controller input validation finished across chat, auth, partner, journal, onboarding and the Elfa webhook *(in review, not on development)*

**Status:** #1159 and #1160 merged into #1154's branch (`fix/chat-body-validation`). #1154 is open against `development`, retitled `fix(api): validate controller inputs with zod and strip NUL bytes`, and closes #1127 and #1158 on merge.

- **Fixed:** #1159 covers #1158 rows 1 to 5: `/chat/execute` body (top-level shape, `TradePlanSchema` reused), partner `sessionId`, self-link `code`/`force` (`"false"` no longer counts as true), journal `cursor`, messages `pageSize` bounded 1 to 50. #1160 covers rows 8 and 12: a number onboarding answer threw a 500 on `.trim()`, and an unsigned Elfa webhook with a non-string title got a 500 instead of a 401.
- **Decided not to fix:** rows 9 to 11 already end in a correct refusal. Row 6 was already done by #1154. Row 15 (trade-ticket Redis parses) skipped because existing null-on-parse-failure and the single-use plan token already contain it. Each decision is recorded in the #1158 table.
- **Verified:** tsc clean, 69 suites / 1071 tests passing on the merged branch. Not run: full suite, `test:e2e`, live API.
- **Open:** `/chat/execute` still forwards undocumented keys (`userId`, `sessionId`, `history`) to the DAG. Whether that should stay is undecided.

**Evidence:** [[21st Oct 2026]], PRs #1154, #1159, #1160, issue #1158. Lessons: [[Engineering Lessons#In zod v4 an object key typed z.unknown() or z.custom() is required unless marked optional]], [[Engineering Lessons#An audit row that says unchecked input reaches X is a lead, not a bug, until the path is traced]].

### 2026-10-21 → 2026-10-23 — Generated baskets: measured them properly, found the picks are about random and the AI chases, built a guarded rule *(rule committed, not pushed, off by default)*

**Status:** #1162 (story, body rewritten 10-23 to current state). Eval script on `feat/basket-eval-script` (draft #1166). Buy rule `398517c5` on `feat/basket-no-top-gainer-buy`, not pushed. Coverage gap filed as #1167.

- **Measured:** read-only eval of ~3.3k baskets (05-27 → 10-22). Basket results follow the market (long-only +2.73% / short-only −6.82% in a rising month, the reverse in a falling one). Against same-platform tradable peers, buys rank 52%, sells 51%: about random.
- **Corrected myself twice:** the first read's "+0.65% selection" depended on the yardstick (BTC, mean coin and median coin all disagreed), and the peer groups were polluted because ~44% of listings have no sentiment category. Both corrections posted on #1162.
- **Built:** "no buy of last week's top-15% gainer" (prompt line + post-generation reject, behind `BASKET_NO_TOP_GAINER_BUY`). Backtest: buy rank 52.0 → 55.8, 4 of 5 months, returns flat. 37/37 specs, tsc, biome, read-only live dry check.
- **Found:** `/categories` is a hard-coded list in pear-stat-arbs unchanged since January. Chip menu is AMD/INTC/MU/NVDA only, likely part of the repeat problem. Filed #1167.
- **Open:** out-of-sample verdict ~11-13 (weekly scheduled check), product calls on net long / leverage / long-only, which weights the web app trades.

**Evidence:** [[22nd Oct 2026]], [[23rd Oct 2026]], [[Basket Quality — Staged Improvement Plan]], issues #1162, #1167, PR #1166.

---

### 2026-10-28 → 2026-10-29 — Elfa replacement step 2: math re-measure root cause, measurer fix, local shadow poller *(in progress, local branch only)*
**Commits:** `e7a82057` on `investigate/elfa-step2` (not pushed) · `Confirmed`

- Found why 34% of math fires failed Pear's re-measure from Railway logs: Elfa's funding values were wrong (Pear matched Hyperliquid), Pear built Bollinger with the live candle, and read liquidation from the just-opened Coinglass hour.
- Fixed Bollinger (closed candles) and liquidation (trailing 60 min of 5m buckets). Liquidation not verified live, Coinglass returns `401 Upgrade plan` since 28 Oct.
- Elfa is paused, so backtested the 598 past math fires against Hyperliquid history: price triggers real in 111 of 112, funding in 22 of 94.
- Built a log-only minute poller (`MathShadowProcessor`) that detects the curated math triggers from Pear's own data. Uncommitted, not run live.

**Evidence:** [[Elfa Triggers, Three Classes (Step 1)]], [[28th Oct 2026]], [[29th Oct 2026]].

**Update 2026-10-29 (end of day):** status above is stale. Work is pushed and in draft PRs, all on the Elfa replacement stack (epic #1190 → `development`, math #1191 on `elfa/math`, narrative #1192, twitter #1193). `Confirmed`
- Math commits on `elfa/math` (11): re-measure fixes (Bollinger, liquidation, RSI, SMA, EMA, Bollinger middle, 2h/4h change, kPEPE casing), shadow poller for curated + 47 personal math triggers, `eth-funding-flip` dead band at −3% APR from a funding backtest, touch sweep and log fixes, and cutover part one (`de8f2a03`, `PEAR_SIGNALS_LIVE`).
- Personal math triggers Pear can measure: 27 → 46 of 47.
- Found that personal triggers depend on Elfa Builder Chat to write their rules, so a Pear rule builder is a launch blocker.

**Update 2026-10-29 (late):** Pear rule builder and math comparison done. `Confirmed`
- Commits on `elfa/math`, pushed: `bb4c16a6` rule builder (Haiku 5.5, keeps a rule only if the measurer can read every leg, 24h backoff on rejects, Kalshi contract picks for Fed/CPI/jobs), `98c4b1d9` OR rules drop an unreadable leg and moves allow 1h to 72h windows, `9aa82cd0` personal fires capped at `maxTriggers` (3) per week.
- Live personal watches: Pear builds 36/36 from the current watch text, Elfa had 5 armed and 0 rules from the current text (100-query plan cap left 30 personal slots, 271 of 295 rows evicted).
- Replay of 46 past personal rules: Pear 0.63 fires per rule per day with the cap vs Elfa 0.85. Curated price: both catch 49 of 49 distinct moves, Pear with 54 pings vs Elfa's 87. Funding: Pear 16/28 flips held vs Elfa 18/94.
- Decisions: a first reading never fires, `maxTriggers` caps personal only. Dashboard: https://claude.ai/artifact/1XaJ5QtNaBFkDTm12AktBJ
- Open: Elfa arming gate, dev rehearsal, Coinglass (#1180 reopened).

**Update 2026-10-30:** math code work done. `d4befdc5` (pushed): with `PEAR_SIGNALS_LIVE` on, the trigger reconcile calls Elfa for nothing, saves every rule for Pear, runs while Elfa is down, and the poller covers the 3 Kalshi alerts and reads only armed rows. Decision: one switch for the whole product, see [[Elfa Replacement — One Switch, Not Math-First]]. PR #1191 description updated. Open: shadow run check (ends ~1 Nov), dev rehearsal, Coinglass (#1180), then the news track. `Confirmed`

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
