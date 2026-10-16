# Planner Fallback Sent a Trade-Idea Request to generalChat After Auto-Task History

Status: diagnosed, no fix yet *(issue state superseded, see Update 2026-10-14 below)*. Issue: [#1118](https://github.com/pear-protocol/agent-pear-v2/issues/1118)

## What the user saw

A trade-idea request got "I don't have a fresh trade signal" with no scan run. Prod trace `67de189f271f8a3af6b4d36c994d3691` (2026-10-13 07:55 UTC, Telegram DM).

## Confirmed

- Primary planner (Cerebras `gpt-oss-120b`) looped in `dependencies.research` and hit the 4096 token cap (`finishReason: length`). Invalid JSON, call failed.
- Fallback (`claude-haiku-4.5`) returned a valid plan: `generalChat: true`, all nodes null.
- Because the plan is valid, the recovery from [[My Work at Pear Protocol]] (2026-07-13, only on `planner_no_output`) never fires.
- Real-path replay (`planner.invoke`, Haiku forced, exact trace messages): full history `generalChat` 20/20. `[auto-task]` messages removed: `tradeExecution` 10/10. Last user message only: `tradeExecution` 8/10, `pair`+`tradeExecution` 2/10.
- Primary loop did not recur in 10 real-path runs with the same history.

## Corrected along the way

First replay used plain `json_object` output and routed correctly 9/9, which wrongly suggested history was not the cause. It skipped the real structured-output path (`Output.object`). The real-path run overturned it. Lesson: replay through the actual `planner.invoke`, not a lookalike request.

## Unknown

- Why `gpt-oss-120b` looped on this input.
- Whether auto-task history also degrades the primary at some rate (10 runs only).

## Possible fixes (none implemented)

Strip or collapse `[auto-task]` messages for the planner, prompt rule that trade-idea requests are never `generalChat`, treat `generalChat` without clarify on a trade-idea query as a failure and retry, log `finishReason: length` on the primary. Needs `PLANNER_TEST_SET=both` and the awareness eval.

Related: [[Kalshi Odds Query Fell Into a Planner Routing Dead Zone With No Empty-Plan Fallback]], [[Generic US-Stock Picks Refused as Crypto-Only, Then Reported as a Data Outage]]

## Update 2026-10-14

- [#1118](https://github.com/pear-protocol/agent-pear-v2/issues/1118) was closed 2026-10-13 13:41 UTC as completed, by me, with no comment and no linked PR. A PR search for #1118 returned nothing, and the PRs merged that day (#1117 chat missing-message guard, #1119 X media retry) are about other failures.
- Not verified: whether the planner fix was shipped some other way, or the issue was closed for another reason. The "Possible fixes" above are still unimplemented as far as GitHub shows. Check the planner code on `development` before treating this as fixed. See [[13th Oct 2026]].
