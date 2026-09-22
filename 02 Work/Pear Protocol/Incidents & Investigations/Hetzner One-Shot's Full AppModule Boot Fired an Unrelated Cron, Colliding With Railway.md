---
type: investigation
area: pear
status: partially-fixed
created: 2026-09-19
---

# Hetzner One-Shot's Full AppModule Boot Fired an Unrelated Cron, Colliding With Railway

## Symptom

During the first real dry runs of the `apps/context` Hetzner one-shots on `development` HEAD (post-PR #800), the Tokenomist one-shot ran its real work correctly but never exited on its own. The following OI one-shot also ran its real work correctly, but while it was still alive, an unrelated cron (`TwitterCronService`) fired on its own schedule, processed a batch of tweets (LLM labeling, DB writes, Discord post) that had nothing to do with the OI job, and roughly a minute later a second `TwitterCronService` run — almost certainly Railway's own regularly-scheduled production instance — collided with it: `stored: 0 failed: 10` on the same batch size (confirmed via a Discord screenshot the user captured independently).

## Context

Both jobs were run manually (not via systemd, which isn't installed/enabled for either yet) against the box's `development` checkout, per [[Harness Integration/Step 3 — Tokenomist + OI Canaries on Hetzner]]. This is the first time either one-shot has run against current `development` (post `#757`/`#758`/`#800` merges, 2026-09-18/19).

## Evidence

1. **Tokenomist one-shot (`--tokenomist-once`, no flock/systemd):** logged `Refreshed 136/136 assets in 111.4s` and `Tokenomist one-shot complete` — matching the known-good baseline exactly. The process then did **not** exit. Left running 26+ minutes with CPU trending to ~0%, then manually `SIGTERM`'d (exit 143). This is a live reproduction of the P1 already flagged (but not fixed) in the 2026-09-16 code review of PR #757 — see Step 3's own "Still open, not fixed this pass" note: *"a delayed/manual/slow invocation spanning a cron boundary can trigger unrelated scheduled writers outside the one job's own flock."*
2. **OI one-shot (`--oi-once`, wrapped in `timeout 240` after finding #1):** logged `Synced 188/188 assets (exchange: Hyperliquid)` and `OI one-shot complete` — correct. But ~1 minute into the run, log output showed `TwitterCronService: 🕐 Batch processing triggered by scheduled cron` firing **on its own**, unrelated to the requested `--oi-once` job. `TwitterBatchProcessorService` then ran a full batch: fetched 15 pending items, called an LLM (`ai-sdk`) to label 14 of them, wrote `stored=10 rejected=4 failed=0 notFound=1` to the context DB, and posted a summary to Discord. The process still didn't exit on its own; killed by the `timeout` wrapper (exit 124).
3. **User-provided Discord screenshot** shows two `TwitterCronService` posts back to back: 1:00 AM IST (`stored: 10 | failed: 0` — matches #2 above) and **1:01 AM IST** (`stored: 0 | failed: 10`, same `analyzed: 14`, same `rejected: 4`) — a second, independent firing one minute later.
4. Checked the Hetzner box directly after the run: no `dist/main` process left alive, no user crontab, no systemd timer matching Twitter anything. Nothing on the box could have produced the second post — pointing to Railway's own regularly-scheduled production `TwitterCronService` instance colliding with the one our OI one-shot accidentally triggered.
5. Code read, `apps/context/src/twitter/services/collector.service.ts` — `getPendingContent()` is a plain `prisma.pendingContextTweet.findMany({ where: { processed: false, notFound: false }, ... })`. No row-level claim (`SELECT ... FOR UPDATE SKIP LOCKED`, an atomic update-then-return, or similar). Two concurrent callers can fetch the identical set of pending rows.
6. Code read, `apps/context/src/twitter/services/batch-processor.service.ts` — the per-item `catch` block for a non-"bad request" error deliberately leaves the item **unmarked** (not added to `processedIds`/`rejectedIds`/`notFoundIds`), with a comment stating it is "re-picked next cycle." This suggests the second run's 10 "failed" items stay `processed: false` and will be retried on Railway's next cycle rather than being permanently lost — **inferred from the code, not independently confirmed** by observing an actual successful retry.

## Finding / Root Cause (two compounding causes)

1. **`apps/context`'s Hetzner one-shots boot the full `AppModule`**, not a scoped module — so `ScheduleModule.forRoot()` and every other `@Cron` job in the app (Twitter batch processing, and presumably others) are live and can fire during the one-shot's lifetime, entirely unrelated to the one job that was actually requested. This exact risk was named in the 2026-09-16 review of PR #757 ("can trigger unrelated scheduled writers outside the one job's own flock") but the fix was not made before #757 (and sibling #758) merged into `development` on 2026-09-18.
2. **`TwitterCollectorService.getPendingContent()` has no per-item claim/lock**, so when two instances of the same job run within the same short window — Railway's regular scheduled instance and our accidental Hetzner-triggered one — both can fetch and attempt to process the same pending rows. This is a pre-existing concurrency gap in the Twitter ingestion path itself, independent of the Hetzner migration; the migration's side effect is what exposed it live.

Neither one-shot's *own* requested work was wrong — Tokenomist and OI both refreshed real data correctly. The problem is exclusively in what else the process boot brought along for the ride, and how the affected write path handles two callers at once.

## Fix / Decision

**Not yet fixed.** Paused further `apps/context` one-shot dry runs (including `apps/indexing`, not yet checked for the same exposure) rather than risk triggering more unplanned production side effects. Two separate fixes are needed:

- Scope `apps/context`'s one-shot entrypoints to a dedicated minimal module per job (mirroring the pattern `apps/agent`'s `GhostReportRunnerModule`/`UserValueScoreRunnerModule`/`SignalFunnelRollupRunnerModule` already use safely for ghost-report/user-value-score/signal-funnel-rollup), instead of `NestFactory.createApplicationContext(AppModule, ...)`.
- Add a per-item claim (`SELECT ... FOR UPDATE SKIP LOCKED` or an atomic conditional update) to `TwitterCollectorService.getPendingContent()` so two concurrent batch runs can't double-process the same pending rows — this fix stands on its own regardless of the Hetzner issue.

## What Fooled Us

Nothing — the exact failure mode was already named in writing three days earlier (PR #757's 2026-09-16 review) and left as a known, unfixed risk when the PR merged anyway on 2026-09-18. This session is the live confirmation, not a new discovery of the underlying exposure.

## Remaining Unknowns

- Whether the second run's 10 "failed" items actually retried successfully on Railway's next cycle — not independently verified (would need a Railway-side DB or log check).
- Whether `--baskets-once`, `--watched-accounts-once`, and `--twitter-ingestion-once` (same `apps/context/src/main.ts` file, same `AppModule` boot pattern) carry the identical exposure — very likely given the shared code path, but not individually re-verified this session.
- Whether `apps/indexing`'s one-shot entrypoint has the same class of risk — not checked this session; dry runs there were paused before reaching it.
- The exact error text behind the second run's 10 failures was not directly observed (only Discord's aggregate counts were visible) — the "will retry next cycle" read is from the code path, not a captured log line.

## Related Work

[[Harness Integration/Step 3 — Tokenomist + OI Canaries on Hetzner#Update — 2026-09-19 (predicted risk confirmed live: an OI one-shot fired an unrelated production Twitter cron, colliding with Railway)]]
[[00 Home/Current Focus]]

## Lessons

[[Engineering Lessons#A one-shot that boots the full app isn't scoped to one job — and an unlocked write path turns a coincidence into a collision]]

## Update — 2026-09-19 (root cause #1 fixed and merged; root cause #2 still open)

**Root cause #1 (full-`AppModule` boot) is fixed.** Every `apps/context` one-shot (`--tokenomist-once`, `--oi-once`, `--watched-accounts-once`, `--twitter-ingestion-once`) and every `apps/indexing` one-shot (`--asset-index-once`, `--coin-index-once`, `--tokenomist-index-once`) now boots a minimal per-job runner module instead of `AppModule`, mirroring `BasketRunnerModule`. Without `ScheduleModule`, none of the reused services' `@Cron` decorators register — PR #803, merged into `development` 2026-09-19.

Live-tested on Hetzner, on the fix branch, before merge: all 7 one-shots ran with correct data, clean process exit, and no side-fired jobs — confirming this resolves both open questions from "Remaining Unknowns" above (`--watched-accounts-once`/`--twitter-ingestion-once` did carry the identical exposure via the same `apps/context/src/main.ts` `AppModule` boot; `apps/indexing`'s one-shots did too). Also live-tested the 4 `apps/agent` one-shots (`ghost-report`, `image-retention`, `signal-funnel-rollup`, `user-value-score`) after the fact — all working, unrelated to this bug (they already used a scoped-module pattern).

**Found a second, unrelated live bug while validating the fix**: `DiscordListenerService` opens a persistent `discord.js` gateway `Client` in `onModuleInit()` with no teardown hook, so `--watched-accounts-once`/`--twitter-ingestion-once` (the only one-shots that load `TwitterModule`) hung indefinitely after logging completion — killed by a 300s safety timeout in testing. Fixed in the same PR #803 by implementing `OnModuleDestroy` on the class.

**Root cause #2 (`TwitterCollectorService.getPendingContent()`'s missing per-row claim) is still NOT fixed** — deliberately deferred, needs either a claim/lease column or a `FOR UPDATE SKIP LOCKED` transaction, both bigger than PR #803's scope. Not yet filed as its own tracked GitHub issue as of this update — the fix in #803 only closes the *exposure window* (an accidental extra Twitter-cron firing can no longer happen from a one-shot process), it does not close the underlying concurrency gap itself, which remains real if two legitimate schedules (e.g. Railway + a future Hetzner Twitter-ingestion timer) ever overlap.

A related, separately-discovered bug was filed as its own issue: [[Engineering Lessons#A framework's graceful-shutdown only tears down what it manages]] links the Discord-client find; **#806** on GitHub tracks a `CoinIndexService` cache-race (`onModuleInit`'s cache-preferring refresh can silently swallow a one-shot's forced-refresh request) — same general family (an eager startup refresh racing an explicit one-shot call), but a distinct bug in a different service, not fixed as part of #803.
