---
type: investigation
area: pear
status: diagnosed — fix not yet implemented
created: 2026-09-11
---

# Stale Recall Memory and an Unpersisted `skipRisk` Flag Resurfaced Old TP/SL Preferences

## Symptom

A user explicitly said "no sl no tp" opening a HYPE long. Within the same session, minutes later, the agent surfaced TWO different unrequested TP/SL figures on the same position-building flow: a code-attached default band (7.5% SL / 12% TP) on one turn, then a stale remembered preference (15% SL / 15% TP) on another. User (via Discord, "Blackie | Rysk"): *"it does keep on happening, seems to be getting a bit worse... it is hallucinating tps and sls now."*

## Context

Investigated a specific production session (`a6045423-d7e1-47df-95e3-722fc9e10374`, a long-lived TG DM session spanning 2026-08-23 → 2026-09-11 with no daily rotation). Traced via Langfuse trace/observation reads (tool I/O, generation inputs, model fields) — read-only, no code changed, no fix shipped this investigation.

## Initial Hypothesis (user-supplied)

That a same-day forced model switch (Cerebras dropped `gemma-4-31b`; composer/modeClassifier/reaction/scheduler-evaluator hotfixed onto `gpt-oss-120b`, PR merged ~2.5h before the incident) caused the regression.

## Investigation

1. Read the actual tool I/O for the 7.5%/12% turn: the model's `open_position` call omitted `skipRisk`; the tool's own response told the model *"Do NOT call set_risk_parameters for it and do NOT tell the user it is missing"* — the default band is attached silently by design.
2. Found the 15%/15% text was NOT produced by tradeExecution or composer — it came from the **planner's `clarify` early-exit shortcut**, which bypasses tradeExecution's tool contract entirely and free-composes clarifying-question text directly from the injected persona/memory block.
3. Git-archaeology on every relevant node's model history (`apps/agent/src/config/models.ts` full log): `tradeExecution` has **never** run Gemma or GPT-OSS-120B in any commit (history: gpt-5.1-codex → gemini-3.6-flash → gpt-5.6-luna → gemini-3.1-flash-lite). `planner` has run `gpt-oss-120b` since 2026-06-15 (switched from `gemini-3.1-flash-lite`, not Gemma) — not recent. Only `composer`/`modeClassifier`/`reaction`/scheduler-evaluator had the actual same-day Gemma→GPT-OSS switch.
4. Live-replayed the exact captured production context (same messages, same tools, same temperature) against the actual models in the causal path:
   - `tradeExecution` on its real model (`google/gemini-3.1-flash-lite`, via OpenRouter): **8/8 trials** omitted `skipRisk`, deterministically — not model noise, a pure consequence of context that no longer contains any risk-preference signal.
   - `planner` on its real model (`gpt-oss-120b`, via Cerebras): **0/5** hallucinated when the raw "no sl no tp" instruction was still in context (turn 1); **3/5 (60%)** injected "SL 15%, TP 15%" once the instruction had rolled out of the raw window and only stale cross-session memory remained (turn 7, the actual incident context).
   - Attempted the Gemma counterfactual directly: **Cerebras rejected `gemma-4-31b` outright** ("Model does not exist or you do not have access to it") — confirming it's genuinely gone from the provider, not a toggle-able choice.

## Finding / Root Cause

Three compounding, independently-real gaps — none of them the model switch itself:

