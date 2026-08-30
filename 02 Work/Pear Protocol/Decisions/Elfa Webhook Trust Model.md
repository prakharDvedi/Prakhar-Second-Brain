---
type: decision
area: pear
status: active
created: 2026-08-20
---

# Decision: Elfa Webhook Trust Model

## Context

Elfa signal-trigger webhooks could arm and execute without HMAC verification under an existing unsigned fallback path.

## Problem

An authentication-oracle / replay risk on production webhooks — anything reaching the endpoint could potentially be treated as a legitimate trigger firing.

## Options / Alternatives

- **Keep the unsigned fallback available everywhere:** simplest, but leaves the trust bypass live in production.
- **Remove the unsigned fallback entirely, no exceptions:** maximally strict, but removes a path some environments may still need for local iteration.
- **Require HMAC on every armed production webhook, but preserve an explicit dev-only unsigned escape hatch (chosen).**

## Decision

Removed the unsigned execution lookup fallback for production. Every armed production webhook now requires Elfa HMAC verification via a per-webhook signing secret set at arm time. The explicit `ELFA_WEBHOOK_ALLOW_UNSIGNED` path remains development-only.

## Why

Production trust must not depend on an unsigned fallback; a development-only escape hatch preserves local iteration speed without extending the same risk to production traffic.

## Tradeoffs

Existing armed queries predate the signing requirement, so `signed-v1` was added to persisted EQL specifically so old rows are detected as drift and re-armed after deployment — a rollout step this decision required, not optional.

## Evidence

[[My Work at Pear Protocol#2026-08-18 — Elfa signal triggers accepted invalid conditions, and close-all could silently downgrade a TWAP to market]]
[[Resume Evidence#Removed an unsigned-webhook fallback from the signals security surface]]

## Later Evidence

Focused controller + TriggerManager suites: 80/80 passing.
