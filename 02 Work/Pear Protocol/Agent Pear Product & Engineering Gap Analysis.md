# Agent Pear Product & Engineering Gap Analysis

Date: 2026-09-20

## Executive conclusion

Agent Pear is not primarily missing another agent, another venue adapter, or another prompt.

It is missing a durable middle layer between:

> user intent → agent reasoning → evidence → plan → execution → outcome → future behavior

Today those concepts exist, but in separate forms:

- chat messages
- planner JSON
- DAG node strings
- tool traces
- `PositionOrigin.plan`
- `PositionOrigin.intent`
- `JournalEntry.payload`
- `ScheduledTask.instruction`
- `ScheduledTask.lastResult`
- `UserTradeProfile.profile`
- signal/watch tables
- Langfuse spans

They are not one coherent, typed, replayable product model.

That is the main architectural ceiling.

The strongest builder opportunity is therefore:

> Build Pear’s durable decision and strategy runtime: an append-only, typed record of what the user wanted, what Pear knew, what it decided, what it did, and what happened afterward.

Start as a run ledger and replay system. Grow it into persistent strategies, portfolio intelligence, proactive monitoring, post-trade learning, and eventually safe autonomous workflows.

This is substantially more valuable than continuing to own deployment reliability or isolated production hardening.

## Scope and evidence

I inspected:

- the current repository branch, `fix/claude-basket-research-diversity`
- `ARCHITECTURE.md`, package boundaries, agents, tools, execution, data services, APIs, background workers, tests, and Prisma schemas
- the adjacent work-history corpus under `C:\Users\prakh\OneDrive\Documents\Desktop\Projects & Code\Projects - Main\agent-pear-v2\docs`
- recent authored history, including basket generation, Lighter integration, signal systems, evals, deployment, observability, and investigations
- the planner investigation around production trace `7e0824d9`
- the V3/Lighter integration test plan and telemetry inventory

The current branch is at:

`b00eeb45 fix(context): restore basket-research novelty signal so a run's baskets diversify`

The work-history corpus is mostly local engineering notes and plans rather than a separate Git repository. I treated those notes as evidence of your context and prior investigations, not as authoritative product requirements.

## 1. Agent Pear as a system

### Concise architecture map

```text
Telegram / Web / Pear Pro / Base / Twitter / MCP / CLI
                    │
                    ▼
            Chat and API surfaces
                    │
                    ▼
        Session + conversation persistence
        Prisma ChatMessage / Session
        Redis recent-context cache
                    │
                    ▼
        Standard or Advanced agent path
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
   Planner + DAG path     Advanced tool loop
          │                   │
          ▼                   ▼
   PlannerDecision       execute_ops / tools
          │
          ▼
   DAG orchestration
   - node registry
   - dependency graph
   - topological layers
   - parallel execution
          │
          ▼
   Specialist agents
   - pair
   - basket
   - research
   - Twitter
   - tokenomics
   - Coinglass
   - market scanner
   - sentiment
   - trade execution
   - scheduler
          │
          ▼
   Tools and data services
   - quant DB
   - context DB
   - asset index
   - market APIs
   - account reads
   - execution tools
          │
          ▼
   Shared execution core
   packages/execution
   - plan / execute protocol
   - venue adapters
   - order families
   - account selection
   - polling and reconciliation
          │
          ▼
   Pear V3 execution engine
   Hyperliquid / Lighter / other engine venues
          │
          ▼
   Results
   - SSE
   - Telegram delivery
   - MCP structured results
   - trade tickets
   - charts
   - citations
   - PnL cards
          │
          ▼
   Persistence and background systems
   - memory recall
   - journals
   - trade profiles
   - schedules
   - signals
   - watches
   - ghost triggers
   - X digests
   - value scoring
```

### Major subsystem assessment

| Subsystem | What it does today | Mature | Primitive or missing |
|---|---|---|---|
| Interfaces | Telegram, web/SSE, Pear surfaces, MCP, Twitter, CLI | Surface gating and protocol contracts are strong | Semantics differ by surface; capabilities are not represented by one shared product model |
| Sessions | Prisma sessions/messages plus Redis recent context | Good persistence and identity migration | Mostly conversation-centric; no durable workflow/run identity |
| Planner | Produces node activation and dependencies | Rich routing rules and extensive tests | Large prompt, brittle structured output, weak typed plan semantics |
| DAG | Executes dependency layers in parallel | Strong orchestration mechanics | Cross-node contracts are primarily text |
| Specialist agents | Domain-specific research and execution agents | Broad coverage and mature tool use | Agent boundaries are partly prompt-defined and duplicated |
| Tools | Many domain tools, execution tools, MCP tools | Execution/MCP schemas are increasingly strong | Agent tool registry and evidence model are fragmented |
| Execution | Plan-token confirmation, account/venue resolution, order families, reconciliation | Strongest subsystem in repository | Missing higher-level strategy, portfolio, lifecycle, and outcome model |
| Asset/data layer | Unified asset index, quant/context databases, ingestion pipelines | Strong data collection and venue awareness | Multiple identifiers and representations still leak upward |
| Memory | Fact recall, journals, preferences, deterministic trade profile | More mature than many agent products | Fragmented; no unified trader model or queryable decision history |
| Background behavior | Schedules, signal alerts, watches, ghosts, digests | Sophisticated proactive notification infrastructure | No unified event/action/workflow abstraction |
| Evals | Planner, node, composer, integration, background, KB, trade evals | Broad and valuable | No single replayable run/eval platform with lineage across the whole system |
| API/SDK | Chat API, SSE, SDK protocols, MCP | Good wire contracts | Mostly exposes chat and operations, not durable agent capabilities |
| UX | Chat, trade tickets, charts, PnL, MCP widgets | Good tactical flows | Powerful backend state is not exposed as durable strategies, decisions, or portfolio intelligence |

## 2. Important current code paths

### Entry and conversation

Relevant paths:

- `apps/agent/src/chat/chat.controller.ts`
  - `POST /chat`
  - `GET /chat/stream`
  - session and message APIs
  - `/chat/execute`
- `apps/agent/src/chat/chat.service.ts`
- `apps/agent/src/ai/services/memory/conversation-store.service.ts`
- `packages/agent-sdk/src/protocol/messages.ts`
- `packages/agent-sdk/src/protocol/events.ts`

The API and streaming protocol are reasonably well defined. The limitation is that the primary resource is still a message/session, not an agent run, strategy, decision, or workflow.

### Planner

Relevant paths:

- `apps/agent/src/ai/agents/planner/agent.ts`
  - `attemptPlan`
- `apps/agent/src/ai/agents/planner/prompt.ts`
- `apps/agent/src/ai/dag/services/planner-adapter.service.ts`
  - `plan`
  - `invokeWithTimeout`
  - `extractStructuredResponse`
  - `normalize`
  - `enforceIdeaQueryDeps`
  - `createDefaultPlan`

The planner prompt contains routing rules for:

- clarification
- pair and basket analysis
- execution intent
- find-then-execute dependencies
- read-only trade questions
- ideas versus actions
- timeframes
- surface restrictions

This is substantial work. The problem is not “the planner has no intelligence.”

The problem is that the planner is acting like a compiler without a real intermediate representation.

The production investigation around `7e0824d9` is especially revealing:

- the model understood the request
- it generated the correct comparison subquery
- it failed to emit the expected `PlannerDecision`
- `extractStructuredResponse()` discarded the intent-correct payload
- `createDefaultPlan()` converted the failure into `generalChat: true`
- the composer correctly refused to fabricate an answer

