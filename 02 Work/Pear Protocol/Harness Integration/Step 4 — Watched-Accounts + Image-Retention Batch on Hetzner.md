# Step 4 — Watched-Accounts + Image-Retention Batch on Hetzner

- Status: In progress — Story #759 code-complete (PR #771 open, #763/#764 live validate+cutover status not confirmed). Story #765 code-complete (PR #772 open), explicitly paused before #769 live validation pending a decision on the RUN_IMAGE_RETENTION finding below.
- Date: 2026-09-13
- Project: Agent Pear harness integration
- Related: [[Harness Integration/Step 3 — Tokenomist + OI Canaries on Hetzner]] · [[Harness Integration/Step 1 — AP Background Processes and Hetzner VPS Inventory]] · [[My Work at Pear Protocol]]

## Objective

Continuation of Epic #717's canary-class migration, next batch after Story #745/#746: the two remaining "no LLM, deterministic" jobs from the original `step1.pdf` audit that hadn't been groomed yet — watched-account fetch (`apps/context`, hourly) and image-retention purge (`apps/agent`, daily). Explicitly excluded from this batch: the 30-min Twitter-ingestion labeling cron, which the audit itself recommends keeping on Railway (LLM cost × high frequency) — not an oversight, a deliberate scope boundary.

## Grooming

Two new Story issues opened under Epic #717, same shape as #745/#746: **Story #759** (watched-account fetch, tasks #760-#764) and **Story #765** (image retention, tasks #766-#770). Epic #717's body updated to list this as migration-order phase 4.

## Story #759 — watched-account fetch — status

