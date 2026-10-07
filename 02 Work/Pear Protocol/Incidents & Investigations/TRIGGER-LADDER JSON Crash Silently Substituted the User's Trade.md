---
type: investigation
area: pear
status: resolved
created: 2026-08-24
---

# TRIGGER-LADDER JSON Crash Silently Substituted the User's Trade

## Symptom

`open_position`'s TRIGGER/LADDER `condition` argument crashed with a raw `TypeError`:
```
Cannot read properties of undefined (reading 'map')
```
The model then silently retried with a *different* condition type and succeeded — replacing the user's actual request with no error ever shown to them.

## Context

Found while triaging five days of Langfuse ERROR-level traces, not from a user report.

## Initial Hypotheses

None initially — the crash itself was the first signal.

## Evidence

Trace `b9164502960cd3af356c603488ee5657`: the model called `open_position` with a `weighted_ratio` condition carrying a `symbol` field (the shape for a *different* condition type, `price`) instead of the required `basket` array. It crashed, then retried with `type: "price"` and succeeded — quietly replacing the user's weighted-basket trigger request with a single-symbol price trigger.

## Investigation

Traced the `condition` argument's path: it's a JSON-encoded string, `JSON.parse`d and spread `as never` directly into `planTriggerOpenUsecase`/`planLadderUsecase` with no schema check. Compared against the MCP adapter, which already uses the real `TriggerOpenInput` Zod discriminated union as its tool input schema and rejects the same malformed shape cleanly.

## Finding / Root Cause

The agent/Telegram-chat adapter took a JSON-string shortcut for `condition` (likely for tool-schema simplicity across models) and never validated the parsed contents before use — one write path, two adapters, only one of which validated the input. A later re-validation step exists (`HttpTypes.CreateTriggerOpen.safeParse`) but runs on the already-built wire condition, after the crash point — positioned too late for this failure mode.

## Fix / Decision

Reused the same schemas the MCP adapter already trusts (`TriggerOpenInput.pick({condition:true})`, `LadderCreateInput.pick({condition:true, config:true})`) to validate the parsed condition JSON in `open-position.tool.ts` before it reaches the usecase. A bad shape now fails as a named `PlanningError` with an actionable message, same propagation path, just correctly diagnosed.

## What Fooled Us

The model's automatic retry with a *different* condition type looked, from the outside, like ordinary self-correction — nothing distinguished "retried the same request" from "silently answered a different question."

## Remaining Unknowns

None noted — new spec confirms both the rejection case and that a well-formed condition still passes through unchanged.

## Related Work

[[My Work at Pear Protocol#2026-08-24 — A single day of hardening the Lighter surface: wiring gaps, a doc that contradicted itself, and a crash that silently substituted the user's request]]
[[24th Aug 2026]]

## Lessons

[[Engineering Lessons#An unvalidated shortcut on one of two parallel adapters reopens a hole the other already closed]]
[[Engineering Lessons#A silent retry that changes the request type is worse than a visible crash]]

## Follow-up — 2026-09-28 (the validation error wasn't actionable)

*Correction to "Fix / Decision" and "Remaining Unknowns" above; the original text is kept as written.*

The 2026-08-24 validation stopped the crash, but its error message was **not** actionable. `parseConditionArg` joined only `issue.message` and dropped `issue.path`, so the model saw `Invalid condition: expected string, received undefined` with no field name. The tool description also only said `...type fields`. As a result, every `ratio` / `weighted_ratio` open failed. The model guessed `symbol`, then `symbolA`/`symbolB`, retried up to 11 times in one turn, and then told users false things ("please confirm" for an unstaged trigger, "market orders only", "baskets not supported"). Traces: `7e74054c7106eb2b01ce1e1d3b2f9c97`, `ea8bd6b98f5231b89f4921f5517b3082`, `4d46e9d1ebf28a2560804b8c795e6fe4`.

**Fix:** the error now names the path (`condition.symbol_a: …`), and the description lists the fields for each type. Commit `a78b6bd7`, PR #1017 (open as of 2026-09-28).

**Still unconfirmed:** the trade-execution prompt already listed the fields, and advanced/direct mode (which lacks that prompt) is suspected to be the failing path. The traces' mode hasn't been checked.

See [[28th Sep 2026]] and [[Engineering Lessons#A validation error that doesn't name the field just moves the model's guessing loop]].