This is a format-boundary failure, not a keyword-classification failure.

### DAG orchestration

Relevant paths:

- `apps/agent/src/ai/dag/dag.service.ts`
- `apps/agent/src/ai/dag/node-registry.ts`
- `apps/agent/src/ai/dag/executor/orchestrator.service.ts`
- `apps/agent/src/ai/dag/executor/node-executor.service.ts`
- `apps/agent/src/ai/dag/types/index.ts`

The DAG has good mechanics:

- dependency normalization
- topological layers
- parallel execution
- per-node failure capture
- streaming
- surface gates
- auth gates
- node traces
- telemetry

But the key cross-node structures are weak:

```ts
interface TextResult {
  text: string;
  steps?: any[];
  modelId?: string;
}
```

`AggregatedFindings` is mostly a collection of specialist-specific result types whose primary payload is prose.

`DependencyContextService` and `ContextAssemblerService` then turn findings into labelled strings such as:

```text
=== QUANT BACKED PAIR IDEAS ===
...
```

That is effective for a small system. It becomes a ceiling when:

- the composer needs machine-verifiable facts
- a later node needs provenance
- a workflow needs to resume
- an eval needs to compare decisions
- a strategy needs to reference prior evidence
- a UI needs to render structured explanations
- the system needs to distinguish stale, estimated, observed, inferred, and executed data

### Execution

Relevant paths:

- `packages/execution/src/domain`
- `packages/execution/src/contracts/plan.ts`
- `packages/execution/src/contracts/plan-intent.ts`
- `packages/execution/src/domain/shared/plan-family.ts`
- `apps/agent/src/ai/services/v3-execution`
- `apps/agent/src/ai/tools/trade-execution`
- `apps/mcp/src/tools/manifest.ts`

This part is materially mature.

The execution layer already supports:

- venue-aware account selection
- Hyperliquid and Lighter
- asset resolution
- open/close/adjust/reverse
- TP/SL
- TWAP
- ladder
- trigger opens
- rebalance
- auto-rebalance
- plan-token confirmation
- venue verdicts
- polling
- reconciliation
- read APIs
- MCP plan/execute tooling

The right conclusion is not “build a venue abstraction.” That abstraction already exists.

The missing layer is above execution:

- what strategy or decision caused the order?
- what evidence supported it?
- what risk policy was active?
- was it user-requested, signal-triggered, scheduled, or autonomous?
- what outcome did it produce?
- did the user follow or override the recommendation?
- how should Pear reason about the next decision?

### Memory and user state

Relevant paths:

- `apps/agent/src/ai/services/memory/conversation-store.service.ts`
- `apps/agent/src/ai/services/user/memory.service.ts`
- `apps/agent/src/ai/background/recall.agent.ts`
- `apps/agent/src/ai/background/journal.agent.ts`
- `apps/agent/src/ai/services/user/personalization.service.ts`
- `apps/agent/src/ai/services/user/trade-profile.service.ts`

Current state includes:

- session history
- fact memory
- memory notes
- user preferences
- onboarding persona
- trade journals
- position origins
- deterministic trade profile
- interests
- signal subscriptions
- ghost triggers
- watches
- activity/value models

This is not a memory vacuum.

The missing piece is an integrated, typed representation of a trader’s active beliefs, strategies, constraints, decisions, and outcomes.

### Background systems

Relevant paths:

- `apps/agent/src/scheduler`
- `apps/agent/src/signals`
- `apps/agent/src/ai/background`
- Prisma models:
  - `ScheduledTask`
  - `SignalTrigger`
  - `SignalSubscription`
  - `GhostTrigger`
  - `WatchScheduleEntry`
  - `GhostFire`
  - `SignalTriggerPlan`
  - `FollowUpHook`

Proactive systems are already surprisingly advanced.

Pear has:

- scheduled monitoring
- signal triggers
- interest-derived alerts
- ghost trigger synthesis
- watch scheduling
- notification funnels
- digests
- background recall
- user-value scoring

The missing capability is not “background agents.”

It is:

> a single user-owned object that explains what is being watched, why, under what conditions, with what action policy, and what happened over time.

## 3. What is genuinely missing

### Missing capability 1: durable agent run and decision state

There is no first-class object for:

```text
Run
  user request
  surface
  session
  planner intent
  plan version
  evidence
  node outputs
  actions
  approval
  execution
  outcome
  final response
  model/prompt versions
  replay references
```

Evidence:

- `ChatMessage` stores final conversational artifacts, not full structured execution state.
- `DagContext` is request-scoped.
- `NodeTrace` is an in-memory result type.
- `PositionOrigin` only stores a plan JSON and a model-written intent string.
- `ScheduledTask.lastResult` is a truncated string.
- Langfuse spans contain useful telemetry but are not a product/domain record.
- No `AgentRun`, `DecisionRecord`, `EvidenceRecord`, `WorkflowRun`, or equivalent exists in the Prisma schema.

### Missing capability 2: durable strategy or goal representation

The database has monitors, signals, watches, and task instructions, but no first-class strategy object.

A strategy should be able to represent:

- thesis
- assets or basket
- timeframe
- entry conditions
- exit conditions
- risk constraints
- target exposure
- venue/account scope
- evidence requirements
- monitoring cadence
- action policy
- human approval policy
- current lifecycle state
- historical decisions
- realized outcomes

Today those ideas are spread across:

- natural-language `ScheduledTask.instruction`
- trigger JSON
- position origin
- journal narrative
- user profile
- signal subscription
- auto-rebalance config

### Missing capability 3: typed evidence and provenance

Pear has good domain evidence in individual places:

- signal evidence
- basket research evidence quality
- research citations
- tool traces
- market data
- execution receipts

But the DAG composer mostly receives rendered text.

There is no universal evidence object such as:

```ts
type EvidenceBlock = {
  id: string;
  kind: "market" | "research" | "social" | "portfolio" | "execution";
  source: string;
  observedAt: string;
  freshness: "live" | "recent" | "historical";
  confidence?: number;
  provenance: string[];
  payload: unknown;
};
```

### Missing capability 4: unified portfolio intelligence

Current execution reads are useful but mostly account/position oriented.

Missing higher-level views include:

- total cross-venue exposure
- asset-normalized exposure
- long/short directional exposure
- basket exposure
- venue concentration
- funding exposure
- leverage distribution
- strategy attribution
- thesis-level PnL
- decision-to-outcome attribution
- risk budget usage
- portfolio state over time

`UserTradeProfile` is a useful behavioral summary, but it is not a portfolio intelligence model.

### Missing capability 5: end-to-end replay

Pear has many evals, but the repository does not appear to have a single deterministic replay path from:

```text
production request
→ planner
→ DAG
→ tools
→ composer
→ action/ticket
→ final response
```

The current evals are valuable but fragmented:

- planner E2E fixtures
- node evals
- composer evals
- background evals
- integration suites
- KB evals
- trade-execution bakeoffs
- context paper-shadow artifacts

The missing system is a unified run fixture and replay harness with:

- redacted production inputs
- exact model/prompt/schema versions
- mocked tool results
- controllable clock
- venue simulator
- expected decision assertions
- before/after comparisons

### Missing capability 6: web-native proactive delivery

The repository has explicit TODOs in:

- `apps/agent/src/scheduler/scheduler.processor.ts`
  - expired web task notification
  - web notification delivery through SSE/WebSocket

Telegram is much more deeply integrated into proactive delivery.

This is a real gap, but it is smaller than the decision/runtime gap.

### Missing capability 7: unified agent capability registry

`NODE_REGISTRY` is primarily a planner routing table.

