---
type: investigation
area: pear
status: resolved
created: 2026-07-10
---

# Composer Fabricated the Current Time

## Symptom

Asked for the current time, the composer answered "It is currently 02:30 UTC" when the real time was ~04:46 UTC, then admitted after correction that it had no live clock.

## Context

Production session, trace showing the composer's system prompt carried a calendar date but never a time-of-day.

## Initial Hypotheses

None recorded beyond the obvious — no deterministic clock existed at the composer boundary.

## Evidence

The wrong time was absent from the composer's input and first appeared only in the output — confirming the fabrication happened inside the generation itself, not upstream.

## Investigation

Checked whether the planner or any DAG node supplied a time value — none did. The composer (generalChat route) had never been given one.

## Finding / Root Cause

Missing deterministic time source at the composer boundary. Not a planner or node issue.

## Fix / Decision

Added a minute-precision UTC clock as its own system message positioned *after* the cacheable base prompt, mirroring the planner-adapter's existing volatile-after-cacheable pattern so a per-minute value can't invalidate a prompt-cache prefix.

## What Fooled Us

Nothing subtle here — the gap was simply never noticed until a user asked and got a fabricated answer.

## Remaining Unknowns

User-local timezone is still unknown and not inferred; all answers stay UTC-labelled.

## Related Work

[[My Work at Pear Protocol#2026-07-10 — The composer had no clock, and a finished PnL card was being thrown away on timeout]]
[[10th Jul 2026]]
