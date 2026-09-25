# Step 1 — AP Harness Integration: Background Processes and Hetzner VPS Inventory

- Status: Open — preparation and inventory only
- Date started: 2026-09-01
- Project: Agent Pear harness integration
- Related: [[Agent Pear Revamp]] · [[1st Sep 2026]]

## Purpose

This is the handoff document for Step 1 of the harness integration. The immediate task is to understand every existing Agent Pear process, identify which background services could potentially move from their current deployment to the Hetzner `ap-tg-fallback` VPS, and prepare a proposal for senior review.

No service should be moved yet. Movement, deployment, infrastructure, and privileged-access changes require senior approval. The goal is to create room for the Oracle harness while preserving reliability and preventing duplicate execution.

## Working architecture

```text
Oracle harness
    ↓
AP pair-trading tools and existing deterministic DAG
    ↓
Pear V3 backend
    ↓
Hyperliquid
```

Proposed responsibility boundary:

- **Oracle harness:** outer agent loop, sessions, memory, skills, specialist lanes, model/profile routing, scheduling, and lifecycle.
- **Agent Pear:** users and tenant/account identity, pair-trading grammar, domain tools, deterministic DAG, confirmation and policy, and product surfaces.
- **Pear V3 backend:** tickets, execution state, venue interaction, and reconciliation.
- **Hyperliquid:** final venue truth for fills, positions, and account state.

The primary architectural risk is two competing agent loops. Oracle should orchestrate long-running agent behavior; AP should remain the owner of pair-trading semantics, safety policy, and deterministic execution boundaries unless the design is explicitly changed.

## AP process map to verify

| Area | Observed responsibility | Dependencies | Initial disposition |
|---|---|---|---|
| `ap-api` | HTTP REST/SSE API | Agent DB, Redis, queues | Keep request-facing until replaced safely |
| `ap-tg` | Telegram polling and delivery | Telegram, Redis leader state, queues | Move only with an explicit failover plan |
| `ap-worker` | BullMQ workers and execution/realtime processing | Redis, Agent DB, Pear V3 | High risk; do not duplicate or move casually |
| `context` | Headless context/data ingestion | Context/Quant DB and external data | Possible candidate after ownership is verified |
| `indexing` | Redis index builders | Redis and source data | Possible lower-risk candidate if one queue owner exists |
| `twitter` | Listeners, posting, and signal work | X, Redis, Agent API | External side effects require review |
| `mcp` | MCP HTTP service | Redis, Pear V3 | Keep request-facing unless deliberately split |
| `connect` | Wallet-linking flow | Browser/auth and Agent API | Keep near its current auth boundary |
| `tg-fallback` | Telegram failover service | Health endpoint, `tg:primary`, Telegram | Existing Hetzner workload; preserve reliability |

This is a starting hypothesis, not an approval list. Confirm every row from repository configuration, deployment configuration, logs, process manager, and queue ownership.

## What to record for every process

1. Start command and working directory.
2. Code location and deployment owner.
3. Supervisor: Docker Compose, systemd, PM2, Railway, or another mechanism.
4. Inputs, outputs, external side effects, and credentials required.
5. Databases, Redis keys/streams/queues, and other shared state.
6. Schedule, concurrency, model/API calls, and resource profile.
7. Retry, timeout, idempotency, and deduplication behavior.
8. Crash/restart behavior and safe-resume behavior.
9. Whether a second instance could duplicate messages, trades, posts, or jobs.
10. Network, firewall, health-check, logging, alerting, and rollback requirements.

Pay special attention to API/Telegram DAG work, scheduled tasks, `execute-ticket`, recall/journal/intent/signal flows, and Telegram delivery. Execution and realtime consumers need special treatment because duplication or partial migration can create financial or reconciliation problems.

## Hetzner `ap-tg-fallback`: confirmed facts

These are point-in-time observations from the working SSH session:

