# Agent Pear — Learning Map

Written 2026-10-11 for Prakhar. Derived from the repo and the engineering vault, not from a generic syllabus.

## 0. How to read this, and what it rests on

**Sources actually read.** Repo: `ARCHITECTURE.md`, `CLAUDE.md`, `packages/execution/README.md` (protocol, staleness, venue-verdict sections), `apps/agent/src/config/models.ts`, `packages/shared/knowledge/quant/{stat-diagnostics,risk-and-sizing}.txt`, and directory layouts of `apps/agent`. Vault: AI Vault Rules, Engineering Lessons (all of it), Current Focus, Knowledge Gaps, Features Owned, Agent Pear Revamp, the Incidents index and the knowledge notes on trading safety, model routing and delivery ordering.

**Not read.** The prompts themselves, `dag.service.ts` and the specialist agents line by line, the `twitter` app, the 40+ daily logs, individual incident notes beyond their index lines, Langfuse data, and the external `pear-stat-arbs` / `pear-stat-rs` / V3 engine code (not in this repo). Claims about those are marked *[unverified]*.

**Evidence tiers** (per your vault rules): `[code]` = read in the repo this session; `[vault]` = your own notes, treated as prior memory; `[infer]` = my inference, not checked; `[rec]` = recommendation.

The vault's *Agent Pear Product & Engineering Gap Analysis* (2026-09-20) is an AI-generated synthesis, so it sits at the bottom of your source hierarchy. I used it for pointers (the `7e0824d9` planner failure, the stringly-typed findings) and re-derived the architecture from `ARCHITECTURE.md` instead. Where I agree with it, I say why.

---

## 1. System map

### 1.1 End to end

```
 SURFACES   Telegram DM/group · Web (SSE) · Pear Pro / Base mini · MCP hosts · X bot · CLI
               │                                                     │
               ▼                                                     ▼
 INGRESS    grammY bot (ap-tg)          REST/SSE (ap-api)      MCP server (apps/mcp)
               │  enqueue                    │  enqueue              │ OAuth, scope gates
               ▼                             ▼                       │
 QUEUES     Redis/BullMQ: agent-dag-tg · agent-dag-api · scheduled-tasks · tg-delivery
               │  (ap-worker drains; results stream back on Redis pub/sub job:{id})
               ▼
 CONTEXT    history (Redis cache + Postgres, ≤40 msgs) · recall notes · journal · prefs
            · runtimeContext (Basket Builder draft) · KB (local embeddings) · hot-facts
               │
               ▼
 ┌─ PLAN ─────────────────────────────────────────────────────────────────────┐
 │ planner LLM (gpt-oss-120b on Cerebras; fallback Claude Haiku via OpenRouter)│  ← INTELLIGENCE
 │ → PlannerDecision: sub-query per node + dependency edges, or clarify, or   │
 │   generalChat. Repair/normalise in code. Total failure → generalChat plan. │  ← DETERMINISTIC
 └────────────────────────────────────────────────────────────────────────────┘
               ▼
 ┌─ EXECUTE (DagOrchestrator) ────────────────────────────────────────────────┐
 │ gates (wallet auth, surface allowedIn, cron) → toposort → layers →          │  ← DETERMINISTIC
 │ Promise.allSettled per layer. Nodes: pair · basket · research · twitter ·   │
 │ tokenomist · coinglass · marketScanner · assetSentiment · tradeExecution ·  │
 │ scheduler. Each node = own model + tool loop. Output = mostly prose.       │  ← INTELLIGENCE
 └────────────────────────────────────────────────────────────────────────────┘
               ▼
 COMPOSE    composer LLM (two-phase: prose + strict extract) · trade links from
            structured tradePlan · charts · PnL cards · internal-leak guard (regex)
               ▼
 ACTION     tradeExecution tools → plan* stages a TICKET → user taps Confirm →
            execute* redeems single-use plan token → V3 engine → Hyperliquid / Lighter
            → execution polling → venue verdict (filled/failed/running/absent/unreported)
               ▼
 DELIVERY   SSE tokens · Telegram (rate-limited tg-delivery queue) · MCP widgets
 ASYNC IN   V3 engine realtime:private (Redis) → hydrate over REST → TG alert card
 BACKGROUND recall/journal/intent agents · scheduler · Elfa signal triggers · ghost triggers
            · watch scheduler · X digest · interest-distiller · user-value score · basket cron
            (≈27 jobs, most moved Railway → Hetzner systemd timers in Sep 2026)
 DATA       3 Postgres (agent / context / quant) · Redis (queues, cache, indices, locks, pub/sub)
 EXTERNAL   OpenRouter · Cerebras · pear-stat-arbs (signals, z-scores) · V3 engine · HL /info
            · CoinGecko · Coinglass · Tokenomist · Tavily/Exa · Polymarket/Kalshi · Elfa · X APIs
 OBS        Langfuse (OTel) · Discord webhooks · per-node timing · tool traces
 DEPLOY     Railway (ap-api, ap-tg, ap-worker, context, twitter, indexing, mcp, connect)
            · Hetzner (tg-fallback + new systemd one-shots) · tg:primary Redis lock for failover
```
`[code]` for the topology, node list, queues, single-pass rule and model assignments. Model names change fast; `models.ts` is the source of truth.

### 1.2 Where the intelligence lives vs. the deterministic logic

| Layer | Intelligence (LLM, stochastic) | Deterministic |
|---|---|---|
| Routing | planner chooses nodes + sub-queries | node registry, surface/auth gates, dependency repair, toposort |
| Analysis | each node's tool loop and prose | quant numbers (computed upstream), asset index, price caches |
| Money path | choosing *which* plan to stage, filling fields | `packages/execution`: plan→token→execute, margin gate, drift gate, price-age gate, venue verdict |
| Reply | composer prose | leak guard, trade links from structured plan, chart generation |
| Memory | recall/journal summarisation | `UserTradeProfile`, atomic callback consume, Redis history |

The strongest part of the design is the boundary: the LLM proposes, deterministic code validates and confirms. The weakest part is the middle: nodes hand each other prose.

