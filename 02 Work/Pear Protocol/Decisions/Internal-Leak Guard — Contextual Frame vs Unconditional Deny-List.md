---
type: decision
area: pear
status: active
created: 2026-08-28
---

# Decision: Internal-Leak Guard — Contextual Frame vs Unconditional Deny-List

## Context

Model/provider names (`gpt-5`, `gemini-3`, `openrouter`, `cerebras`, `mimo`, ...) are simultaneously real internal identifiers and ordinary public AI-industry vocabulary. An unconditional substring deny-list on these names blocked a legitimate AI-industry news summary in production.

## Problem

Loosening the match to stop false positives risks reopening false negatives (real self-disclosure phrasings) the stricter version used to catch.

## Options / Alternatives

- **Keep the unconditional substring deny-list for these names:** catches every disclosure, but hard-blocks the whole reply on legitimate reporting too.
- **Drop these names from the guard entirely:** would stop the false positives, but removes protection against genuine disclosure.
- **Sentence-scoped, frame-gated check (chosen):** only fires when a first-person/internal-operation disclosure frame ("we run on X," "powered by X," "routed through X," "based on X," "running on X") appears in the same sentence as the name.

## Decision

Moved model/provider names out of the unconditional `INTERNAL_TOKENS` deny-list into a separate `MODEL_PROVIDER_TOKENS` check, gated on disclosure framing. Distinctive internal identifiers (`execute_ops`, `base_mini`, `DAG`, etc.) stayed hard-blocked under any framing — this decision only reshapes handling for public-vocabulary-overlapping names.

## Why

The false-positive-driving names are exactly the ones that also appear in ordinary tech reporting; a frame-based check lets benign mentions through while still catching actual first-person disclosure.

## Tradeoffs

A first attempt at the frame narrowed too far and missed real direct phrasings ("based on X," "running on X," "the underlying model is X") — caught same-day via review and widened again, without restoring any unconditional blocking. Cross-sentence reach is deliberately limited to the immediately-following sentence, and only when it's a back-referencing continuation.

## Evidence

[[My Work at Pear Protocol#2026-08-28 — The internal-leak guard blocked legitimate AI-industry reporting; signal failures collapsed two mechanisms into one alert; delivery ordering had no visibility (open)]]
[[Internal-Leak Guard Blocked Legitimate AI-Industry Reporting]]

## Later Evidence

60/60 tests including adversarial and benign near-miss cases, as of the same-day follow-up widening.
