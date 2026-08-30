---
type: decision
area: pear
status: active
created: 2026-08-24
---

# Decision: Confirm-Time Margin Gate

## Context

`open_basket` had no check that a fresh margin read still covered the plan at the moment of confirmation — margin could have moved between planning and confirming.

## Problem

Whether to add a safety check at confirm time, and how to scope it so it doesn't block legitimate reduce-only adjustments (which release margin rather than consume it).

## Decision

Added a fail-closed gate reading fresh `availableToTrade` before any leverage or order write, with a configurable buffer (100bps), and explicitly exempted reduce-only adjustments from the check.

## Why

A margin check that also blocked reduce-only adjustments would be checking the wrong direction of risk — reduce-only actions free up margin, they don't consume it, so gating them the same way as an open would add friction with no corresponding safety benefit.

## Tradeoffs

Fail-closed means a transient margin-read failure blocks the trade rather than letting it through — accepted as the safer default on a money-moving path, with replayable diagnostics so a rejection is debuggable rather than opaque.

## Evidence

[[My Work at Pear Protocol#2026-08-24 — A confirm-time margin gate, a recurring ambiguous-symbol crash, and a hidden 2x-budget confirm]]
[[Resume Evidence#Designed a confirm-time margin safety gate]]
[[Features Owned#Trade execution]]

## Later Evidence

Focused tests cover insufficient / unavailable / changed-balance cases.
