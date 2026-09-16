# Step 3 — Tokenomist + OI Canaries on Hetzner

- Status: In progress — Story #745 fully proven (code + systemd + live-tested), PR #757 open awaiting merge, only cut-over (#751) left. Story #746: kill-switches (#752) and one-shot flags (#753) both coded and pushed on PR #758 (open), but #753 is NOT yet live-validated and systemd units (#754) were reverted mid-write pending a reviewed plan.
- Date: 2026-09-12
- Project: Agent Pear harness integration
- Related: [[Harness Integration/Step 1 — AP Background Processes and Hetzner VPS Inventory]] · [[Harness Integration/Step 2 — Claude Basket Generation on Hetzner]] · [[My Work at Pear Protocol]]

## Objective

Epic #717 ("Migrate background jobs to the Claude harness on Hetzner") scoped phase 3 as the "canaries" — the non-LLM, deterministic jobs the Step 1 analysis flagged as lowest-risk (no Opus cost question, no research-benefit question, easily verified/rolled back). This session deliberately did the canaries **before** phase 2 (sentiment), reversing the epic's originally-stated order, specifically because Step 1's own cost analysis argued sentiment shouldn't move to the Claude/Opus harness at all (no web-research benefit, high volume). That tension is still unresolved — phase 2 is deferred, not decided.

## Scope split — two independent stories, not one

The five "canary" jobs from the `step1.pdf` audit turned out to span two architecturally different apps, so they were groomed as two separate GitHub stories under Epic #717 rather than one combined story:

- **Story #745** — Tokenomist refresh + OI refresh, both cron jobs living inside `apps/context` (an always-on app that also runs sentiment, Twitter, baskets, etc.). Same shape as the basket migration: peel one job's cron out of a busy app.
- **Story #746** — asset index (hourly) + coin index (6h) + Tokenomist index (daily 13:00), all living in `apps/indexing` — a **completely separate** always-on app that does nothing else (no HTTP listener — `main.ts` uses `NestFactory.createApplicationContext` with no `.listen()`). This is closer to "retire a whole Railway service" than "peel one job out." **Not started.**

## Story #745 — status

Branch: `feat/context-cron-hetzner-canaries`, based off `origin/development` — **not** off `feat/basket-hetzner-migration`. This was deliberate (independent stories should merge/rollback independently, not be stacked on an unmerged sibling branch), confirmed via `git merge-base --is-ancestor` returning false in both directions between the two branches.

| Task | Status |
|---|---|
| #747 kill-switches | Done — commit `3e756a75` |
| #748 one-shot flags | Done — commits `53b2a8f6` + `f5b3a2f8` (bugfix) |
| #750 validate | Done, informally, via live Hetzner runs (see below) |
| #749 systemd units | Done — commit `a28777b8`, installed + smoke-tested on the real Hetzner box (see below) |
| #751 cut-over + rollback | Not started — blocked only on deploying this branch to Railway `context` |

### #747 — kill-switches

`TOKENOMIST_CRON_DISABLED` / `OI_CRON_DISABLED`, env-gated, default `false`, mirroring `BASKET_CRON_DISABLED`. Guards the top of `TokenomistCronService.refresh()` and `OiCronService.runDaily()` — covers both the `@Cron` tick and the `RUN_ON_STARTUP` re-run path (same method). Unit tests added, `tsc --noEmit` clean.

