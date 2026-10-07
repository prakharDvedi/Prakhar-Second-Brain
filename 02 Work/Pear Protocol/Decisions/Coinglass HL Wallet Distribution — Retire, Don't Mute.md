---
type: decision
area: pear
status: active
created: 2026-10-07
---

# Decision: Coinglass HL Wallet Distribution — Retire, Don't Mute

## Context

The Coinglass plan went from standard to start. `hyperliquid/wallet/position-distribution` and `pnl-distribution` (one tool, `coinglass_hl_distribution`) now return "Upgrade plan". We are not upgrading.

## Problem

Keep the tool and mute the errors with a fallback, or remove it.

## Decision

Remove the tool, the two service methods and their types. Drop "HL PnL distribution" from the planner description. Add one line to the coinglass node prompt saying wallet-level data isn't available, and answer with the top-trader long/short ratio, liquidation history and taker volume that still work. No Hyperliquid API replacement for now.

## Why

Prod Langfuse, 14 days: 51 calls, 48 failed, all as an extra on routine positioning queries. A kept tool fails every time, costs a tool-loop step, tokens and latency, and the planner keeps advertising it. `git revert` restores it if the plan changes.

## Tradeoffs

Users lose the whole-exchange "are big or profitable wallets long or short" view. It has been dead since 26 Sep, so nothing new is lost. A Hyperliquid-API approximation (top wallets, then positions) is a sample, untested, and not built.

## Evidence

[[7th Oct 2026]]
Issue [#1089](https://github.com/pear-protocol/agent-pear-v2/issues/1089)
