---
type: investigation
area: pear
status: mitigated
created: 2026-10-08
---

# Chart and Chip Extractor Picked Up an Earlier Turn's Asset

## Symptom

In a group chat, a HYPE/ETH answer shipped a QNT chart and "How would you like to proceed with the short QNT trade?" Execute buttons. Another user had asked about QNT a few turns earlier. The written answer was correct; only the chart and buttons were wrong. Langfuse trace `bc38a48fb77dbe1c1c27c42e9280b673`.

## Evidence

- The phase-2 extractor (`extractExtras` in `composer/agent.ts` and `advanced/agent.ts`) builds the tradePlan, charts and suggestedAction after the answer is written. It got the chat history through `messages.filter((m) => m.role !== "system")`.
- That filter also removed `CURRENT_TURN_DIRECTIVE`, the only marker of which user message is the current ask.
- Replay of the incident on gpt-oss-120b at temperature 0: QNT chart and chip 8 of 8 before the change, HYPE/ETH pair chart after.

## Finding / Root Cause

The extractor could not tell which user message was the current ask, so it took an asset from an earlier turn. Prompt and input problem, not a model-capability one.

## Fix / Decision

PR #1091 (merged into `development` 2026-10-08): `markCurrentAsk` keeps the history (so "same but 4h" still resolves), drops system prompts, and inserts a marker before the latest user message. `EXTRACT_INSTRUCTION` now says to extract for the current ask only, and to take the chart interval from the ask, default 1d. Model unchanged.

Model switch tried and rejected, on the same 100 prod extractor inputs from the last 7 days: Claude Haiku 5.5 was worse (trade plans with an empty or USDC leg, invented legs, Execute chips after the user declined, about twice the latency); Sonnet 5.5 matched on trace cases at about 20x Haiku's cost; both Gemini Flash models flipped trade direction.

On the 100 inputs, chart assets matched prod in 85 and trade plans in 96. Prod leaked a stale asset in about 4 of 100; with the marker, none.

## What Fooled Us

The written answer was right, so the failure looked like a composer problem. It only showed up in the second pass that builds charts and chips.

## Remaining Unknowns

- **Known weak point, not fixed:** when the reply directly before the ask is a headlined trade pitch with a direction ("short QNT"), then HYPE/ETH is asked, the extractor still adds a QNT chart and chip (5 of 5). That shape did not leak in the sample (0 of 33 inputs where the previous reply named an unrelated asset), so the real rate is unknown but could be a few percent. Quoting the current ask at the end of the instruction fixed it but was worse on 7 of the 100 and better on 4, so it was not shipped. The likely difference (a headlined pitch vs tables/recaps) is untested.
- The awareness eval and a live Telegram end-to-end turn were not run.
- 3 apps/agent suites (26 tests) fail on `development` too; unrelated.
- `tradePlan` direction and chip wording stay model judgment with no deterministic check.

## Related Work

[[8th Oct 2026]]
