# Step 5 — Ambient-Agent Job Migration Plan (apps/agent, 12 jobs)

- Status: Planning only — no code written, no Story opened, no scope decision made. Blocked on the Epic #717/#438 scope question below before any task in this plan starts.
- Date: 2026-09-15
- Project: Agent Pear harness integration
- Related: [[Harness Integration/Step 1 — AP Background Processes and Hetzner VPS Inventory#Analysis — 2026-09-15 (apps/agent ambient-agent jobs: difficulty ranking + Claude-harness benefit)|Step 1 §2026-09-15 analysis]] (risk ranking + effort spot check this plan builds on) · [[Harness Integration/Step 4 — Watched-Accounts + Image-Retention Batch on Hetzner|Step 4]] (the `apps/agent` one-shot precedent every job below reuses) · [[00 Home/Current Focus]]

## Objective

Turn the 2026-09-15 difficulty ranking into an executable, per-job task plan, ordered easiest-first, using the exact five-task decomposition already proven on Stories #745/#746/#759/#765: **kill-switch → one-shot flag → systemd unit → live validate → cutover**. This is a plan, not a grooming decision — GitHub Stories/tasks should be opened from this once the scope question is resolved.

## Precondition — must be resolved before task #1 starts

Epic #717 (yours) explicitly excludes "touch/\*, X posting, signal path" from this migration. 8 of the 12 jobs below (`watch-scheduler`, `ghost-writer`, `ghost-synthesis`, `x-digest`, `touch-brief`, `ghost-shadow`, arguably `global-feedpull`, `signal-funnelrollup`) fall inside that line as written. **Do not open Stories under #717 for those 8 until you've decided whether to amend #717's scope or open a separate epic.** The 4 that are unambiguously in scope regardless (no signal/touch/X involvement) are `ghost-report`, `user-valuescore`, `touch-priors` — wait, `touch-priors` is a `touch-*` job by name; treat it as in the same "confirm scope" bucket as the others. Only `ghost-report`, `user-valuescore`, and `signal-funnelrollup` are clearly outside the excluded categories by function (pure analytics, no touch/X/signal-firing behavior) even though `signal-funnelrollup` has "signal" in its name — worth explicitly confirming that reading before starting, not assuming it.

## Open architecture question — one shared runner or 12 dedicated entrypoints?

Image-retention (Story #765) set the precedent of a **dedicated minimal entrypoint** (`main-image-retention.ts` + a slim runner module) because `apps/agent`'s single `AppModule` boots one monolithic Zod env schema. Doing that 11 more times means 11 more near-duplicate `main-<job>.ts` + `<job>-runner.module.ts` pairs. Two options, not decided here:

- **(A) Replicate the pattern per job** — consistent with precedent, zero shared-abstraction risk, but real duplication across 11 files (each just wires a different DB/queue/service subset).
- **(B) One shared lean entrypoint** parameterized by job name (`main-background-once.ts --job=user-value-score`), with a small per-job registry mapping job name → module + runner function. Less duplication, but it's a new abstraction with no precedent yet in this codebase, and Step 1's other notes have flagged effort-vs-safety mismatches before — a shared entrypoint bug could affect every job's one-shot at once instead of just one.

**Recommendation, not a decision: build (A) for the first 2–3 easiest jobs (ghost-report, user-valuescore, touch-priors) using the proven pattern, then decide whether the duplication is annoying enough to justify (B) once the shape of 3 real instances is visible** — same "prove it 2-3 times before abstracting" instinct the codebase already follows elsewhere (`BasketRunnerModule` was job-specific, not generalized). Revisit this line item once #2–3 in the order below are done.

## Kill-switch pattern (confirmed, reuse exactly)

Per Story #765's real design decision (`#766`): the kill-switch must call `queue.removeJobScheduler()` on disable, not skip-only — skip-only leaves a stale-but-inert Redis scheduler entry still spawning unconsumed jobs. Guard the `OnApplicationBootstrap` scheduling call in each processor (all 12 already implement `OnApplicationBootstrap`, confirmed via code read on 2026-09-15).

## Per-job plan, easiest → hardest

Ordering reuses the 2026-09-15 risk ranking as-is (validated against the effort spot check — only #7 and #10 need an effort caveat, noted inline; the rest of the order is unaffected).

### 1. `ghost-report` — monthly — **DONE (code), cutover pending**
- Queue: `GHOST_REPORT_QUEUE`. Deps: `AgentPrismaService` only — thinnest in the set.
- Kill-switch: `GHOST_REPORT_CRON_DISABLED`.
- One-shot: `--ghost-report-once` → calls the same report-build method the scheduler calls.
- Systemd: `agent-ghost-report.service` (no `.timer` cadence urgency — monthly; `OnCalendar=monthly` or explicit day, confirm current schedule config before wiring).
- Validate: read-only report generation — no upsert-vs-append ambiguity to resolve, lowest-stakes validation of the whole set.
- Cutover: trivial — monthly cadence means even a mistake has a month to be caught before it repeats.

**Update — 2026-09-15 (shipped):** Branch `feat/agent-ghost-report-hetzner` off `development`, [PR #787](https://github.com/pear-protocol/agent-pear-v2/pull/787) open. Kill-switch (`GHOST_REPORT_CRON_DISABLED`, explicit `removeJobScheduler()` not skip-only, unit-tested both ways) and one-shot (`--ghost-report-once` via `main-ghost-report.ts` + `GhostReportRunnerModule`, sharing `buildAndLogGhostReport()` with the BullMQ path) both done. **Live-validated on Hetzner**: 874 active ghosts, 124 owners processed, clean exit, no hang — no `redis.quit()` fix needed, confirming this plan's guess that a Redis-free job wouldn't need it. Systemd files (`agent-ghost-report.service`/`.timer`, monthly `07:00 UTC` on the 1st, flock-guarded, `.env.example` scoped to just `AGENT_DATABASE_URL`, README) written but **not installed/enabled** — deliberately deferred, per this plan's own task split. `GHOST_REPORT_CRON_DISABLED` not yet flipped anywhere; Railway's in-process scheduler is still the live one. Cutover (install timer, flip flag, monitor) is the next, separate step — not started.

### 2. `user-valuescore` — daily 04:30 — **DONE (code), cutover pending**
- Queue: `USER_VALUE_SCORE_QUEUE`. Deps: `AgentPrismaService` only.
- Kill-switch: `USER_VALUE_SCORE_CRON_DISABLED`.
- One-shot: `--user-value-score-once`.
- Systemd: `agent-user-value-score.service/.timer`, `OnCalendar=*-*-* 04:30:00`.
- Validate: confirm idempotency (upsert keyed on user id, matching the Tokenomist/OI before/after-`updatedAt` validation method from Story #745) — not yet confirmed in code, check `computeUserValueScores`/`user-value-score.compute.ts` before assuming upsert semantics.
- Cutover: same two-command pattern as #745/#765.

**Update — 2026-09-15 (shipped):** Branch `feat/agent-user-value-score-hetzner` off `origin/development`, [PR #788](https://github.com/pear-protocol/agent-pear-v2/pull/788) open, CI green, `mergeStateStatus: CLEAN`. Kill-switch (`USER_VALUE_SCORE_CRON_DISABLED`, `removeJobScheduler()`) and one-shot (`--user-value-score-once` via `main-user-value-score.ts` + `UserValueScoreRunnerModule`, extracted `computeAndPersistUserValueScores` shared with the BullMQ processor) both done, 10/10 unit tests. **Live-validated on Hetzner** against dev DB: `scored=273 top=0.99 median=0.207`, clean exit. Systemd files written, not installed.

**Resolved, correcting this plan's own flagged uncertainty:** `computeUserValueScores` persists via `createMany` — **append-only, not an upsert**, no unique key on `userId`. Story #745's before/after-`updatedAt` diff validation method does **not** apply here; the eventual cutover validation for this job needs a different method (e.g. row-count delta or a dedicated read-back), not yet designed.

**Real bug found and fixed here, but still latent in PR #787 — needs a follow-up fix:** `ConfigService` was imported as `import type` in the runner-module scaffold (copied from the ghost-report reference file), which erases the runtime reference NestJS's `emitDecoratorMetadata` needs for `design:paramtypes`, breaking DI. Fixed in this PR by using a real import. **PR #787 has the same latent bug**, uncaught there because nothing e2e-tests `AiModule` with `GhostReportProcessor` registered — flagged as a pre-merge fix for #787, not yet done. *(Superseded — see Correction below: this specific claim about #787 was wrong.)*

**Correction — 2026-09-15:** Investigated the "PR #787 has the same latent bug" claim above directly on that branch. **The bug does not exist in PR #787** — `ghost-report.processor.ts`'s `ConfigService` import was already a real `import { ConfigService } from "@nestjs/config"`, not `import type`, and neither `ghost-report-runner.module.ts` nor `main-ghost-report.ts` reference `ConfigService` at all. The claim above was inaccurate; #787 got the import right the first time, #788 apparently didn't (on its own copy of the scaffold) and fixed it there. What #787 *was* missing, and what got added instead: a compile-time DI regression test (`ghost-report.processor.compile.spec.ts`) that resolves `GhostReportProcessor` through real `Test.createTestingModule(...).compile()` DI resolution (reading `design:paramtypes`), rather than the hand-constructed `new GhostReportProcessor(...)` the existing unit spec uses, which can't catch this bug class at all. Verified the new test actually works, not just that it exists: temporarily reintroduced `import type { ConfigService }`, confirmed the test failed with Nest's own diagnostic pointing at the type-only import, then reverted and confirmed both suites pass (5/5). Full `AiModule` compilation (as originally asked for) wasn't practical — it registers 16 BullMQ queues via `registerQueueAsync` factories that open real Redis connections at compile time plus ~50 unrelated providers — so the test targets `GhostReportProcessor` directly with real dependencies, matching the existing `x-share.module.compile.spec.ts`/`ws.module.compile.spec.ts`/`signals-core.module.compile.spec.ts` precedent of testing smaller standalone slices rather than the whole module. **PR #787 needs no further DI fix; the regression-test push is the only change from this task.**

**Git-hygiene note:** branched fresh off `origin/development` rather than off the ghost-report branch, since `development` doesn't yet have PR #787's flag/script anchors (that PR is still unmerged) — two merge conflicts resolved by hand rather than assuming ghost-report's state. Confirms these per-job branches are independent siblings, same pattern Story #745/#746 used, not a stack.

### 3. `touch-priors` — daily 04:50
- Queue: `TOUCH_PRIORS_QUEUE`. Deps: `AgentPrismaService` + one `ConfigService<EnvConfig>` slice — confirm exactly which env keys before scoping the dedicated entrypoint's schema subset.
- Kill-switch: `TOUCH_PRIORS_CRON_DISABLED`.
- One-shot: `--touch-priors-once`.
- Systemd: `agent-touch-priors.service/.timer`, `OnCalendar=*-*-* 04:50:00`.
- Validate: same upsert-diff method as #2, pending confirmation of write shape.
- Cutover: standard two-command pattern. Flag before starting: confirm this doesn't fall under Epic #717's "touch/\*" exclusion despite being purely internal-state (no outbound send) — see Precondition above.

### 4. `signal-funnelrollup` — weekly (Mon 06:00) — **DONE (code), cutover pending**
- Queue: `SIGNAL_FUNNEL_ROLLUP_QUEUE`. Deps: `AgentPrismaService` + `RedisService` + `ElfaCreditBudgeterService` (→ Redis + `ElfaClient` + config, one hop deeper).
- Kill-switch: `SIGNAL_FUNNEL_ROLLUP_CRON_DISABLED`.
- One-shot: `--signal-funnel-rollup-once`. Needs the raw-Redis `redis.quit()` fix in `finally` (same bug class as #748/#753 — apply proactively, don't rediscover it live).
- Systemd: `agent-signal-funnel-rollup.service/.timer`, `OnCalendar=Mon *-*-* 06:00:00`.
- Validate: needs an Elfa API credential in the dedicated entrypoint's env subset — confirm which key (`ELFA_API_KEY` or similar, not yet confirmed) before building the minimal schema.
- Cutover: weekly cadence gives a comfortable observation window; standard two-command pattern.

**Update — 2026-09-15 (shipped):** Branch `feat/agent-signal-funnel-rollup-hetzner`, [PR #789](https://github.com/pear-protocol/agent-pear-v2/pull/789) open. Kill-switch (`SIGNAL_FUNNEL_ROLLUP_CRON_DISABLED`, `removeJobScheduler()`) and one-shot (`--signal-funnel-rollup-once` via `main-signal-funnel-rollup.ts`, extracted `runSignalFunnelRollup()` shared with the BullMQ processor) both done. First Redis-backed one-shot in the batch — the raw-`REDIS`-token `.quit()` fix was applied proactively as planned, not rediscovered live. 10/10 on the file, 290/290 across the full `background/*` suite, `tsc --noEmit` clean. **Live-validated on Hetzner**: booted, connected to all three Prisma DBs, exited cleanly; the run itself was a no-op because the watermark was already caught up — correct behavior, not a failure.

**Two new reusable lessons for jobs 5–12, apply proactively:**
1. **Build command**: `pnpm --filter agent build` bypasses Turbo's dependency graph and fails on `Cannot find module '@agent-pear/execution'`. Use `pnpm build:agent` instead.
2. **Runner-module env schema**: loading the full app-wide `createEnvConfig()` in a dedicated runner module pulls in 15+ unrelated required vars (`TELEGRAM_BOT_TOKEN`, `LANGFUSE_*`, etc.) — the exact monolithic-schema problem Story #765 originally solved by going minimal. Use a bare `ConfigModule.forRoot()` (raw env, no schema validation) in every new runner module, not `createEnvConfig()`. Jobs 1–2's runner modules didn't hit this because their dependency lists were even thinner; this will matter more as later jobs (`ghost-synthesis`, `x-digest`, `global-feedpull`) add more config-reading services.

**New sequencing dependency found, not in the original cross-job list:** `signal-funnel-rollup` soft-reads `touchCandidate` rows that `touch-brief` (job 11) writes — wrapped in try/catch, so not a hard blocker, but worth sequencing awareness once `touch-brief`'s migration comes up. Added to the cross-job dependency list below.

### 5. `interest-distiller` — daily 04:00
- Queue: `INTEREST_DISTILLER_QUEUE` (confirmed, `background.types.ts`). Deps, **confirmed by reading the full processor file 2026-09-15** — deeper than originally guessed: `ConfigService<EnvConfig>`, `AgentPrismaService`, `OPENROUTER_MODEL_FACTORY` (wraps the Cerebras call via `resolveNodeModel`), `AssetIndexReader` (the **same cross-app dependency on `apps/indexing`'s asset index that makes `global-feedpull` the deepest of the no-LLM jobs** — this job also needs that index hydrated, and already skips its whole run cleanly if `assetIndex.isReady()` is false), `SignalAdaptivePolicyService`, `SignalUserGateService`. Effort tier: closer to `global-feedpull`/`ghost-writer` than the thin jobs 1–4.
- Writes: wipes and rewrites `UserInterest` rows per user (`deleteMany` + `createMany` in a `Serializable` transaction with retry-on-conflict for `P2034`/`P2002`), preserving rows owned by `xdigest`. Also writes capped `FollowUpHook` rows separately. **Destructive-write shape** — a genuine dual-scheduler race here isn't just a stale row, it's two concurrent wipe-and-rewrite transactions on the same user; the existing retry logic handles ordinary conflicts but hasn't been proven against an actual Railway+Hetzner overlap.
- Volume: per-active-user loop (active = memory or journal touched in the last 30 days), **no fixed cap** — unlike made-baskets' bounded daily target, this scales with active-user count. Rough proxy from other jobs' live-validation output (`user-value-score` scored 273, `ghost-report` processed 124 owners): likely low-hundreds of calls/night today, growing with the user base.
- **Useful existing hook**: the processor already supports a single-user run via `job.data.userId` — the one-shot CLI should expose an equivalent `--user-id=<id>` flag for a safe single-user validation before ever doing a full-user-base run, mirroring the caution Story #765 wishes it had applied to `RUN_IMAGE_RETENTION`'s first real purge.
- Kill-switch: `INTEREST_DISTILLER_CRON_DISABLED`.
- One-shot: `--interest-distiller-once` (+ optional `--user-id`). First LLM-calling job in this plan — confirm the Cerebras API key env var and check whether `AssetIndexReader`/`SignalAdaptivePolicyService`/`SignalUserGateService` hold a raw Redis client needing the `.quit()`-in-`finally` fix (established pattern since Story #745) — don't assume either way, verify.
- Systemd: `agent-interest-distiller.service/.timer`, `OnCalendar=*-*-* 04:00:00`.
- Validate: use the new `--user-id` flag for a single-user dry pass first. Confirm whether interest-distiller has ever actually run at its current cadence before assuming steady-state behavior.
- Cutover: standard pattern, but monitor Cerebras cost/latency post-cutover since this is the first LLM job moved in this batch, and confirm no concurrent-wipe race during the cutover window specifically.
- **Scope note:** this job is the one not covered by Epic #717's named exclusion list (not touch/X/signal-shaped) — user confirmed 2026-09-15 to proceed with it without waiting on the full #717 scope resolution.

**Update — 2026-09-15 (shipped, including systemd):** Branch `feat/agent-interest-distiller-hetzner`, [PR #791](https://github.com/pear-protocol/agent-pear-v2/pull/791) open against `development`. Kill-switch (`INTEREST_DISTILLER_CRON_DISABLED`, `removeJobScheduler()`) done and tested. One-shot: extracted the full batch loop into a standalone `runInterestDistiller(deps, userId?)` shared with the BullMQ processor, `main-interest-distiller.ts` (`--interest-distiller-once` + optional `--user-id=<id>` for staged single-user validation — the flag this plan asked for), `interest-distiller-runner.module.ts` (bare `ConfigModule.forRoot()`). Confirmed `AssetIndexReader` does hold a raw Redis client with no `OnModuleDestroy` — the `redis.quit()`-in-`finally` fix was applied proactively, not discovered live, and **the live Hetzner run confirms it actually works** (process exited on its own after a real run), not just that it compiles. `tsc --noEmit` clean, 960+/965 jest (5 pre-existing unrelated failures confirmed via `git stash`), biome clean.

**Live-validated on Hetzner, staged correctly**: first attempt failed cleanly on missing `CEREBRAS_API_KEY` (confirmed per-user error isolation doesn't crash the batch on a config problem), then a real single-user run (`--user-id`) succeeded — `1/1 users written, 0 failed`, real Cerebras call, real `UserInterest` write.

**Systemd shipped too** (asked for mid-session, beyond this task's original stop-after-one-shot scope): `agent-interest-distiller.service/.timer` (04:00 UTC, flock-guarded), env template, README. **Not installed/enabled** — deliberately, cutover is still a separate step.

**Real production-parity footgun found and flagged (not yet fixed):** the minimal runner module skips the Zod env schema entirely, so `SIGNAL_ADAPTIVE_POLICY`'s **default-`true`** (which applies when the full schema validates it) does **not** apply in the one-shot — if that env var isn't set explicitly in the Hetzner env file, the one-shot silently runs in **legacy non-adaptive mode**, diverging from what the Railway cron actually does. This is the same class of risk as `RUN_IMAGE_RETENTION` being silently unset in Story #765, but caught before a live cutover instead of after months of running — confirm `SIGNAL_ADAPTIVE_POLICY=true` is explicitly set in `agent-interest-distiller.env` before any real cutover.

**Not done, correctly per scope:** systemd not installed/enabled on the VPS, `INTEREST_DISTILLER_CRON_DISABLED` not flipped on Railway — production still runs the nightly job unchanged. PR description was mid-update (to mention the systemd addition) when this update was recorded.

**Update — 2026-09-16 (separate model-quality question, not a migration-cutover task):** picked up the Cerebras→Claude-API swap question this section's risk entry didn't cover and Step 1's 2026-09-15 analysis flagged as a deferred follow-up. Produced a research plan only (CLI invocation via `ClaudeCodeClient`, structured-output design, a real-measurement methodology using this job's own `--user-id` flag, an offline quality eval against the `isStyleTopic()` violation rate, risk list) — no code written, no decision made, independent of and does not block this job's own Hetzner cutover above. Confirmed live via `gh pr view 791`: still OPEN, unmerged, as this entry already states. Full detail: [[Harness Integration/Step 7 — Claude CLI Feasibility Research for interest-distiller]].

### 6. `watch-scheduler` (sync) — daily 05:30
- File: `watch-scheduler-sync.processor.ts`. Queue/deps not yet traced beyond the file name — confirm before building (this plan didn't do even a shallow constructor check on this one; don't assume it's as thin as #1–4).
- Kill-switch: `WATCH_SCHEDULER_SYNC_CRON_DISABLED`.
- One-shot: `--watch-scheduler-sync-once`.
- Systemd: `agent-watch-scheduler-sync.service/.timer`.
- Validate: "user-originated workflow" per the risk ranking — validate against a real user's watch entries, not just row counts, since correctness here is user-visible (a watch scheduled wrong is a product bug, not just a data-quality issue).
- Cutover: standard pattern, extra caution on the first live cycle given user-visible blast radius.

### 7. `ghost-writer` — daily 05:15 — **effort caveat**
- Queue: `GHOST_WRITER_QUEUE`. Deps: `AgentPrismaService` + `TriggerSynthesizerService`, which itself needs `SignalAdaptivePolicyService`, `WatchSchedulerService`, `SignalUserGateService`, config — the deepest dependency subtree of the no-LLM jobs (confirmed 2026-09-15 spot check).
- Kill-switch: `GHOST_WRITER_CRON_DISABLED`.
- One-shot: `--ghost-writer-once`. Building the dedicated entrypoint here means wiring 4 services, not 1 — expect this to take noticeably longer than #1–6 despite being risk-ranked lower than #8/#9. Good candidate to reconsider option (B) from the architecture question above, since duplicating this much DI wiring per job stops looking cheap.
- Systemd: `agent-ghost-writer.service/.timer`, `OnCalendar=*-*-* 05:15:00`.
- Validate: confirm no duplicate-trigger-write risk if a Railway consumer briefly overlaps a Hetzner one-shot during cutover — this job seeds the trigger set two steps upstream of `ghost-shadow`'s live signal fires, so a double-write here propagates.
- Cutover: standard pattern, but confirm `ghost-synthesis` (downstream) isn't reading mid-write.

### 8. `ghost-synthesis` — daily 05:45
- Deps: Cerebras (`alert-matcher` judge + `signal` author calls) + `OpenRouterEmbedder` for similarity — the only OpenRouter touchpoint in this whole batch.
- Kill-switch: `GHOST_SYNTHESIS_CRON_DISABLED`.
- One-shot: `--ghost-synthesis-once`. Needs both a Cerebras key and an OpenRouter key in the dedicated entrypoint's env subset — first job in this plan needing two LLM-adjacent credentials.
- Systemd: `agent-ghost-synthesis.service/.timer`, `OnCalendar=*-*-* 05:45:00`.
- Validate: writes to **shared** signal context (not per-user) — validate with a read-only dry-run first if one exists; if not, that's a gap worth adding before this one-shot ever does a real write, given the shared-blast-radius risk called out in the ranking.
- Cutover: highest-caution cutover of the daily jobs so far — confirm `ghost-writer`'s output is stable before this job's Hetzner run reads it.

### 9. `x-digest` — daily 05:00
- Deps: Cerebras + `TwitterApiClient` (twitterapi.io, paid).
- Kill-switch: `X_DIGEST_CRON_DISABLED`.
- One-shot: `--x-digest-once`. Needs the Twitter API credential in the entrypoint's env subset — confirm rate-limit/quota handling isn't tied to something Railway-specific (IP allowlisting, etc.) before assuming a Hetzner-origin call behaves identically.
- Systemd: `agent-x-digest.service/.timer`, `OnCalendar=*-*-* 05:00:00`.
- Validate: per-user loop over X-linked users — same "confirm real production volume before a full run" caution as Story #773 applied to Twitter-ingestion (that story measured ~280–310 calls/day before deciding to move it).
- Cutover: standard pattern, but this is also the job most likely affected by Epic #717's "X posting" exclusion line — resolve the scope question specifically for this one before building anything.

### 10. `global-feedpull` — 2x daily — **effort caveat**
- Queue: `GLOBAL_FEED_PULL_QUEUE`. Deps: `AgentPrismaService` + `ConfigService` + `RedisService` + `ElfaDataService` (→ Redis + `AssetIndexReader`, the same Redis-backed asset index `apps/indexing` builds under Story #746) + `ElfaCreditBudgeterService` — deepest dependency footprint of the whole batch, confirmed 2026-09-15.
- Kill-switch: `GLOBAL_FEED_PULL_CRON_DISABLED`.
- One-shot: `--global-feed-pull-once`. Real precondition: this one-shot needs `apps/indexing`'s asset-index Redis cache already warm — if Story #746's `AssetIndexService` cache is stale or that migration hasn't cut over yet, this job's Hetzner run could read stale asset data. **Sequencing dependency on Step 3's Story #746, not just an independent job — confirm #746 has fully cut over before scheduling this one's live validation.**
- Systemd: `agent-global-feed-pull.service/.timer`, twice-daily cadence — confirm exact UTC times before writing `OnCalendar`.
- Validate: writes the actual global feed content every user sees — validate against a shadow/dry-run write path if one exists before a real cutover; this is directly user-facing, unlike #1–7.
- Cutover: highest-caution cutover of the no-LLM jobs — external API budget + user-facing + cross-app dependency stack together here.

### 11. `touch-brief` — daily 06:40
- Deps: Cerebras, reads `GlobalFeedItem` rows `global-feedpull` already wrote — **sequencing dependency on #10**, don't build this before #10 is stable.
- Kill-switch: `TOUCH_BRIEF_CRON_DISABLED`.
- One-shot: `--touch-brief-once`.
- Systemd: `agent-touch-brief.service/.timer`, `OnCalendar=*-*-* 06:40:00`.
- Validate: this is the last internal step before an actual outbound notification (per Epic #717's own boundary) — validate with notification-send explicitly disabled/dry-run first; do not let a Hetzner one-shot's first real run also be the first time it sends something to a live user.
- Cutover: treat as the highest-caution cutover among the daily jobs; confirm the scope question resolves this one in-scope before starting at all, since it sits closest to #717's exclusion line.

### 12. `ghost-shadow` — every 15 minutes
- Deps: `SignalMeasurementService`, deterministic, no LLM.
- Kill-switch: `GHOST_SHADOW_CRON_DISABLED`.
- One-shot: `--ghost-shadow-once`. Mechanically probably the simplest LLM-free job to wrap (deterministic evaluator, likely thin deps like #1–4) — but building the one-shot is not the hard part here.
- Systemd: `agent-ghost-shadow.service/.timer`, `OnCalendar=*:0/15` (same pattern as Story #773's `context-twitter-ingestion.timer`).
- Validate: **this is the live trigger-evaluation/signal-fire loop** — a duplicate consumer during cutover fires signals twice, 96 times/day, not once. Do not attempt a real cutover without a locking/mutual-exclusion check at least as strong as the `node dist/main --tokenomist-once --oi-once` mutual-exclusion guard from Story #745, and ideally a longer parallel-run observation window than any other job in this plan given the frequency.
- Cutover: last job in this plan for a reason — do not schedule this until every other job's one-shot/kill-switch pattern has been proven live at least once, since this is the closest analog to Step 1's `ap-worker`/`execute-ticket` "do not duplicate or move casually" bucket.

## Sequencing dependencies across jobs (not just within one)

- `global-feedpull` (#10) needs Story #746 (`apps/indexing` asset index) fully cut over first.
- `touch-brief` (#11) needs `global-feedpull` (#10) stable first — reads its output.
- `ghost-synthesis` (#8) needs `ghost-writer` (#7) stable first — reads its output.
- `ghost-shadow` (#12) should be last regardless of its own simplicity, per the frequency/blast-radius argument above.
- `signal-funnelrollup` (#4, shipped) soft-reads `touchCandidate` rows `touch-brief` (#11) writes — found 2026-09-15, wrapped in try/catch so not a hard blocker, but confirm `touch-brief`'s cutover doesn't change that table's shape/availability before assuming `signal-funnelrollup` stays unaffected.

## Correction — 2026-09-15 (precondition section above was internally inconsistent)

The original Precondition section named "8 excluded" jobs (including `signal-funnelrollup`), then separately said "the 4 unambiguous" ones were `ghost-report`/`user-valuescore`/`touch-priors`, then contradicted that in the very next sentence by naming `ghost-report`/`user-valuescore`/`signal-funnelrollup` as the 3 clear ones instead — swapping `touch-priors` out and `signal-funnelrollup` in without flagging the inconsistency. That error wasn't caught before jobs 1, 2, and 4 were built and shipped under the "3 confirmed clear" reading (which is how it played out in practice — `signal-funnelrollup` proceeded, `touch-priors` didn't).

**Net effect, stated plainly now:** `ghost-report`, `user-valuescore`, and `signal-funnelrollup` are the 3 jobs treated as clear-of-exclusion and are now all shipped (PRs #787/#788/#789). `touch-priors` was held back as ambiguous. **`interest-distiller` (job 5, next in risk order) was never actually run through this scope check at all** — it's not touch/X/signal-named or -shaped (it analyzes user memory/preferences, not touch delivery, X posting, or signal firing), so the same "functionally outside the excluded categories" reasoning that cleared `signal-funnelrollup` would plausibly clear it too — but this plan asserted that for `signal-funnelrollup` without asking first, and got lucky that it held up. Do not repeat that pattern silently for `interest-distiller` — this is a real decision point for the user, not something to infer again.

## What this plan does NOT cover

- No code has been written for any of these 12 jobs.
- The scope question (Epic #717 vs a new epic) is not resolved — this plan assumes it will be, but doesn't decide it.
- Exact env var names for Cerebras/Elfa/Twitter API credentials are not all confirmed against `app.config.ts` — flagged per-job above where uncertain, not guessed.
- `watch-scheduler-sync`'s actual dependency depth was never checked (this plan's own 2026-09-15 effort spot check only covered the 6 lowest-risk jobs) — confirm before treating it as equivalent in effort to its risk rank.
