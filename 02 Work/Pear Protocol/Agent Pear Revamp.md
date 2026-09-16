# Agent Pear Revamp

Status: open — research phase done, verdict reached, not yet brought back to senior for scope alignment.
Filed: 2026-08-31, from [[../../99 Inbox/Inbox|Inbox]]. Original conversation is undated in the source (Discord), so the filing date is not the same as when the conversation happened.

## What this is

Team is planning to redesign Agent Pear (AP). A colleague shared research on **Oracle** (`oracle.demi.la`, product name "DEMI") — a competing all-in-one multichain agent — as reference material, specifically asking about the **agent harness side**: how Oracle structures execution, signing, and its agent architecture, and what of that is worth bringing into the AP redesign.

My own framing at the time: "i want powerful agent harness system for AP" — "harness" here means the execution/agent-runtime layer, not test harnesses (the vault's existing "harness" hits are all test/replay harnesses — [[My Work at Pear Protocol]] — unrelated to this).

## Source material (preserved verbatim from Inbox)

Colleague's message, evaluating where Oracle fits relative to Pear:

> Read the docs end-to-end. Pear = pair-trade execution layer on Hyperliquid (plus GMX/Vertex/SYMM): splits a basket into long/short legs, executes both same-block via HL builder codes, gives ratio-level TP/SL/TWAP, synthetic basket PnL accounting, and a stat-arb layer (correlation/cointegration/z-score/beta). REST API + TS SDK (@pear-protocol/hyperliquid), trade-ideas endpoint (GET /markets/v2 — active pairs, watchlists, AI picks). Fees: 0.06% open + 0.06% close on both legs, on top of HL taker fees.
>
> Where Oracle fits — four honest options:
>
> 1. Pair-trade lane in Oracle, executed directly on HL (recommended). The hard part of Pear — same-timestamp long+short legs on HL with an agent wallet — is capability Oracle already has (armed-local HL signer, delta-neutral tooling, TWAP sim). What we lack is the pair grammar: ratio charting, ratio TP/SL, beta-weighted sizing, basket PnL. Building that as an oracle pair surface gives DEMI Pear's product without Pear's 12bps round-trip fee — our admin lane is zero-fee by law. Their Agent-Pear stats (z-score entry at ±2, rolling beta hedge ratio) plug straight into the event-study/prior machinery we've been building all week: fund-flip priors → pair-spread priors is the same math.
>
> 2. Pear as a routed venue (prepare-only integration). Add Pear's API as a venue surface: Oracle quotes the same pair both ways — direct-HL legs vs Pear execution — and ranks net of Pear's 6bps+6bps. Honest read: direct-HL wins on cost every time; Pear only wins if their synced execution measurably beats our two-order dispatch on slippage. That's testable before building.
>
> 3. Signal cross-feed. Their ai-picks stat-arb baskets are a free read (public endpoint). Worth a cron that pulls them, runs them through our own prior/backtest gate, and pings only survivors — same law as everything else: no ping without a measured edge. Also a calibration source for the fingerprinter, like Agent Arcade was.
>
> 4. Custody red flag if we ever use Pear to execute: their model stores the user's HL agent-wallet private key "encrypted within the Pear Protocol system." That's a third party holding a trading key — directly against our self-custody law. Oracle's own HL agent wallet is local. So Pear is fine as a data/routing comparison, but executing through them means trusting their key custody; I wouldn't arm that for your wallet.

Note: this message is written from *Oracle/DEMI's* perspective, evaluating whether/how to integrate with Pear — not written about AP. It's being used here as competitive reference material for AP's own harness design, not as a plan Pear itself is executing.

My response at the time: "ahh okay. interesting. ill run the cli as well. check how this works 🫡"

## What needs studying

- Oracle's docs end-to-end (not yet done)
- Oracle's CLI — run it, see how the harness actually behaves hands-on
- Specifically the harness/execution side: armed-local signer model, delta-neutral tooling, TWAP simulation — what's structurally reusable as a pattern for AP vs. Oracle-specific
- Their custody model (local agent wallet, no third-party key storage) as a contrast point — worth checking how AP currently handles signing/custody before assuming anything needs to change

## Findings — AP-vs-Oracle fit assessment (codebase-grounded)

Ran a codebase-grounded comparison against the actual `packages/execution`, `apps/agent`, and `apps/mcp` source (not just Oracle's docs). Full file-level citations kept in the source chat, condensed here.

**Bottom line: AP should not adopt Oracle wholesale.** AP already implements the equivalent of most of Oracle's valuable safety ideas, through its own plan-token, ticket, execution-status, and reconciliation protocols — just expressed as feature-specific mechanisms rather than one generic abstraction.

Mechanism-by-mechanism:
- **Plane separation** — AP's `packages/execution` is already framework-free and mechanically forbidden from importing app adapters/HTTP/MCP/env access, enforced by `architecture-boundary.test.ts`. Functionally equivalent to Oracle's data/policy/exec split. Credentials are handled at the composition root (`v3-engine-client.service.ts`), never in the execution/model domain.
- **Bound grants** — AP's plan-token protocol (`plan-token-protocol.ts`, `confirm-protocol.ts`) is a strong "one approved action" primitive (dry-run → stored plan → single-use token → exact replay on execute), but it's narrower than Oracle's grants: it approves one already-created plan, not a durable allowance ("$5k/hr on BTC/ETH"). AP's checks (margin recheck, reduce-only validation, price drift) are feature-specific by design and shouldn't be collapsed into one generic policy language — different trade types have materially different invariants.
- **Receipts or it didn't happen** — mostly already covered via `venueStatusOf`, execution polling, reconciliation (`reconcile-after-execution.ts`), and preserving unresolved states instead of forcing them into pass/fail. AP trades positions on execution engines, not raw chain txs, so tx-hash/balance-delta isn't the right receipt shape anyway — position/order/fill state is.
- **Prepare-before-execute** — already the normal flow (`plan → dry-run → confirm → execute stored plan`, both agent-side and MCP-side). Gap is real but narrow: AP doesn't fully simulate fills/slippage/matching, which is a normal limitation, not a missing architecture.
- **Model routing + MCP/agent-loop separation** — both already exist (`models.ts` per-lane routing; `dag.service.ts` agent loop vs `request-server.ts` MCP adapter, sharing the same execution package so they can't drift on safety logic).

**The one real gap: autonomous automation.** Scheduled/autonomous runs are currently read-only by design (`writesAllowed = writeEnabled && !autonomous` in `agent.ts`) — AP avoids the "unattended execution" problem rather than solving it. This is where Oracle's bound-grant idea actually applies.

**Recommendation (if revamp scope includes automation):**
1. Prototype a **bounded execution policy** for future pre-authorized/autonomous trading — reusing the existing plan-token/ticket machinery as the artifact, with a new executor-side policy check (account, caller, allowed plan families/symbols, max notional/leverage, order styles, expiration, rate/budget limits) enforced by the scheduler/executor, never the model, fail-closed, audited.
2. Optional secondary cleanup: a unified receipt/audit envelope consolidating the venue verdict, execution IDs, statuses, reconciliation result, and unresolved-state warnings that already exist but aren't standardized into one shape.

**Not worth transferring:** multichain wallet/signing, NFT/meme-sniper features, self-hosted signer/key-vault model (AP's agent never receives user keys — handled at the execution-engine integration layer), generic grants for the already-well-served interactive-confirm case, Oracle's balance-delta receipt semantics, their license model.

**Unrelated finding surfaced during this pass, worth its own follow-up:** the execution README notes instrument-cache age exists but isn't wired into a hard gate — flagged as a more immediate, concrete safety improvement than anything from the Oracle comparison. Not filed as its own note yet — separate task if you want it tracked.

## Current direction: use Oracle as the harness, not as the whole AP replacement

The useful shape is:

```text
Oracle harness
→ AP pair-trading tools and DAG
→ Pear V3 backend
→ HL / Lighter
```

"use Oracle as the brain" means Oracle can own the outer agent runtime: sessions, memory, skills, model selection, specialist lanes, scheduling and lifecycle. AP should still own the pair-trading grammar, DAG execution path, confirmation flow, tool contracts and venue-specific behavior. Pear V3 remains the backend for users, accounts, auth and execution.

This keeps the DAG as the inner execution system. The harness decides what needs to happen and which AP capability to call. The DAG decides the concrete order of analysis, parallel work, validation, charts and execution. There is no reason to replace a DAG that is already balanced across speed, intelligence and execution.

The main benefit is modularity. Telegram, web and future surfaces can call the same AP-facing interface while the harness, model or surface changes behind it. Models can be swapped at the harness routing layer without changing the pair tools or execution backend.

The main risk is creating two agent loops. The boundary needs to be explicit: Oracle owns conversation and high-level orchestration; AP owns deterministic task execution. Oracle should not independently recreate trade planning or bypass AP's confirmation and receipt rules.

## Update — 2026-09-01 — operational integration and VPS study

The intended harness was clarified: Oracle is the agent harness/control plane, not the existing AP integration-test harness. The working shape remains:

```text
Oracle harness → AP pair-trading tools/DAG → Pear V3 backend → Hyperliquid
```

The current operational assignment is to inventory AP's existing processes and background workers, then classify which could safely move to the Hetzner `ap-tg-fallback` VPS to create room for the Oracle integration. Nothing has been approved for movement yet. The first deliverable is a read-only process/deployment map; the senior will decide what moves.

SSH access to the Hetzner server was successfully established as Linux user `prakh`. The server is Ubuntu 24.04 and runs Docker/containerd. A Node process running `apps/tg-fallback/dist/main.js` was visible on the host; no custom application systemd unit was visible in the inspected service list. The host snapshot was healthy at inspection time (low load, about 8% memory use, about 31% root-disk use). The user is not in the Docker group, so `docker ps` is denied; Docker inspection still needs senior-approved access. These are observations from one point in time, not a complete deployment inventory.

The expected next study is to map each AP service/consumer by start mechanism, queue or cron ownership, dependencies, state, side effects, crash/retry behavior, and migration risk. In particular, `ap-worker` and any execution/realtime consumers must not be moved or duplicated casually. The fallback host must also remain reliable when it is needed for Telegram failover.

The operational handoff is tracked in [[Harness Integration/Step 1 — AP Background Processes and Hetzner VPS Inventory]].

The coding-tool installation task was also clarified as separate from the architecture study: install the requested Cloud/Claude Code tool on the approved target, while the senior handles authentication. The exact product name, target machine, version, and installation command still need confirmation before running an installer.

## Oracle cost and ownership

Oracle is being considered as an open-source or self-hosted harness, not as a venue that automatically takes a percentage of every Pear trade. The docs and CLI setup did not establish a mandatory Oracle trading fee. Actual costs still include model calls, embeddings, APIs, VPS or other hosting, monitoring and engineering maintenance. A BUSL license question remains if Oracle is embedded in a competing hosted product, so this needs a direct licensing check before committing to it.

The Oracle CLI was attempted with Codex OAuth. A later PowerShell session could not resolve `oracle` or the expected npm-global `oracle.cmd`, so the installation state was not confirmed. No trading wallet or credentials were added and no funds were at risk.

## Open questions

- Whether "revamp" scope actually includes autonomous/unattended execution — that's the only place this comparison found a real gap. Still not clarified with the senior.
- Whether the senior agrees with the "don't adopt wholesale" verdict, or was expecting a bigger architectural swing.
- Whether Oracle will be used as a thin outer harness or will be allowed to own any AP planning logic.
- Whether the Oracle BUSL terms permit the planned hosted AP product and whether the team's deployment model changes that answer.

## Next action

Bring back the narrower recommendation: use Oracle as an outer harness if its boundaries are explicit, keep the DAG and AP execution layer intact, and prototype bounded execution policy only if autonomous execution is in scope.
