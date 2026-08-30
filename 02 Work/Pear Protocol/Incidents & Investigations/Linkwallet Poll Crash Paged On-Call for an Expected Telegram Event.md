---
type: investigation
area: pear
status: resolved
created: 2026-07-21
---

# Linkwallet Poll Crash Paged On-Call for an Expected Telegram Event

## Symptom

Production error paging Discord:
```
Bootstrap:TG Unhandled rejection:
GrammyError: Call to 'sendMessage' failed! 403: Forbidden: bot was blocked by the user
```

## Context

The `/linkwallet` confirmation poll ran as an unguarded `setInterval(async () => ...)`.

## Initial Hypotheses

A real bug in the wallet-linking flow.

## Evidence

`setInterval` never awaits its callback — when a user blocked or abandoned the bot mid-flow, the poll's `ctx.reply` threw a grammY 403 that escaped as a process-level unhandled rejection.

## Investigation

Confirmed neither of the two existing safety nets caught this: grammY's `bot.catch` only wraps update middleware, not timer callbacks; the delivery worker's try/catch guards the notification queue, not this interactive polling path.

## Finding / Root Cause

An uncaught rejection inside a `setInterval` callback with no `await`-based error boundary around it.

## Fix / Decision

Extracted the interval body into `tick()`, invoked as `void tick().catch(...)`. A 403 (user blocked/deleted/deactivated the bot) is treated as an expected Telegram lifecycle event and logged quietly at `warn`; any other failure still logs at `error`.

## What Fooled Us

Nothing subtle — a straightforward missing-await-boundary bug that only a real user-abandonment event would trigger, so it didn't show up in typical happy-path testing.

## Remaining Unknowns

None noted.

## Related Work

[[My Work at Pear Protocol#2026-07-21 — A trade-execution trio: fractional leverage, a wrong root-cause string, and an unhandled Telegram rejection]]
[[21st Jul 2026]]
