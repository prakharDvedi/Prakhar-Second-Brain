# Leaderboard & Campaign Context Ingestion

Status: open — pre-existing AP work, separate from the Oracle revamp.
Filed: 2026-08-31.

## What this is

The team is working on making campaign context available to AP automatically. This is not part of the Oracle revamp.

The current split from the team discussion is:

- campaign information from Twitter and GH goes through the background/context worker and updates AP's context or KB layer
- live leaderboard rankings and other dynamic leaderboard details come from the backend API for charting
- AP trade ideas annotate entry, TP and SL. This was considered fine for now

## Backend findings

The backend already has a public `GET /statistics/leaderboard` endpoint. It calculates the leaderboard when requested from positions, fills, funding and X-account data. It is not a static KB document and does not need to be copied into the KB by a cron.

Relevant backend files:

- `apps/core/src/controllers/statistics.controller.ts`
- `apps/core/src/services/usecases/statistics/get-leaderboard.usecase.ts`
- `docs/leaderboard-design.md`
- `apps/crons/src/services/jobs/index.ts`

The backend cron service currently registers the OI job, not a leaderboard-ingestion job.

## Likely AP path

```text
Twitter + GH campaign updates
→ AP context/background worker
→ extract and normalize campaign facts
→ store fresh context with source and timestamp
→ agent and charting surfaces consume the right data
```

AP already has a Twitter cron and batch-processing pattern in:

- `apps/context/src/twitter/services/cron.service.ts`
- `apps/context/src/twitter/services/batch-processor.service.ts`
- `apps/context/src/app.module.ts`

One storage constraint: `packages/shared/src/kb/kb-index.ts` loads a pre-built local KB artifact. Live leaderboard data should remain API-driven. Campaign context may use the context/KB pipeline, but it needs source URLs, timestamps, deduplication and stale-data handling.

## Next steps (superseded — see Update below)

1. Trace the existing AP context worker and find whether a GH adapter already exists.
2. Inspect the VPS process and confirm whether it runs the AP context worker. The AP repo docs mention Railway for the context service, so the VPS ownership is not assumed.
3. Identify the campaign sources and fields to extract.
4. Add the automated campaign ingestion job using the existing worker pattern.
5. Expose the stored campaign context to the agent. Leave live leaderboard API reads and TP/SL chart annotations to their separate paths.

## Update — 2026-08-31

Findings above confirmed, plus:

- No GH adapter exists anywhere in `apps/context` (grep-clean) — GitHub ingestion is a from-scratch build, no existing pattern to extend.
- **VPS question resolved as "not in either repo."** Grepped both repos for `VPS`/`vps` — no real hits. `apps/context` has no Dockerfile, only Railway config. Only off-Railway service is `apps/tg-fallback` (Telegram failover, unrelated purpose). If a VPS genuinely runs something, it's untracked in-repo — flagged to the team rather than assumed.
- **Composer plug-in point found**, confirming no DAG node is needed: `ComposerService.buildExtraTools()` (`apps/agent/src/ai/dag/services/composer.service.ts:62-66`) is composer's on-demand toolset, independent of the planner/DAG graph — same place the existing `query_sm` tool is registered. Both the leaderboard tool and the campaign-info tool hook in here.
- **No chart↔leaderboard integration point exists** in `chart.service.ts` (only unrelated match was HIP-3 volume-ranked-listing resolution) — the planned chart-integration PR was dropped rather than fabricated.
- Ranking/scoring/PnL math on the backend confirmed **fully dynamic per-request** (positions + fills + funding + live Redis mark prices + X-linkage), no cache, no precompute table — `docs/leaderboard-design.md` §7.1/§7.7 explicitly accept "no rate limiting, no cache" as a known risk. AP-side TTL cache is therefore AP's own responsibility, not something backend provides.

### Scope decision (via GitHub grooming)

Groomed into **Epic #643** → **Story #644** (live leaderboard) / **Story #645** (campaign ingestion). Chart PR dropped from Story #644 per the finding above.

Three of the five original open questions got MVP defaults rather than stakeholder sign-off, explicitly flagged as such in the issues (not silently treated as final):
- Twitter/GH source lists → manual curated watch list (reusing the existing `/watch` mechanism), no auto-discovery
- Campaign ↔ asset linkage → optional `campaignId`, best-effort, not forced
- GH ingestion scope → releases only, no issues/PRs/changelog parsing