Note: `BASKET_CRON_DISABLED` itself (PR #725, `feat/basket-cron-disable-flag`) is **still open/unmerged** as of this session — CI green, but not landed on `development`. Both kill-switch branches are independent siblings off `development`.

### #748 — one-shot flags, and a real bug found + fixed

Added `--tokenomist-once` / `--oi-once` to `apps/context/src/main.ts`: build an `AppModule` application context, call the same service method the in-process cron calls (`TokenomistCronService.refresh()` / `OiCronService.runDaily()`), `app.close()`. Reuses `AppModule` directly (unlike baskets, which needed a dedicated `BasketRunnerModule`) since Tokenomist/OI already live in the normal app.

**Bug found live on Hetzner:** both one-shots ran their real logic successfully and logged completion, but the process never returned to the shell — required `Ctrl+C` every time. Root cause: `app.close()` tears down NestJS-lifecycle providers, but the raw `ioredis` clients bound to the `REDIS`/`ASSET_INDEX_REDIS` tokens are plain factory providers (`new Redis(url)`, no `OnModuleDestroy`) — their open TCP sockets kept the Node process alive indefinitely. Fixed by explicitly calling `redis.quit()` / `assetIndexRedis.quit()` in a `finally` block after `app.close()` — the exact same pattern `runBasketsFull()` (on `feat/basket-hetzner-migration`) already uses, for the identical reason. Confirmed fixed: both one-shots now exit cleanly on their own.

### #750 — validation (done informally)

Live on Hetzner, in order:
1. `--tokenomist-once`: refreshed **136/136** assets in ~110s. Verified via a direct read-only Prisma query (`tokenomistData.findMany`, ordered by `updatedAt`) that rows moved from the `01:01` scheduled-Railway-run timestamp to the manual run's timestamp.
2. `--oi-once`: synced **188/188** assets (batches of 50) against Hyperliquid via Coinglass. Same write path as the existing 03:00 cron.
3. Mutual-exclusion check: `node dist/main --tokenomist-once --oi-once` throws immediately as intended.

No `--dry-run` mode exists for either job (unlike baskets) — both are idempotent upserts keyed on `assetId`, so the before/after `updatedAt` diff was the chosen validation method instead, matching the original task decomposition's own noted trade-off.

**Confirmed: no Claude/OpenRouter usage anywhere in either job.** `OPENROUTER_API_KEY` is only required because `AppModule` boots the whole app's env schema — neither job's code path touches it.

### Environment gotcha for whoever runs this next on Hetzner

`apps/context`'s env schema (`app.config.ts`) is one monolithic Zod object validated all-at-once (`envSchema.parse(process.env)`). Booting via `AppModule` (as both one-shots do) requires **every** field the whole app needs, not just what Tokenomist/OI use. `/home/prakh/pear/.env.basket-development` (built originally for the leaner basket runner) was missing 8 keys required by the full schema:

- **Actually used** by these jobs, need real values: `COINGLASS_API_KEY`, `TOKENOMIST_API_KEY`.
- **Not used** by these jobs at all, only required because the schema is all-or-nothing: `DISCORD_BOT_TOKEN`, `DISCORD_TWITTER_CHANNEL_ID`, `DISCORD_GUILD_ID`, `DISCORD_WEBHOOK_ERROR_URL`, `DISCORD_WEBHOOK_STATS_URL`, `TWITTER_API_KEY`. Confirmed safe to fill these with placeholder/dummy values by reading the actual call sites: `DiscordListenerService.onModuleInit()`'s `client.login()` is wrapped in try/catch that only logs on failure; `TwitterApiProvider` is a lazy axios factory with no eager call; `InternalLogger`'s Discord webhook posts are fire-and-forget with their own `.catch()`. All confirmed live — a bogus Discord token produced exactly one harmless logged error, nothing else.

**Open design question, deferred, not blocking:** should these one-shots use a dedicated lean module (mirroring `BasketRunnerModule`) instead of full `AppModule`, so they don't need Discord/Twitter credentials at all for jobs that never touch Discord/Twitter? Not resolved this session; #748 shipped using `AppModule` as-is.

### #749 — systemd units, installed + smoke-tested

`context-tokenomist.service/.timer` and `context-oi.service/.timer` (commit `a28777b8`) followed the `context-baskets.service/.timer` convention exactly — including its placeholder `agentpear` user and `/etc/agent-pear/context.env` path, which don't match this box. Installed by generating the real files at install time via `sed` substitution rather than editing the checked-in templates: `User`/`Group` → `prakh`, `WorkingDirectory` → `/home/prakh/pear/agent-pear-v2/apps/context`, `EnvironmentFile` → `/home/prakh/pear/.env.basket-development` (the file already confirmed working for `--tokenomist-once`/`--oi-once` earlier this session).

One more substitution the templates' own comments flagged as a risk and which turned out to be real: `ExecStart` used `/usr/bin/env node`, but this box's Node is an NVM install (`~/.nvm/versions/node/v24.20.0/bin/node`) — not on systemd's minimal `PATH` (systemd doesn't source `.bashrc`/`.profile`). Substituted to the absolute NVM path.