- SSH works to Ubuntu 24.04.4 LTS as Linux user `prakh` at `46.224.111.239`.
- The hostname shown at login is `ap-tg-fallback`.
- Docker/containerd tooling is present, and `apps/tg-fallback/dist/main.js` was visible during inspection.
- No custom application systemd unit was identified in the inspected service list; the real supervisor/deployment path still needs confirmation.
- The initial visible listening surface was limited to SSH on port 22.
- The snapshot showed low load, about 8% memory use, about 31% root-disk use, and no swap.
- `docker ps` was denied because `prakh` is not in the `docker` group; the socket was `root:docker` with mode `srw-rw----`.
- No process, container, firewall, queue, database, deployment, or credential state was changed.

This is not a capacity guarantee. Measure actual services and peak usage before placing anything on the fallback host.

## SSH and access notes

- The Windows machine generated an ED25519 key pair at `C:\Users\prakh\.ssh\id_ed25519` and `id_ed25519.pub`.
- The public key may be shared with the administrator. Never share the private key, passphrase, or a private-key file.
- The key comment `prakh@BetterHalf` is local metadata. It is not the Linux username and is not part of the SSH destination.
- Normal command shape: `ssh -i "$env:USERPROFILE\\.ssh\\id_ed25519" prakh@46.224.111.239`.
- SSH opens a remote shell; it is not a ping. `exit` returns to local PowerShell.
- Current access is enough for permitted read-only inspection, not Docker inspection or system-wide administration. Docker-group membership is highly privileged and should be granted only deliberately.

## First read-only study procedure

Run these on the VPS and save the output for the inventory. Do not restart, stop, modify, or install anything during this phase.

```bash
whoami
hostname
pwd
uptime
df -h
free -h
ps aux --sort=-%cpu | head -n 20
ps aux --sort=-%mem | head -n 20
systemctl --type=service --state=running --no-pager
ss -tulpn
```

Docker metadata and logs should be inspected only after the administrator grants appropriate access and confirms the scope. Do not request broad access merely for convenience.

## Migration decision framework

Potentially lower-risk candidates: headless ingestion, isolated Redis index builders, research-only workloads, and maintenance jobs with clear schedules and no financial side effects. Potentially higher-risk workloads: `ap-worker`, `execute-ticket`, Telegram polling/delivery, Twitter posting, and any shared Redis consumer.

For each candidate, answer:

| Question | Required answer |
|---|---|
| Can it run on the VPS? | CPU, memory, disk, network, runtime, and OS fit |
| Can it be isolated? | Separate process/container, environment, logs, and restart policy |
| Can it have one owner? | No simultaneous Railway and Hetzner consumer |
| Can it be rolled back? | Stop/drain/redeploy path and known-good version |
| Can it be observed? | Health check, logs, metrics, and alerts |
| Can it be proven safe? | No duplicate trades, posts, jobs, or state mutations |

Background work is not automatically harmless. Any worker touching queues, execution tickets, user state, social posting, wallet operations, or venue operations needs explicit review.

## Safeguards before any move

- One authoritative consumer for each queue or consumer group.
- No accidental Railway-plus-Hetzner duplicate.
- Explicit secret injection and credential ownership; no secrets in chat, source, shell history, or vault notes.
- A supervisor with restart behavior and a documented shutdown/drain procedure.
- Logs, health checks, alerts, and resource monitoring.
- A tested rollback path.
- Idempotency and locking verified for state-changing work.
- Capacity measured under realistic load.
- Telegram fallback reliability preserved.

## Separate coding-tool task

Coding-tool installation is separate from process migration. The spoken product name was ambiguous between Cloud Code and Claude Code. Before installation, confirm the exact product, official command, target version, `sudo` requirement, and credential storage. The senior will handle authentication. Do not put tokens or authentication material in this document.

## Step 1 deliverables

1. Inventory AP services, workers, queues, schedules, and supervisors.
2. Inventory the Hetzner VPS using permitted read-only commands.
3. Build a candidate movement matrix with dependencies and risk.
4. Call out duplicate-consumer, financial, auth, and reliability risks.
5. Propose target placement and ownership boundaries for senior review.
6. Obtain explicit approval before changing deployments or server access.
7. After the product is confirmed, install the coding tool and have the senior complete authentication.