MCP has a separate typed `TOOL_MANIFEST`, `ToolDefinition`, `defineTool`, and `toManifest`.

Agent tools are assembled locally by specialist agents:

- `buildReadTools`
- `buildWriteTools`
- `create...Tool`
- per-agent toolsets

What is missing is a shared registry describing:

- capability name
- input schema
- output schema
- permissions
- surfaces
- freshness
- cost
- side effects
- required account
- supported venues
- planner eligibility
- evidence type
- composability

This is important, but it should follow or support the run/evidence model, not become a standalone framework project.

## 4. Architectural ceilings

| Current design | Why it works today | What breaks later | When | Better abstraction | Migration difficulty |
|---|---|---|---|---|---|
| Planner prompt encodes routing policy | Fast to iterate; all rules visible | Prompt size and exception count grow; behavior becomes model-dependent | More nodes, surfaces, and action types | Typed intent + plan compiler + capability metadata | Medium |
| `extractStructuredResponse()` accepts narrow shapes | Keeps planner contract simple | Intent-correct outputs can be discarded; failures become `generalChat` | Already occurred in trace `7e0824d9` | Versioned `PlanEnvelope` with repair/recovery | Low-medium |
| DAG findings are prose | Easy for composer to consume | No provenance, validation, replay, or machine composition | More cross-node workflows | Typed `Finding`/`EvidenceBlock` plus rendered view | Medium |
| Context assembly is hardcoded per node | Explicit and easy to debug | Every new node requires edits in multiple switches and prompt paths | 15–20+ nodes | Capability/evidence registry with renderers | Medium |
| `ScheduledTask` stores instruction and result strings | Good for simple monitors | No structured state, event triggers, policy, or resumable workflow | Strategy automation and richer notifications | Strategy/Workflow/Run entities | Medium-high |
| `PositionOrigin` is one JSON plan + intent string | Enough for trade journal | Cannot explain full reasoning or compare thesis to outcome | More product analytics and automation | Link positions to decision/run/evidence records | Medium |
| Memory, journal, trade profile, interests are separate | Each feature can evolve independently | Pear cannot build a coherent trader model | More personalization and proactive behavior | Unified trader-state projections with provenance | Medium-high |
| Venue identity leaks through multiple types | Two venues are manageable | Venue #3/#4 multiplies conditionals and inconsistent symbols | More connectors or cross-venue portfolio | Canonical `InstrumentRef`, `AccountRef`, `PositionRef` | Medium-high |
| Model choice is per-node config | Practical latency/cost control | Quality/cost policy remains scattered; hard to compare versions | More models and more autonomous paths | Model policy registry + run lineage | Medium |
| Langfuse is the main execution history | Excellent observability | Observability is not a product/domain record and can be truncated/disabled | Replay, user explanations, compliance | Durable run ledger linked to telemetry | Medium |
| Telegram is the strongest proactive surface | Natural notification channel | Web users cannot access the same lifecycle | Web/mobile/product expansion | Shared event/notification layer | Low-medium |
| Advanced mode and standard DAG differ | Each mode can optimize independently | Semantics and guarantees diverge | More surfaces and actions | Common typed capability/action layer | Medium |

## 5. Latent product capabilities already close to possible

These are not speculative greenfield ideas. Pear already has most of the ingredients.

### A. Persistent execution-aware trade intelligence

Existing ingredients:

- execution history
- accounts and positions
- `PositionOrigin`
- journals
- user preferences
- trade profile
- market and research tools
- signals
- scheduled monitors

Missing ingredient:

- a durable thesis/decision/outcome object.

Result:

- “Show me every open position and the thesis that created it.”
- “Which of my trades contradicted my own stated rules?”
- “Tell me what changed since I opened this basket.”
- “Monitor this thesis, not just this ticker.”
- “If the thesis breaks, explain exactly why.”
- “Compare current portfolio exposure with my declared risk profile.”
- “Turn this conversation into a paper strategy.”
- “Show me how this strategy would have behaved historically.”
- “Ask for approval only when a durable strategy reaches an action condition.”
- “After a trade closes, explain whether the original thesis was right, wrong, or simply invalidated by an external event.”

### B. Strategy lifecycle management

Existing ingredients:

- scheduling
- signals
- watches
- trigger plans
- auto-rebalance
- execution plan tokens
- background agents

Missing ingredient:

- a persistent strategy specification and lifecycle.

Result:

```text
draft → paper → monitoring → approval-needed → active → paused → completed → postmortem
```

### C. Personalized market monitoring

Existing ingredients:

- user interests
- X digests
- signal subscriptions
- ghost triggers
- watch scheduler
- DTP
- portfolio reads

Missing ingredient:

- a unified watch/strategy object that can combine personal context, portfolio state, market conditions, and action policy.

### D. Replayable agent improvement

Existing ingredients:

- telemetry
- NodeTrace
- planner fixtures
- integration tests
- model config
- production investigations

Missing ingredient:

- one canonical run fixture format and replay CLI.

## 6. Highest-leverage foundational primitives

### Primitive 1: durable decision/run record

#### Current missing piece

A typed, versioned record connecting:

```text
request → intent → plan → evidence → action → outcome → response
```

#### Features unlocked

- replay
- debugging
- explainability
- post-trade analysis
- evaluation datasets
- prompt/model comparison
- strategy history
- user-facing “why”
- causal links between signals and trades
- multi-surface continuity
- safer future autonomy

#### Current code that benefits

- `DagService`
- `PlannerAdapterService`
- `NodeTrace`
- `PositionOrigin`
- `JournalEntry`
- `ScheduledTask`
- `TelemetryService`
- `TradeTicketsController`
- planner and composer evals

#### Migration cost

Medium. It can begin additively without rewriting the DAG.

### Primitive 2: Strategy/Goal/Workflow specification

#### Current missing piece

A structured user-owned object with:

- thesis
- conditions
- scope
- risk
- cadence
- action policy
- approval policy
- lifecycle

#### Features unlocked

- durable strategies
- saved research workflows
- scheduled strategy checks
- event-driven monitoring
- paper trading
- strategy backtests
- strategy-specific notifications
- safe autonomous actions
- strategy outcome reports

#### Current code that benefits

- `ScheduledTask`
- `SignalTrigger`
- `GhostTrigger`
- `WatchScheduleEntry`
- `PositionOrigin`
- scheduler agent
- signal systems
- trade execution
- memory/personalization

#### Migration cost

Medium-high. Start with read-only monitoring strategies.

### Primitive 3: canonical trading domain model

#### Current missing piece

Shared types and read models for:

- person
- account
- venue
- instrument
- position
- portfolio
- exposure
- decision

#### Features unlocked

- cross-venue portfolio views
- easier venue additions
- unified risk
- simpler prompts
- cleaner UI
- strategy-to-position linking
- asset-aware evidence
- fewer venue conditionals

#### Current code that benefits

- `AssetIndexReader`
- indexing
- `V3EngineClientService`
- `packages/execution`
- trade profile
- portfolio tools
- MCP
- context/basket generation

#### Migration cost

Medium-high. Avoid a rewrite; introduce boundary types and adapters.

## 7. Builder-sized opportunities

### Opportunity 1: Pear Run Ledger and Replay Lab

#### Problem

The team can inspect traces and run feature-specific evals, but cannot consistently replay a full production interaction with controlled tools and exact lineage.

#### What exists today

- `TelemetryService`
- Langfuse spans
- `NodeTrace`
- planner fixtures
- integration test suites
- model configuration
- extensive incident investigations

#### What is missing

