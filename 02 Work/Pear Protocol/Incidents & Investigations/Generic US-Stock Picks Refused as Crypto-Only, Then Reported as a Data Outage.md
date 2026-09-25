---
type: incident
area: pear
status: fix in review (PR #986 open, not merged)
created: 2026-09-24
---

# Generic US-Stock Picks Refused as Crypto-Only, Then Reported as a Data Outage

## Symptom

A user (RyanLee, via Discord with a screenshot) asked Agent Pear on web: *"check last US session. What stocks to long and to short? give me 3 longs 3 shorts"*. The reply was *"I'm unable to retrieve the latest U.S. session data right now… try again later"*. His follow-up, *"don't you have market data in your charts?"*, got *"I don't have any market data available at the moment."* Both replies are false: Pear lists US stocks as HIP-3 perps (`xyz:NVDA`, `xyz:SP500`) and marketScanner has live data for them.

## Investigation

Found the prod traces in Langfuse by searching trace inputs for the query text: `0601633e1409f9264a45caafcc6dd43c` and `fa00f69db57a21bb60e6a45f6bafe17c` (web, `environment=production`).

- **Turn 1:** the planner (gpt-oss-120b on Cerebras) returned an all-null plan with `generalChat: false`. Its reasoning: *"The system is a crypto trading assistant… This is out of scope. So we must refuse."* It misapplied the `common-rules.ts` rule 7 scope boundary, although its own prompt says `HIP-3 / equities / commodities → marketScanner`. The planner output has no "refuse" field, so it nulled every node. The composer got "No findings available from any node" and no tools, and described that as a fetch failure.
- **Turn 2:** routed correctly to generalChat, but the composer made no tool calls and said it has no market data.
- **Reproduced on TG** (my own session): `c82e480477c8c044d46beed74cc619de` and `258ea38716e8d1c0615cd2a41ad232da` failed; "give me a long/short pair on stocks" worked. A separate fresh-session web trace of the exact prompt (`d19ae506`) also worked.
- **Replaying the exact prod planner inputs** (8 samples each) through the real `PlannerAgent`: RyanLee's input was 8/8 correct on the current prompt, so his failure was a sampling fluke. My TG session's input was 0/8, a deterministic failure. Removing the four `[auto-task]` position-update messages at the top of that history mostly fixed it (7/8), but no smaller slice of the history reproduced the failure; only the full 10-message history did.

## Finding

- **Primary cause (confirmed by replay):** the planner treats generic stock-pick asks as outside a crypto-only scope and returns an empty plan or generalChat. Long execution-flow history (auto-task updates, a staged close, a metals-portfolio build) makes this deterministic.
- **What made it user-visible as a false outage:** the same empty-plan → composer-with-no-findings path documented in [[Kalshi Odds Query Fell Into a Planner Routing Dead Zone With No Empty-Plan Fallback]]. See the 2026-09-25 update there.

## What Fooled Us Initially

- **"It's transient":** the first fresh-session repros passed, and RyanLee's own input replayed 8/8 correct. Only the long TG history failed every time.
- **"The eval harness reproduces prod":** the regression harness passed 5/5 on the unfixed prompt. My first TG-shaped fixture used a condensed history and also passed. Only the verbatim prod history reproduced the failure.
- **"The v3 wording is weaker":** mid-comparison, Cerebras failed on about half the calls and the planner silently fell back to `claude-haiku-4.5` through OpenRouter. That produced misleading numbers. The comparison was redone pinned to Cerebras with `modelOverride`; see the lesson below.

## Fix / Decision

[PR #986](https://github.com/pear-protocol/agent-pear-v2/pull/986) (`fix/planner-us-stocks-in-scope` → `development`; open, not merged at close-out). One planner prompt line: tickerless stock-pick asks are in scope and go to marketScanner, even mid-execution-flow; never generalChat or an empty plan; named tickers keep their normal route (`bb/sp500` → pair). `common-rules.ts` was deliberately left untouched. Changing it also fixed the replay, but it reaches every agent's prompt.

After a blast-radius review, a second commit (`74d7e90f`) made three changes:
- dropped the "US stocks ARE tradeable here" claim, since the planner routes and must not claim tradeability on surfaces like `base_mini`
- added the named-tickers clause, because the first wording sent `bb/sp500` to marketScanner 2 times in 8
- replaced the real account figures in the TG history fixture with synthetic ones; it still reproduces the bug

The final wording was chosen from 4 tested variants. One variant written after the review halved a curated-baskets routing case (4/8) and was dropped.

**Verification (final wording):**
- prod-input replay: 24/24 routed to marketScanner
- stock fixtures: 20/20 (main prompt: 11/15)
- `bb/sp500` → pair: 8/8; curated baskets → basket: 8/8
- `PLANNER_TEST_SET=both`: 325/347 (main: 323, 323)
- 12 diverse stock asks, Cerebras-pinned: 90.3% (first wording: 88.6%)
- full `test:node-eval` on web, 4 runs: the scanner called `hyperliquid_market_overview({dex:"xyz"})`, got real stock listings, and the composer answered from them

## Remaining Unknowns

- **Not verified after merge in prod.** The only evidence is replays and eval runs.
- **`base_mini` leaks Hyperliquid `xyz` stocks.** The scanner ignores the SYMM-only whitelist and attaches a trade link. The same happens on the main prompt, so it's pre-existing and not fixed here.
- **"is TSLA a short here?"** mostly routes to tradeExecution (read as a position check) under every wording.
- **Earlier probe runs may include some Haiku-fallback samples.** The harness couldn't show which model answered. The final v1-vs-v3 comparison was pinned to Cerebras.

## Related Work

[[24th Sep 2026]]

## Lessons

- [[Engineering Lessons#A model switch that lands right before an incident is a lead, not a verdict]]: the same point, that replaying the exact captured context is what separates deterministic from sampling failures.
- [[Engineering Lessons#An eval harness with a silent model fallback measures a different model when the primary is overloaded]]
