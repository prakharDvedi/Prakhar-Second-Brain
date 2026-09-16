# Resume Evidence

A factual evidence bank for work that may later be useful in resumes, interviews, status updates, or performance discussions. Draft entries, not polished bullets — pull from here when actually writing something.

> Fuller technical narrative for every entry below lives in [[My Work at Pear Protocol]]; entries here are the condensed, reusable shape. Ranked source list: [[My Work at Pear Protocol#Strongest Resume / Career Evidence]].

---

## Root-caused a silent trade-execution defect from a cross-lingual production trace

**Date:** 2026-07-09
**PR(s):** #248

### Problem
The streaming chat entry point (`dag.runAsync`) built its DAG context with hard-coded empty `userInput`/`history`, so an explicit user instruction never reached nodes that read those fields.

### What I actually did
Traced a Dutch-language production session where a user's explicit "I want 5% profit" turn was overridden by a documented 100%-PnL silent default one turn later. Root-caused it to the missing arguments and fixed it with a one-line-pair change mirroring the unaffected synchronous path.

### Confirmed outcome
New regression spec (`dag.service.context-propagation.spec.ts`) verified to fail on the pre-fix code and pass after, by reverting the change and re-running it.

### Technical significance
Found via direct trace reading across a language barrier, not from a bug report — and root-caused to the actual mechanism rather than patched at the symptom.

### Evidence
- Work history: [[My Work at Pear Protocol#2026-07-09 — DAG context bugs: a fake telemetry alarm, and a real silent TP/SL default]]
- Daily log: [[9th Jul 2026]]
- Tests: `dag.service.context-propagation.spec.ts`, 12 suites / 66 tests green

### Confidence
High

---

## Designed a confirm-time margin safety gate

**Date:** 2026-08-24
**PR(s):** #569

### Problem
`open_basket` had no check that a fresh margin read still covered the plan at the moment of confirmation.

### What I actually did
Added a fail-closed gate reading fresh `availableToTrade` with a configurable buffer (100bps), positioned before any leverage or order write, with reduce-only adjustments deliberately exempted since they release margin rather than consume it.

### Confirmed outcome
Focused tests cover insufficient / unavailable / changed-balance cases.

### Technical significance
A genuine safety feature added ahead of a failure mode, not a reaction to one — the judgment to add friction only where real risk exists (and explicitly not on reduce-only).

### Evidence
- Work history: [[My Work at Pear Protocol#2026-08-24 — A confirm-time margin gate, a recurring ambiguous-symbol crash, and a hidden 2x-budget confirm]]
- Feature note: [[Features Owned#Trade execution]]
- Daily log: [[24th Aug 2026]]

### Confidence
High

---

## Found and closed a crash that was silently substituting a user's trade request

**Date:** 2026-08-24
**PR(s):** #584

### Problem
`open_position`'s TRIGGER/LADDER `condition` argument was a JSON string spread unchecked into the plan usecase — a malformed shape crashed with a raw `TypeError`.

### What I actually did
Found this while manually triaging five days of Langfuse ERROR-level traces. Traced a specific case where the model's malformed request crashed, then silently retried with a *different* condition type and succeeded — replacing the user's weighted-basket trigger with a single-symbol price trigger, with no error ever shown. Fixed by validating the parsed condition against the same Zod schema the MCP adapter already trusted, before it reached the usecase.

### Confirmed outcome
New `open-position.tool.spec.ts`: malformed shape now rejected pre-usecase with a named `PlanningError`; well-formed shape still passes through unchanged.

### Technical significance
The failure mode — silent correctness divergence rather than a visible crash — is the harder class of bug to even notice, and this one was found by deliberate manual investigation, not a user report.

### Evidence
- Work history: [[My Work at Pear Protocol#2026-08-24 — A single day of hardening the Lighter surface: wiring gaps, a doc that contradicted itself, and a crash that silently substituted the user's request]]
- Incident: [[TRIGGER-LADDER JSON Crash Silently Substituted the User's Trade]]
- Trace: `b9164502960cd3af356c603488ee5657`

### Confidence
High

---

## Closed a recurring 13-in-3-days production error at its remaining call sites

**Date:** 2026-08-13 – 08-24
**PR(s):** #431, #514

### Problem
An ambiguous HIP-3 ticker (listed under more than one deployer) crashed chart generation with a generic fallback error. After the chart path was fixed, the exact same underlying error kept recurring through every *other* DAG entry point (pair analysis, opens, general compose).

### What I actually did
Fixed the chart-rendering call site first (#431), then confirmed via a live Discord alert and a named trace that the identical message recurred 13 times in the 3 days after that fix — and closed the two remaining `DagService` entry points that shared the same uncaught error type (#514).

### Confirmed outcome
Full DAG suite green (8/8 files, 60/60 tests) after the second fix; new spec explicitly proves the fix isn't a blanket catch-all (an unrelated error still falls back to the generic message).

### Technical significance
Demonstrates checking whether a narrow fix actually covered the whole failure surface, rather than declaring done after the first observed occurrence.

### Evidence
- Work history: [[My Work at Pear Protocol#2026-08-13 – 08-14 — Chart edge cases: paging on bad input, an ambiguous ticker crashing the whole reply, and wrong display tickers]], [[My Work at Pear Protocol#2026-08-24 — A confirm-time margin gate, a recurring ambiguous-symbol crash, and a hidden 2x-budget confirm]]
- Incident: [[Ambiguous HIP-3 Symbol Crashed the Whole Reply, Repeatedly]]
- Trace: `1848c72d…` (initial), `0e25d734e9eb599b94a8b4c30ee51249` (recurrence)

### Confidence
High

---

## Disclosed a hidden ~2x margin exposure on leverage-clamped trades

**Date:** 2026-08-24
**PR(s):** #593

### Problem
When a leg's leverage cap forced a downgrade, the execution engine's notional-preserving margin re-derivation could roughly double a user's stated budget — the ticket only mentioned the leverage adjustment, never the margin delta.

### What I actually did
Traced to a specific user's actual numbers, confirmed the underlying engine behavior (notional preservation) was intentional rather than a bug, and fixed it at the correct layer — a prompt requirement that the composer state the margin change as its own sentence before the leverage-cap note.

### Confirmed outcome
Traced case: user asked for "$1,000 margin at 10x leverage," one leg capped to 5x, resulting margin came out to $1,999.95 with no standalone callout prior to the fix.

### Technical significance
A financial-trust issue traced to real numbers, and correctly diagnosed as a disclosure problem rather than a math bug — fixed without touching the (correct) underlying engine behavior.

### Evidence
- Work history: [[My Work at Pear Protocol#2026-08-24 — A confirm-time margin gate, a recurring ambiguous-symbol crash, and a hidden 2x-budget confirm]]
- Decision: [[Confirm-Time Margin Gate]]
- Trace: `5a3192cee9b685c8a3e6e8201caf58ad`, session `TG_DM1110963912`

### Confidence
High

---

## Quantified and fixed retry-storming on rebalance weight validation

**Date:** 2026-08-30 (open)
**PR(s):** #642

### Problem
Rebalance weights were hard-rejected whenever they didn't sum to exactly 1, even though the sum is arithmetic the LLM derives to 4 decimals across 13-15 legs.

### What I actually did
Measured the actual cost of the strict check — 4 traces, 21 rejected `rebalance_position` attempts, drift up to 15%, one trace showing 10 back-to-back self-correction attempts — before deciding to rescale proportionally instead of rejecting.

### Confirmed outcome
Rescaling is provably safe: per-leg weights are already `zod .positive().max(1)`, so the sum is always > 0 before division, and a clean sum stays bit-exact (no regression on currently-passing inputs).

### Technical significance
A data-backed validation-design decision with a stated, provable safety property — not a guess at what "feels" more permissive.

### Evidence
- Work history: [[My Work at Pear Protocol#2026-08-30 — Rebalance weights were rejected instead of rescaled (open, current branch)]]
- Decision: [[Rebalance Weight Validation — Reject vs Rescale]]
- Status: not yet merged as of 2026-08-30

### Confidence
High (evidence); status still open

---

## Removed an unsigned-webhook fallback from the signals security surface

**Date:** 2026-08-20
**PR(s):** #555

### Problem
Elfa signal triggers could arm without HMAC verification under a fallback path, an authentication-oracle / replay risk on production webhooks.

### What I actually did
Removed the unsigned execution lookup fallback so every armed production webhook requires HMAC verification, added a per-webhook signing secret at arm time, and tagged persisted queries so pre-existing armed rows are detected as drift and re-armed post-deploy.

### Confirmed outcome
Focused controller + TriggerManager suites: 80/80 passing.

### Technical significance
A concrete security-hardening decision — closing a real trust bypass with an explicit rollout/re-arm plan, not just a code change.

### Evidence
- Work history: [[My Work at Pear Protocol#2026-08-18 — Elfa signal triggers accepted invalid conditions, and close-all could silently downgrade a TWAP to market]]
- Decision: [[Elfa Webhook Trust Model]]

### Confidence
High

---

## Built an asymmetric, downgrade-only verification gate for reduce-only failures

**Date:** 2026-07-27 – 07-29
**PR(s):** #337

### Problem
Every "Reduce only order would increase position" rejection was treated as a hard FAILED, even when the user's actual requested end-state had already been achieved by other means (a sibling close, a direct-exchange trade).

### What I actually did
Built a gate that re-reads the freshest live position and downgrades FAILED to benign *only* when every targeted leg is provably at target — explicitly designed so it can only ever downgrade, never upgrade uncertainty into success.

### Confirmed outcome
Fully backward compatible — no wire/status/SDK contract change; only one previously-FAILED outcome class now settles benign, and only under a positively-confirmed condition.

### Technical significance
Careful, explicitly-reasoned safety design — the PR itself enumerates exactly which conditions must hold for the downgrade to fire and which don't.

### Evidence
- Work history: [[My Work at Pear Protocol#2026-07-27 – 07-29 — A "would increase position" rejection didn't mean the trade failed; Twitter labeling failures were unreadable]]
- Decision: [[Reduce-Only Verification Source]]
- Lesson: [[Engineering Lessons#A downgrade-only gate is safer than a symmetric recheck]]

### Confidence
High

---

## Tuned an LLM-output security guard against a reproduced false-positive/negative pair

**Date:** 2026-08-28 (open)
**PR(s):** #639 + same-day follow-up

### Problem
The internal-leak guard's unconditional substring deny-list blocked legitimate AI-industry reporting (a reproduced production incident: a news summary naming "GPT-5.6" and "OpenRouter" got its whole reply nuked).

### What I actually did
Moved model/provider names into a sentence-scoped check firing only on an actual first-person disclosure frame. A same-day follow-up review then caught that the widened version had narrowed too far — missing real phrasings like "based on X" — and widened the frame again without restoring unconditional blocking.

### Confirmed outcome
60/60 tests including adversarial and benign near-miss cases (news about GLM/GPT-5 passes; "based on X" now correctly flagged).

### Technical significance
Shows the harder half of security tuning — catching an over-correction via review before it ships, verified in both directions.

### Evidence
- Work history: [[My Work at Pear Protocol#2026-08-28 — The internal-leak guard blocked legitimate AI-industry reporting; signal failures collapsed two mechanisms into one alert; delivery ordering had no visibility (open)]]
- Decision: [[Internal-Leak Guard — Contextual Frame vs Unconditional Deny-List]]
- Lesson: [[Engineering Lessons#Tuning a safety check in one direction can silently break the other]]

### Confidence
High (evidence); status open as of 2026-08-30

---

## Built the shared LLM-error observability layer later used to diagnose a Gemini incident

**Date:** 2026-07-15, extended 2026-07-20
**PR(s):** #258, #300

### Problem
Every LLM call failure logged only `err.message` — the fields that actually explain a provider failure (status, body, provider, request id) were dropped.

### What I actually did
Built one shared `formatLlmError()` utility, deliberately best-effort (never throws itself even on a circular error shape), and wired it through every failure site. Extended it after validating against a live Gemini "Corrupted thought signature" failure that needed a `data`-field fallback OpenRouter-specific quirk to surface the real provider name.

### Confirmed outcome
The work history entry states this enrichment was "later used to diagnose the Vertex↔AI-Studio reroute behind the Gemini pinning work" — noted here as stated in that entry; whether that follow-on branch (`fix/pin-gemini-backend`) fully resolved the underlying reroute is not established in currently available notes.

### Technical significance
Infrastructure investment that paid into a real later diagnosis, not observability added for its own sake.

### Evidence
- Work history: [[My Work at Pear Protocol#2026-07-15 — LLM failures went dark past "the model errored"; the trade-execution prompt contradicted itself on leverage]], [[My Work at Pear Protocol#2026-07-20 — Chasing a recurring Gemini failure meant enriching the error pipeline twice]]
- Incident: [[Chasing the Corrupted Thought Signature Failure]]
- Tests: `llm-error.spec.ts`, 6/6 then 9/9

### Confidence
High (the tool and its adoption); Medium (the downstream "diagnosed the reroute" claim — sourced from the work-history note's own wording, not independently re-verified here)

---

## Breadth across a full new execution venue (Lighter integration)

**Date:** 2026-08-18 – 08-24
**PR(s):** #556, #557, #558, #559, #560, #564, #574, #587, #588, #589 (own) + authored commits inside teammate-opened #545–548

### Problem
Adding Lighter as a second execution venue surfaced Hyperliquid-only assumptions across pricing, margin mode, account binding, KB wording, and Telegram surface copy.

### What I actually did
Touched the problem in five or six structurally different places: a venue-aware price-read tool, invocation-scoped account binding, replace-not-merge asset-cache semantics, KB doc correction, Telegram multi-account UX, and platform-scoping fixes across two agent tools — spanning `apps/agent`, `packages/execution`, and the KB pipeline.

### Confirmed outcome
Verified live against the gateway that 227/227 Lighter instruments report `marginMode="any"`; KB eval held at retrieval @1/@3/@5 97/100/100 with all new Lighter eval queries ranking first; dozens of focused tests passing across the individual PRs (see work-history entries).

### Technical significance
Marked **Derived**: the individual facts (which files/PRs touched what) are each explicitly confirmed; the summary claim of "breadth across a full venue" is a synthesis across those confirmed pieces, not a single stated fact.

### Evidence
- Work history: [[My Work at Pear Protocol#2026-08-18 – 08-20 — Bringing up Lighter as a second execution venue]]
- Feature note: [[Features Owned#Multi-venue execution: Lighter integration]]

### Confidence
Medium — breadth claim is derived across multiple confirmed PRs, not a single stated outcome

**Update (2026-09-10):** User-stated business outcome — Lighter integration behind over $63M in trading volume in the month since launch. Not independently re-derived from a dashboard/export in this vault; treat as user-reported pending a linked metrics source.

---

## Live-verified every emergency model swap before merging

**Date:** 2026-08-05 – 08-13
**PR(s):** #384, #386, #387, #436, #437, #399, #478, #479

### Problem
Three different upstream models hit provider-side rate limits or capacity walls in the same production window, each requiring an emergency swap.

### What I actually did
Distinguished rate-limiting from deprecation before each swap, and validated the replacement live before merging every time — a bakeoff run, an 8-request concurrent-burst smoke test, or a tool-loop latency check, depending on the node.

### Confirmed outcome
Flash Lite bakeoff 1/1; Luna 8/8 concurrent burst with no 429s; Gemini advanced-mode smoke test ~5.4s with streamed reasoning.

### Technical significance
Operational discipline under repeated pressure — the same verification rigor applied on the fifth emergency swap of the week as on the first.

### Evidence
- Work history: [[My Work at Pear Protocol#2026-08-05 – 08-13 — Model rate-limit whack-a-mole: Gemini → Luna → Flash Lite, and MiMo → Gemini → Luna]]
- Decision: [[Model Routing Under Provider Instability]]
- Lesson: [[Engineering Lessons#Live-verify a replacement before trusting it, especially under pressure]]

### Confidence
High

---

## Root-caused and hotfixed a prod config fan-out that silently nulled basket stats

**Date:** 2026-09-02
**PR(s):** #677 (→ `main`), #678 (→ `development`)

### Problem
A production `MadeBaskets` cron kept raising a `No baskets generated` alert while retrying, but baskets were in fact being generated. One `context` env var (`STATS_BASE_URL`) fed two consumers on two different services — `GET /categories` (categories service) and `QuantService.getBasketStats()` → `POST /stats/basket` (Rust stats service) — and an earlier change repointed both onto the categories host, so `/stats/basket` 404'd and basket stat columns persisted null.

### What I actually did
Root-caused it to the shared-var fan-out (the alert named a failure that wasn't happening), confirmed the two routes live on different services with live probes (`GET /categories` 200 vs 404; `POST /stats/basket` 201 vs 404), then split the config into `CATEGORIES_BASE_URL` and `STATS_BASE_URL`, renamed the categories-only consumer's var, and hardened both to `z.url()`. Shipped as an emergency hotfix to `main`, then backported to `development` to re-level the branches. Verified prod Railway vars and deploy health via CLI without reading any secret values.

### Confirmed outcome
`tsc` clean both apps; tokenomist-index spec green; both prod services redeployed SUCCESS with the new vars present; 8/8 prod services Online. Forward-only — already-nulled rows are not backfilled.

### Technical significance
Production reliability + end-to-end ownership: took a misleading alert, found the real layer, and shipped + deployed + verified across code, prod config, and deploy under hotfix pressure — while respecting a no-read-secrets boundary throughout.

### Evidence
- Incident: [[One Service URL Fed Two Different Services, Nulling Basket Stats]]
- Work history: [[My Work at Pear Protocol#2026-09-02 — One service URL fed two different services, nulling two-sided basket stats]]
- Daily log: [[2nd Sep 2026]]
- Lesson: [[Engineering Lessons#One config value shared by two consumers on different services is a hidden fan-out]]

### Confidence
High (evidence + prod deploy); backport #678 still open