### 1.3 Where failures are most likely (ranked by what your incident record shows)

1. **LLM ↔ code boundaries.** Planner output that doesn't match the schema (`7e0824d9` `[vault]`), an all-null decision reaching nobody (Kalshi `[vault]`), a tool-validation error that doesn't name the field (PR #1017 `[vault]`), a safety tag dropped by a summarising node (#794 `[vault]`).
2. **Persisted state that exists only in prompt context.** `skipRisk` forgotten once the 3-turn window and a stale recall note aged out `[vault]`.
3. **Two paths that should agree but don't.** Streaming vs sync context (TP/SL 100% default), MCP vs agent validation of TRIGGER/LADDER, `validate_assets` vs `quant_basket_stats` on venue `[vault]`.
4. **Provider behaviour.** Rate limits vs deprecation, HTTP-200 error bodies, Vertex→AI Studio reroute ("corrupted thought signature"), streaming errors that never throw, response cache replaying a bad answer `[vault]`.
5. **Stale caches that look fresh.** Price/instrument feeds; instrument age is exposed but not gated (`packages/execution/README.md` "Available, not wired") `[code]`.
6. **Non-atomic multi-leg execution.** Partial fills leave a user unhedged; rollback/compensation is explicitly undocumented `[vault]`.
7. **Operational glue.** Env var shared by two services, provider redeclared per module (4 Redis clients), full-AppModule one-shots firing unrelated crons, Prisma 5 s transaction default over a slower network `[vault]`.
8. **Async delivery.** Telegram send ordering (open, no root cause), unguarded timer callbacks `[vault]`.

---

## 2. Fundamental problems vs. implementation accidents

Be willing to disagree with the architecture. My read:

**Fundamental (they stay, you design around them)**
- A language model sits between user intent and a deterministic API. It will sometimes emit a degenerate plan. You can't prompt that away; you need a validated representation and a recovery policy.
- Two-leg pair trades are not atomic on a perp venue. Leg risk is a property of the product.
- Model providers are unreliable and non-uniform. Routing, fallback and eval comparability are permanent concerns.
- "Done" ≠ "filled." The venue verdict design is the right answer and is a good model for other places.

**Accidents (fixable, and where most of your leverage is)**
- **Prose as the inter-node format.** `TextResult { text }` + labelled strings means the composer re-parses what a node already knew structurally (BNB–AVAX orientation lost this way `[vault]`). This is the single biggest source of recurring bugs.
- **A planner that is a compiler with no IR.** Routing policy, dependency generation, sub-query writing, safety and surface gating in one prompt plus parser heuristics. The empty-plan and format-boundary failures are consequences.
- **Single pass, no retry** (`ARCHITECTURE.md` §2.3 `[code]`). Defensible for WRITE nodes (a retry re-fired real trades). But it is applied to the whole pipeline, including read-only nodes where retry is safe. The right abstraction is *idempotency class per node* (read / idempotent-write / non-idempotent-write), not one global rule. `[rec]`
- **Telemetry that isn't a record.** Langfuse spans are excellent for debugging and not durable domain history (truncated, can be disabled). Several tables each hold a partial story (`PositionOrigin.plan`, `JournalEntry`, `ScheduledTask.lastResult`).
- **Config and wiring fan-out.** Adding a node touches planner prompt, node registry, orchestrator, context assembler, composer prompt, surface policy, tests. The vault lists the same.
- **Redis lock for failover without fencing** (`tg:primary`, `ARCHITECTURE.md` §5 `[code]`). Adequate if the only harm of split-brain is duplicate canned replies; worth understanding precisely why, rather than assuming. `[rec: understand, don't rebuild]`
- **Staleness gate exists for prices, not for instruments.** A known, written-down gap `[code]`.

**Things I'd push back on if asked to "improve" them**
- Don't rewrite the DAG; the mechanics (layers, allSettled, gates) are sound.
- Don't build a generic vector-memory platform; your missing memory is *structured state* (preferences, flags, strategy), which embeddings won't fix.
- Don't treat "more evals" as the answer before you can say how noisy an eval is (see §4, Evaluation statistics).

---

## 3. Knowledge, by subsystem

Format per topic: **What** · **Where in Agent Pear** · **Helps solve** · **Realistic improvement** · **Depth** (L1 awareness, L2 working, L3 strong competence, L4 specialist) · **Tier**.

### 3.1 Agent and LLM engineering

**Typed boundaries: schemas, discriminated unions, parsing at trust boundaries** · Tier A · L3
- What: modelling "what can this value legally be" so illegal states are unrepresentable; parse, don't validate; a schema is also a truthfulness boundary.
- Where: planner `PlannerDecision` (every field nullable → all-null is legal), `AggregatedFindings`/`TextResult`, `open_position` condition union, MCP vs agent validation parity, structured extract in composer.
- Solves: Kalshi dead zone, `7e0824d9` format loss, silent substitution of the user's trade, unvalidated JSON-string shortcut.
- Improve: a `PlanEnvelope` (clarify | answer | research | act | schedule) with a refinement that forbids "nothing selected"; error messages carrying `issue.path`; typed evidence blocks with `observedAt`/`freshness`/`source` for the first one or two domains.
- Search: "parse don't validate", zod discriminated unions and refinements, "make illegal states unrepresentable", JSON schema constrained decoding, Vercel AI SDK structured output failure modes.

**Tool calling and structured output behaviour across providers** · Tier A · L3
- What: how models choose tools, how `tool_choice` is honoured (or not), how streaming surfaces errors, how "reasoning" and thought-signature state pass between steps.
- Where: every node's tool loop; `twophase:` model wrappers; DeepSeek Flash ignoring `tool_choice: none` `[vault memory]`; ai@6 streamText swallowing later-step errors.
- Solves: fallback that never fires, DSML markup leaks, thought-signature reroutes, retry loops that burn calls.
- Improve: a small conformance harness each candidate model must pass (tool call, forced final step, streaming error after a tool step, empty 200); fallback keyed on "nothing emitted".
- Search: "OpenAI function calling vs structured outputs", "tool_choice semantics", "Vercel AI SDK streamText error handling", "OpenRouter provider routing allow_fallbacks".

**Context engineering and memory design** · Tier A · L3
- What: deciding what goes into the window, in what form, with what precedence; separating durable structured state from re-derivable prose.
- Where: 3-turn raw history + recall notes + journal + hot-facts + KB + dependency context; `skipRisk` incident; stale recall precedence enforced only in prompt.
- Solves: preferences forgotten or resurrected; prompt bloat; contradictory instructions.
- Improve: persist per-flow trade state (risk protection, size intent) server-side with the model able to override; give recall notes timestamps and expiry the code enforces rather than the prompt.
- Search: "context engineering for agents", "memory in LLM agents state vs retrieval", "lost in the middle", "prompt caching".

**Agent architectures and planning** · Tier B · L2→L3
- What: planner/executor vs ReAct vs plan-and-execute, DAG scheduling, supervisor patterns, where each fails.
- Where: standard DAG vs "advanced" single tool loop (Qwen won the blind A/B, per `models.ts` comment), Oracle-harness question in the Revamp note.
- Solves: the open question of two agent loops; consistency of semantics across modes.
- Improve: write down what each mode guarantees (writes at most once? confirmation? gating?) and test the same fixtures against both.
- Search: "plan-and-execute agents", "LLMCompiler", "ReAct", "agent harness design".

**Retrieval and RAG** · Tier B · L2→L3
- What: chunking, embeddings, hybrid lexical+dense, recall@k / MRR, reranking, evaluation of retrieval separately from generation.
- Where: `packages/shared/knowledge/*.txt` embedded into a committed artifact; `kb-index.ts` affordance list; `kb:eval`.
- Solves: "agent doesn't know / invents a command" class; drift between KB index and docs.
- Improve: add a hard-negative set (near-duplicate docs) to `kb:eval`; consider BM25 alongside embeddings for exact tokens like command names.
- Search: "hybrid search BM25 dense", "MRR recall@k", "retrieval evaluation hard negatives".

**LLM evaluation: design, grading, drift** · Tier A · L3 (see also Evaluation statistics in §3.6)
- What: building test sets, judging, deterministic invariants vs LLM judges, regression vs held-out, pairwise preference, judge bias.
- Where: planner fixtures (regression/holdout/prod), awareness eval with grammar invariants, KB eval, interest-distiller A/B with neutral judges.
- Solves: "did the prompt change help or did sampling variance?"
- Improve: report confidence intervals; pin the model during A/B (your fallback-contaminated run is the lesson); keep the invariant checker (mechanical) as the primary gate.
- Search: "LLM-as-a-judge bias", "pairwise evaluation", "paired bootstrap", "eval set contamination".

**Prompt injection and untrusted content** · Tier B → A if autonomy grows · L2→L3
- What: indirect injection through tweets, web pages, tool output; confused deputy; capability scoping.
- Where: twitter ingestion → LLM labeller → context DB → agent; research node (Tavily/Exa); Claude basket research whose evidence quality is self-attested (open review thread on #786 `[vault]`).
- Solves: a poisoned tweet influencing a basket or an alert; SSRF-class bugs.
- Improve: treat anything from X/web as data with provenance; validator or approval gate before persisting model-attested evidence; keep writes behind tickets.
- Search: "indirect prompt injection", "lethal trifecta", "OWASP LLM top 10", "confused deputy".

**Model routing, cost/latency/quality** · Tier A · L3
- What: choosing per-node models from measured latency, error rate and quality; classifying failures (429 vs 502 vs HTTP-200 error); fallback on a different provider and key bucket.
- Where: `models.ts` (planner on Cerebras, tradeExecution on Flash Lite, pair/basket on gpt-6-luna), emergency swaps Aug–Sep.
- Improve: a routing table built from historic Langfuse p50/p95/error per node; this ends emergency swaps being the default process.
- Search: "LLM routing", "tail latency p99 budgets", "hedged requests".

**Transformers, attention, KV cache, quantization, batching, LoRA** · Tier C/D · L1→L2
- See §6 for the self-hosting analysis. You need conceptual literacy to *read* provider docs and pricing (why input tokens are cheap with caching, why long contexts cost latency); you don't need to implement any of it.

### 3.2 Software and systems

**State machines and lifecycles** · Tier A · L3
- What: modelling entities as explicit states with legal transitions and one place that enforces them.
- Where: trade ticket (pending→confirmed→executed/expired…), plan token (minted→consumed→recorded), execution polling, signal trigger→ticket, scheduled tasks, ghost triggers, watch entries, Hetzner timers.
- Solves: the "binding_mismatch is not proof nothing was placed" class; double-consume; ambiguous terminal states.
- Improve: draw the ticket and plan-token FSM from the code; list every transition and what a crash between two of them leaves behind; add the missing property tests.
- Search: "finite state machine modelling backend", "statecharts", "TLA+ for engineers" (light), "saga pattern".

**Idempotency, at-least-once delivery, claims and locks** · Tier A · L3
- What: making repeated processing safe; dedup keys; `SELECT … FOR UPDATE SKIP LOCKED`; lease columns; fencing tokens; outbox pattern; dead-letter queues.
- Where: BullMQ jobs, Twitter `getPendingContent()` with no row claim (still open `[vault]`), atomic Redis `getDel` for callbacks, plan-token single-use, `tg:primary` lock, `append-only createMany` in user-value-score.
- Solves: duplicate Twitter processing, dual-scheduler races during Railway↔Hetzner cutover, Telegram ordering investigation.
- Improve: add the claim to the pending-content query; write an idempotency checklist for every new cron (what happens if two run at once; what if the run dies halfway).
- Search: "idempotency keys", "transactional outbox", "SKIP LOCKED queue Postgres", "Redlock critique Kleppmann fencing token", "BullMQ job retries stalled jobs".

**Concurrency in Node/TS: event loop, backpressure, cancellation** · Tier B · L3
- What: the event loop, timers, `AbortSignal` propagation, unhandled rejections, resource handles that keep a process alive, `AsyncLocalStorage`.
- Where: composer/planner timeouts, per-tier abort in planner, one-shots that hung 28 min on open Redis/Discord handles, the Linkwallet unhandled rejection, ALS for Telegram send correlation.
- Improve: standard "every async callback has an error boundary" lint/test; a shared shutdown helper.
- Search: "Node.js event loop phases", "AbortController timeouts", "unhandled promise rejection process", "libuv handles active".

**Databases: Postgres internals, transactions, migrations** · Tier B (your own vault flags High) · L3
- What: query plans, indexes, isolation levels, lock modes, connection limits, zero-downtime migrations, Prisma's interactive-transaction timeout.
- Where: three Postgres DBs, append-only vs overwrite tables (`AssetSentiment` no unique key), `$transaction` 5 s overrun over a public proxy.
- Improve: `EXPLAIN` the hot queries; batch per-row loops; add the missing unique constraints deliberately.
- Search: "Postgres EXPLAIN ANALYZE", "isolation levels anomalies", "Prisma interactive transactions timeout", "expand-contract migrations".

**Redis semantics and caching** · Tier B · L3
- What: data structure choices, TTL and eviction, pub/sub being fire-and-forget, atomic ops, cache invalidation, stale-while-revalidate, response caches in front of LLMs.
- Where: conversation cache, rate limits, `asset_index:full`, plan-token store, `realtime:private` pub/sub, OpenRouter 5-minute response cache.
- Improve: define freshness contracts (max age, behaviour on stale) per cached thing; wire the instrument-age gate.
- Search: "cache invalidation strategies", "Redis pub/sub vs streams", "stale-while-revalidate", "cache stampede".

**Distributed-systems awareness** · Tier B · L2
- What: partial failure, clocks, ordering, consensus at a "what problem does it solve" level.
- Where: engine → Redis → agent alerts, failover host, multiple workers on one queue.
- Improve: the vocabulary to reason about the Telegram ordering bug (concurrent jobs vs completion order vs client rendering).
- Search: "Designing Data-Intensive Applications chapters 5, 8, 9", "Lamport clocks", "exactly-once is a lie".

**Linux, systemd, SSH, networking for operations** · Tier B · L2→L3
- What: units/timers, journald, environment inheritance, PATH in non-interactive SSH, DNS/TLS basics, latency between hosts.
- Where: the Hetzner migration (27 timers), the deploy script's `pnpm: not found`, `/run` permissions.
- Improve: a runbook per timer with expected duration, expected side effects and kill-switch; basic alert when a timer's last result isn't success.
- Search: "systemd timers", "journalctl", "non-interactive ssh environment".

**Memory allocation, pointers/ownership, OS internals** · Tier D · L1
- Honest read: Node/TS service code, with the stat service in Rust *[vault: `pear-stat-rs`; unverified here]*. You'd touch ownership only if you read or change that service. Learn heap/GC behaviour and handle leaks (Tier B above), not allocators.

**Data structures and algorithms** · Tier D beyond L2
- Where it appears: toposort (already implemented), alias resolution, ranking, dedup/overlap checks. L2 is enough. Don't grind.

### 3.3 Observability and production engineering

**Tracing, structured logs, metrics (OTel, Langfuse)** · Tier A · L3
- What: spans vs logs vs metrics, correlation IDs, sampling, cardinality, trace-to-code navigation, what must never be truncated.
- Where: Langfuse OTel, `formatLlmError`, ALS-based per-turn Telegram logs, Discord webhooks for error alerts, the 500-char `finalText` cap that looked like a truncation bug.
- Gap: background jobs have *zero* Langfuse tracing (checked by direct query, 2026-09-15 `[vault]`) and OpenRouter calls carry no per-job attribution tag.
- Improve: tag every OpenRouter call with job/node (`X-Title` or metadata) so cost per job becomes answerable; give background jobs spans; add a run ID that travels request→job→ticket→execution.
- Search: "OpenTelemetry semantic conventions for GenAI", "Langfuse tracing sessions", "correlation ID propagation", "high cardinality metrics".

**Error taxonomy and SLOs** · Tier B · L2→L3
- What: classifying errors by who can act (user, provider, our bug, venue), defining SLIs (plan success, tool success, time-to-first-token, confirm→filled), error budgets.
- Where: one Discord channel for "error" mixes expected Telegram 403s with real faults (the on-call page); alerts that name a failure the evidence contradicts (`No baskets generated`).
- Improve: a short error taxonomy enum, used in logs; three SLIs measured from existing traces.
- Search: "Google SRE book SLI SLO", "error budget", "alert fatigue".

**Incident response and post-incident practice** · Tier B · L2
- You already do this unusually well (hypothesis lists, replay before blame). The gap is making it repeatable: a template and a rule that every incident ends in either a regression fixture or a written reason it can't.

**Replay and production evaluation** · Tier A · L3
- What: capturing a run (input, history, model/prompt versions, tool results, clock) and re-running it deterministically with stubs.
- Where: fragmented today: planner fixtures, node evals, composer evals, integration suite, replay scripts per cron.
- Improve: one fixture format; start with only the standard DAG path; turn `7e0824d9` into the first fixture.
- Search: "record and replay testing", "golden master tests", "deterministic simulation testing".

### 3.4 Quantitative finance and trading

Important orientation `[code][vault]`: signal generation, z-scores and cointegration are computed by the external `pear-stat-arbs` service; this repo consumes and explains them. So the leverage for you is **judging, validating, explaining and sizing off those numbers**, and understanding whether the numbers deserve trust, not writing the estimators.

**Returns, volatility, correlation, covariance** · Tier A · L3
- Log vs simple returns, annualisation, sample vs shrinkage covariance, rolling windows, outliers in crypto.
- Where: KB `stat-diagnostics.txt`, correlation on log returns, volatility regimes (LOW/NORMAL/HIGH), basket stats.
- Improve: read a few stat outputs against raw data by hand once; know why correlation 0.3–0.7 is normal for cointegrated crypto pairs (the KB already says it).
- Search: "log returns vs arithmetic", "covariance shrinkage Ledoit-Wolf", "volatility clustering".

**Regression, beta, hedge ratios** · Tier A · L3
- OLS of A on B, errors-in-variables (OLS is asymmetric: regress A on B ≠ B on A), total least squares, rolling vs long-term beta, Kalman-filter dynamic hedge.
- Where: `lt_beta` for sizing vs dynamic `hedge_ratio` for z-score; pair orientation (BNB–AVAX incident was about orientation).
- Solves: "which leg is the dependent variable" bugs; sign errors; why beta divergence is a warning.
- Search: "hedge ratio OLS vs TLS", "Kalman filter pairs trading", "regression direction asymmetry cointegration".

**Stationarity, cointegration, mean reversion, half-life** · Tier A · L3
- Engle–Granger vs Johansen, ADF on *residuals* needs different critical values from plain ADF (MacKinnon), unit-root test power is low, AR(1) half-life estimate noise, Hurst on spread residuals.
- Where: KB `stat-diagnostics.txt`; pair and basket nodes.
- Questions to ask of the stat service *[unverified]*: does it use residual-based critical values? what lookback? how does it handle structural breaks? how are p-values reported to users?
- Search: "Engle-Granger two-step", "MacKinnon critical values cointegration", "Ornstein-Uhlenbeck half-life", "Johansen test".

**Multiple testing, selection bias, backtest overfitting** · Tier A · L3 (the highest-value item people skip)
- What: if you scan thousands of pairs and keep the ones that pass at 5%, many pass by chance. FDR control, deflated Sharpe, out-of-sample walk-forward, "Sharpe 5+ is an artifact" (your KB already says so).
- Where: pair/basket discovery, `ai-picks`, forward-evaluation `paper-shadow` (positive on 10 baskets, small and simulated `[vault]`), familiar-asset ranking.
- Improve: for any "top pairs" feature, ask how many candidates were screened and report that; require out-of-sample persistence before surfacing.
- Search: "Probability of backtest overfitting Bailey López de Prado", "deflated Sharpe ratio", "Benjamini-Hochberg false discovery rate", "walk-forward analysis".

**Perpetual futures mechanics: funding, basis, open interest, leverage, margin, liquidation** · Tier A · L3
- Cross vs isolated margin, maintenance margin, liquidation price for a multi-leg book, funding as carry, OI + funding as positioning proxies, mark vs index vs last.
- Where: `concepts/{funding-rate,liquidation,leverage-margin-notional,hyperliquid-mechanics,account-unification,hip-3,lighter}.txt`; incidents on leverage cap changing stated margin, HIP-3 collateral, No-Dex-Abstraction accounts; Coinglass node.
- Improve: be able to compute by hand a liquidation price and funding cost for a two-leg trade; check the agent's numbers against it.
- Search: "Hyperliquid docs margining", "perpetual futures funding rate derivation", "cross margin liquidation".

**Order books, liquidity, slippage, transaction costs, microstructure** · Tier A (basics) / Tier C (modelling) · L2→L3
- Where: TWAP, ladder, trigger orders, builder fees (0.06% per leg per the vault), price-drift gate (100bps default), slippage in entry decisions.
- Improve: know how to estimate slippage for a given notional from depth; check whether signal "edge" survives costs (KB says small edges get eaten).
- Search: "limit order book basics", "market impact square-root law", "TWAP VWAP execution".

**Execution risk: leg risk, partial fills, compensation** · Tier A · L3
- What: what to do when leg 1 fills and leg 2 doesn't; unwind policy; hedge-first ordering; reduce-only semantics.
- Where: partial-fill alert, reduce-only verification, "rollback/compensation unresolved" in your own Knowledge Gaps.
- Improve: write the failure matrix (leg A/B × filled/partial/rejected/unknown) and what the product does in each cell today.
- Search: "saga compensation trading", "legging risk", "reduce-only order".

**Portfolio construction and risk** · Tier B · L2→L3
- Position sizing off stop distance, vol targeting, correlation of stop-outs, sector concentration, exposure aggregation, Kelly as a warning not a rule, drawdown.
- Where: KB `risk-and-sizing.txt`; absence of a cross-venue portfolio view.
- Search: "volatility targeting", "risk parity basics", "maximum drawdown", "fractional Kelly".

**Regimes and structural breaks** · Tier B · L2
- Detecting breaks, rolling re-estimation, CUSUM, hidden Markov models at awareness level. Pear explicitly does not predict regime change; know why the time-limit exit (3× half-life) exists.

**Prediction markets (Polymarket/Kalshi)** · Tier B · L2
- Probability ≠ direction without spot and strike; implied distribution from strike ladders; resolution risk; thin-book prices. Your own lesson (#1054) is the starting point; the chat agent's odds tool was not rechecked.

**Crypto-specific structure** · Tier B · L2
- HyperEVM/HIP-3 builder-deployed perps, USDH vs USDC collateral, Lighter's zk-based order book, builder codes, agent wallets and signing, on-chain vs off-chain matching, oracle/mark price risk, delisting windows, token unlocks (projected vs confirmed).
- Where: `hip-3.txt`, `hip-4.txt`, `lighter.txt`, tokenomist node, "Token-Unlock Amounts Shown as Confirmed Facts."

**Backtesting engineering** · Tier B · L3
- Look-ahead, survivorship, cost modelling, funding in PnL, bar-size/annualisation errors (the KB calls this "the most common mistake"). The vault gap-analysis warns a naive backtester would mislead; treat that as correct.

### 3.5 Product and UX

**Confirmation flows, safe execution UX** · Tier A · L3
- Where: ticket + Confirm button, group-vs-DM suggested actions, `/menu` "Confirmations off" risk (pre-merge review for #1010 `[vault]`), disclosure when leverage caps change margin.
- Learn: confirmation fatigue, destructive-action design, showing deltas ("you asked X, I'll do Y"), default-safe, undo/expiry. Search: "confirmation dialog fatigue", "Nielsen error prevention", "trading UI risk disclosure".

**Honesty of conversational output** · Tier A · L2→L3
- Promises the system can't keep (retry, watching), invented commands/UI paths, fabricated time, a data outage claim that was really an empty plan, projected vs confirmed numbers. This is the largest *user-visible* class of defect in your record and is mostly a systems problem (give the composer true facts) rather than a prompt problem.

**Streaming and latency perception** · Tier B · L2
- Time-to-first-token, status events per node, skeletons, partial results; p99 budgets (pair node timeout 85 s). Search: "perceived performance", "streaming UX for LLMs".

**Showing uncertainty and explaining recommendations** · Tier B · L2→L3
- Confidence intervals vs a single z-score, "why this pair," freshness stamps, evidence citations. Needs the typed evidence work above to be more than prose.

**Notification design** · Tier B · L2
- Alert fatigue, batching, relevance gating (signal funnel rollup), per-surface delivery (web delivery TODO). Search: "notification design principles", "alert fatigue".

**Charting/visualisation** · Tier C/D · L1→L2. Worth it only when entry/TP/SL annotations or spread charts are on the roadmap (#665).

---

## 4. Hidden areas you didn't list

These are the fields your incidents are secretly about. Ranked by how much of your record they explain.

1. **Interface design between probabilistic and deterministic components (a mini compiler problem).** Intermediate representations, grammars, error recovery, "lowering." Your planner/DAG/composer chain is a compiler pipeline where each stage's contract is a string. Study: *Crafting Interpreters* (parsing and IR chapters), "compiler IR design," "error-recovering parsers." This reframes half your bugs.

2. **Experimental design and statistics for your own evals.** Your own data: old sentiment prompt flips direction on 7/17 assets between identical runs; "11% on 27 calls" accuracy; v2's neutral count 4–13 of 16–17 on identical code. At those sample sizes most A/B differences are noise. Study: confidence intervals, paired tests, bootstrap, power analysis, variance of LLM outputs, multiple comparisons. This also trains the same muscle as pair-selection statistics, so it pays twice.

3. **Cache semantics as a correctness issue.** The OpenRouter response cache replayed empty answers and contaminated repeat-run variance; price/instrument caches look fresh when frozen; a stale doc "lies confidently." Study: consistency models, freshness contracts, negative caching, cache keys.

4. **Process lifecycle and resource handles.** Four incidents (hung one-shots, duplicate Redis clients, Discord gateway not closed, full-AppModule boot) are one topic: what keeps a Node process alive and what a DI container does and doesn't own. Study: libuv handles, NestJS provider scopes and lifecycle hooks.

5. **Provenance and data lineage.** "Where did this number come from and how old is it?" is behind orientation loss, projected-vs-confirmed unlocks, stale recall, tool tags dropped in synthesis. Study: data lineage, provenance models, W3C PROV at concept level.

6. **Selection bias in anything that ranks or recommends.** Pair discovery, familiar-asset ranking ("pick by statistical edge, not return %"), baskets. See multiple-testing above.

7. **Trust boundaries and capability security.** Plan tokens are capability tokens. Agent never holds user keys. Webhook trust model (Elfa decision note). Study: capability-based security, confused deputy, replay attacks, HMAC webhook verification.

8. **Time as an input.** The composer fabricated the current time; retries that "re-measure" change the fact being retried; deterministic clocks for replay. Study: bitemporal data (event time vs processing time), clock injection.

9. **Human factors in high-stakes automation.** Automation bias, trust calibration, mode confusion (the standard vs advanced vs surface differences). Study: "ironies of automation" (Bainbridge), aviation human-factors summaries.

10. **Control theory / feedback intuition for polling, timeouts and retries.** Retry storms, timeout stacking (planner 22 s tier + node 85 s + composer 30 s), backpressure, rate-limit buckets shared between primary and fallback. Study: "retry budgets", "timeouts, deadlines and propagation," queueing basics (Little's law).

---

## 5. Self-hosting and fine-tuning models: a realistic read

**Starting facts.** `[code]` The planner and composer already run **open-weight `gpt-oss-120b`**, served by Cerebras (direct) with OpenRouter as the fallback. Other nodes use proprietary models via OpenRouter. So "run our own models" doesn't mean "go open-source"; it means "operate the serving layer ourselves."

**Where self-hosting could plausibly create an advantage**
- *Latency/throughput on the planner*: Cerebras is very fast; matching it with GPUs you run is unlikely. No advantage here. `[infer]`
- *Provider independence*: your record shows repeated provider shocks (Gemini, MiMo, Cerebras pulling `gemma-4-31b`, Vertex reroutes). Owning the weights removes deprecation risk, not capacity risk (you'd own GPU supply instead).
- *Narrow classifiers*: intent gate, alert matcher, leak classification, label/sentiment tagging, interest-tag filtering are small well-defined tasks. A small fine-tuned model (distilled from your own logged traces) could replace a general API call at lower cost and with stable behaviour. This is the one place I'd call it a plausible *engineering advantage*, later.
- *Data privacy*: only matters if you decide user conversation or position data shouldn't go to third parties. Not evidenced as a requirement `[unverified]`.
- *Embeddings*: you embed ~100 KB docs and ~$0.63/month of other embeddings `[vault]`. No case.

**Where it would be infrastructure pain with little return**
- Running a 120B-class model for chat: a single-GPU-class deployment is possible with quantized weights *[from memory; verify memory footprint before relying on it]*, but you then own autoscaling, cold starts, batching, GPU availability, observability, upgrades, and incident response at the same time you are a small team still fixing agent-contract bugs.
- Tool-calling quality: open models differ widely in tool-call fidelity; your incidents are mostly *contract* failures that a different host won't fix.
- Cost: your measured spend is small relative to a GPU reservation. Known numbers `[vault]`: Twitter ingestion ≈ $0.52/day, sentiment jobs ≈ $0.011/day, embeddings ≈ $0.63/month, basket generation ≈ $2.18/basket on Claude. Interactive chat costs are not in the vault.
- Evaluation comparability: every model change is a new eval baseline; self-hosting adds a second dimension (quantization level, serving engine version).

**Recommendation `[rec]`.** Don't self-host in the next two quarters. Learn enough to be a good *consumer*: inference basics (prefill vs decode, KV cache, batching, quantization at L1–L2) so provider docs, pricing, TTFT and context-length tradeoffs make sense. Revisit when (a) you have a logged corpus for a narrow classifier and (b) a measured cost or reliability number says an API is the bottleneck. At that point a small-model fine-tune (LoRA on a 3–8B model, behind a vLLM or hosted-inference endpoint) is a bounded experiment, not an infrastructure program.

---

## 6. Prioritised tiers

Ranking criteria: impact on Agent Pear, likelihood you'll use it, your current gap, cross-system leverage, time to usefulness.

### Tier A — learn now (1–3 months)

| # | Topic | Depth | Why now |
|---|---|---|---|
| A1 | Typed boundaries / parse-don't-validate / IR design | L3 | explains most recurring LLM-boundary bugs; low cost, high reuse |
| A2 | Eval statistics and replay (noise, CIs, paired comparison, fixtures) | L3 | every prompt/model decision depends on it; your data already shows noise swamping effects |
| A3 | State machines + idempotency/claim patterns | L3 | money path, queues, cutovers; vault flags this as High gap |
| A4 | Tracing/OTel/Langfuse + error taxonomy + run IDs | L3 | background jobs untraced; cost attribution impossible today |
| A5 | Pair-trading statistics: regression direction, cointegration testing, half-life, multiple testing | L3 | judge the numbers you present; quality of trade suggestions |
| A6 | Perp mechanics: margin, liquidation, funding, HIP-3/Lighter specifics | L3 | the agent explains and sizes these daily |
| A7 | Execution microstructure and leg risk | L3 | partial-fill and compensation questions are unresolved |
| A8 | Context engineering + structured state vs prose memory | L3 | `skipRisk`-type bugs recur otherwise |
| A9 | Provider/tool-calling behaviour and model routing from data | L3 | removes emergency-swap mode |
| A10 | Confirmation/honesty UX for trading agents | L2→L3 | user-visible trust |

### Tier B — build over 3–12 months

Postgres internals and migrations · Redis/caching semantics and freshness contracts · Node concurrency and process lifecycle · Linux/systemd/network operations · distributed-systems vocabulary (DDIA) · prompt-injection and trust-boundary threat modelling · SLO/SLI practice · portfolio construction and risk · backtest engineering · regime/structural-break detection (awareness) · retrieval evaluation (hybrid search) · agent architecture patterns · prediction-market semantics · streaming/latency UX · uncertainty display · notification design.

### Tier C — specialised, only if Agent Pear moves there

Fine-tuning a small classifier and serving it · order-book / depth modelling for slippage · Kalman filters and dynamic hedge estimation · Johansen multivariate cointegration for baskets · formal methods (TLA+ light) for the ticket FSM · market-making/maker-taker fee optimisation · event sourcing as a full pattern · charting.

### Tier D — interesting, low ROI now

Implementing transformers from scratch · LoRA/QLoRA internals · quantization algorithms · CUDA and kernel work · memory allocators, pointers, ownership as a topic in itself (unless you start reading the Rust stat service) · low-latency/HFT engineering (your latency budget is seconds, dominated by LLM and venue calls) · consensus protocols beyond awareness (Raft/Paxos) · generic DSA grinding · MEV and zk internals · building your own vector DB.

---

## 7. Roadmap

Each phase is ~4–6 weeks of part-time study alongside normal work. Order is by dependency, not by prestige.

### Phase 1 — Typed boundaries and the planner IR
- **Learn:** parse-don't-validate; discriminated unions and refinements; schema evolution; why "all optional" schemas permit degenerate values; error messages that guide a model.
- **Search:** "Parse, don't validate" (Alexis King), zod refine/superRefine, JSON Schema oneOf, *Crafting Interpreters* IR chapter.
- **Inspect:** `apps/agent/src/ai/dag/services/planner-adapter.service.ts` (`extractStructuredResponse`, `normalize`, `createDefaultPlan`), `planner/agent.ts`, the `PlannerDecision` zod type, `NODE_REGISTRY`.
- **Exercise:** write a property test generating random `PlannerDecision` values and list every legal-but-degenerate shape.
- **Real improvement:** a refinement or `PlanEnvelope` that rejects "nothing selected," emits a classified failure reason, and routes to a safe read-only default instead of silent `generalChat`. Closes the Kalshi and stock-pick class (PR #986 follows the same line).

### Phase 2 — Replay and evaluation statistics
- **Learn:** variance of LLM outputs; confidence intervals for proportions; paired tests; bootstrap; power; why pinning the model matters; golden-master fixtures.
- **Search:** "bootstrap confidence interval", "McNemar test", "LLM eval variance", "record and replay testing".
- **Inspect:** `apps/agent/test/planner.e2e-spec.ts` and fixtures, `awareness-eval.ts`, `awareness-invariants.ts`, `kb-eval.mjs`, sentiment playground scripts, telemetry service.
- **Exercise:** take one fixture set, run it 20 times, compute per-fixture flip rates and the confidence interval on the pass rate; decide how large a change you could even detect.
- **Real improvement:** a `RunFixture` format and one replay command for the standard DAG path with mocked tools; the `7e0824d9` failure as fixture #1; add CIs to the planner and awareness eval summaries.

### Phase 3 — State machines, idempotency, queues
- **Learn:** FSM modelling; at-least-once delivery; outbox; `SKIP LOCKED`; fencing tokens; saga/compensation; BullMQ stalled-job behaviour.
- **Search:** "transactional outbox", "Postgres SKIP LOCKED queue", "How to do distributed locking" (Kleppmann), "BullMQ stalled jobs".
- **Inspect:** ticket and plan-token code in `packages/execution` (`plan-token-protocol.ts`, `confirm-protocol.ts`, `reconcile-after-execution.ts`), `trade-tickets`, `tg-delivery`, the `tg:primary` lock, `TwitterCollectorService.getPendingContent()`.
- **Exercise:** draw the ticket and plan-token state diagram from code, then for each transition note "what exists in the DB and venue if the process dies here."
- **Real improvement:** add the row claim to Twitter pending-content (open item); write the leg-risk failure matrix; wire the instrument-age gate (documented as available, not wired).

### Phase 4 — Observability as a product
- **Learn:** OTel concepts, GenAI semantic conventions, correlation IDs, error taxonomy, SLIs from traces, cost attribution.
- **Search:** "OpenTelemetry GenAI semantic conventions", "Langfuse sessions and scores", "SRE workbook SLOs".
- **Inspect:** `apps/agent/src/ai/telemetry/telemetry.service.ts`, `formatLlmError`, the OpenRouter factory (headers), background processors (no tracing), Discord logging.
- **Exercise:** from existing Langfuse data, compute p50/p95/error rate per node for a week; identify the three nodes where the budget is tightest.
- **Real improvement:** tag OpenRouter calls with job/node so per-job cost is answerable; trace the 27 background jobs; add a run ID through request→ticket→execution; derive a model routing table from the numbers.

### Phase 5 — Pair-trading statistics and perp mechanics
- **Learn:** OLS direction asymmetry; Engle–Granger with correct critical values; half-life estimation noise; multiple testing; funding, margin and liquidation maths; slippage estimation.
- **Search:** "Pairs trading Vidyamurthy" / "Ernie Chan Algorithmic Trading ch.2–3", "MacKinnon critical values", "Probability of backtest overfitting", Hyperliquid docs (margining, funding).
- **Inspect:** `packages/shared/knowledge/quant/*.txt`, `concepts/*.txt`, the shared `QuantService`, `made-baskets-cron.service.ts` (overlap/frequency rules), `paper-shadow.ts`, the pair and basket node prompts.
- **Exercise:** pick one live signal, recompute hedge ratio, ADF on residuals and half-life from raw prices in a notebook; compare with the service. Separately compute a liquidation price and 7-day funding cost for a two-leg position.
- **Real improvement:** a "candidates screened" count and out-of-sample persistence check beside any ranked-pair feature; an orientation test that asserts a pair's sign survives the agent boundary (BNB–AVAX class); a numeric self-check the composer can cite instead of inventing.

### Phase 6 — Context engineering, structured memory, trust
- **Learn:** structured state vs recalled prose; precedence and expiry enforced in code; provenance; indirect injection and capability scoping; human factors of confirmation.
- **Search:** "context engineering", "indirect prompt injection", "capability-based security", Bainbridge "Ironies of Automation".
- **Inspect:** `recall.agent.ts`, `recall-preferences.ts`, `journal.agent.ts`, `trade-profile.service.ts`, `PositionOrigin`, twitter ingestion → context DB → agent path, `common-rules.ts`, `internal-leak-guard.ts`.
- **Exercise:** list every piece of state the agent "remembers"; mark each as persisted-structured, persisted-prose, or window-only; find the ones where loss would change a trade.
- **Real improvement:** server-persisted trade-flow state (`riskProtection`) with model override, closing the open `skipRisk` diagnosis; an expiry rule for recall notes enforced in code.

### Later (only on a trigger)
- Small-model distillation experiment — trigger: a narrow task with 10k+ logged examples and a measured API cost or reliability problem.
- Portfolio snapshot / exposure model — trigger: product commits to cross-venue portfolio views.
- Strategy object — trigger: product direction decided; see Gap Analysis for the sketch.

---

## 8. Top 15, ranked for you and Agent Pear

1. **Typed interfaces between LLM stages (IR, schema refinements, parse-don't-validate).** Explains the planner, composer and tool-validation incidents.
2. **Statistics of LLM evaluation (variance, CIs, paired comparison, pinned models).** Without it, prompt and model decisions are partly guesses.
3. **Record-and-replay testing for agent runs.** Turns each incident into a permanent regression.
4. **State machines for tickets, plan tokens and executions.** The money path's correctness argument.
5. **Idempotency, at-least-once delivery and claim/lock patterns.** Queues, cutovers, Twitter double-processing, Telegram ordering.
6. **Cointegration, hedge-ratio estimation and half-life, including their failure modes.** The core of what you're explaining to users.
7. **Multiple testing and backtest overfitting.** Protects every "top pairs / ranked baskets / Sharpe" claim.
8. **Perpetual-futures mechanics (margin, liquidation, funding, HIP-3, Lighter).** The agent talks about these constantly and has been wrong before.
9. **Leg risk and compensation in non-atomic multi-leg execution.** The largest unresolved safety question in your own notes.
10. **Tracing/OTel/correlation and error taxonomy, including background jobs.** Currently the blind spot.
11. **Context engineering with structured, server-persisted state.** Prevents the `skipRisk` family.
12. **Provider and tool-calling behaviour; routing from measured data.** Ends emergency swaps as a process.
13. **Cache semantics and freshness contracts (prices, instruments, LLM response cache, docs).** Four incidents trace to stale-looking-fresh data.
14. **Postgres/Redis operational semantics (transactions, locks, timeouts, migrations).** Your vault ranks it High and the Hetzner migration keeps hitting it.
15. **Trust boundaries and prompt-injection threat modelling.** Becomes urgent the moment any path gets autonomy or ingests more untrusted text.

Deliberately not on the list: transformer implementation, self-hosting, LoRA, quantization, allocators, HFT latency. Revisit self-hosting only on the triggers in §5.

---

## 9. Open questions I couldn't resolve from here

- What exactly do `pear-stat-arbs` / `pear-stat-rs` compute (critical values, lookbacks, break handling)? Reading that service would sharpen Phase 5. *(external, unverified)*
- What does interactive chat actually cost per turn? The vault has background-job costs but not a per-node chat figure, which any model-routing or self-hosting argument needs.
- Is the `tg:primary` lock safe against split-brain in the way that matters (duplicate replies only), or can both hosts act on the same update? Worth reading `apps/tg-fallback` before changing anything.
- How much of the Revamp's "bounded execution policy" is in scope? That decides whether Trust boundaries (§8 #15) is Tier A or B for you.
