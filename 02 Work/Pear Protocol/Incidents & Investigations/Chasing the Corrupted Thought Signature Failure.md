---
type: investigation
area: pear
status: diagnosed (not confirmed resolved)
created: 2026-07-20
---

# Chasing the Corrupted Thought Signature Failure

## Symptom

`google/gemini-3.1-flash-lite` intermittently failed with `Corrupted thought signature.` — recurring roughly every 1-2 days in dev, 27 times in a sampled production window.

## Context

Discovered while validating the newly-built `formatLlmError()` observability helper against a live dev failure.

## Initial Hypotheses

Unclear at first — the existing logs gave only a bare error message, not enough to form a hypothesis.

## Evidence

- OpenRouter reports body-level errors as HTTP 200 with an error object in the JSON body; the provider SDK throws `APICallError` with the payload in `.data`, never `.responseBody`.
- The most diagnostic field (`errorData`, carrying the upstream provider name, e.g. "Google AI Studio") was being silently dropped by the existing formatter.
- A separate bug compounded the diagnosis gap: a loose request-id header match recorded the literal string `"MISS"` as if it were a real id.

## Investigation

Fixed the two formatter gaps (`data` fallback, strict header matching) so the next occurrence would log the upstream provider name directly, rather than inferring it from sibling-step metadata.

## Finding

The observability fix makes the *next* occurrence diagnosable — it does not, by itself, confirm what causes the corruption. The Pear Protocol Work note this was drawn from states the enrichment was "later used to diagnose the Vertex↔AI-Studio reroute" behind a referenced follow-on branch (`fix/pin-gemini-backend`) — that attribution is carried here as stated in that note, not independently re-verified against a specific merged PR in the authored-work record.

## Fix / Decision

`llm-error.ts`: fall back to `APICallError.data` when `responseBody` is absent; strict `x-request-id` / `x-openrouter-request-id` header matching only.

## What Fooled Us

The provider's HTTP-200-with-error-body pattern meant a genuinely failed call looked, at the transport level, like a success — the failure only existed inside the JSON payload the old formatter wasn't reading.

## Remaining Unknowns

Whether the referenced Vertex↔AI-Studio reroute was ever conclusively confirmed as *the* root cause, and whether it was fully resolved, is not established in the currently available notes.

## Related Work

[[My Work at Pear Protocol#2026-07-20 — Chasing a recurring Gemini failure meant enriching the error pipeline twice]]
[[20th Jul 2026]]

## Lessons

[[Engineering Lessons#Observability helpers must never throw]]
