---
type: investigation
area: pear
status: resolved
created: 2026-07-09
---

# Silent TP-SL Default on the Streaming Chat Path

## Symptom

A user's explicit prior-turn TP/SL instruction was silently overridden by a documented 100%-PnL default on the following turn, on the streaming chat path only.

## Context

Traced from a production session, Dutch-language: turn N said *"ik wil 5% winst maken waarom dan tp op 100% zetten?"* (why is TP set to 100% if I want 5% profit?), turn N+1 staged a ticket carrying the 100% default anyway.

## Initial Hypotheses

The trade-execution agent itself misreading the user's stated percentage.

## Evidence

- The synchronous `run()` entry point was unaffected — only the streaming `runAsync()` path showed the bug.
- `runAsync` called `buildDagContext(sessionId, userId, "", [], …)` — hard-coded empty `userInput`/`history`.

## Investigation

Traced the DAG context construction on both entry points. The node-executor injection gates that read `ctx.history`/`ctx.userInput` (notably `tradeExecution`) never received real values on the streaming path, so the user's actual instruction never reached the agent that would have honored it.

## Finding / Root Cause

`runAsync` never threaded the real `userInput`/`history` into `buildDagContext` — a hard-coded `"", []` — while `run()` passed the real values. Composer and planner read their own args directly, so this was isolated to the node-executor injection seam.

## Fix / Decision

Pass the real `userInput`/`history` into `buildDagContext` on the streaming path, mirroring the sync path exactly — a one-line-pair fix, no redesign of context threading.

## What Fooled Us

The bug only manifested on one of two structurally near-identical entry points, and only for nodes that specifically opted into reading `ctx.history`/`ctx.userInput` — an easy thing to miss without a test driving both paths side by side.

## Remaining Unknowns

None noted — scope confirmed contained to the node-executor injection gates.

## Related Work

[[My Work at Pear Protocol#2026-07-09 — DAG context bugs: a fake telemetry alarm, and a real silent TP/SL default]]
[[9th Jul 2026]]

## Lessons

[[Engineering Lessons#Two entry points meant to behave identically can silently diverge]]