Retrieval trigger resolved technically (not just recommended): on-demand tool call, matching the `query_sm` precedent — `ContextAssemblerService.build()` only formats findings from nodes that already ran, so "auto-inject every turn" would need either a new DAG node (excluded) or a standing prompt instruction: deferred, not built into MVP.

### PR stack (all opened as empty scaffolds, stacked per-dependency — superseded, see Update below)

Story #644: PR #646 → #647 → #648
Story #645: PR #649 → #650 → #651 → #652 → #653

Umbrella `development → main` promotion PR planned once all 8 land. See [[31st Aug 2026]] for the session log.

## Update — 2026-08-31 (later same day) — PR stack corrected

Verified against live GitHub state. All 8 stacked scaffolds above were closed the same day — confirmed unnecessary/premature (see [[31st Aug 2026]] for the check and full breakdown). Actual current state:

- **Story #644 (leaderboard)**: reopened leaner as **#659 → #660** (2 PRs instead of 3 — the separate hardening/test PR was folded in). Both open.
- **Story #645 (campaign ingestion)**: closed, no replacement PR opened yet — this axis is back to unstarted as of this check.
- **#665** — `feat(charts): entry/TP/SL level annotations on position-aware charts` — opened same day, base `development`, open. Possibly related to the trade-idea annotation behavior noted above, not confirmed as the same initiative.

Status as of this check: leaderboard axis in progress under #659/#660; campaign axis not yet restarted.

## Update — 2026-09-01 — leaderboard axis: read stack + TG command landed on #659

Leaderboard axis (Story #644) progressed. All merged into `feat/leaderboard-read-tool` (PR **#659**, still open to `development`, in review):

- **read_leaderboard tool + cached read client + deterministic composer pre-fetch** — the agent answers live-standings questions by pre-fetching the board and injecting it as context (removes the model's discretion, which it dropped once history held a prior "unavailable" reply). Confirmed the composer plug-in point predicted on 2026-08-31 (`buildExtraTools`) was the actual seam used.
- **TG `/leaderboard` command** (PR **#675**, merged `ba61ddd4`) — current weekly Orchard board, per venue (Hyperliquid / Lighter), with an inline venue toggle. Works as slash command AND natural-language chat. Added to the DM wallet-gate allowlist so unlinked users can see standings (the board is public). *(Originally volume-ranked; default changed to ROI same day — see the Metric correction below.)*
- **Read-stack review fixes** (commit `35d28ffb`) — non-active venue returns an explicit "not in this week" state (not an empty board); custom date ranges include unlinked-X (platform stats); single-venue-week no-merge documented; `sortBy` default is scope-aware.

### Eligibility decision (senior call — resolves an MVP open question)

The FE runs **two** leaderboard reads with opposite eligibility:
- `use-weekly-leaderboard.ts` → `includeUnlinkedX: false` — linked-X only, the weekly campaign board.
- `use-trading-competition.ts` → `includeUnlinkedX: true` — everyone, volume-only draw.

The agent mirrors the **weekly** board (`currentWeeklyLeaderboard` hard-codes `includeUnlinkedX: false`). Senior (GPSxtreme) confirmed weekly params are fine and **no separate global/competition board is needed**. So the all-accounts competition board is deliberately NOT wired into the agent — out of scope by decision.

### Metric correction — default is ROI (huf, later same day)

Second senior (huf, business) clarified the **competition week is judged on ROI**, so the board leads with ROI, not volume. Refines the metric only — eligibility (weekly, linked-X) is unchanged.

- Flipped both defaults to ROI (commit `629e48a8`): `currentWeeklyLeaderboard` defaults `sortBy` to `roi` (NL composer pre-fetch inherits it); the `/leaderboard` command fetches ROI.
- TG card made metric-aware (was volume-hardcoded) — header "Ranked by <metric>", rows show the ranked metric (ROI %, "—" for unmeasured ROI). Backend already supports all sorts, so this was a default change, not new capability.

Status: leaderboard axis feature-complete on #659 (ROI default), awaiting review + merge to `development`. Campaign axis (Story #645) still unstarted.