Branch `feat/context-watched-accounts-hetzner`, stacked on `feat/context-cron-hetzner-canaries` (PR #757, still unmerged) because the one-shot flag reuses that branch's `main.ts` scaffolding — same dependency risk that implies (inherits #757 until it merges).

Code-complete: #760 kill-switch (`WATCHED_ACCOUNT_FETCH_CRON_DISABLED`, guards both the `@Cron(EVERY_HOUR)` tick and the `onModuleInit` `RUN_ON_STARTUP` path without touching the sibling `processDiscordTweets` startup call in the same block), #761 `--watched-accounts-once` one-shot (with the proactive `redis.quit()` fix), #762 systemd units. PR [#771](https://github.com/pear-protocol/agent-pear-v2/pull/771) open. **#763 (live Hetzner validate) and #764 (cutover) status not confirmed** — user is running these directly, no evidence back yet either way.

Queue-dependency check done during grooming: `PendingContextTweet` rows this job writes are read only by `TwitterBatchProcessorService.processBatch()` (same app, same DB, staying on Railway) — no orphaned-consumer risk from moving the producer to Hetzner.

## Story #765 — image retention — status

Branch `feat/agent-image-retention-hetzner`, fresh off `development` (independent, unlike #759).

**Architecturally different from every prior story in this epic**, discovered during grooming: BullMQ `upsertJobScheduler`-based, not `@Cron` — lives in `apps/agent` (not `apps/context`/`apps/indexing`), the biggest/busiest app in the repo, with two bootstrap entrypoints (`main-tg`, `main-api`) both pulling `ImagesModule` in transitively via `AiModule`, plus a third (`main-worker`) via `WorkerConsumerModule`. `apps/agent` also had zero one-shot CLI precedent before this story.

Code-complete, two commits on the branch (no attribution trailer — see [[Engineering Lessons#A session-level instruction can silently override a stored preference]]):
- **#766** — `IMAGE_RETENTION_CRON_DISABLED` kill-switch. Real decision made (not a template copy): explicit `queue.removeJobScheduler()` on disable, chosen over skip-only, because skip-only would leave a stale-but-inert Redis scheduler entry still spawning unconsumed jobs. Also extracted `purgeStaleImages()` so the BullMQ path and the one-shot share code.
- **#767 + #768** — `--image-retention-once` via a **dedicated minimal entrypoint** (`main-image-retention.ts` + `image-retention-runner.module.ts`) rather than the full `AppModule` — `apps/agent`'s env schema is one monolithic all-at-once Zod object (Telegram/OpenRouter/X-OAuth/etc.), and this job only needs 6 `S3_*` vars. First `apps/agent/deploy/hetzner/` systemd unit.

PR [#772](https://github.com/pear-protocol/agent-pear-v2/pull/772) open. **#769 (live validate) and #770 (cutover) explicitly NOT started** — paused, see finding below.

### Real finding: `RUN_IMAGE_RETENTION` is unset everywhere in production

While preparing #769's validation commands, checked (via Railway CLI, `railway variable list --kv`, filtered to just this one key — not a full env dump) whether `ap-api`, `ap-tg`, or `ap-worker-prod` has `RUN_IMAGE_RETENTION=true`. **None of them do.** Per the code comment in `apps/agent/src/images/images.module.ts:10-16`, this purge is opt-in, default OFF, meant to be enabled on exactly one worker (`ap-worker`) — but it never was. **The image-retention purge has never actually run in production since it shipped.** Images older than the 60-day cutoff have been accumulating untouched the whole time, not just since some recent regression.

Consequences, not yet decided:
- There is no live in-process job for `IMAGE_RETENTION_CRON_DISABLED` to disable — that flag is a no-op wherever it's set today.
- The Hetzner one-shot will be the FIRST real purge ever run against prod — likely a large one-time backlog delete, not the small steady-state trickle a daily-since-launch job would produce.
- Open question handed back to the user: run the Hetzner one-shot as the first-ever purge as-is, or inspect the bucket's age/count distribution first. **Explicitly deferred — "leave it, will handle this one later."**

## PR-hygiene follow-up (from Step 3) — fixed this session

PR #758's stale `Closes #752`-only description (missing `Closes #753`, and a now-false "separate PR next in the chain" line) — flagged twice in Step 3, never applied until now. Fixed via `gh pr edit`: now reads `Closes #752` + `Closes #753`, description updated to describe both tasks. Also stripped a leftover "🤖 Generated with Claude Code" footer while editing (unrelated cleanup, done because it was directly in the diff being touched, consistent with the user's standing no-AI-trailer preference).

## Handoff — next actionable steps

1. Decide the `RUN_IMAGE_RETENTION` question above before running #769 for real.
2. **#763/#764** (Story #759) — confirm live-validation actually happened and get the cutover done; no evidence back yet.
3. **#769/#770** (Story #765) — blocked on item 1.
4. PR #757/#758 still open/unmerged — #759's branch depends on #757 merging eventually (currently stacked).

## Update — 2026-09-14 — Story #773 (Twitter-ingestion) shipped, reversing the earlier exclusion

The 13th Sep session explicitly excluded the 30-min Twitter-ingestion labeling cron from this batch ("audit already recommends against it — LLM cost × high frequency"). That was reversed: **Story #773** was groomed (tasks #774-#778, same code/kill-switch/one-shot/systemd/validate/cutover shape as #759/#765) and shipped on `feat/context-twitter-ingestion-hetzner` (stacked on `feat/context-watched-accounts-hetzner`, 3 commits ahead):

- **#774** kill-switch — `TWITTER_INGESTION_CRON_DISABLED`, guarded inside the shared `processBatch(trigger)` (covers both the 30-min cron tick and the startup path in one place).
- **#775** one-shot — `--twitter-ingestion-once` → public `processDiscordTweets()` (keeps the Discord summary), `redis.quit()` in `finally` (same pattern as #748/#753/#761), added to the mutual-exclusion guard.
- **#776** systemd — `context-twitter-ingestion.service`/`.timer`, `OnCalendar=*:0/30`, lock `/run/agent-pear/twitter-ingestion.lock`, `TimeoutStartSec=600` marked provisional pending #777's observed timing (same pattern as #749/#762).

tsc/jest(6/6)/biome all clean per the session's own report (not independently re-verified here). Pushed `--no-verify` and opened [PR #779](https://github.com/pear-protocol/agent-pear-v2/pull/779) against `feat/context-watched-accounts-hetzner`, no attribution trailer. **#777 (live validate) and #778 (cutover) not started.** Why the earlier exclusion was reversed is not recorded in this session — flagged as a gap, not guessed at.