- canonical redacted run fixture
- one replay command
- deterministic tool stubs
- model/prompt/schema version capture
- end-to-end assertions
- replayable execution tickets without real writes

#### Evidence

- `apps/agent/src/ai/telemetry/telemetry.service.ts`
- `apps/agent/src/ai/dag/types/index.ts`
- `apps/agent/test/planner.e2e-spec.ts`
- `apps/agent/test/advanced-eval.ts`
- `apps/agent/test/node-eval.ts`
- `packages/shared/scripts/kb-eval.mjs`
- planner investigation of trace `7e0824d9`

#### Why users/company care

It reduces regressions in the highest-risk product area: natural language turning into research or money-moving actions.

#### What you could build

A `RunFixture` format:

```ts
{
  runId,
  userInput,
  history,
  surface,
  authState,
  runtimeContext,
  plannerOutput,
  nodeInputs,
  toolResults,
  finalOutput,
  modelVersions,
  promptVersions,
  clock
}
```

#### Smallest useful version

- capture a redacted standard DAG run
- replay planner + DAG with mocked tool outputs
- assert selected nodes, dependencies, final action type
- CLI or Jest command

#### Full version

- Langfuse import
- production trace redaction
- fixture browser
- model/prompt A/B comparison
- deterministic clock
- venue simulator
- replay of scheduler and signal runs
- automatic regression corpus

#### Expected modules

- `apps/agent/src/ai/run-record/*`
- `apps/agent/src/ai/telemetry/*`
- `apps/agent/src/ai/dag/*`
- `apps/agent/test/*`
- possibly `packages/shared`
- CLI command under `apps/cli`

#### Dependencies

- redaction policy
- stable schemas
- tool mocking boundaries
- access to representative traces

#### Risk

Medium. The main risk is building a test framework no one uses. Keep the first version narrow and attach it to one known planner failure class.

#### Effort

2–4 weeks for a useful MVP; 1–3 months for the full lab.

#### Business upside

Fewer broken or misleading agent experiences; faster safe iteration.

#### Technical leverage

Very high.

#### Product leverage

Medium initially, high once replay powers personalized explanations and strategy history.

#### Learning value

Very high: planner, orchestration, tools, telemetry, testing, execution safety.

#### Visibility

High. Every agent engineer benefits.

#### Ownership

Likely unowned or weakly owned as a cross-cutting capability.

#### First PR

Add a versioned redacted `RunFixture` schema and export one standard DAG run into it.

#### Second PR

Add deterministic replay with mocked findings and assertions for planner/DAG behavior.

#### 30-day success

Five real production regressions can be turned into fixtures and replayed locally in under five minutes.

#### Six-month potential

Become the owner of Pear’s agent quality and replay infrastructure.

### Opportunity 2: Typed planner contract and plan compiler

#### Problem

The planner combines intent interpretation, routing, dependency generation, focused-query generation, safety policy, and surface gating in one large prompt and a narrow parser.

#### What exists today

- rich planner prompt
- `NODE_REGISTRY`
- `PlannerDecision`
- normalization and dependency repair
- extensive planner tests
- fallback models

#### What is missing

- versioned discriminated plan envelope
- explicit intent type
- structured task parameters
- safe repair of partial but intent-correct output
- planner failure classification
- capability-driven planner prompt generation

#### Evidence

- `apps/agent/src/ai/agents/planner/prompt.ts`
- `apps/agent/src/ai/agents/planner/agent.ts`
- `apps/agent/src/ai/dag/services/planner-adapter.service.ts`
- `docs/investigations/planner-trade-intent-routing.md`

#### Smallest useful version

Keep the current output shape but add:

```ts
type PlanEnvelope =
  | { kind: "clarify"; ... }
  | { kind: "answer"; ... }
  | { kind: "research"; nodes: ... }
  | { kind: "act"; nodes: ... }
  | { kind: "schedule"; ... };
```

Then:

- validate it
- preserve partial intent-correct payloads
- emit a planner failure reason
- avoid silently converting every failure to `generalChat`

#### Full version

- capability registry generates routing table
- plan compiler validates dependencies
- structured intent separate from node selection
- safe read-only recovery path
- planner versioning and offline comparison

#### Effort

2–4 weeks initially; 1–3 months for compiler/registry integration.

#### Risk

Medium-high because planner behavior is central and ownership may be sensitive.

#### First PR

Introduce `PlanEnvelope` and failure telemetry without changing successful routing behavior.

#### Second PR

Recover valid `subQueries` or partial intent from failed planner generations into a safe read-only route.

#### Success

The `7e0824d9` failure reproduces as a recoverable planner-format error rather than a silent general-chat turn.

#### Six-month potential

Pear’s planner becomes an actual validated plan compiler instead of a large prompt with post-hoc repairs.

### Opportunity 3: Strategy and workflow runtime

#### Problem

Pear can monitor, notify, research, and execute, but cannot represent a user’s ongoing strategy as a first-class object.

#### What exists today

- `ScheduledTask`
- signals and subscriptions
- ghost triggers
- watch schedules
- auto-rebalance
- execution plans
- journals
- personal interests
- background processors

#### What is missing

A durable strategy object with lifecycle and action policy.

#### Evidence

- `ScheduledTask.instruction`
- `ScheduledTask.userQuery`
- `ScheduledTask.lastResult`
- `SignalTrigger.eql`
- `GhostTrigger.statement`
- `WatchScheduleEntry.state`
- `PositionOrigin.plan`
- `apps/agent/src/ai/tools/scheduler`
- `apps/agent/src/signals`

#### Smallest useful version

Read-only strategy monitoring:

```text
Strategy
- name
- thesis
- instruments/basket
- conditions
- timeframe
- risk constraints
- monitoring cadence
- notification policy
- status
```

Each run produces a typed `StrategyObservation`.

No autonomous writes.

#### Full version

- approval policies
- paper trading
- actions
- execution-aware state transitions
- strategy history
- backtesting
- post-trade analysis
- user-specific adaptation

#### Effort

1–3 months for the initial product slice.

#### Risk

High. This could become too broad or collide with product priorities.

#### First PR

Create read-only strategy/observation models and expose them through one conversational flow.

#### Second PR

Have the scheduler evaluate a strategy into structured observations rather than only a text instruction/result pair.

#### Success

A user can create, inspect, pause, and review one strategy without Pear treating it as a frozen prompt.

#### Six-month potential

Pear becomes a lifecycle manager for execution-aware trading strategies rather than a chat agent with monitors.

### Opportunity 4: Cross-venue portfolio intelligence

#### Problem

Pear has venue/account/position reads, but lacks a unified, higher-order view of a user’s portfolio and exposure.

#### What exists today

- `get-account`
- `get-portfolio`
- `get-positions`
- trade history
- multiple trade accounts
- DTP
- asset index
- execution reconciliation
- MCP portfolio tools

#### What is missing

- canonical exposure aggregation
- asset-normalized positions
- strategy attribution
- cross-venue risk
- portfolio snapshots
- historical exposure

#### Evidence

- `packages/execution/src/domain/portfolio/read/*`
- `apps/agent/src/ai/tools/trade-execution/read/get-portfolio.tool.ts`
- `apps/agent/src/ai/services/user/trade-profile.service.ts`
- `packages/shared/src/asset-index/types.ts`

#### Smallest useful version

A deterministic read model:

```text
PortfolioSnapshot
- total equity
- available margin
- long/short exposure
- per-asset exposure
- per-venue exposure
- leverage
- open positions
- concentration
- timestamp
```

Expose it to the composer and MCP.

#### Full version