## Open questions for senior review

- Which AP processes currently consume the most Railway resources?
- Which services are actually deployed on Railway, Hetzner, or elsewhere?
- What supervisor owns each process, and how are releases and rollbacks performed?
- Can Docker metadata/log access be granted without broad administration?
- Which queues have exactly one consumer, and what locking/deduplication protects them?
- Where should the Oracle loop end and AP’s deterministic execution loop begin?
- Is Oracle local-first only, or how will sessions/memory/scheduling be isolated per AP user?
- How will Oracle receipts map to Hyperliquid fills and positions?
- What licensing and hosted-product implications apply to Oracle?
- Which exact coding tool is being installed?

## Safety boundary

This document records preparation and read-only study only. No AP repository code, Hetzner deployment, Docker configuration, firewall, queue, database, credential, or authentication state has been modified as part of Step 1.

## Update — 2026-09-10 (user-reported, migration in progress)

Basket generation has since moved onto the Hetzner VPS as part of this migration effort (overall migration still in progress, not fully complete). User reports the new architecture's baskets score 1.14% better on average than the prior architecture's. This is stated directly by the user; no dashboard/eval export is linked in this vault yet — treat as user-reported pending a linked metrics source, consistent with the $63M Lighter-volume note in [[Resume Evidence]]. The deployment wiring, environment contract, current validation failure, and cutover plan are tracked in [[Harness Integration/Step 2 — Claude Basket Generation on Hetzner]].

## Analysis — 2026-09-11 (which migrated jobs take compute, and where Claude's research actually helps)

Working through the full migration inventory to separate two things that get conflated: **box compute** (Hetzner CPU/RAM/net — the data/ingestion/index jobs) vs **LLM API cost** (the real money — only the jobs that call a model). And, critically: **"uses an LLM" ≠ "benefits from web research."** Most LLM jobs here are *synthesis/classification over already-fetched data* — they want a good model's judgment, not live web search.

Grounded in the code read (sentiment prompts analyze PRE-FETCHED context; the category-sentiment synth prompt literally says "synthesizing existing analysis, not doing new research"; made-baskets research stage is the only path with WebSearch/WebFetch as its core):

| Job | LLM? | Web-research benefit? | Cost driver |
|---|---|---|---|
| **made baskets** (03:00) | **Opus ×3, web** | **YES — the only one** | LLM $ (Opus + web), ~$2.18/basket all-in |
| asset sentiment (12:00) | yes (cheap model) | **No** — synthesizes pre-fetched web/twitter/tokenomics | LLM $ × whole asset universe (volume) |
| category sentiment (Wed/Sun 12:30) | yes | **No** — aggregates existing asset sentiments | LLM $, low frequency |
| weekly sentiment refresh (Sun 12:00) | yes | **No** — same synthesis on stale assets | LLM $, weekly |
| Twitter context ingestion (30 min) | yes (cheap model) | **No** — classifies/extracts fetched tweets | LLM $ × high frequency |
| tokenomist / OI refresh | no | no | API + DB IO (light) |
| tokenomist / asset / coin index | no | no | Redis build (light, verifiable, rollback-able) |
| watched-account fetch (hourly) | no | no | Twitter API poll + queue |
| image retention (03:00, opt-in) | no | no | object-store deletes (maintenance) |

**Conclusion: web research genuinely benefits exactly ONE job — made baskets.** Establishing a *current* catalyst (unlock date, filing, listing, fresh narrative) needs live external facts. The other LLM jobs get their web/twitter/tokenomics data *before* the model runs, so they benefit from model *quality*, not research — and they're the **high-volume** ones (asset sentiment across the universe; Twitter every 30 min), exactly where Opus would be expensive overkill. They already run on cheaper models (sentiment prompt notes A/B tuning on `deepseek-v4-pro`) and should stay there. The no-LLM jobs (indexes, refreshes, fetch, retention) take only light box compute — plain cron/systemd, no model.

