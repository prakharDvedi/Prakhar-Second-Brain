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

## Next steps

1. Trace the existing AP context worker and find whether a GH adapter already exists.
2. Inspect the VPS process and confirm whether it runs the AP context worker. The AP repo docs mention Railway for the context service, so the VPS ownership is not assumed.
3. Identify the campaign sources and fields to extract.
4. Add the automated campaign ingestion job using the existing worker pattern.
5. Expose the stored campaign context to the agent. Leave live leaderboard API reads and TP/SL chart annotations to their separate paths.