Validated via `sudo systemctl start context-tokenomist.service` / `context-oi.service` (without enabling either `.timer` — cut-over is still #751, not now) — **both succeeded**, same real refresh (136/136, 188/188) now running under systemd's process supervision instead of an interactive shell.

## Story #746 — status

Branch: `feat/indexing-hetzner-canaries`, off `origin/development`.

| Task | Status |
|---|---|
| #752 kill-switches | Done — PR [#758](https://github.com/pear-protocol/agent-pear-v2/pull/758) (commit `16a06190`), not yet merged |
| #753 one-shot flags | Done — commit `c9cb1647`, same branch/PR as #752. `tsc --noEmit` clean; **not yet live-validated on Hetzner** (unlike #748, no real run has confirmed it works) |
| #754 systemd units | Attempted, then reverted before committing — see "#754 attempt" below |
| #755–#756 | Not started |

### #752 — kill-switches

`ASSET_INDEX_CRON_DISABLED` / `COIN_INDEX_CRON_DISABLED` / `TOKENOMIST_INDEX_CRON_DISABLED`, each `z.stringbool().default(false)`, mirroring `TOKENOMIST_CRON_DISABLED`/`OI_CRON_DISABLED`. Confirmed real difference from Story #745's pattern while implementing: `apps/indexing`'s three services each have an eager `onModuleInit` that calls the SAME shared refresh method as their `@Cron` tick and as `ensureReady()` (the self-heal path `AssetIndexService.rebuildAll()` awaits on the other two before building) — so gating *inside* the shared method, as `apps/context` does, would have also silently broken `ensureReady()` for any future caller. Guarded each `@Cron` tick and `onModuleInit` call site individually instead, leaving `refreshIndex()`/`runRefresh()`/`ensureReady()` themselves ungated. `AssetIndexService` has no `ensureReady()` of its own (no other caller found via grep), so both its call sites were gated with no such caveat.

Verified: `pnpm --filter indexing exec tsc --noEmit` clean, `pnpm --filter indexing test` — 3 suites / 34 tests pass (new guard tests per service: no-op when flag true, unaffected when false, `ensureReady()` still does real work when its owning flag is true).

**Also live-validated on Hetzner** (`node apps/indexing/dist/main.js`, real prod Redis/QuantDB/APIs, no systemd/one-shot yet since #753 doesn't exist): all three flags `=true` → all three services logged their skip line and nothing else, correctly idle (initially looked like a hang — it wasn't; with no cron due and no HTTP server, silence after the skip lines IS the correct end state). Flags unset → unchanged real behavior: `AssetIndexService` hydrated 777 entries from the Redis cache, `CoinIndexService` loaded 2160 coins from cache, `TokenomistIndexService` did a live refresh (matched=136, missing=51, ambiguous=1) and rewrote the mapping with a fresh TTL — confirms the boot order is Coin/Tokenomist-index before Asset-index (matches `AssetIndexService.rebuildAll()`'s `ensureReady()` dependency), and that neither cache-hydrate path nor the live Tokenomist fetch were touched by the new guards.

One env gap hit and fixed while validating: `apps/indexing` needs `V3_GATEWAY_BASE_URL` and `COINGECKO_API_KEY`, neither of which `~/pear/.env.basket-development` had (that file was built for the `apps/context` basket runner, which doesn't use either). Added `V3_GATEWAY_BASE_URL=https://pro-gateway.pearprotocol.io` (public, no auth) and a real `COINGECKO_API_KEY` to that same env file.

### #753 — one-shot CLI entrypoints, code-complete but not yet live-tested

Added `--asset-index-once` / `--coin-index-once` / `--tokenomist-index-once` to `apps/indexing/src/main.ts` (commit `c9cb1647`, same branch/PR as #752). Confirmed genuinely new bootstrap code, not a variant of an existing pattern — this file had zero argv handling before. Each flag builds an `AppModule` context and calls the exact same method its `@Cron` tick calls (already gated by #752's kill-switches): `AssetIndexService.scheduledRebuild()`, `CoinIndexService.hourlyRefresh()`, `TokenomistIndexService.dailyRefresh()`.

Learned from #748's hang and applied proactively this time instead of discovering it live: `apps/indexing` also registers a raw `REDIS` factory provider (via `RedisProvider` in each index module) with no `OnModuleDestroy`, so `app.close()` alone wouldn't quit it. Added the `redis.quit()` call in a `finally` block from the start. `tsc --noEmit` clean.

**Not yet proven on the real box** — unlike every other piece of code in this migration so far (#747, #748, #749, #752 were each live-validated on Hetzner before being called done), #753 has only been typechecked locally. That live-validation step is #755, still ahead.

### #754 attempt — reverted, plan requested first

Started writing 5 systemd files (`indexing-asset.service`/`.timer`, `indexing-coin.service`/`.timer`, `indexing-tokenomist.service`) mirroring the `context-*` convention, but was asked to stop and revert before committing anything, and share a detailed plan for the rest of Story #746 first. All 5 files were deleted before ever being staged — nothing to undo in git, no commit exists.

The plan discussed (**not yet built, not approved as final** — just what was on the table when paused):
- Same `agentpear`/`/etc/agent-pear/...` placeholder convention as #749, substituted with real values (`prakh`, `/home/prakh/pear/agent-pear-v2/apps/indexing`) at actual install time — consistent with how #749 was installed.
- A **separate** `indexing.env` file, not shared with `apps/context`'s env file — `apps/indexing` has its own, different env schema (confirmed: needs `V3_GATEWAY_BASE_URL`, `COINGECKO_API_KEY`, doesn't need Discord/Twitter bot credentials at all, unlike `apps/context`).
- The `indexing-asset.timer` should be enabled **last** at cut-over, after `indexing-coin`/`indexing-tokenomist` are already live — `AssetIndexService.rebuildAll()`'s `ensureReady()` call depends on their data already being fresh, confirmed via the live #752 validation (Coin/Tokenomist-index boot before Asset-index).
- `TimeoutStartSec` would need a placeholder value (no real one-shot has ever run on Hetzner for these three) until #755 produces observed timing, same pattern as #749's baskets/tokenomist/OI units.

## A near-miss worth remembering

Mid-session, a long stretch of confusing "hang" debugging (Redis connectivity checks, auth theories, ioredis diagnostics) turned out to be chasing the wrong problem: the Hetzner box was accidentally back on `feat/basket-hetzner-migration` (via an earlier `git stash pop` done to recover unrelated WIP) while the commands being tested (`--tokenomist-once`) only exist on `feat/context-cron-hetzner-canaries`. That branch doesn't recognize the flag at all, so it was silently ignored and the process fell through to the **default boot path** — which starts the app as a live, long-running server. That's not a hang, that's correct behavior for a daemon that was never told to exit. See [[Engineering Lessons#Silence after switching branches on a remote box can mean the wrong code is running, not a hang]].

## PR hygiene gap — found and fixed (twice now)

Checked live GitHub state (not just this note) for both open PRs and found **PR #757's description closed #747/#748/#750 but was missing `Closes #749`**, even though the #749 systemd-unit commit (`a28777b8`) was on that same branch/PR — merging it would have left issue #749 open forever. Fixed via `gh pr edit --body`: #757 now reads `Closes #747 · Closes #748 · Closes #749 · Closes #750`. PR #758 was already correct (`Closes #752`) **at the time**.

**Update — 2026-09-13:** The same gap reappeared on PR #758 itself. After #753 (`c9cb1647`) landed on the same `feat/indexing-hetzner-canaries` branch, PR #758's body still says "Closes #752" only and explicitly describes #753 as "a separate PR next in the chain" — no longer true, since #753's commit is now on this PR too. **Not yet fixed** — found and recorded here, deliberately not edited via `gh pr edit` in the same pass this was noticed, since that's a GitHub-visible edit outside this note-taking task's scope. See [[Engineering Lessons#A PR's Closes list can go stale the moment a new commit lands on its branch]] — this is now a confirmed pattern (twice, two different PRs, same session), not a one-off.

## Update — 2026-09-13 (batch confirmed run+verified; next batch groomed)

Per user confirmation: Story #745 and Story #746's work (including #753's one-shot flags, previously noted above as "not yet live-validated") has now been run and verified on Hetzner. This supersedes the "not yet proven on the real box" line under #753 above — preserved as-written since it was accurate at the time; this line is the correction.

Next batch of canary-class jobs (same "no LLM, deterministic" audit category) groomed under Epic #717:
- **Story #759** — watched-account fetch (hourly, `apps/context`) — same architecture as #745 (`@Cron`, kill-switch + one-shot + systemd), tasks #760–#764.
- **Story #765** — image-retention purge (daily 03:00, `apps/agent`) — architecturally different from every prior story in this epic: BullMQ `upsertJobScheduler`-based, not `@Cron`, lives in the much larger `apps/agent` app (both `main-tg`/`main-api` entrypoints pull it in transitively), and `apps/agent` has no one-shot CLI precedent at all. Flagged as a real design task, not a copy-paste, in tasks #766–#770.
- Epic #717's body updated to list this as migration-order phase 4.

## Handoff — next actionable steps

1. **#751** (Story #745) — cut-over: requires PR #757 to merge to `development` and deploy to Railway `context` first (the kill-switch flags only work once Railway is running code that checks for them), then flip `TOKENOMIST_CRON_DISABLED=true`/`OI_CRON_DISABLED=true`, `systemctl enable --now` both timers on Hetzner, monitor a few cycles, keep the two-command rollback documented.
2. **PR #758's description needs updating** to add `Closes #753` and correct the summary/scope line before it merges — see the PR-hygiene section above. Small, mechanical, but real.
3. **#754** (Story #746) — systemd units for the 3 indexing jobs. A concrete plan exists (see "#754 attempt" above) but was explicitly paused for review before more code gets written — confirm the plan (or revise it) before resuming.
4. **#755** — live-validate #753's one-shot flags on Hetzner (never actually run yet) before or alongside installing #754's units, same order #745 followed (#748 validated live, then #749 wired systemd on top of proven code).
5. **#756** — cut-over, same shape as #751, blocked on PR #758 merging + Railway `indexing` deploying first.

## Update — 2026-09-16 (code review found the #748/#753 Redis fix was incomplete; root-caused and fixed)

A code review of PR #757 and PR #758 (reviewer: GPSxtreme) found that the `redis.quit()` fix from #748/#753 — recorded above as "Confirmed fixed: both one-shots now exit cleanly on their own" — did not actually make the one-shots exit cleanly under systemd. Real VPS logs from 12 Sep show both jobs logging completion, then hanging: OI logged "one-shot complete" at 13:44:57 UTC, systemd killed it at 14:13:28 (`result=timeout`); Tokenomist logged completion at 13:43:16 UTC and was killed the same way. The earlier "Confirmed fixed" claim was based on interactive/manual runs (#750) and a `systemctl start` check (#749) that didn't wait long enough to observe the subsequent hang — preserved as-written above since it was an honest read of what was checked at the time; this is the correction.

**Root cause (new, distinct from the #748/#753 lesson already recorded):** `TwitterModule`, `SentimentModule` and `StatsModule` (in `apps/context`) and `AssetIndexModule`, `CoinIndexModule`, `TokenomistIndexModule` (in `apps/indexing`) each declared their own `RedisProvider` directly inside their own module's `providers` array. In NestJS, importing the same *module class* into several places gives one shared singleton — but repeating the same *provider literal* across different modules' own `providers` arrays does not get deduplicated; each module silently got its own separate ioredis client. That meant 4 real clients in `apps/context` and 3 in `apps/indexing`, while `main.ts`'s existing `redis.quit()`/`assetIndexRedis.quit()` calls (correct for the two tokens the #748/#753 author knew about) only ever closed one of them. The review's repro used a real Nest container + module declarations + mocked I/O to demonstrate the exact instance count.

**Fix:** added a shared `RedisModule` (`packages/shared/src/providers/redis.module.ts`) and switched every affected feature module to `imports: [RedisModule]` instead of redeclaring `RedisProvider`/`RedisService` locally. `main.ts` needed no changes in either app — the existing single `quit()` call now closes the only client that exists. Added `redis-module-wiring.spec.ts` in both apps (inspects each module's Nest metadata for the shared import) as a regression guard, sanity-checked by temporarily reverting one module's import and confirming the test fails.

**Verified:** `tsc --noEmit` clean in both apps. Full existing suites pass — `apps/context` 11 suites / 57 tests (was 10/54 before the new spec), `apps/indexing` 4 suites / 37 tests (was 3/34). Pushed: `feat/context-cron-hetzner-canaries` @ `c73c9a4c`, `feat/indexing-hetzner-canaries` @ `e7348b29`. Summary posted as PR comments on both [#757](https://github.com/pear-protocol/agent-pear-v2/pull/757#issuecomment-5698515101) and [#758](https://github.com/pear-protocol/agent-pear-v2/pull/758#issuecomment-5698518226).

**Still open, not fixed this pass:** the same review found a second P1 in each PR, unrelated to the Redis fix:
- `apps/context`: the one-shot runners still boot the full `AppModule`, including `ScheduleModule.forRoot()` and the Discord listener — a delayed/manual/slow invocation spanning a cron boundary can trigger unrelated scheduled writers outside the one job's own flock.
- `apps/indexing`: an un-awaited, cache-preferring `refreshIndex(true)` fired at startup can be joined by the explicit `--coin-index-once` (`refreshIndex(false)`) call, so a "fresh rebuild" can silently return stale cached data.

PR #786 (Claude basket generation, [[Harness Integration/Step 2 — Claude Basket Generation on Hetzner]]) received the same review pass — 4 P2 correctness findings, a `tsc --noEmit` validation-claim correction, 3 needs-validation security leads, and a noted merge conflict with #757 in `apps/context/src/main.ts`. Not yet worked on; tracked in Step 2, not here.

**Handoff correction:** item 1 above (#751 cut-over gated only on PR #757 merging) understated the gate — merging either PR before this session's fix would have shipped a job that still can't exit cleanly under systemd. That specific risk is now closed for both PRs; the second P1 in each (above) is a separate, still-open gate before either should merge.