1. **`skipRisk` is pure LLM-context state, never persisted structurally.** [`open-position.tool.ts`](apps/agent/src/ai/tools/trade-execution/write/open-position.tool.ts) attaches a leverage-scaled default SL/TP band on any MARKET open unless the model re-emits `skipRisk: true` — on every single call, with no flow-level memory of an earlier opt-out.
2. **tradeExecution's raw history window is `ctx.history.slice(-6)`** (3 turns, 500-char cap per message) — [`node-executor.service.ts:414-425`](apps/agent/src/ai/dag/executor/node-executor.service.ts:414). Any instruction older than ~3 turns is guaranteed gone from raw context.
3. **The "recall" note meant to cover the gap is stale by design in a long-lived session.** Refreshed every 15 cumulative messages ([`background.types.ts:44-47`](apps/agent/src/ai/background/background.types.ts:44)), but the counter is cumulative across calendar days on a TG session that never rotates — confirmed the note attached to the *very first message* of the 09-11 session was still describing the *previous day's* session verbatim.
4. **Precedence ("current instruction overrides memory") is prompt text only, never code-enforced.** [`session-state.util.ts:64-66`](apps/agent/src/ai/utils/session-state.util.ts:64) — one sentence, repeated identically at three independent injection sites (composer, planner-adapter, tradeExecution-via-node-executor), each with its own copy of the same possibly-stale note and no shared reconciliation.

## Model-Switch Hypothesis: Rejected As Stated, Partially Right By Accident

The user's premise doesn't hold for either node actually in the causal path (`tradeExecution` never ran Gemma; `planner`'s GPT-OSS switch is 3 months old, not recent). But `composer` genuinely was forced onto `gpt-oss-120b` same-day due to Cerebras retiring `gemma-4-31b` entirely — and the team's own prior internal eval (comment in `composer/agent.ts:39-41`) already found GPT-OSS-120B "over-compresses... drops supporting information" vs Gemma for that role. And the live replay *did* show GPT-OSS-120B is genuinely unreliable (60%) at honoring "current instruction overrides memory" once that instruction is out of view — real, but conditional on the context already being corrupted by the other three gaps, not the primary cause.

## Fix / Decision

Not yet implemented. Recommended: promote risk-protection intent to structured, server-persisted trade-flow state — `riskProtection: 'enabled' | 'disabled' | 'unspecified'`, scoped to the in-progress position/flow — with `skipRisk` derived from it as a default rather than re-derived from prose every call. Removes the dependency on raw-window survival and recall-refresh timing for this specific flag.

## What Fooled Us (would have, without the replay)

The timing correlation (hotfix merged ~2.5h before the incident) was a genuinely reasonable first hypothesis and easy to stop at. Only replaying the *exact* captured context against the *actual* models in the causal path — not just noting which models were live — separated the real, deterministic application bug (skipRisk, 100% reproducible) from the real-but-secondary model reliability gap (60%, conditional on the context already being broken).

## Remaining Unknowns

- Full blast radius not reliably quantified. A naive keyword sweep (40 sampled sessions with a `skipRisk`-omitted `open_position` call) found 5/40 with an earlier same-session risk-opt-out phrase, but on inspection at least 2/5 were false positives (user quoting a prior assistant message, not stating a preference) and the other 2/5 had multi-day/multi-week gaps too large to confidently attribute to the same trade-flow. A same-flow-scoped query (bounded turn window, same symbol/venue) would be needed for a trustworthy count.
- Whether the 60% GPT-OSS-120B failure rate generalizes beyond this one captured context (single scenario, 5 trials) wasn't tested further.
- Direct DB query of `MemoryNote`/`ChatMessage.recallCompactedAt` for this session (exact recall-refresh timestamps, exact message counts) was not completed — Langfuse-derived turn counts (87 turns, ~174 messages) are an estimate, not DB-verified.

## Related Work

[[Silent TP-SL Default on the Streaming Chat Path]] — same user-visible symptom class ("explicit TP/SL instruction silently overridden"), different root cause (that one: hard-coded empty context on the streaming entry point, fixed 2026-07-09). This is the second time this symptom shape has appeared from a structurally different cause — worth watching for a third occurrence as a signal that the underlying pattern (risk-protection intent living only in re-derived LLM context) needs the structural fix rather than another point patch.

## Lessons

[[Engineering Lessons#A model switch that lands right before an incident is a lead, not a verdict]]
[[Engineering Lessons#State that must survive across turns cannot live only in re-derived LLM context]]
