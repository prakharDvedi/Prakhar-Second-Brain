---
type: investigation
area: pear
status: open
created: 2026-08-28
---

# Telegram Outbound Delivery Ordering — Unresolved Investigation

## Symptom

An older turn's media (e.g. a chart image) appeared to land on Telegram Desktop *after* a newer turn's text — unclear whether this is a real backend ordering bug or a client rendering quirk.

## Context

No prior instrumentation existed around outbound send ordering per conversation turn.

## Initial Hypotheses

Two live possibilities, explicitly not yet distinguished:
1. Outbound send ordering genuinely inverted (a photo completing after a later turn's text, or carrying a higher `message_id` than it should).
2. Send order was correct and the Telegram Desktop client simply rendered it oddly.

## Evidence

None yet — this PR adds the instrumentation needed to gather it; no data from it has been analyzed as of this note.

## Investigation

Added a single grammY API transformer (`delivery-observer.ts`) logging `send_start`/`send_complete`/`send_error` around every terminal outbound send, correlated to the originating turn via `AsyncLocalStorage` context set in `handleText` after enqueue. Captures chat id, job id, session id, response type, timestamps, returned `message_id`, media count, and (best-effort) the outbound image URL — no message content.

## Finding

No root cause established yet — this work is observability-only, specifically so the *next* occurrence can be diagnosed from logs instead of re-guessed. `AsyncLocalStorage.enterWith` was verified (via an adversarial spec) to isolate two genuinely overlapping turns without one inheriting or overwriting another's ids.

## Remaining Unknowns

Whether the symptom is a real ordering bug or a client-side rendering artifact is exactly the open question this instrumentation exists to answer — not yet answered as of the note's creation.

## Related Work

[[My Work at Pear Protocol#2026-08-28 — The internal-leak guard blocked legitimate AI-industry reporting; signal failures collapsed two mechanisms into one alert; delivery ordering had no visibility (open)]]
[[Features Owned#Observability / agent infrastructure]]
[[28th Aug 2026]]
