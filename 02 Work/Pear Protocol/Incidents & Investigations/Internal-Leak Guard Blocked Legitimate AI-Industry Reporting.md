---
type: investigation
area: pear
status: resolved (as of last known state — PR open at time of writing)
created: 2026-08-28
---

# Internal-Leak Guard Blocked Legitimate AI-Industry Reporting

## Symptom

A Z.AI/Zhipu monitor task's news summary — saying "outperforming some GPT-5.6 benchmarks" and "anonymous model appearing on OpenRouter" — had its entire reply replaced by the guard's canned notice.

## Context

Model/provider names like `gpt-5`, `gemini-3`, `openrouter`, `cerebras`, `mimo` are simultaneously real identifiers of Pear's own inference stack and ordinary public AI-industry vocabulary any news summary can name-drop.

## Initial Hypotheses

None needed — the mechanism was immediately clear: an unconditional substring match on the deny-list.

## Evidence

Reproduced directly in the guard's own spec suite as a condensed version of the production incident.

## Investigation

Reviewed the guard's matching logic — a bare substring hit anywhere in the reply hard-blocked the whole response, regardless of surrounding context.

## Finding / Root Cause

The deny-list treated distinctive internal identifiers and ordinary public industry vocabulary the same way — both were unconditional substring matches.

## Fix / Decision

Moved model/provider names out of the unconditional `INTERNAL_TOKENS` deny-list into a separate, sentence-scoped `MODEL_PROVIDER_TOKENS` check that only fires when a first-person/internal-operation disclosure frame ("we run on X," "powered by X," "routed through X") appears in the *same sentence*. Distinctive internal identifiers stayed hard-blocked under any framing. See [[Internal-Leak Guard — Contextual Frame vs Unconditional Deny-List]] for the full decision.

## What Fooled Us

A same-day follow-up review found the first widened version had over-corrected: it now missed real direct self-disclosure phrasings ("based on X," "running on X," "the underlying model is X") that the original stricter version used to catch. The frame was widened again without restoring unconditional blocking.

## Remaining Unknowns

Whether every legitimate disclosure phrasing has been captured, or whether a future review will again find a gap in one direction or the other, isn't something a single tuning pass can guarantee — this is an ongoing tuning surface by nature.

## Related Work

[[My Work at Pear Protocol#2026-08-28 — The internal-leak guard blocked legitimate AI-industry reporting; signal failures collapsed two mechanisms into one alert; delivery ordering had no visibility (open)]]
[[28th Aug 2026]]

## Lessons

[[Engineering Lessons#Tuning a safety check in one direction can silently break the other]]