**Recommendation:** reserve Claude/Opus + web research for `made baskets`; keep sentiment + Twitter on cheap models; migrate the data/index/maintenance jobs as ordinary scheduled tasks. Don't let a "good Claude candidate" label pull Opus onto a synthesis job with no research upside.

### Sustainability of the made-baskets Opus cost (the open concern)

The user runs made-baskets via the Claude **$200 subscription plan** (Max) through the CLI, not API billing — so the constraint is the plan's rolling 5-hour usage windows, not a dollar invoice. Data point: on the OLD 3-call path, **10–11 baskets consumed ~18% of a 5h window** → ~58 baskets per full window. Measured all-in is **~$2.18/basket** (category $0.62 + research $1.45 + selection $0.11).

A full daily run is large: `GENERAL_TARGET_COUNT=18` × 3 platforms + equity + directional passes ≈ **60–75 baskets/day**. So:

- **Pre-optimization: a full daily run does NOT fit one 5h window** (~58 capacity vs 60–75 needed) — it would throttle/stall mid-run.
- **Post-memo (~28% cheaper): ~80 baskets/window**, so a full run *just* fits — but with near-zero headroom and no room for retries or any other Claude usage.
- The user **also does interactive Claude Code work on the same plan** (competes for the same window). So in practice, running the full daily basket cron on the same $200 plan the user codes on is **not comfortably sustainable** — it's fragile even after the memo.

**The real lever is basket COUNT, not per-basket cost.** The research call ($1.45) is the irreducible web-research core; you can't cheapen it without losing the research. Sustainable options, in order of preference: (1) **lower the daily basket target** (18/platform × 3 is a lot); (2) move basket gen to **API pay-as-you-go** (removes the window cap, but ~$95–130/day ≈ $3–4k/mo — only worth it if baskets drive that much value); (3) a **dedicated Claude plan/account** for the cron, separate from the user's coding plan. The prompt trims + memo were the right first step but do not, by themselves, make a full-scale daily Opus run sustainable on a shared $200 plan. *(Sustainability figures are estimates from one usage data point + the measured $2.18 — directional, not exact.)*

## Analysis — 2026-09-15 (apps/agent ambient-agent jobs: difficulty ranking + Claude-harness benefit)

Extends this note's own migration-decision framework and its 2026-09-11 LLM/web-research analysis to a second cluster of 12 background jobs the user surfaced for the "what's next after the canaries" question, living in `apps/agent` (not `apps/context`/`apps/indexing`, the app already covered by Steps 3–4): `interest-distiller`, `user-valuescore`, `touch-priors`, `x-digest`, `ghost-writer`, `watch-scheduler`, `ghost-synthesis`, `global-feedpull`, `touch-brief`, `signal-funnelrollup`, `ghost-report`, `ghost-shadow`, plus `image-retention` (already covered — see [[Harness Integration/Step 4 — Watched-Accounts + Image-Retention Batch on Hetzner|Step 4]], not re-ranked here).

**Scope note, confirmed via `gh issue view`:** these 12 jobs already have a home — Epic #438 "Ambient Agent — Algorithmic User Retention," authored by a teammate (`Yadheedhya06`), not the user. Per the user's explicit instruction, issues not opened by them were treated as irrelevant to this analysis — the ranking below is derived from Epic #717's (the user's own) risk framework and direct code reading, not from #438's issue tree. Separately, and worth flagging as a real tension: Epic #717 (also the user's own) carries an explicit "out of scope" line — *"User-facing / event-driven jobs (Telegram delivery, touch/\*, X posting, signal path) stay on Railway per the audit"* — which pre-excludes 8 of these 12 (`touch-*`, `x-digest`, `ghost-*`, `watch-scheduler`) by name. **Not resolved here — a scope decision (widen #717, or open a separate epic) is needed before any of these becomes a Story, and is the recommended next actionable step.**

### Difficulty ranking (easiest → hardest to migrate), applying Step 1's risk axes

