---
type: investigation
area: pear
status: diagnosed (not confirmed resolved)
created: 2026-09-11
---

# Kalshi Odds Query Fell Into a Planner Routing Dead Zone With No Empty-Plan Fallback

## Symptom

A real production TG DM user (`telegram:1789835514`, session `TG_DM1789835514`) asked *"what are the odds on kalshi that there is NO FED RATE increase next week"* and got a bare refusal: *"I'm not able to provide odds for a Kalshi contract on the Fed rate decision at this time."* One turn earlier, in the same conversation, a general Kalshi news question ("you dont track info on kalshi?") had routed correctly and produced a real, sourced answer.

## Context

Found while doing a general Langfuse trace review of this user's TG session (requested ad hoc, not from an incident report). No error, no exception, no ERROR/WARNING-level observation anywhere in the trace — this was a silent behavioral gap, not a crash.

## Investigation

Traced end-to-end via Langfuse observation reads (trace `1c69ecd4e5fc700b6900cf218910120b`, failing; `524d9c69a6bfd675ee79e035205568d1`, the preceding successful turn) plus a full read-only code audit of the planner/DAG pipeline (`apps/agent/src/ai/agents/planner/{prompt.ts,schema.ts}`, `apps/agent/src/ai/dag/{dag.service.ts,node-registry.ts,executor/orchestrator.service.ts,services/planner-adapter.service.ts}`, `apps/agent/src/ai/surface/surface-profile.ts`).

**Ruled out first (all confirmed, not just assumed):**
- Not a model/provider/infra difference — both planner calls used the identical `internalModelId`, `cerebras.chat`, temperature 0.1, `maxTokens: 4096`, and both finished with `finishReason: stop` (no truncation/error).
- Not a prompt-version drift — the planner system prompt sent to the model was byte-for-byte identical (49,734 chars, zero-diff) between the failing and succeeding calls.
- Not missing context — the failing call's message history included the full prior assistant turn about Kalshi verbatim (2,737 chars, regulatory/legal content), so the planner had just discussed the same entity one turn earlier.
- Not the known "planner emits the literal string `'false'`" incident (already patched — see `planner-adapter.false-activation.spec.ts`) — the raw structured output was genuine JSON `null`/`false`, not placeholder strings.

**What was found:**
1. The planner returned a fully empty decision: every node `null`, `generalChat: false`, `clarify: null`.
2. `research`'s own trigger description (`node-registry.ts`, `invokeWhen`) contains both an *inclusion* clause ("macro updates" — a Fed-rate decision is macro) and an *exclusion* clause ("NOT for live price... (marketScanner)") in the same sentence. A Kalshi contract's "odds" **is** its live price, so the query pattern-matches both directions of the same trigger text at once.
3. The node the exclusion redirects to (`marketScanner`) is, by its own `invokeWhen` text, scoped to two named tools (CoinGecko + Hyperliquid price feeds) for crypto tokens/HL-perps/HIP-3 markets — it claims no Kalshi/prediction-market coverage. So the redirect target doesn't own the query either — a genuine routing dead zone, not just an absence of matching words.
4. Independently of why the planner produced this: the DAG has no contract enforcement rejecting an all-null-plus-`generalChat:false`-plus-`clarify:null` decision. `PlannerDecisionSchema` (Zod) has no `.refine()` requiring at least one of {node, `generalChat`, `clarify`}; `extractStructuredResponse()` validates only that the right *keys* are present, never that any value is truthy; `orchestrator.service.ts`'s `executePlan`/`streamExecutePlan` explicitly early-returns empty findings when `activeNodes.length === 0` with no log/metric; `dag.service.ts`'s only two post-plan gates (`plan.clarify`, `shouldDeclineDialedPlan`) both no-op here — the latter unconditionally returns `false` for any surface without a `nodes` dial, and `tg_dm`'s profile has none.
5. The system already has a purpose-built fallback for "nothing to route": `createDefaultPlan()` (`planner-adapter.service.ts`) returns this exact shape except `generalChat: true`. It is wired only to the *missing/invalid structured response* failure mode (`dag.service.ts` line ~501-505: `if (!planFromAdapter) ... return this.plannerAdapter.createDefaultPlan()`), never to a *structurally valid but semantically empty* response.

## Finding

Two separable, both-confirmed root-cause layers, not one bug:
- **Routing/prompt gap:** `research`'s trigger text simultaneously includes and excludes this query shape, and the node it redirects to doesn't cover the topic either — a genuine dead zone for prediction-market/odds-style queries specifically.
- **DAG contract gap:** regardless of why the planner landed on an empty decision, nothing downstream catches it. The one field separating what happened from the system's own known-good fallback (`createDefaultPlan()`) is `generalChat: true` vs `false`.

## What Fooled Us Initially

The first read of this treated it as "research's trigger list is just missing the words odds/probability" — a simpler, weaker explanation. A closer read of the exact deployed prompt text (not just a paraphrase) showed the real mechanism is more specific: an active *exclusion* clause pointing away from research toward a node that doesn't claim the territory either, plus a downstream contract that was never built to catch the resulting empty decision.

## Remaining Unknowns

Causality was not proven by live reproduction. It was not verified, by re-invoking the planner with controlled variants, that the trigger-text conflict is what actually drove this specific LLM output on this turn (temperature 0.1 is not fully deterministic; only one real occurrence was observed). Whether this is a one-off sampling fluke or a systematic gap for this query class is unknown — no broader sample of other sessions/users was checked.

## Fix / Decision

None implemented this session — read-only investigation only. No PR opened.

## Update — 2026-09-25 (new evidence: the empty-plan state recurs)

A second, unrelated query class hit the same all-null plan (`generalChat: false`, `clarify: null`): generic US-stock pick asks judged crypto-only. See [[Generic US-Stock Picks Refused as Crypto-Only, Then Reported as a Data Outage]]. With no findings, the composer described the empty plan to the user as a data outage ("unable to retrieve… try again later").

This partly answers the "one-off vs systematic" unknown above, for the empty-plan state in general rather than for the Kalshi case specifically:
- Planner-only probes (Cerebras gpt-oss-120b; main prompt and fixed prompt pooled, 18 samples each; some early samples may have fallen back to Haiku) mostly returned an all-null plan for off-topic asks, although the planner prompt routes those to generalChat:
  - "write me an essay on the history of the stock market": 14 empty, 3 clarify, 1 correct
  - "calculate the CAPM beta for my finance homework": 12 empty, 4 clarify, 2 correct
- The DAG contract gap is unchanged: still no empty-plan fallback on web/TG. PR #986 fixed only the stock-routing trigger, not this gap.

Still not re-verified for the Kalshi query itself.

## Related Work

[[11th Sep 2026]]

## Lessons

[[Engineering Lessons#A schema that allows "nothing selected" needs to be caught structurally, not left to the model's next mistake]]
