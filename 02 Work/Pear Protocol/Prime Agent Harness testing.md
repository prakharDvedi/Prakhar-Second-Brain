# Prime Agent Harness testing

- Status: finished for now. One full run of the final setup (full-4a). A second run (full-4b) did not complete. Decision: drop Prime Agent as a runtime for now. (extended 15 Oct evening: full-4c run and quality judging, see Update below. Decision unchanged.)
- Dates: 2026-10-14 to 2026-10-15
- Project: harness evaluation for Agent Pear, side study next to [[Agent Pear Revamp]]
- Notion write-up (private draft, same content in short form): https://app.notion.com/p/3fa4641ea90c81c3974fff12245cb4f4
- Work files (outside the repo): `C:\scratch\pear-replay` (start with `REPORT.md`) and `C:\scratch\trial-harness`

## tl;dr

Tested Prime Agent (Prime Intellect's loop plus Python scratchpad agent) in two ways. First as a coding tool on the #1096 task, then as a possible runtime for long, multi step Agent Pear chats by replaying one real production session through it.

- It was slower than our advanced mode in every run. Final setup: 2,762 s against 1,403 s for the 13 turns, slower on all 13.
- Input tokens were 5.7M in the first run and 1.87M in the last, against 1.97M for production. So that part got fixed.
- It writes about 1.4x the output (136k against 95k). Output is most of the turn time, so this is why it stays slow.
- Answer quality ended up about level with production. Better on honesty and on catching changes early for most of the session, weaker on the last two turns, which are the stale ETH close test.
- By the bar I set before starting (faster and more accurate, or not worth it), it fails on faster.

## Why I tested it

Wanted to know if a runtime built around a live Python scratchpad and subagents helps long tasks that need memory, like the multi step book clean up users do in chat. I did not want to adopt it on the strength of the vendor blog alone. The blog summary is in `prime-agent-blog-explained.md` in the repo root (untracked, not committed). Every performance number in the blog is the vendor's own claim.

No branch for it exists in this repo. Checked local and remote branches after a fetch on 15 Oct for `prime`, `rlm`, `repl`, `refine`, `continual`, `improv` and found nothing related.

## Work log, in order

### 1. Coding trial on the #1096 task (14 Oct, found on disk, I did not run or review these)

Same task, same model (`openai/gpt-6-luna`), scratch clone `C:\scratch\pear-trial` at the parent of PR #1096. Task: find why the Hetzner asset-index job logs `Tokenomist mapping refresh failed: Connection is closed.`, fix it, add tests, do not commit. A typecheck gate ran after each attempt.

| run | time | cost | files changed | note |
|---|---|---|---|---|
| claude-1 | 803 s | $0.091 | 2 (+60/-1) | gate passed |
| claude-2 | 618 s | $0.068 | 2 (+45/-1) | gate passed |
| prime-1 | 834 s | $0.003 | none | provider 520 error early |
| prime-1b | 1,801 s | $0.079 | 2 (+55/-1) | hit the 30 minute cap. The gate hit a Windows path bug (exit 127) once, then passed |
| prime-2 | 710 s | $0.039 | 2 (+57/-2) | last exit code 127 in its meta, final gate passed |

The real PR touched 5 files (`coin-index` service and spec, `main.ts`, and the two tokenomist files). Every trial run touched only the two tokenomist files. I only compared the file lists. I have not read the diffs, so whether the root cause matched is unknown.

### 2. Looked at real production traces (15 Oct, Langfuse, environment production, last 14 days)

- Standard mode (`dag.runAsync`): 1,500 turns, median 10 s, p90 38 s, p99 63 s, median cost about $0.013.
- Advanced mode (`dag.runAdvancedAsync`): 266 turns, median 77 s, p90 143 s, p99 286 s.
- The advanced mode cost recorded in Langfuse is not trustworthy (about $0.002 for multi minute runs). Used token counts instead.
- Searched 1,465 user turns across 255 sessions for memory complaints. Almost none. The real long task failure found was in one session (below). Another user retyped the same "analyze all my trades" request 11 times over four days, which is a standing task done by hand.
- Production keeps a 15 message window plus a recall summary. Input tokens grow inside a turn (each model call resends everything so far), not across the session.
- One advanced turn traced end to end (205 s): six main model calls (13k to 35k input each) did about 95% of the time, tools about 12 s. Two long reasoning passes were 131 s of it. Speed is mostly output tokens and the number of sequential model rounds.

Session chosen for the replay: `46e70278-0b42-49b5-96c3-9c6612950a7d`, advanced mode, turns 2 to 14 (Oct 5, 16:38 to 17:09 UTC). Turn 1 is a different day, skipped. A user manages a large Lighter book through a 5 item plan, one confirm ticket per turn. At the end the ETH short closed at 17:05:18 and the stop confirm landed at 17:05:33.

### 3. Built the replay harness (15 Oct)

- A local MCP server that serves the recorded tool results. 28 tools with the real names, descriptions and schemas pulled from the traces. Never serves a result recorded in a later turn. Logs every call and whether it was served from the same turn, an earlier turn, or not at all.
- First version served a result only if it fully covered the request. Later versions added a `USDT` alias, the schema default interval, nearest recorded interval (flagged), valid JSON when combining recordings, and a neutral reply for `stage_trade`.
- Same model as production advanced: `qwen/qwen3.8-flash`, thinking high, through OpenRouter. Could not pin the provider like production does.
- Prompt: the real production system prompt from the trace (turn 2 version) with the injected "conversation so far" summary left out, because that summary is the memory layer being tested. The recent trade tickets block is put in each user message instead.
- Prime reaches the tools from its Python scratchpad over MCP. Production calls them natively. That extra hop is part of the comparison.
- Not in the Prime setup: the two helper models production uses (`tradeExecution`, `extract`), the leak guard, the grounding check, the forced final answer budget, the provider fallback.

### 4. Runs

| run | what it was | result |
|---|---|---|
| smoke, smoke2 | turn 2 only | worked. 14 to 18 model calls, 356k to 514k input, 281 s |
| full-1 | one Prime call per turn, each forking the previous session | 1,842 s, 5.69M in, 94k out, $0.30 |
| full-2 | one long running Prime process, shorter runtime prompt, retry on stream drops | 1,637 s, 5.13M in, 98k out, $0.15 |
| full-3 | history shaped like production's | started, partial files only, not analysed by me |
| full-4a | fresh session per turn with recent turns as plain text, saved notes carried over, neutral `stage_trade` reply, circuit breaker on repeated failed calls | 2,762 s, 1.87M in, 136k out, $0.13 |
| full-4b | repeat of full-4a | did not complete (empty turn 2 in its log, nothing running) |

Production for the same 13 turns: 1,403 s, 1.97M input (1.31M in the main loop), 95k output, 84 model calls (52 in the main loop).

full-1 faster than production on 3 of 13 turns, full-2 on 5 of 13, full-4a on 0 of 13. full-2 never compacted its context (0 compactions, 153k at the end). Cause unknown. full-4a keeps each call between 24k and 44k.

full-4a is not stock Prime. A small circuit breaker was patched into its MCP code. It also passes history the way production does, so it no longer tests Prime's own memory.

## Quality

I read all 13 pairs of answers for full-1 and full-2 with the full production text, and all 13 for full-4a. My first pass used production answers cut off at 500 characters, so I re scored production. Not blind. Scores 1 to 5:

| | production | full-1 | full-2 | full-4a (13 turns) |
|---|---|---|---|---|
| grounding in the book | 3.5 | 3.5 | 4 | 3.5 |
| tracking plan and changes | 4 | 4.5 | 4.5 | 4.5 |
| honesty and calibration | 3 | 3.5 | 4.5 | 3.5 |
| concise, finishes the task | 4.5 | 3 | 3 | 4 |
| caution before money moves | 3.5 | 4.5 | 4.5 | 4 |

For turns 2 to 10 full-4a was higher (grounding 4, honesty 4.5, caution 4.5).

What Prime did better:
- Flagged the ENA position growing from 500k to 900k in turn 4 (full-2 and full-4a). Production's turn 4 table shows it without comment and first flags it in turn 7.
- Right margin percentages. Production's "% margin" column is really uPnL divided by entry notional (ETH short shows +0.16% where margin based is +7.9%). Confirmed by arithmetic on the trace data. Worth checking in code.
- Turn 13 order of events: close at 17:05:18, stop at 17:05:33. Production's turn 13 says it armed the stop "seconds before" the close.
- Turn 14 (full-2): said the cause of the close is unidentified, did not promise to arm a stop after a fill.
- Says plainly when data is missing.

What production did better:
- Shorter, clear "2 of 5, 4 of 5" progress lines, staged the re entry in turn 14.

Where each went wrong:
- full-1, turn 14: "someone or something holding your keys is trading". Not supported.
- full-4a, turn 13: "closed at market by you". Only the USER tag supports that, and the user says they did nothing.
- full-4a, turn 14: said Pear was tracking 44.6% of the real leg and that the position had been added to outside Pear. Not supported. Entry quantity is about 206.78 ETH and exit quantity is 231.78, a gap of exactly 25 ETH. The 103.39 was only the half left after the 16:44 trim. Production and full-2 flagged the gap as unreconciled and did not guess.
- full-4a, turn 14: staged an ENA stop at $0.2150 that would cost about 61.7% of equity if hit.
- production, turn 9: swapped the user's ETH/BTC pair for XRP/BTC on its own after the exchange refused. Turn 14 promised to arm a stop the moment a trade fills, which it can't do.

## Problems hit and fixes

- Prime panics on Windows when resuming a session (`-c` and `-r`, `windows_pipe.rs:116`). `--fork` of the previous session file works. Later runs avoided resume by passing history as text.
- The session "lease" entries are folders, so `rm -f` never cleaned them. Needs `rm -rf`.
- `prime-agent -p` prints the answer but the process keeps running. The driver ends a turn once output and the session file are idle for 20 s.
- A fresh state folder had no model catalog, so Prime capped output at 4096 tokens. Copying the cached catalog fixed it (131072 for this model).
- Windows quirks: UTF-8 stdout needs forcing, bash strings eat single backslashes (use `cygpath`).
- Langfuse rate limits (429) when pulling many traces. Needs backoff.

## Findings about production, not fixed

- Suspected display bug: the "% margin" column is uPnL over entry notional.
- Advanced mode cost in Langfuse is under recorded (suspected, tokens used instead).
- The `tradeExecution` helper wrote "closed by your manual action" in its staging text with no evidence for who closed it.
- The confirm receipt "Exit armed on the position" is code written. It said applied on a position that was already closed 15 s earlier. Not changed.
- The closed trade record has a 25 ETH gap between entry and exit quantity. Cause unknown.
- Production swapped a trade leg without asking (turn 9).

## Caveats

- One run of the final setup, my own harness, answers not scored blind.
- Even after full-4a, the user's recorded Confirms refer to production's tickets, so from about turn 5 Prime's world diverges from its own plan. That shows how it copes with surprises. It is not a like for like test of plan tracking.
- Wall time depends on provider load. Token and call counts are steadier. The run notes said the provider was a bit slower during full-4a (about 53 tokens per second against 62), roughly 15% of the gap.
- Prime has the scratchpad and MCP hop and none of production's helper models or guards.

## Open

- Why compaction never fired in full-2.
- No second full run of the final setup (full-4b did not complete). full-3 was not analysed.
- Not checked: the coding trial diffs, and whether Prime's numbers hold up against the recorded book one by one (spot checked only).

## Decision

Drop Prime Agent as a runtime for now. Input and quality got to about level with production, but it is about 2x slower because it writes more, and faster was part of the bar. Revisit only if the output gap closes.

Ideas worth taking for the current agent instead:
- a stored plan checklist for multi step jobs, so users stop asking "done?"
- batch tool calls to cut the number of sequential model rounds
- re check live state right before saying anything is done or armed

## Related

[[Agent Pear Revamp]] · [[13th Oct 2026]] · [[14th Oct 2026]] · [[15th Oct 2026]]

## Update 2026-10-15 (evening): full-4c and quality judging

Added after the sections above, which stay as written. Work files are still in `C:\scratch\pear-replay` (`runs\full-4c`, `score.py`, `blind\`).

### full-4c: output-cutting changes on top of full-4a

Changes (all outside the repo):
- `book()` helper in Prime's `rlm/mcp.py` (4a version kept as `mcp.py.4a`): account, positions and open orders in one parallel call, returned as about 2k characters against about 25k raw.
- Per-turn cache for repeated identical market-data calls. Account, history and `stage_trade` calls are never cached. `save_carry` clears it.
- Carry note capped at 800 characters saved and 900 passed on (was 2,500 and 3,000).
- Prompt (`prime_tailored_prompt4c.txt`): use `book()`, keep reasoning short, print only needed numbers, notes about 600 characters. Driver `run_rpc4c.py`.
- Turn 2 smoke test first: 278 s against 287 s in full-4a.

Result, 13 turns, `runs/full-4c`. No errors, no retries, no dropped responses:

| | production | full-4a | full-4c |
|---|---|---|---|
| answer time | 1,403 s | 2,722 s | 1,709 s |
| input tokens | 1.97M | 1.87M | 1.64M |
| output tokens | 95k | 136k | 93k |
| turns faster than production | | 0 of 13 | 3 of 13 (8, 12, 14) |

4c is about 1.22x production's time, down from about 1.97x. Slowest gaps: turn 2 (297 s against 205 s) and turn 13 (189 s against 151 s). One run of this setup, still patched Prime, provider speed varies between runs.

### Quality judging

Three different readers, each on production vs 4c answers. Results disagree, so none of them settles it alone.

- **gpt-6-luna, all 13 turns, both A/B orders, partial recorded data** (account, positions, orders, trade history only). Cost about $0.07. Picked 4c on 7 turns, production on 1 (turn 13), 5 ties. Scores 1.5 to 2.4 for both and 227 unsupported claims flagged for each side, because it could not see TA or derivatives data. Discounted.
- **gpt-6-sol, turns 9 and 11 to 14, all recorded tool data, both orders.** Cost $0.59 (186k in, 22k out). Production won 3 (turns 9, 12, 13), 4c won 2 (11, 14), same pick in both orders on all five. Mean 1 to 5: grounding 2.40 vs 1.90, honesty 2.00 vs 1.70, plan tracking 2.90 vs 2.30, concision 2.40 vs 1.80, caution 2.00 vs 2.10 (production vs 4c). Numbers in answers not found in the recorded data: 28 of 140 for production, 74 of 185 for 4c (derived numbers count too). Its turn 12 claim that 4c misstated the staged stop price could not be confirmed from the ticket log, which carries no price.
- **My own read (Claude), all 13 turns, shuffled labels, key opened after scoring.** Not fully blind, I recognised some production answers from these notes. Picks: 4c on 9 turns (2, 4, 7, 8, 9, 10, 11, 12, 13, several only slightly), production on 1 (turn 6), ties on 3, 5, 14. Scores 1 to 5, production vs 4c: grounding 3.5 vs 4, plan tracking 4 vs 4, honesty 3 vs 3.5, concise and finishes 4 vs 3.5, caution 3 vs 4. Average 3.5 vs 3.8.
- **Opus 5.5 was NOT used as a judge.** It was proposed but never run (about $2 on the first design), so no Opus 5.5 numbers exist.

Turns 13 and 14 (stale ETH close), my read: 4c gets the order right in turn 13 (close 15 s before the stop landed) but stages an unasked 400k ENA trim. In turn 14 it guesses the close came from outside the chat and tells an unsupported partial-fills story. Production gets the order wrong in turn 13, then in turn 14 admits the "armed" claim was wrong and flags the entry and exit size mismatch, but asserts what `PARTIAL` means, promises to arm a stop on fill and stages a new 50x short. I scored it a tie.

Production-side errors seen while reading: turn 9 swapped the user's pair for XRP/BTC blaming an exchange rejection, while its own turn 10 says the real cause was a $20.94 margin shortfall.

### Reading of the three together

Speed: 4c is 1.22x production, not level, so it still fails the "faster" bar. Quality: roughly level. Luna and my read lean 4c, Sol leans production on its five turns. The decision to drop Prime as a runtime stands.

### Still open

- No single judge is trusted. A stronger judge (Sol or Opus 5.5) over all 13 turns with full data is not run.
- One run of 4c. Turns 11 to 14 are still confounded by the replay serving production's tickets.
- full-3 not analysed, #1096 coding trial diffs not read (unchanged).

## Update 2026-10-16: latency work, runs 5 and 6

- Latency analysis of 4c (`C:\scratch\pear-replay\latency\ANALYSIS.md`): the extra time was the Python the model wrote between tool calls (about 23k tokens vs about 4k of native tool args in production), 38% of it the model writing its own notes. 4c already thought less than production (57.0k vs 64.3k reasoning tokens, exact counts from OpenRouter).
- Provider: OpenRouter's generation lookup shows 68 of 68 traceable 4c calls were served by Alibaba, the same provider production pins. The earlier line "could not pin the provider like production does" still holds for config, but the provider was the same anyway.
- Correction to the production-errors line above: the turn 9 data shows the ETH/BTC open really was refused for an opposite-side conflict on ETH. The $20.94 margin shortfall was the XRP/BTC attempt in turn 10.
- Run 5 (helper bug fixes, `risk()`/`at()`, harness-written notes) and run 6 (+ prefetch, status and staging rules, closed-trade read), both on turns 2, 3, 4, 5, 9, 11, 13. Same 7 turns: 4c 1,057 s / 57.9k tokens / 37 calls, run 5 699 s / 39.2k / 37, run 6 747 s / 36.0k / 25 (provider ran at 48 tok/s that run), production 675 s / 38.0k / 28.
- Blind judge (Claude, separate session, scores written before the key): run 6 4.40, run 5 4.09, production 3.63. Turn 9 is unfair (judge data included production's own staging attempts). Without it, run 6 and run 5 tie at 22.3 per turn vs production 18.5.
- Team write-up (private draft): https://app.notion.com/p/3fa4641ea90c81a0a452e26b69669ecf
- Decision not changed by this note. Still open: full 13-turn runs back to back, replay `stage_trade` that refuses what the engine refuses, judge data without production's write actions.