| # | Job | Schedule | Why |
|---|---|---|---|
| 1 | `ghost-report` | monthly | Pure reporting (`buildGhostReport`), no LLM, no downstream trigger, lowest frequency of the set. |
| 2 | `signal-funnelrollup` | weekly (Mon) | Same shape as the proven Tokenomist/OI canaries: deterministic aggregation (`aggregateSignalFunnel`), no LLM. |
| 3 | `user-valuescore` | daily 04:30 | Pure arithmetic (`computeUserValueScores`), no LLM, single-table upsert, no external API. |
| 4 | `touch-priors` | daily 04:50 | Same internal-state shape as #3 (pure Bayesian/CTR stats, no LLM), one hop closer to the notification pipeline since `touch-brief`/`touch-governor` read it. |
| 5 | `interest-distiller` | daily 04:00 | Calls an LLM (Cerebras-direct `gpt-oss-120b`) over already-stored user memory/persona/journal — writes back into the agent's own reasoning input for that user, so a bad write silently degrades every later interaction, not just one score. |
| 6 | `watch-scheduler` | daily 05:30 | LLM classification (dated-vs-open horizon) of one stored ghost statement per entry; "user-originated workflow" — a duplicate-writer bug is user-visible (a watch mis-firing), unlike #3/#4. |
| 7 | `ghost-writer` | daily 05:15 | No LLM (delegates to `TriggerSynthesizerService`, pure DB scoring/ranking) — but seeds the trigger set two steps upstream of a live signal fire; two writers racing here diverges the trigger set, not just a stale row. |
| 8 | `ghost-synthesis` | daily 05:45 | LLM merge/author calls (Cerebras-direct) writing to **shared** signal context across users, not a per-user row — corrupts one shared artifact everyone downstream reads. |
| 9 | `x-digest` | daily 05:00 | LLM classification, but the only job in the set that fetches live external data (via `twitterapi.io`, a paid Twitter API client) immediately before the LLM call — same "personalized + external API budget × volume" profile this note's 2026-09-11 analysis already flagged for sentiment/Twitter-context jobs as unsuited to the Opus/Claude harness. |
| 10 | `global-feedpull` | 2x daily | No LLM (fetches Elfa trending/news data, writes raw rows) — but external API budget **and** directly user-facing (the global feed users see); a duplicate consumer here burns shared quota and shows stale/duplicate content to everyone, not one user. |
| 11 | `touch-brief` | daily 06:40 | LLM prose synthesis over headlines `global-feedpull` already fetched — sits directly on Epic #717's own drawn boundary (last internal step before an actual outbound notification). |
| 12 | `ghost-shadow` | every 15 min | No LLM (deterministic condition evaluator, `SignalMeasurementService`) — hardest anyway: ~96 runs/day, an order of magnitude above every other job here, and it *is* the live trigger-evaluation/signal-fire loop — Step 1's `ap-worker`/`execute-ticket` "do not duplicate or move casually" bucket by definition. |

Argument in one line: risk tracks hops-to-a-user-visible-action, multiplied by run frequency (duplicate-consumer blast radius) and by external-API/shared-artifact exposure — not by "uses an LLM," which turned out to be a weak signal here (7 of 12 have no LLM call at all).

### Claude-harness-vs-current-provider benefit — code-grounded, none found

Checked whether any of these 12 would benefit from Claude-harness dependency (WebSearch/WebFetch/MCP tool orchestration), the same test this note's 2026-09-11 section applied to `apps/context` jobs (only made-baskets passed it there).

**Premise correction, confirmed by code:** none of these 12 jobs use OpenRouter for generation. All 5 that call an LLM (`interest-distiller`, `x-digest`, `watch-scheduler`, `ghost-synthesis`, `touch-brief`) route through `resolveNodeModel()` to **Cerebras-direct** `gpt-oss-120b`, per a deliberate comment in `apps/agent/src/config/models.ts` ("Cerebras-direct ON PURPOSE... don't switch providers without re-running the background-eval"). OpenRouter appears exactly once in this cluster, for `ghost-synthesis`'s embedding step (`OpenRouterEmbedder`) — not a generation call.

