---
type: investigation
area: pear
status: resolved
created: 2026-07-21
---

# Partial Fill Alert Blamed the Wrong Cause

## Symptom

A production alert for ticket `dcfb9d9c-967b-42e0-8958-a24fde7218ab` reported a genuine partial fill (kPEPE short filled, CASHCAT long did not, leaving the user unhedged) with the stated cause: "one or more legs' notional fell below the $11-per-asset minimum after weight distribution."

## Context

That explanation was hardcoded into `execute-core.ts` and appended to *every* partial fill, regardless of the actual numbers.

## Initial Hypotheses

Initial reading of the alert suggested a sizing problem (notional too small).

## Evidence

The specific incident's own numbers disproved the hardcoded guess: Long CASHCAT / Short kPEPE, 3x leverage, $170 margin → $510 notional → $255 per leg — 23x above the stated $11 floor. The freeze-time min-notional pre-check had already passed correctly.

## Investigation

Traced the error string to its source and confirmed it was a static guess, not computed from the actual execution response — the Pear BE `openPosition` response carries no per-leg rejection reason at all.

## Finding / Root Cause

The real cause was a venue-side rejection of the thin CASHCAT leg, unrelated to notional size. The alert text was asserting a cause the code had no way to actually verify.

## Fix / Decision

Rewrote the message to report only what fills actually tell us: which legs filled, which are missing, that the position is now unhedged, and a generic "one or more legs failed to execute" — no specific cause asserted.

## What Fooled Us

The message read as a computed diagnosis rather than a static guess, so it directed both the user and on-call toward the wrong fix ("increase margin/notional") for every occurrence, not just this one.

## Remaining Unknowns

The underlying architectural question (non-atomic multi-leg execution, whether rollback/compensation should exist) was explicitly left as a separate design discussion.

## Related Work

[[My Work at Pear Protocol#2026-07-21 — A trade-execution trio: fractional leverage, a wrong root-cause string, and an unhandled Telegram rejection]]
[[21st Jul 2026]]

## Lessons

[[Engineering Lessons#A hardcoded root-cause string in an error message is a guess wearing a lab coat]]