- time-series snapshots
- strategy attribution
- scenario analysis
- funding and liquidation risk
- portfolio recommendations
- cross-account rebalancing

#### Effort

2–6 weeks for MVP; 1–3 months for richer analytics.

#### Risk

Medium. It may overlap with Pear web portfolio ownership; position it as an agent/domain read model rather than a competing UI.

#### First PR

Canonicalize and aggregate current positions across supported trade accounts into one typed DTO.

#### Second PR

Persist snapshots and expose “what changed?” comparisons.

#### Success

A user with Hyperliquid and Lighter accounts receives one coherent portfolio answer without the model manually merging venue-specific prose.

#### Six-month potential

Pear becomes an execution-aware portfolio intelligence layer for all interfaces.

### Opportunity 5: Evidence and capability registry

#### Problem

The system has many tools and nodes, but no shared description of what a capability can do, what it returns, how fresh it is, or what permissions it requires.

#### What exists today

- MCP `TOOL_MANIFEST`
- `ToolDefinition`
- `defineTool`
- `NODE_REGISTRY`
- agent-specific tool builders
- tool timing and traces

#### What is missing

A shared internal capability descriptor and typed evidence output.

#### Evidence

- `apps/mcp/src/tools/manifest.ts`
- `apps/mcp/src/tools/base/tool-definition.ts`
- `apps/agent/src/ai/dag/node-registry.ts`
- `apps/agent/src/ai/tools/*`
- `apps/agent/src/ai/dag/builders/context-assembler.service.ts`

#### Smallest useful version

Define descriptors for a small set of read tools:

- name
- input
- output
- side effects
- permissions
- supported surfaces
- venue support
- freshness
- evidence type

Use them to generate planner documentation and test metadata.

#### Full version

- shared registry for agent + MCP
- dynamic tool selection
- typed `EvidenceBlock`
- generated prompt/tool manifests
- capability compatibility checks

#### Effort

2–4 weeks for a narrow scope.

#### Risk

Medium-high. There is already an MCP manifest; avoid duplicating it. Start by sharing metadata, not rewriting all tools.

#### Success

Adding one new read capability requires fewer manual edits across planner, node registry, context assembly, and tests.

### Opportunity 6: Web-native proactive event delivery

#### Problem

Web schedules exist conceptually, but the scheduler is Telegram-first.

#### Evidence

- TODOs in `apps/agent/src/scheduler/scheduler.processor.ts`
- `ScheduledTask.appType`
- Telegram delivery service
- SDK SSE protocol

#### Smallest useful version

- persistent web notification event
- SSE subscription by user/session
- delivery state
- unread notification endpoint

#### Full version

- unified notification center
- email/mobile/Discord adapters
- event preferences
- action buttons
- strategy and signal notifications

#### Effort

1–3 weeks.

#### Risk

Low technical risk, moderate product ownership risk.

#### Why it is not the primary recommendation

It is useful but not a signature project. It exposes existing backend value rather than creating the deepest new capability.

### Opportunity 7: Strategy evaluation, simulation, and paper trading

#### Problem

Pear can produce historical statistics and execute live plans, but cannot systematically evaluate a user-defined strategy through a common lifecycle.

#### What exists today

- quant statistics
- backtest knowledge and tools
- basket stats
- execution plan models
- integration tests
- paper-shadow artifacts
- real venue test plans

#### Missing

- strategy specification
- event replay
- realistic fills
- fees/slippage
- venue constraints
- paper positions
- result attribution

#### Effort

1–3 months.

#### Risk

High. A naive backtester would produce misleading confidence and could become commercially useless.

#### Recommendation

Build only after the strategy object and run ledger exist.

## 8. Opportunity scoring

Scores are 1–5. Higher is better except for the two risk columns, where higher means more risk.

| Opportunity | Product | Business | Tech leverage | Arch leverage | User | Learning | Ownership | Visibility | Feasibility | Reuse | Strategic | Wasted effort risk | Ownership conflict risk |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Run ledger + replay lab | 4 | 4 | 5 | 5 | 3 | 5 | 5 | 4 | 4 | 5 | 5 | 2 | 2 |
| Typed planner contract | 3 | 3 | 5 | 5 | 3 | 5 | 4 | 4 | 4 | 5 | 5 | 2 | 3 |
| Strategy/workflow runtime | 5 | 5 | 5 | 5 | 5 | 5 | 5 | 5 | 3 | 5 | 5 | 3 | 4 |
| Portfolio intelligence | 5 | 5 | 4 | 4 | 5 | 4 | 4 | 4 | 4 | 5 | 5 | 2 | 4 |
| Capability/evidence registry | 3 | 3 | 5 | 5 | 4 | 4 | 4 | 3 | 3 | 5 | 5 | 3 | 4 |
| Web proactive delivery | 3 | 3 | 2 | 2 | 4 | 2 | 2 | 2 | 5 | 3 | 3 | 2 | 4 |
| Strategy simulation/paper trading | 5 | 5 | 4 | 5 | 5 | 5 | 4 | 4 | 2 | 5 | 5 | 4 | 4 |

Tradeoff summary:

- The highest raw product upside is Strategy/Workflow.
- The highest near-term leverage and feasibility is Run Ledger/Replay.
- The best user-facing wedge is Portfolio Intelligence.
- The best AI architecture wedge is Typed Planner Contract.
- The riskiest idea is full Backtesting/Paper Trading before the underlying strategy and decision models exist.

## 9. Category-by-category assessment

### Agent architecture

#### Exists

- standard planner → DAG → composer
- advanced agent loop
- specialist agents
- node registry
- dependency execution
- surface gates
- tool loops

#### Missing

- typed intermediate representation
- universal capability registry
- consistent semantics between standard and advanced paths
- durable workflow state
- model-independent action semantics

#### Build now?

Yes, but narrowly: typed plan envelope and run record.

Do not rewrite the DAG.

### Memory and state

#### Exists

- sessions
- recent history
- recall
- memory notes
- user preferences
- journals
- DTP
- interests
- position origins

#### Missing

- queryable trader state
- provenance and confidence
- unified decision history
- strategy state
- relationship between beliefs, actions, and outcomes

#### Build now?

Yes, as a projection on top of existing state. Do not start with a generic vector-memory rewrite.

### Tool system

#### Exists

- many well-tested tools
- strong MCP manifest
- execution tool schemas
- tool timing
- output contracts

#### Missing

- shared registry
- typed evidence
- dynamic capability compatibility
- freshness and cost metadata
- common agent/MCP representation

#### Build now?

A narrow registry/evidence slice is worthwhile. A full tool platform is not yet.

### Autonomous behavior

#### Exists

- scheduled monitors
- signal delivery
- watch scheduling
- ghost triggers
- background processors
- autonomous runs are explicitly read-only

#### Missing

- strategy lifecycle
- event-to-action policy
- approval state
- durable action plans
- safe deferred execution model

#### Build now?

Build the strategy/observation layer. Do not build autonomous money movement yet.

### Trading intelligence

#### Exists

- pair and basket analytics
- research
- sentiment
- tokenomics
- Coinglass
- signals
- portfolio reads
- journals
- user trade profile

#### Missing

- thesis lifecycle
- portfolio-level reasoning
- decision attribution
- personal strategy analytics
- research-to-execution continuity
- systematic post-trade learning

#### Build now?

Yes. This is the largest product opportunity.

### Execution architecture

#### Exists

- mature shared execution core
- plan/execute
- venue/account routing
- order families
- reconciliation
- venue verdicts

#### Missing

- strategy/decision linkage
- portfolio read model
- event-sourced execution history at Pear product layer
- structured filled quantity/outcome contract in some venue cases

