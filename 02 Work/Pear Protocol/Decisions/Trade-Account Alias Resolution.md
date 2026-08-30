---
type: decision
area: pear
status: active
created: 2026-08-17
---

# Decision: Trade-Account Alias Resolution

## Context

Models can relay a user's human-readable trade-account alias instead of the UUID shown in the choice list; the previous UUID-only lookup rejected that retry outright.

## Problem

A naive alias fallback risks selecting the wrong account when aliases collide across supported and unsupported venues.

## Options / Alternatives

- **UUID-only lookup (previous behavior):** safe, but rejects a very common model retry pattern.
- **Naive alias fallback (rejected):** would resolve any alias match without checking uniqueness, risking a wrong-account pick on collision.
- **Unambiguous, case-insensitive alias match evaluated against the full account list before the venue gate (chosen).**

## Decision

Resolve explicit account selections by alias only when the match is unique across the user's full account list, and only evaluate the supported-venue gate after that uniqueness check. Restrict typo recovery to users with exactly one total supported account.

## Why

Duplicate aliases needed to fail closed rather than guess; checking uniqueness before the venue gate prevents an unsupported-venue account from silently absorbing an ambiguous alias.

## Tradeoffs

Unique aliases now resolve to their account UUID and persist as the resolved UUID (not the alias) going forward. Duplicate aliases fail closed with the existing venue error; existing exact-UUID selection is unchanged.

## Evidence

[[My Work at Pear Protocol#2026-08-17 — Stale close-all SDK contract, unsafe alias resolution, and the development→main promotion review]]

## Later Evidence

Focused agent account-resolution suite 18/18, MCP selector suite 7/7, both passing at review time.
