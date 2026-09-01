# Telegram Local Startup

Goal: `@prakhar_pear_bot` running off my machine, talking to a local DB, so I can test the agent by DMing it.

### The mental model (this is what tripped me up)

There are **two separate Postgres** in play and they are not the same thing:

- **`pear-postgres`** (Docker, host port **5432**) holds the agent's own `agent` + `context` databases. This is what the bot needs.
- The **`backend` docker project** (the pear engine stack) runs a *different* Postgres (`pear-backend-v3`, host port **5433**). That's the engine DB, unrelated to the agent's schema. Don't confuse it for the agent DB.
- There is also a **native Windows service** `postgresql-x64-18` on 5432. It's set to **Disabled** and I leave it that way. The agent does NOT use it. The docker `pear-postgres` is what answers on 5432.

So `AGENT_DATABASE_URL=...@localhost:5432/agent` is correct — it hits the docker `pear-postgres`, not the native service.

### Prereqs

1. **Docker Desktop running.**
2. Start the pear containers: **`pear-postgres`** and **`pear-redis`**.
   - `docker start pear-postgres pear-redis`
   - Or the play buttons in Docker Desktop.

> **Gotcha — only run ONE postgres container.** `pear-postgres` and `pear-postgres-alt` share the same data volume. If both run, they fight over the data dir and each self-terminates ~60s later with:
> ```
> could not open file "postmaster.pid": No such file or directory
> performing immediate shutdown because data directory lock file is invalid
> ```
> Keep `pear-postgres-alt` stopped. Confirm `pear-postgres` survives past 60s before continuing.

### Start the bot

```bash
cd agent-pear-v2/apps/agent
pnpm dev:tg
```

Nest compiles for ~1–2 min. Success looks like:

```
Nest application successfully started
Telegram bot polling started
```

Then DM `@prakhar_pear_bot` (my chat id is the only one whitelisted via `TELEGRAM_DM_WHITELIST`). It replies = it works.

### Env notes

- `AGENT_DATABASE_URL` / `CONTEXT_DATABASE_URL` → `localhost:5432` = docker `pear-postgres`.
- `REDIS_URL`, quant/hl-engine DBs, OpenRouter → already point at remote dev, leave them.
- The bot is my own dev bot, so no 409 polling conflict with the deployed dev bot (different token).

### Troubleshooting

- **`Can't reach database server at localhost:5432`** → `pear-postgres` isn't running. `docker ps` to check. If it starts then dies ~60s later, see the one-postgres gotcha above.
- **Bot boots then exits** → almost always the DB. The Nest app loads all modules first, then dies on `AgentPrismaService`/`ContextPrismaService` connect. Check the DB before anything else.

---

## Optional: see entry / TP / SL chart annotations

The level-annotation feature (entry/TP/SL lines on position-aware charts) needs (a) the feature branch and (b) a local chart service that has the drawing code — the deployed prod chart service does NOT have it yet.

### 1. Agent branch

```bash
cd agent-pear-v2
git checkout feat/chart-level-annotations
```

### 2. Run chart-image-generator locally

```bash
# clone the chart repo, feature branch feat/level-annotations
git clone <chart-image-generator> && cd chart-image-generator
git checkout feat/level-annotations
npm install
# if npm's allow-scripts policy blocks postinstall, chromium won't download.
# install it explicitly:
npx puppeteer browsers install chrome
npm run dev     # builds + serves on http://127.0.0.1:3000
```

Needs no env: Discord is optional (skips itself if `DISCORD_BOT_TOKEN` unset), market/candle data comes from the prod gateway. The proprietary TradingView `charting_library` is committed in the repo, so no separate asset to fetch.

Smoke test the render (should return a PNG with the three lines):

```bash
curl -o test.png "http://127.0.0.1:3000/chart?long=BTC&longLabels=BTC&interval=1h&chart=asset&levels=entry:60000,tp:66000,sl:57000"
```

### 3. Point the agent at it

In `apps/agent/.env`:

```
CHART_IMAGE_GENERATOR_BASE_URL="http://127.0.0.1:3000"
```

(back up first — `cp .env .env.bak.chartlevels`). Restart `pnpm dev:tg`.

### 4. Actually seeing the lines

The lines only draw when the **whitelisted wallet holds an open position matching the charted asset** — levels are derived from my open positions, not invented. No matching position → chart still renders, just no lines. So ask the bot to chart an asset I'm currently in.

> Known limitation at time of writing (from the PR review): on single-asset charts the **TP/SL lines won't appear** even with a position, because the matching compares the base ticker against the venue instrument id. Entry line works. Pair charts draw the entry ratio only. So if you see entry but no TP/SL, that's the known bug, not your setup.

### Revert when done

```bash
cd apps/agent && cp .env.bak.chartlevels .env   # restore prod chart url
```