#### Build now?

Build above execution, not inside the core unless a specific domain contract is missing.

### Multi-venue architecture

#### Exists

- Hyperliquid and Lighter support
- unified asset index
- venue-aware account selection
- shared execution package
- venue-specific constraints

#### Missing

- complete canonical instrument/account/portfolio domain model
- capability-driven venue differences
- cross-venue portfolio projections

#### Build now?

Introduce shared boundary types incrementally. Do not abstract every venue condition prematurely.

### Personalization

#### Exists

- onboarding persona
- memory
- journals
- deterministic trade profile
- interests
- X data
- signal behavior

#### Missing

- unified personal model
- explicit user goals
- strategy preferences attached to durable strategies
- belief/outcome learning

#### Build now?

Yes, but through the decision/strategy layer.

### Evals

#### Exists

- broad feature-specific evals
- planner holdouts
- node tests
- composer evals
- integration matrix
- KB evals
- paper-shadow artifacts

#### Missing

- canonical run fixture
- end-to-end replay
- version lineage
- cross-subsystem quality score
- product metrics tied to agent decisions
- deterministic venue simulator

#### Build now?

Yes. This is one of the clearest team-velocity projects.

### Model layer

#### Exists

- per-node model config
- OpenRouter/Cerebras
- fallback models
- provider routing
- timeouts
- response caching
- reasoning settings

#### Missing

- unified model policy and quality registry
- model/prompt lineage attached to user outcomes
- automatic model selection based on task and cost
- reliable cross-model evaluation

#### Build now?

Not as a provider abstraction rewrite. Add lineage and evaluation support through the run ledger.

### Context architecture

#### Exists

- recent session history
- memory blocks
- user profile
- journals
- node dependency context
- KB retrieval
- runtime context

#### Missing

- relevance-ranked context
- typed context blocks
- provenance
- context budgets by semantic value
- durable cross-run references
- context compaction beyond fact memory

#### Build now?

Typed evidence and run records first.

### Data architecture

#### Exists

- agent/context/quant DBs
- Redis indexes
- asset index
- market feeds
- ingestion pipelines
- basket generation

#### Missing

- canonical domain identity used consistently at all boundaries
- portfolio snapshot history
- unified market/evidence freshness metadata
- explicit data quality contracts

#### Build now?

Canonical domain types and portfolio read model.

### Developer experience

#### Exists

- many tests
- integration suite
- telemetry
- good package boundary in execution
- MCP manifest tests

#### Missing

- local replay
- production fixture extraction
- deterministic mocks
- one-click new-agent harness
- unified trace-to-test workflow

#### Build now?

Yes. Highest internal leverage.

### Replay/simulation

#### Exists

- execution protocol tests
- integration tests
- paper-shadow outputs
- unit-level mocks

#### Missing

- full system replay
- deterministic clock
- market/venue simulator
- strategy replay
- action outcome simulation

#### Build now?

Replay now. Simulation later.

### API/platform potential

#### Exists

- chat API
- SDK
- SSE
- MCP service
- structured tool contracts

#### Missing

- public run/decision/strategy resources
- durable capability APIs
- workflow subscriptions
- event streams
- reusable agent execution primitives

#### Build now?

Start with internal records. Productize later.

### User experience

#### Exists

- chat
- tickets
- charts
- PnL cards
- MCP widgets
- signals
- Telegram notifications

#### Missing

- strategy dashboard
- decision history
- thesis monitoring
- portfolio reasoning
- “why” views
- cross-venue exposure views
- durable workflow controls

#### Build now?

A thin UX on top of a new primitive, not a UI-first project.

## 10. Differentiated product opportunities

Generic AI assistants can:

- summarize news
- answer market questions
- generate strategy ideas
- write research reports

Pear can do more because it has proximity to:

- user accounts
- actual positions
- actual venues
- actual fills
- actual PnL
- user conversations
- execution plans
- signal history
- trade journals
- personal behavior

The differentiated opportunity is:

> an execution-aware, persistent trading intelligence system that understands the user’s decisions over time.

Specific product capabilities:

- “Show me every open position and the thesis that created it.”
- “Which of my trades contradicted my own stated rules?”
- “Tell me what changed since I opened this basket.”
- “Monitor this thesis, not just this ticker.”
- “If the thesis breaks, explain exactly why.”
- “Compare current portfolio exposure with my declared risk profile.”
- “Turn this conversation into a paper strategy.”
- “Show me how this strategy would have behaved historically.”
- “Ask for approval only when a durable strategy reaches an action condition.”
- “After a trade closes, explain whether the original thesis was right, wrong, or simply invalidated by an external event.”

That is difficult for generic ChatGPT because Pear owns the execution and personal history context.

## 11. Five best ownership wedges

### 1. Agent run/replay infrastructure

#### Company need

Every agent engineer needs confidence that changes improve Pear instead of shifting failures around.

#### Your existing context

- incident investigation
- Langfuse trace analysis
- planner routing investigation
- evals and model comparisons
- deployment and production context

#### Why it is not merely operations

The output is a durable engineering platform: replayable agent behavior and regression prevention.

#### Six-month trajectory

- Month 1: run fixture schema, redaction, one replay path
- Months 2–3: production trace importer, planner/DAG assertions, model comparison
- Months 4–6: execution-safe simulator, strategy replay, team-facing quality dashboard

#### What you become known for

The engineer who makes agent behavior reproducible and improvable.

### 2. Typed decision and planner runtime

#### Company need

Reduce prompt-driven routing fragility and make action semantics explicit.

#### Your existing context

- planner investigation
- execution semantics
- trade tickets
- Lighter/venue reasoning
- prior cross-cutting fixes

#### Six-month trajectory

- Month 1: `PlanEnvelope`, parser recovery, structured failure telemetry
- Months 2–3: planner compiler and capability metadata
- Months 4–6: plan/run persistence, explainability, strategy integration

#### What you become known for

The engineer who turns Pear’s agent from prompt choreography into a validated decision system.

### 3. Cross-venue portfolio intelligence

#### Company need

Pear’s execution engine supports multiple venues, but users need one mental model.

#### Your existing context

- Lighter integration
- asset index
- venue-aware routing
- execution and account semantics
- market/basket systems

#### Six-month trajectory

- Month 1: canonical position/account DTO
- Months 2–3: portfolio snapshot and exposure aggregation
- Months 4–6: strategy attribution, risk views, agent and web surfaces

#### What you become known for

The engineer who makes multi-venue trading feel like one coherent portfolio.

### 4. Strategy lifecycle and proactive workflows

#### Company need

Pear has monitors and alerts, but not user-owned strategies.

#### Your existing context

- background systems
- signals
- scheduled tasks
- ghost/watch systems
- interest and personalization pipelines

#### Six-month trajectory

- Month 1: structured read-only strategy object
- Months 2–3: observation runs, lifecycle, notifications
- Months 4–6: paper mode, approval policies, execution linkage

#### What you become known for

The engineer who turns Pear from conversational trading assistance into persistent strategy infrastructure.

### 5. Context/evidence/provenance system

#### Company need

Pear has rich sources but often collapses them into prose before composition.

#### Your existing context

- context ingestion
- basket research
- sentiment
- signal evidence
- evals
- model grounding work

#### Six-month trajectory

- Month 1: typed `EvidenceBlock`
- Months 2–3: provenance and freshness propagation
- Months 4–6: source-aware decision explanations and strategy monitoring

#### What you become known for

The engineer who makes Pear’s intelligence grounded, inspectable, and reusable.

