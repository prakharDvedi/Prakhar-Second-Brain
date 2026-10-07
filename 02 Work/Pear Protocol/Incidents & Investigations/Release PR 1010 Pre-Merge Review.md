---
type: investigation
area: pear
status: open
created: 2026-09-26
---

# Release PR #1010 Pre-Merge Review

## Context

[#1010](https://github.com/pear-protocol/agent-pear-v2/pull/1010) promotes `development` → `main` (target merge Mon 2026-09-28). 216 files: single-loop advanced mode (Qwen primary, Luna fallback), auto mode on TG/web, the `HL_API_PROXY` rename (#1011), the `/menu` hub (#973), Hetzner one-shots (#988), prompt/KB audit.

Ran `/code-review high` (8 angles, **no verify pass**). Everything below is a **candidate from code reading, not a reproduced failure**. Line numbers are on `origin/development` at review time.

Focus: advanced loop, provider fallback, proxy rename, `/menu`, DAG leak guards, interest distiller. Prompt/KB wording only skimmed.

## Findings

### Pre-merge (highest risk)

1. **Proxy rename can break live trades on deploy.** `hl-api-proxy.ts:12` — `get("HL_API_PROXY") || get("PROXY_URL")`, now feeding the V3 execution SDK (`v3-execution.service.ts:85`). `HL_API_PROXY` previously meant the v2 reverse proxy for HL info reads. If a service still holds the old value on deploy, trade calls go to a proxy without the `X-Proxy-Target` contract until the env is fixed. Fix: set the env before merging, or prefer `PROXY_URL` until migrated.
2. **`/menu` "Confirmations off" is one tap.** `dm-menu.ts:55` confirms only `unlinkwallet` / `clear`; `confOff` (`dm-menu.ts:123`) runs `handleTradeCommand("confirmations off")` directly (`telegram.update.ts:944`). A stray tap turns off confirm-before-write. Same for `execOn`. Fix: add both to `CONFIRMED_ACTIONS`.

### Correctness

3. **Leak guard misses `get_account` and Qwen/Alibaba.** `get_account` is exposed to the model (`direct-tools.ts:521`) but not in `INTERNAL_TOKENS`; model tokens got `claude-opus` (`internal-leak-guard.ts:154`) but no `qwen`/`alibaba`, although Qwen is now primary (`models.ts:89`). Violates the repo CLAUDE.md rule to update the list on new tools/models.
4. **Forged-Confirm regex strips analysis lines.** `agent.ts:173` alternative `confirm…(?::|to)…(open|execute|…)` matches "- Confirm: open interest above $2B…". With no staged ticket: line deleted, "no ticket staged" notice appended, chips dropped. A different variant of the confirm-line false positive found in the #987 review (see [[25th Sep 2026]]).
5. **Fallback model gets Qwen-only reasoning options.** Budget check keys on `"qwen/"` (`final-budget.ts:29`) against the primary id (`direct-loop.ts:310`); after fallback to Luna every step still sends `reasoning.max_tokens` (`direct-loop.ts:354`) with a 16k output budget. Wrapper only strips `temperature` (`provider-fallback.ts:146`). Repo note (`final-budget.ts:28`): OpenAI maps that cap to an effort level.
6. **Leak-guard trip hides a staged ticket.** Advanced stream trip (`dag.service.ts:1619`) emits `pendingTicket: null` and returns; if `stage_trade` already ran, a live ticket exists with no Confirm/Cancel card shown.
7. **TG live draft can stick on a stale frame.** `flush()` sends directly (`tg-rich-streamer.ts:129`); a frame arriving meanwhile is parked as `pending` (`:258`) with no timer scheduled. Frames within 700ms of the last draw are dropped (`:264`). Cosmetic.
8. **Wrong model id recorded after fallback/retry.** `agent.ts:820` returns `finalProse.response?.modelId` (not updated on the direct retry path); the wrapper is built on the primary (`provider-fallback.ts:232`), so Luna-served turns log as Qwen.

### Cleanup

9. `POST_TOOL_REASONING_CAP = 4096` declared in `direct-loop.ts:43` and exported from `final-budget.ts:33`.
10. Reasoning-budget policy duplicated: `proseLoopSettings` (`final-budget.ts:42`) and `directPrepareStep`. #5 would need fixing in both.

## Checked and looked fine

- AI SDK `ai@6.0.205` merges `prepareStep` `providerOptions` with the call's (`mergeObjects`), so the per-step reasoning override does not drop the Alibaba provider pin.
- `stage_trade` / `manage_tasks` run through `execute_ops`, which sets `executeOpsTrace`, so the retry policy does not throw on a successful direct-mode stage.
- `/menu`, the portfolio aliases and the short DM command list are reflected in `awareness-invariants.ts`, the `telegram-commands.txt` KB doc and `pear-hot-facts.ts`.
- `quant.service.ts` now requires both rolling β and `lt_beta` > 0 for a direction — the fix suggested in the #992 review ([[25th Sep 2026]]).

## Status

Open. Findings reported to the session only; not posted to the PR, no fixes applied. Next: decide #1 and #2 before the Monday merge.

## Update (2026-09-26)

Posted findings #2 to #10 as a PR comment on [#1010](https://github.com/pear-protocol/agent-pear-v2/pull/1010), stated without proposed fixes. #1 (proxy rename) was left out on purpose.

Fixes for #2 to #10 opened as [#1013](https://github.com/pear-protocol/agent-pear-v2/pull/1013) (branch `fix/release-1010-review-findings` off `development` b24e5756). #1 still open.

## Update (2026-09-28)

[#1013](https://github.com/pear-protocol/agent-pear-v2/pull/1013) merged into `development` (2026-09-28 04:47 UTC). Prudhvi opened [#1021](https://github.com/pear-protocol/agent-pear-v2/pull/1021) (`fix/promotion-review-gaps`) as a further follow-up to the #1010 review. Among other changes it moves the interest-distiller per-day Redis claim from #988 into a DB state table.

Safety review of #1021 (code reading, not reproduced): no regressions found. Two gaps it does not close, both already on `development`:
- confirmations-off inline execution: a generic thrown error after submission returns `{ok:false}` without `doNotRetry`, so the trade node's inner loop can issue a second write.
- advanced delivery: chart/link generation in `dag.service.ts` after the agent returns can drop the staged ticket or trade outcome if it throws.

Posted both gaps as a PR comment on #1021 (problem + evidence only, no fixes): https://github.com/pear-protocol/agent-pear-v2/pull/1021#issuecomment-5864608217
