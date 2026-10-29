---
type: decision
area: pear
status: active
created: 2026-10-30
---

# Decision: Elfa Replacement — One Switch, Not Math-First

## Context

Pear is replacing Elfa for signal alerts in three tracks: math (price, TA, funding, liquidations, Kalshi), news and twitter. By 30 Oct the math track was code-complete on `elfa/math` (PR #1191) behind `PEAR_SIGNALS_LIVE`. The hourly reconcile still armed the same alerts at Elfa, so turning the flag on could send math alerts twice, and it stopped completely whenever Elfa's key or credit status was unavailable.

## Problem

How the flag should hand alerts from Elfa to Pear: per track as each one is ready, or for the whole product at once.

## Options / Alternatives

- **Math first:** gate only math-only alerts off Elfa, keep news, tweet and mixed rules on Elfa until their tracks land. Restores the alerts behind 77% of past taps weeks earlier, but runs two systems side by side with two budgets, two failure modes, and mixed rules stuck on Elfa.
- **One switch for the whole product (chosen):** with the flag on, the reconcile calls Elfa for nothing. Math rules go to Pear's poller, news and tweet rules are saved for engines still to be built, and the flag stays off until all three tracks can fire.

## Decision

One switch. `d4befdc5` makes the reconcile call Elfa for nothing when `PEAR_SIGNALS_LIVE` is on: every rule is still chosen and saved as `armed` with no Elfa query id, the weekly rebuild goes through Pear's rule builder, it runs while Elfa is down, and the personal budget is no longer clamped by Elfa's plan cap. Flag off is unchanged.

Two smaller calls made alongside:
- A new alert never fires on its first reading: a watch means "from now on", and a lost Redis state must not replay every true rule. Elfa did fire at arm time.
- `maxTriggers` caps personal alerts only (3 per 168h from the first fire). Curated alerts keep their cooldown, since a 720h window would cap crude-oil-shock at 3 fires a month where Elfa fired it 58 times in about 10 weeks.

## Why

Trigger signals are one product. One meaning for the flag gives one source of truth and simpler code than per-alert routing. Elfa is already paused in production, so there is no live traffic that a math-first release would protect.

## Tradeoffs

- Alerts stay dark until news and twitter are built too. Accepted, since Elfa is paused anyway.
- News rows saved under the flag may need a slightly different shape once the news engine exists (for example `events` leaves). Expected to be a small follow-up in the same reconcile.
- Leftover Elfa queries are left to expire instead of being deleted, since the flag means no Elfa calls at all.

## Evidence

[[29th Oct 2026#Update 2026-10-30: Elfa arming gate built (uncommitted, `elfa/math`)]] · commits `9aa82cd0`, `d4befdc5` · PR #1191 · comparison dashboard https://claude.ai/artifact/1XaJ5QtNaBFkDTm12AktBJ