Applying the research-benefit test to the 5 LLM-calling jobs: all 5 are classification/synthesis over data already fetched earlier in the pipeline (stored memory, stored ghost statements, headlines `global-feedpull` already pulled). `x-digest` is the closest exception — it fetches live data right before its LLM call — but that data comes from a paid Twitter API client, which WebSearch/WebFetch tools don't substitute for.

**Conclusion: no job in this cluster would benefit from Claude-harness dependency.** The harness's differentiator (live web research) has no work to do on any of the 12 — same structural finding as the 2026-09-11 sentiment analysis, now confirmed for a second, larger job cluster. The only adjacent, separate argument is model-quality (not harness/tooling) for `ghost-synthesis`'s merge judgment or `interest-distiller`'s persona extraction — a Cerebras→Claude-API model swap question, independent of Hetzner migration, and one that would need its own volume/sustainability math (per-user daily loops, likely worse than made-baskets' already-tight 5h-window budget) before being worth pursuing.

**Follow-up — 2026-09-16:** the deferred model-quality question above was picked up for `interest-distiller` specifically, prompted by a real code-confirmed quality gap (`isStyleTopic()`'s deterministic backstop against small-model style-topic drift). Delivered as a research plan (CLI invocation design, structured-output approach, a real-measurement methodology, offline eval design, risk list) — not yet executed, no decision made. See [[Harness Integration/Step 7 — Claude CLI Feasibility Research for interest-distiller]].

### Implementation-effort spot check — the 6 lowest-risk jobs (#1–4, #7, #10 above)

Separate from the risk ranking: checked whether these 6 are actually as thin to carve into a dedicated Hetzner entrypoint as image-retention was (its own constructor plus one hop into whatever service it injects — deliberately shallow, not a full transitive trace).

| Job | Direct deps | One hop deeper | Verdict |
|---|---|---|---|
| `user-value-score` | `AgentPrismaService` + queue | — | Thinnest — same shape as image-retention. |
| `ghost-report` | `AgentPrismaService` + queue | — | Thinnest — same as above. |
| `touch-priors` | `AgentPrismaService` + queue + one `ConfigService` slice | — | Thin. |
| `signal-funnelrollup` | DB + queue + Redis + `ElfaCreditBudgeterService` | that service needs Redis + `ElfaClient` + config | Moderate — Redis (known pattern from Step 3, needs the `redis.quit()` fix) + an Elfa API credential to carve into the env schema. |
| `ghost-writer` | DB + queue + `TriggerSynthesizerService` | that service needs `SignalAdaptivePolicyService`, `WatchSchedulerService`, `SignalUserGateService`, config | **Deceptive.** Looks thin from its own processor file, but the one service it calls pulls in three more domain services — not a copy-paste of image-retention's shape. |
| `global-feedpull` | DB + queue + Redis + config + `ElfaDataService` + `ElfaCreditBudgeterService` | `ElfaDataService` needs Redis + `AssetIndexReader` — the same Redis-backed asset index `apps/indexing` builds (Story #746) | **Deepest of the six.** Two Elfa services plus a cross-app dependency: this one-shot needs `apps/indexing`'s asset-index Redis cache already warm, not just its own env vars. |

**Revised build order for the easiest batch: `user-value-score` / `ghost-report` first (genuinely as thin as image-retention), then `touch-priors`, then `signal-funnelrollup`, then `ghost-writer`, then `global-feedpull` last** — the last two looked cheap by job description but carry real service subtrees once traced one hop.

### Handoff — next actionable step

Get the scope question answered (widen Epic #717 vs. new epic) before opening any Story for these 12 jobs — per the tension noted above, 8 of the 12 fall inside #717's own "stays on Railway" exclusion as currently written. Once that's settled, `user-value-score` and `ghost-report` are the cleanest starting points per the effort check above. Full per-job execution plan (kill-switch/one-shot/systemd/validate/cutover, plus cross-job sequencing dependencies like `global-feedpull` needing Story #746 cut over first): [[Harness Integration/Step 5 — Ambient-Agent Job Migration Plan (apps agent)]].

## Full-epic status snapshot — 2026-09-15 (every job, old and new, in one place)

Steps 2–5 each track their own slice of Epic #717; this is the first single view across all of it, built by checking live GitHub state (`gh pr view`) for every PR rather than trusting any one Step note's last-known status. **Headline finding: nothing in this entire epic has merged to `development` yet** — 11 jobs across 8 PRs are code-complete and live-validated on Hetzner, and every single one is still sitting in an open (or stacked-but-unmerged) PR.

| Job | App | What it does | PR | Real status (verified 2026-09-15) |
|---|---|---|---|---|
| `made-baskets` | context | AI-researched pair-trading baskets — web research + sentiment + stats | [#786](https://github.com/pear-protocol/agent-pear-v2/pull/786) | Open → `development`. Also depends on [#725](https://github.com/pear-protocol/agent-pear-v2/pull/725) (`BASKET_CRON_DISABLED` kill-switch) merging first — also still open. |
| Tokenomist refresh | context | Refreshes tokenomics data for every tracked asset | [#757](https://github.com/pear-protocol/agent-pear-v2/pull/757) | Open → `development`. **Blocks #771 and #779 below — highest-leverage merge in the whole epic.** |
| OI refresh | context | Refreshes open-interest data from Hyperliquid/Coinglass | same [#757](https://github.com/pear-protocol/agent-pear-v2/pull/757) | Same PR as above. |
| asset/coin/Tokenomist index | indexing | Rebuilds the Redis-backed lookup indexes several other jobs read | [#758](https://github.com/pear-protocol/agent-pear-v2/pull/758) | Open → `development`. |
| watched-account fetch (Story #759) | context | Hourly poll of X accounts the system watches | [#771](https://github.com/pear-protocol/agent-pear-v2/pull/771) | **Merged 2026-09-14, but only into `feat/context-cron-hetzner-canaries`** — that branch IS PR #757, still unmerged into `development`. Not actually landed despite showing MERGED. |
| Twitter-ingestion (Story #773) | context | Labels/classifies incoming tweets for context ingestion | [#779](https://github.com/pear-protocol/agent-pear-v2/pull/779) | **Merged 2026-09-14, but only into `feat/context-watched-accounts-hetzner`** — stacked two levels on `feat/context-cron-hetzner-canaries` (#757). Same situation, one level further down the chain. |
| `image-retention` (Story #765) | agent | Purges object-storage images older than 60 days | [#772](https://github.com/pear-protocol/agent-pear-v2/pull/772) | Open → `development`. |
| `ghost-report` | agent | Monthly ghost-analytics report | [#787](https://github.com/pear-protocol/agent-pear-v2/pull/787) | Open → `development`. |
| `user-valuescore` | agent | Daily per-user value/segmentation score | [#788](https://github.com/pear-protocol/agent-pear-v2/pull/788) | Open → `development`. |
| `signal-funnelrollup` | agent | Weekly signal/engagement analytics rollup | [#789](https://github.com/pear-protocol/agent-pear-v2/pull/789) | Open → `development`. |
| `interest-distiller` | agent | Nightly per-user memory/journal → interest-tag distillation | [#791](https://github.com/pear-protocol/agent-pear-v2/pull/791) | Open → `development`. |

**Dependency chain, not just a flat list:** `#757 → #771 → #779` is a real three-deep stack on the context side. Merging #757 to `development` doesn't just land Tokenomist/OI — it's the single thing standing between all three of those PRs' work (5 jobs total: Tokenomist, OI, watched-account fetch, Twitter-ingestion) actually reaching `development` at once. Nothing else in the epic has this kind of chain — every `apps/agent` job (image-retention through interest-distiller) merged independently off `development` with no stacking.

**The 8 `apps/agent` jobs still held back**, unchanged from the 2026-09-15 risk-ranking analysis above: `touch-priors` (ambiguous, not confirmed-excluded), `watch-scheduler`, `ghost-writer`, `ghost-synthesis`, `x-digest`, `global-feedpull`, `touch-brief`, `ghost-shadow` (all named or implied in Epic #717's "touch/\*, X posting, signal path" exclusion). See the difficulty ranking table earlier in this note for the per-job reasoning.

## Correction/Update — 2026-09-22 (all PRs above actually merged since; one PR closed unmerged — verified live via `gh`)

Re-checked live PR state for every row in the "Full-epic status snapshot" table above, prompted by a user question cross-referencing an external `step1.pdf` inventory (a 12-job table: Tokenomist/OI refresh, made baskets, asset/category/weekly sentiment, Tokenomist/asset/coin index, Twitter context ingestion, watched-account fetch, image retention — a **different 12** from Step 5's 12 `apps/agent` ambient-agent jobs, though `image-retention` appears in both this note's tracking and that table).

**9 of the 12 in that table are now code-merged to `development` AND `main`** (release PR [#813](https://github.com/pear-protocol/agent-pear-v2/pull/813), merged 2026-09-21): `#757` (Tokenomist+OI), `#758` (asset/coin/Tokenomist index), `#771` (watched-account fetch), `#779` (Twitter ingestion), `#772` (image-retention), `#786`+`#725` (made-baskets+kill-switch). The 09-15 snapshot's "nothing has merged yet" finding is now stale — corrected here, original prose above left untouched per vault rules.

**Correction — 2026-09-22, later same day (the "not cut over" claim directly above was WRONG, corrected via live SSH check, not another document):** verified directly on the Hetzner box (`ssh prakh@46.224.111.239`, `systemctl is-enabled`/`is-active`/`show -p Result` on all 12 timers) — **every one of the 12 timers is `enabled`, `active`, and last-result `success`**, including all 9 from this table plus `agent-ghost-report`/`agent-signal-funnel-rollup`/`agent-user-value-score`. Cutover has actually happened since #813 merged (2026-09-21) or since `HertznerBg.pdf`'s 2026-09-19 snapshot (which showed only `context-baskets.timer` enabled) — #804's "no cutover" language described the CI *deploy* workflow's own scope (pull/build/report only), not the state of the kill-switches themselves, and PR-body/doc snapshots were wrongly treated as current fact instead of being live-checked. Lesson: for this box specifically, `systemctl` state on the live host is the only source of truth for cutover status — PRs and handoff docs describe intent/history, not current state, and go stale within days.

**3 of the 12 (asset sentiment, category sentiment, weekly sentiment refresh) were never migrated — by design, not blockage.** No PR exists for them; this note's own 2026-09-11 analysis recommended they stay put (cheap model, no Claude-harness research benefit, ~$0.011/day combined per Step 6). Expected, not an oversight.

**Real, previously-undocumented finding: `interest-distiller`'s PR [#791](https://github.com/pear-protocol/agent-pear-v2/pull/791) is CLOSED, not merged** (`gh pr view 791`: `state: CLOSED, mergedAt: null, closedAt: 2026-09-20T11:04:07Z`, no comments, no successor PR, branch `feat/agent-interest-distiller-hetzner`). The Full-epic snapshot above and [[Harness Integration/Step 5 — Ambient-Agent Job Migration Plan (apps agent)|Step 5]]'s job-5 entry both still say "Open → development" — both now stale on this point. Practical effect: the Hetzner-cutover prerequisite that [[Harness Integration/Step 7 — Claude CLI Feasibility Research for interest-distiller|Step 7's]] Claude-CLI-swap plan is gated on isn't just unmerged, its own migration PR is dead with no replacement in flight — the gap is larger than "pending," it's "not currently being worked."

The other 3 `apps/agent` jobs from Step 5's cluster that DID merge via #813: `#787` (ghost-report), `#788` (user-valuescore), `#789` (signal-funnelrollup) — same "code-merged, not cut over" status as the 9 above. The 8 held-back jobs list is unchanged (still blocked on the Epic #717 scope question, no PRs opened).