## 12. Signature project candidates

### Candidate A: Pear Decision Ledger and Replay Runtime

#### Current state

Reasoning, evidence, execution, and outcomes are spread across chat records, traces, JSON blobs, and background tables.

#### Vision

Every meaningful agent run becomes a durable, inspectable, replayable decision record.

```text
request
→ interpreted intent
→ validated plan
→ evidence
→ proposed action
→ approval
→ execution
→ outcome
→ explanation
```

#### MVP

- versioned `RunEnvelope`
- planner output
- node status and traces
- typed action summary
- tool/result references
- model/prompt versions
- redacted replay fixture
- read-only replay CLI

#### Architecture

- append-only run metadata
- separate evidence/action/outcome records or JSON envelopes initially
- links from `PositionOrigin`, `JournalEntry`, `ScheduledTask`
- no DAG rewrite
- no event-sourcing religion
- retain existing prose rendering for compatibility

#### Why it matters

It creates a common substrate for:

- debugging
- evals
- explanations
- strategy history
- post-trade learning
- autonomous safety
- product analytics

#### Risks

- building a telemetry mirror nobody consumes
- privacy/redaction complexity
- storage growth
- unclear ownership between agent and platform teams

#### First three PRs

1. Add `RunEnvelope` schema and emit it for standard DAG runs.
2. Add redacted fixture export and deterministic replay for planner/DAG with mocked tools.
3. Link execution tickets and `PositionOrigin` to `runId`; show a basic “why this happened” read.

#### Six-month end state

Pear has a durable decision graph that supports replay, post-trade review, strategy lifecycle, and eval lineage.

### Candidate B: Pear Strategy Runtime

#### Current state

Monitoring and alerts exist as separate features, mostly represented by strings or trigger-specific rows.

#### Vision

A strategy is a first-class object with thesis, conditions, evidence, risk, lifecycle, and action policy.

#### MVP

- create/read/pause/resume a read-only strategy
- run scheduled observations
- notify on meaningful changes
- persist structured observations

#### Architecture

- Strategy
- StrategyVersion
- StrategyObservation
- StrategyEvent
- NotificationPolicy
- optional ActionApproval

#### Why it matters

This is the strongest direct product expansion.

#### Risks

- too broad
- overlap with existing signal/watch owners
- premature autonomous execution
- difficult product definition

#### First three PRs

1. Strategy and observation schema, no writes.
2. Scheduler adapter that evaluates structured strategy observations.
3. Chat flow that turns a conversation into a strategy draft and lets users inspect/edit it.

#### Six-month end state

Users can maintain durable strategies and Pear can monitor, explain, and eventually act on them safely.

### Candidate C: Unified Portfolio Intelligence Layer

#### Current state

Pear can read accounts, positions, balances, history, and journals, but each surface sees a different slice.

#### Vision

A canonical portfolio model used by agents, MCP, web, and background systems.

#### MVP

- normalize accounts/positions across venues
- aggregate exposure
- persist timestamped snapshots
- expose through agent and MCP

#### Architecture

- `PersonRef`
- `TradeAccountRef`
- `VenueRef`
- `InstrumentRef`
- `PositionRef`
- `PortfolioSnapshot`
- `ExposureView`

#### Why it matters

It makes multi-venue Pear coherent and creates a foundation for risk and strategy analytics.

#### Risks

- overlap with web portfolio teams
- domain ambiguity around PnL/cost basis
- cross-venue asset identity complexity

#### First three PRs

1. Canonical cross-account portfolio DTO.
2. Deterministic aggregation and snapshot persistence.
3. Agent/MCP “what changed in my portfolio?” capability.

#### Six-month end state

Pear understands the user’s portfolio as a whole rather than as separate execution-account reads.

### Strongest candidate

Candidate A is the best first signature project.

Candidate B is the strongest product destination.

The recommended framing is:

> Build the Decision Ledger and Replay Runtime first, then use it to make Strategies first-class.

This avoids starting with an oversized strategy platform while still moving toward a major product capability.

## 13. Build now / later / probably do not build

### Build now

#### 1. Run ledger and replay

Real evidence, strong internal leverage, bounded MVP.

#### 2. Typed planner contract

A known production failure already proves the need.

#### 3. Portfolio intelligence read model

Execution and multi-venue foundations already exist.

#### 4. Structured strategy observations

Begin read-only; do not start with autonomous writes.

#### 5. Evidence/provenance envelope

Improves agents, evals, explanations, and strategies simultaneously.

### Build later

#### 1. Full strategy automation

Prerequisites: run ledger, strategy model, approval policy, portfolio model.

#### 2. Paper trading and backtesting

Prerequisites: structured strategy, deterministic data, venue simulator, outcome model.

#### 3. Dynamic tool discovery

Useful once capability metadata and evidence contracts are stable.

#### 4. Full web notification center

Worth doing, but lower architectural leverage.

#### 5. Public Agent Pear platform APIs

First make the internal primitives durable.

### Probably do not build

#### 1. Rewrite the DAG

The DAG mechanics are not the main problem.

#### 2. Generic multi-agent framework

Pear already has a working specialist architecture. A framework rewrite would consume time without creating differentiated product value.

#### 3. Generic vector-memory platform

Pear’s main missing memory is structured decision/strategy state, not “more embeddings.”

#### 4. New venue abstraction from scratch

Execution and asset indexing already contain substantial venue abstractions.

#### 5. Autonomous trading before strategy/action policy is explicit

This would create safety and product ambiguity.

#### 6. More deployment/runner infrastructure as your primary direction

Useful operationally, but it compounds your existing profile rather than expanding your builder identity.

#### 7. Pure Telegram command cleanup

The RFC about reducing 27 DM commands is sensible, but it is evidence/product instrumentation work, not a major six-month builder wedge.

## 14. Ten most important things Agent Pear currently lacks

### 1. A durable decision/run record

Why: Pear cannot consistently connect intent, evidence, action, and outcome.

Evidence: no corresponding Prisma model; state is distributed across `ChatMessage`, `PositionOrigin`, `JournalEntry`, traces, and tickets.

Severity: Critical architectural gap.

Type: Architectural change and internal platform.

### 2. A validated planner intermediate representation

Why: planner output can be intent-correct but structurally discarded.

Evidence: `PlannerAdapterService.extractStructuredResponse()` and trace `7e0824d9`.

Severity: High.

Type: Architectural change.

### 3. Typed cross-node evidence

Why: the DAG mostly passes strings, losing provenance and structure.

Evidence: `ContextAssemblerService`, `DependencyContextService`, `TextResult`, `AggregatedFindings`.

Severity: High.

Type: Architectural change.

### 4. A first-class strategy/goal object

Why: monitoring exists, but durable user intent does not.

Evidence: `ScheduledTask`, `SignalTrigger`, `GhostTrigger`, and `WatchScheduleEntry` are separate primitives.

Severity: Critical product gap.

Type: Product feature plus architecture.

### 5. Unified portfolio intelligence

Why: Pear has execution data but not a single cross-venue portfolio model.

Evidence: account/position tools and DTP are separate read paths.

Severity: High.

Type: Product feature plus domain architecture.

### 6. End-to-end replay

Why: the team has tests and traces but not one reproducible agent lab.

Evidence: fragmented planner/node/composer/integration evals.

Severity: High developer-experience gap.

Type: Internal tool.

### 7. Shared capability registry

Why: adding an agent/tool requires edits across many unrelated files.

Evidence: `NODE_REGISTRY`, specialist tool builders, MCP `TOOL_MANIFEST`, context switches.

Severity: Medium-high.

Type: Architectural change.

### 8. Unified notification/event model

Why: schedules, signals, ghosts, and web/TG delivery are separate.

Evidence: multiple background tables and web delivery TODOs.

Severity: Medium-high.

Type: Architectural change.

### 9. Decision-to-outcome learning

Why: journals and profiles exist, but Pear cannot reliably evaluate whether a thesis worked.

Evidence: `PositionOrigin.intent`, `JournalEntry`, `UserTradeProfile`, but no common causal record.

Severity: High product opportunity.

Type: Product capability.

### 10. A canonical domain identity layer

Why: venue/account/instrument identities still require specialized translation.

Evidence: `AssetIndexEntry`, quant asset IDs, venue symbols, account IDs, and V3 client resolution.

Severity: Medium-high scalability issue.

Type: Architectural change.

## 15. The biggest gaps by category

### Biggest architectural debt

Stringly typed boundaries.

The repository has strong local schemas, but the central agent flow frequently turns structured data into prose and then asks another model to reconstruct meaning.

### Biggest product capability gap

Persistent, execution-aware strategy intelligence.

Pear can research, trade, monitor, and remember facts, but cannot yet maintain a coherent lifecycle for a user’s strategy.

### Biggest developer-experience gap

No unified trace-to-replay-to-regression workflow.

The ingredients exist; the connecting system does not.

### Biggest AI/agent architecture gap

The planner is an untyped compiler implemented as a large prompt plus parser heuristics.

The next step is not “make the prompt longer.” It is to introduce a validated plan representation.

### Biggest trading-domain gap

No durable strategy/portfolio/outcome model above individual positions and orders.

### Biggest long-term scalability problem

Every new capability risks requiring edits across:

- planner prompt
- node registry
- orchestrator
- executor
- context assembler
- composer prompt
- surface policy
- tools
- telemetry
- tests
- notifications
- persistence

### Most underrated opportunity

Unifying the existing personalized and proactive systems:

- memory
- trade profile
- journals
- interests
- signals
- ghost triggers
- watches
- schedules
- position origins

Pear already has perhaps 60–80% of the ingredients for a persistent personalized trading intelligence layer.

### Opportunity everyone is likely overlooking

The connection between context-generation work and execution outcomes.

Pear’s basket/research systems produce hypotheses. Its execution system produces real outcomes. The missing product is the causal bridge:

```text
research thesis → user decision → executed position → outcome → learned preference
```

### Thing that looks important but you should not spend time on

Another large deployment or reliability subsystem.

You should continue contributing operationally when necessary, but making that your primary ownership path would keep you in a dependency-heavy operator role instead of moving you toward durable product infrastructure.

## 16. The five best things you personally could build

1. Pear Run Ledger and Replay Lab
2. Typed Planner/Decision Runtime
3. Cross-Venue Portfolio Intelligence
4. Strategy Observation and Lifecycle Substrate
5. Evidence/Provenance Layer for Research and Execution

The recommended sequence is:

```text
Run ledger
  → typed planner contract
  → evidence/provenance
  → portfolio intelligence
  → strategy runtime
```

This gives you progressively larger ownership without requiring a rewrite.

## 17. Recommended primary builder identity

> Prakhar should become the engineer who builds Pear’s persistent decision and strategy infrastructure: the systems that turn conversation, market evidence, execution, and trader outcomes into durable, replayable, personalized intelligence.

That is specific enough to be meaningful and broad enough to grow with the company.

## 18. The first thing you should propose/build

### Exact problem

Pear does not have a durable, structured record of:

- what the user asked
- what Pear inferred
- what plan Pear selected
- what evidence it used
- what tools ran
- what action was proposed or executed
- what actually happened
- which model and prompt versions produced the result

### Relevant code

- `apps/agent/src/ai/dag/dag.service.ts`
- `apps/agent/src/ai/dag/services/planner-adapter.service.ts`
- `apps/agent/src/ai/dag/executor/node-executor.service.ts`
- `apps/agent/src/ai/dag/types/index.ts`
- `apps/agent/src/ai/telemetry/telemetry.service.ts`
- `packages/database/prisma/agent.schema.prisma`
- `PositionOrigin`
- `JournalEntry`
- `ScheduledTask`
- `apps/agent/src/chat/trade-tickets.controller.ts`

### Proposed design

Introduce a versioned `AgentRunEnvelope`:

```ts
type AgentRunEnvelope = {
  version: 1;
  runId: string;
  sessionId: string;
  userId: string;
  surface: string;
  input: {
    userMessage: string;
    historyRef?: string;
    runtimeContext?: unknown;
  };
  plan: {
    kind: string;
    raw?: unknown;
    normalized?: unknown;
    plannerModel?: string;
    plannerVersion?: string;
  };
  nodes: Array<{
    node: string;
    status: "succeeded" | "failed" | "gated" | "skipped";
    input?: unknown;
    output?: unknown;
    evidenceRefs?: string[];
    trace?: unknown;
  }>;
  actions: Array<{
    kind: string;
    status: string;
    ticketId?: string;
    executionRef?: string;
  }>;
  outcome?: unknown;
  response?: {
    finalText?: string;
    sources?: unknown;
    suggestedAction?: unknown;
  };
  versions: {
    prompt?: string;
    schema?: string;
    model?: string;
  };
  timestamps: {
    startedAt: string;
    completedAt?: string;
  };
};
```

Initially:

- preserve current string-based composer behavior
- emit the envelope alongside it
- do not rewrite the DAG
- do not persist every raw model token
- redact sensitive fields
- make action/execution links optional

### Smallest useful scope

One standard DAG run:

- planner output
- active nodes
- node status
- tool traces
- final response metadata
- model IDs
- run ID

Persist or export it as a replay fixture.

### First PR

Add the schema, run ID propagation, and envelope creation at the standard DAG boundary.

Success criterion:

- a real production-like turn produces a validated envelope
- no product behavior changes
- planner/node/composer outcomes are inspectable without manually navigating several trace layers

### Second PR

Add deterministic replay:

```text
pear agent replay <fixture>
```

with mocked tool results and assertions for:

- selected nodes
- dependencies
- gated actions
- final response shape
- no live writes

Success criterion:

- reproduce the `7e0824d9` planner-format failure locally
- demonstrate that a parser/recovery change fixes it without touching live execution

### After that

Link:

- `TradeTicket`
- `PositionOrigin`
- `JournalEntry`
- `ScheduledTask` runs
- strategy observations

to `runId`.

That is where the internal tool becomes a product primitive.

## 19. Your next 30 days

1. Propose the Run Ledger/Replay project as a cross-cutting builder initiative, explicitly scoped away from deployment ownership.

2. Implement `AgentRunEnvelope` for the standard DAG path, preserving existing product behavior.

3. Convert the planner investigation around `7e0824d9` into a permanent replay fixture and regression test.

4. Add a minimal typed `EvidenceBlock` for one domain—pair research or trade execution—without migrating every node.

5. Prototype a read-only “Why did Pear recommend or execute this?” view using the new run record and existing `PositionOrigin`/`JournalEntry` data.

The important strategic choice is to begin with a durable internal primitive that immediately improves engineering velocity, then let it grow into the product layer Pear currently lacks.

## Related vault notes

- [[My Work at Pear Protocol]]
- [[Agent Pear Revamp]]
- [[Engineering Lessons]]
- [[Features Owned]]
- [[LLM Observability & Evaluation]]
- [[Structured Outputs & Validation]]
- [[Multi-Adapter & Multi-Venue Consistency]]
- [[Trading Execution Safety]]
