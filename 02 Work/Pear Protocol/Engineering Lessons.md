# Engineering Lessons

Reusable lessons extracted from documented Pear Protocol work. Each generalizes beyond the specific incident it came from — the incident is the evidence, not the point.

---

### A capped telemetry field can manufacture a false incident

A monitoring field that truncates for display, but is named as if it carries the full value, will get misread as evidence of a real truncation.

**Context:** Langfuse traces capped `finalText` at 500 chars under that exact field name. A raw 753-char, fully-streamed, fully-persisted generation looked truncated to any consumer reading the trace.

**Why it matters:** A display cap and a real failure need visibly different signals (a length + a "capped for telemetry" flag), or the two become indistinguishable to whoever's debugging from the trace later.

**Evidence:** [[My Work at Pear Protocol#2026-07-09 — DAG context bugs: a fake telemetry alarm, and a real silent TP/SL default]]

---

### Two entry points meant to behave identically can silently diverge

Parallel code paths built for the same purpose (a sync call and a streaming call, in this case) are not guaranteed equivalent just because they call the same downstream function.

**Context:** The streaming chat path built its DAG context with hard-coded empty `userInput`/`history`; the synchronous path passed the real values. Both called the same node executor, so nothing at the seam complained.

**Why it matters:** A test asserting both entry points is only real coverage if it actually drives both and compares what they hand downstream — not just that each path "returns something."

**Evidence:** [[My Work at Pear Protocol#2026-07-09 — DAG context bugs: a fake telemetry alarm, and a real silent TP/SL default]]

---

### A capability that doesn't exist must never be implied to a user

Under pressure from a user's stated expectation, an LLM can fabricate an operational capability it doesn't have — including phrasing that sounds like polite compliance.

**Context:** After a timeout, the composer told a user "I'll keep attempting it... I'll let you know as soon as it's confirmed." No retry loop, scheduler, or notification hook exists behind a failed step. A standing user instruction ("if it times out, assume I want to retry") made the fabrication more likely, not less.

**Why it matters:** The forbidden-claim rule has to cover acknowledgment-shaped compliance too ("noted, I'll auto-retry from now on" is the same false promise in different clothing), not just the obvious first-person promise.

**Evidence:** [[My Work at Pear Protocol#2026-07-17 — Charts silently failed to text on HIP-3 assets; the composer promised retries it could never keep]]

---

### A downgrade-only gate is safer than a symmetric recheck

When re-verifying an outcome against a fresh read that might itself be stale or ambiguous, restricting the gate to one direction only keeps the failure mode safe.

**Context:** A reduce-only rejection was re-verified against a fresh live position read, but the gate could only ever move FAILED → benign, never the reverse. Every ambiguous case (non-zero target, still-open exposure, malformed data) preserved the original FAILED.

**Why it matters:** If the "fresh" data source can itself be wrong, a symmetric gate risks turning a real failure into a false success. A one-directional gate can only ever make things safer than they already were reported.

**Evidence:** [[My Work at Pear Protocol#2026-07-27 – 07-29 — A "would increase position" rejection didn't mean the trade failed; Twitter labeling failures were unreadable]]

---

### A hardcoded root-cause string in an error message is a guess wearing a lab coat

An error message that names a specific cause on every occurrence of a failure type is asserting something the code hasn't actually verified.

**Context:** Every PARTIAL FILL error blamed "$11 minimum" notional. One real incident's own numbers proved the guess wrong by 23x — the actual cause was an unrelated venue-side rejection.

**Why it matters:** A guessed cause in a user-facing or on-call-facing message actively misdirects the response. Report only what's verified (which legs filled, which didn't) — a generic "one or more legs failed" beats a specific wrong answer.

**Evidence:** [[My Work at Pear Protocol#2026-07-21 — A trade-execution trio: fractional leverage, a wrong root-cause string, and an unhandled Telegram rejection]]

---

### Fail-closed only where the account's actual state says so — not a blanket rule

A safety fix for one account population has to be checked against every other population sharing the same code path before it ships.

**Context:** A margin-backfill branch was wrongly firing for No-Dex-Abstraction accounts. The fix gated the branch on the account's real abstraction mode rather than removing the backfill outright — removing it would have broken the unified/dex-abstraction accounts it was originally built for.

**Why it matters:** "Remove the thing that's causing the bug" is often the wrong fix when the same code path serves multiple populations correctly. Gate precisely; don't delete broadly.

**Evidence:** [[My Work at Pear Protocol#2026-07-22 — HIP-3 orders were passing margin validation they could never actually fund]]

---

### Fresh measurements can change what a retry is actually retrying

A retry that independently re-measures state, rather than replaying the original evidence, may no longer be retrying the same logical event it was triggered by.

**Context:** A signal-execution retry could receive authenticated crossing evidence from a webhook, then take a fresh independent market measurement — by which point the value had already crossed back. The retry "failed" against a condition that had already reversed, not the one that triggered it.

**Why it matters:** Retries that repeat observation instead of preserving event evidence can amplify timing drift instead of confirming the original signal. Worth checking, for any retry: is it replaying the same fact, or re-deriving a possibly-different one?

**Evidence:** [[My Work at Pear Protocol#2026-08-28 — The internal-leak guard blocked legitimate AI-industry reporting; signal failures collapsed two mechanisms into one alert; delivery ordering had no visibility (open)]]

---

### A validation rule can be arithmetically correct and still be the wrong check

When a check's real purpose is expressing relative intent rather than exact arithmetic, enforcing the arithmetic precisely just burns retries without protecting anything.

**Context:** Rebalance weights were rejected whenever they didn't sum to exactly 1 — correct as a rule, but four traces showed 21 rejected attempts and up to 15% drift from an LLM sizing weights to 4 decimals. The caller's actual intent was the *relative* shares; the exact sum was arithmetic noise.

**Why it matters:** Before hardening a validation rule, check whether real callers are failing it for reasons that don't actually violate its intent — and measure the retry cost before deciding reject-vs-rescale is the right validation.

**Evidence:** [[My Work at Pear Protocol#2026-08-30 — Rebalance weights were rejected instead of rescaled (open, current branch)]]

---

### Distinguish rate-limiting from deprecation before swapping anything

A 429 (retriable) and a flat 502 (likely deprecation) are different failure modes that call for different responses, even though both look like "the model stopped working."

**Context:** Across a week of provider instability (Gemini → Luna → Flash Lite, MiMo → Gemini → Luna), each swap explicitly checked which failure mode was in play before choosing a replacement.

**Why it matters:** Treating the two as interchangeable risks swapping away from a model that would have recovered on its own, or sticking with one that never will.

**Evidence:** [[My Work at Pear Protocol#2026-08-05 – 08-13 — Model rate-limit whack-a-mole: Gemini → Luna → Flash Lite, and MiMo → Gemini → Luna]]

---

### Live-verify a replacement before trusting it, especially under pressure

Time pressure to ship a fix fast is exactly when skipping verification is most tempting — and exactly when a bad swap does the most damage.

**Context:** Every emergency model swap in the rate-limit sequence included a live check before merge — a bakeoff run, a concurrent-burst smoke test, a tool-loop latency check — even on the third or fourth swap of the same week.

**Why it matters:** Verification discipline that only holds up when there's no pressure isn't real discipline. The value of the check is highest exactly when it's most tempting to skip.

**Evidence:** [[My Work at Pear Protocol#2026-08-05 – 08-13 — Model rate-limit whack-a-mole: Gemini → Luna → Flash Lite, and MiMo → Gemini → Luna]]

---

### A narrow fix should be checked against sibling call sites of the same root cause

Fixing the one call site you happened to observe doesn't prove the underlying error type isn't reachable elsewhere in the same system.

**Context:** An ambiguous-symbol crash was fixed for the chart-rendering path first. It then recurred 13 times in 3 days through two *other* DAG entry points that threw the exact same caller-facing error, which the first fix never touched.

**Why it matters:** When an error type is caught at one call site, actively look for other places the same throw can propagate to, instead of waiting for the next occurrence to reveal them.

**Evidence:** [[My Work at Pear Protocol#2026-08-24 — A confirm-time margin gate, a recurring ambiguous-symbol crash, and a hidden 2x-budget confirm]]

---

### An unvalidated shortcut on one of two parallel adapters reopens a hole the other already closed

In a "one write path, two adapters" architecture, validation has to be enforced at the shared boundary — or drift between adapters quietly reintroduces the exact hole the architecture was meant to prevent.

**Context:** A TRIGGER/LADDER condition was validated with a real Zod discriminated union on the MCP adapter, but taken as an unchecked JSON-string shortcut on the agent/Telegram adapter for the same underlying write path.

**Why it matters:** "We already validate this" is only true if every adapter reaching the shared usecase actually does — check each adapter against the same schema, don't assume parity.

**Evidence:** [[My Work at Pear Protocol#2026-08-24 — A single day of hardening the Lighter surface: wiring gaps, a doc that contradicted itself, and a crash that silently substituted the user's request]]

---

### A silent retry that changes the request type is worse than a visible crash

An automatic retry that substitutes different parameters after a failure can look like graceful recovery while actually changing what the user asked for.

**Context:** A malformed weighted-basket trigger condition crashed with a raw `TypeError`. The model silently retried with a different condition type (a single-symbol price trigger) and succeeded — the user's actual request was replaced with no error ever shown.

**Why it matters:** Validate the input so a retry either succeeds on the *same* request or surfaces a real, visible error — never lets the system quietly answer a different question than the one asked.

**Evidence:** [[My Work at Pear Protocol#2026-08-24 — A single day of hardening the Lighter surface: wiring gaps, a doc that contradicted itself, and a crash that silently substituted the user's request]]

---

### Two checks meant to agree on the same fact need to derive it from the same source

If two tools reachable in the same conversation each independently default an unstated input, they can contradict each other on a fact the user never sees them disagree about.

**Context:** `validate_assets` told the model a leg was tradeable on Lighter; `quant_basket_stats`, reached from the same turn, defaulted the same unstated platform to Hyperliquid and flagged the same leg unsupported.

**Why it matters:** When two checks are supposed to agree on venue/scope/permission, bind them to the same resolved input rather than letting each pick its own default.

**Evidence:** [[My Work at Pear Protocol#2026-08-24 — A single day of hardening the Lighter surface: wiring gaps, a doc that contradicted itself, and a crash that silently substituted the user's request]]

---

### A wiring gap can look identical to broken logic

Before redesigning logic that "isn't working," confirm the existing logic is actually being invoked with the inputs it needs.

**Context:** `validate_assets` already supported Lighter through an `opts.allowLighter` flag — but the two call sites only passed it when `forcePlatform` was explicitly set, and on exactly the surfaces meant to use Lighter, `forcePlatform` is never set. The capability existed; the wiring to reach it didn't.

**Why it matters:** A feature that "doesn't work" is sometimes fully built and simply unreachable from the path that was supposed to use it — worth checking before rebuilding.

**Evidence:** [[My Work at Pear Protocol#2026-08-24 — A single day of hardening the Lighter surface: wiring gaps, a doc that contradicted itself, and a crash that silently substituted the user's request]]

---

### Observability helpers must never throw

A logging/error-formatting helper that can itself fail turns an observability improvement into a new failure mode, exactly on the path you can least afford one.

**Context:** The shared `formatLlmError()` utility was deliberately built best-effort: a circular or unexpected error shape falls back to a minimal safe object instead of throwing, verified with its own regression test.

**Why it matters:** Anything that runs inside a catch block has a stricter bar than normal code — it has to degrade, never propagate a new exception.

**Evidence:** [[My Work at Pear Protocol#2026-07-15 — LLM failures went dark past "the model errored"; the trade-execution prompt contradicted itself on leverage]]

---

### A stale knowledge doc doesn't fail loudly — it just starts lying confidently

A documentation fact describing product capability can become false the moment the underlying system changes, and nothing about the system will signal that it happened.

**Context:** KB docs stated triggers were "enforced by HL" specifically. That became actively false the moment Lighter existed as a second execution venue, and the agent kept repeating it to Lighter users until the doc was caught and corrected.

**Why it matters:** Docs describing what the product can do need to be revisited whenever the underlying system changes — a stale doc doesn't error, it just becomes a confidently wrong answer.

**Evidence:** [[My Work at Pear Protocol#2026-08-18 – 08-20 — Bringing up Lighter as a second execution venue]]

---

### Tuning a safety check in one direction can silently break the other

Loosening a detector to fix false positives can reopen false negatives it used to catch — the two failure modes need to be checked together, not sequentially.

**Context:** The internal-leak guard was first widened to stop blocking legitimate AI-industry news. A same-day follow-up review found the widened version now missed real self-disclosure phrasings ("based on X," "running on X") that the original, stricter version used to catch.

**Why it matters:** After loosening a binary safety check, run adversarial cases for *both* directions before shipping — not just the false-positive case that motivated the change.

**Evidence:** [[My Work at Pear Protocol#2026-08-28 — The internal-leak guard blocked legitimate AI-industry reporting; signal failures collapsed two mechanisms into one alert; delivery ordering had no visibility (open)]]

---

### A structured-output schema is also a truthfulness boundary

A field the model structurally cannot answer honestly at generation time should not exist in the schema at all — asking for it guarantees fabrication.

**Context:** Basket tag taxonomy explicitly excluded state labels (Trending / New / Hot) because the model has no way to know them when generating a brand-new basket. Making them structurally impossible removed the temptation to invent them.

**Why it matters:** When designing structured output, ask not just "what field would be useful" but "can the model possibly know this at the moment it's asked" — if not, the schema itself is the fix.

**Evidence:** [[My Work at Pear Protocol#2026-07-28 – 08-01 — Basket generation could only produce two-sided baskets, and had no way to tag them]]

---

### A shared try/catch turns a benign "message is not modified" into a fake outage

A Telegram inline-keyboard toggle that re-fetches and re-renders will produce byte-identical content when the user re-taps the already-selected option; `editMessageText` then fails with 400 "message is not modified". If the fetch and the edit sit in one try/catch, that harmless edit error lands in the same handler as a real data-fetch failure — and the fallback edit overwrites good content with an error notice.

**Context:** The `/leaderboard` venue toggle wrapped fetch + `editMessageText` in a single try/catch. Re-tapping the current venue re-rendered identical content → 400 not-modified → the catch replaced the correct board with "temporarily unavailable". Reproduced live in the dev bot log.

**Why it matters:** Only a fetch failure means the data is unavailable; a delivery failure (not-modified, or an uneditable too-old message) is not an outage. Split them — fetch in one try/catch that decides the content, delivery in its own swallowing catch — so a re-render or stale-message edit can't masquerade as a backend failure. (Same reason the existing `linkvenue` toggle wraps each edit in its own swallow.)

**Evidence:** PR #675, commit `618d646b` + regression test `telegram.update.leaderboard.spec.ts`; see [[1st Sep 2026]].

---

### One config value shared by two consumers on different services is a hidden fan-out

A single env var named for one of its consumers can silently route a second consumer too — and if those consumers hit different services, repointing the var breaks the unnamed one with no signal at either call site.

**Context:** `context`'s `STATS_BASE_URL` fed both `GET /categories` (categories service) and `QuantService.getBasketStats()` → `POST /stats/basket` (a different, Rust, service). A change repointed it on the reasonable-looking "only categories uses this" assumption; `/stats/basket` then 404'd, nulling basket stats, while a `No baskets generated` alert described a failure that wasn't happening.

**Why it matters:** Name a config value for the route/service it addresses, not for one of its consumers — and when a var is repointed, grep every reader, not just the one the var is named after. Two routes on two services must not share one base-URL var. A validating type (`z.url()`) catches a malformed value at boot; it does not catch a well-formed value pointed at the wrong service — only enumerating the consumers does. Bonus tell: an alert that names a failure mode which the evidence contradicts ("no baskets generated" while rows are being written) is pointing at the wrong layer.

**Evidence:** [[One Service URL Fed Two Different Services, Nulling Basket Stats]]; PRs #677 / #678; see [[2nd Sep 2026]].

---

### A model switch that lands right before an incident is a lead, not a verdict

Timing correlation between a model swap and a new failure is a reasonable first hypothesis, but proving it requires replaying the exact captured production context against the actual models in the causal path — not just confirming which models were live at the time.

**Context:** A user's "hallucinating TP/SL" report lined up with a same-day emergency Gemma→GPT-OSS-120B swap (Cerebras retired `gemma-4-31b`). But the two nodes actually producing the bad output had never run that swap: one had never run Gemma or GPT-OSS in its history, the other had been on GPT-OSS for three months already. Live-replaying the exact captured context found the real cause was a deterministic context bug (100% reproducible regardless of model), with the model only a secondary, conditional factor.

**Why it matters:** A correlated timing story is cheap to construct and easy to stop at. The replay is what tells you whether the model is actually in the causal path, and if so, whether its failure rate is 100% (deterministic bug wearing a model's face) or genuinely conditional (a real model-reliability gap).

**Evidence:** [[Stale Recall Memory and an Unpersisted skipRisk Flag Resurfaced Old TP-SL Preferences]]

---

### State that must survive across turns cannot live only in re-derived LLM context

A flag or preference that the model has to re-notice and re-emit on every call, from raw conversation history alone, will eventually be forgotten — not because the model is unreliable in general, but because the instruction that set it can silently age out of whatever window or summary is feeding the model that turn.

**Context:** A user's "no sl no tp" instruction governed a multi-turn trade-building flow (open, then several resizes). The flag enforcing it (`skipRisk`) was never persisted anywhere outside conversation history. Once the raw history window (3 turns) and a stale multi-day recall summary both stopped carrying the instruction, the flag was gone — the model had no way to know it had ever been set, and a live replay confirmed this was 100% reproducible given that context, on the actual production model.

**Why it matters:** Before relying on "the model will remember and re-state it," ask whether the state in question needs to survive longer than the window that's actually guaranteed to carry it. If yes, it belongs in structured, server-persisted state with the model able to override it, not in re-derived prose.

**Evidence:** [[Stale Recall Memory and an Unpersisted skipRisk Flag Resurfaced Old TP-SL Preferences]]

---

### A schema that allows "nothing selected" needs to be caught structurally, not left to the model's next mistake

When a structured-output schema types every routing field as optional/nullable, a classifier can legally emit an answer that selects nothing at all — and if no downstream check requires at least one field to be meaningfully set, that degenerate output silently reaches the consumer with zero context instead of triggering the system's own known fallback.

**Context:** A planner returned every routing node `null`, `generalChat: false`, and `clarify: null` for a real user query. The Zod schema allowed it (no invariant requiring at least one truthy field); the DAG's only two post-plan gates both no-op'd for this shape; the executor treated zero active nodes as a normal, unlogged no-op. The system already had a purpose-built fallback for "nothing to route" (`createDefaultPlan()`, which is `generalChat: true`) — it just wasn't wired to this trigger, only to a fully-missing structured response.

**Why it matters:** "The prompt says to always pick something" is not enforcement. If a schema can structurally represent a null/empty/contradictory decision, something downstream — a refinement, a validation step, a deterministic fallback — has to reject or repair it, or a single bad sample silently reaches the user with no signal that anything went wrong.

**Evidence:** [[Kalshi Odds Query Fell Into a Planner Routing Dead Zone With No Empty-Plan Fallback]]

---

### A PR's Closes list can go stale the moment a new commit lands on its branch

A PR description's `Closes #N` list is only accurate as of the moment it was written — if the branch it's attached to gets a new commit for a different issue afterward, nothing re-checks whether the description still matches what the branch actually contains.

**Context:** Happened twice in the same session, on two different PRs. PR #757 closed 3 of the 4 issues its branch's commits actually resolved (missing the systemd-unit task, added on a later commit). PR #758 was opened correctly scoped to one issue, then a second issue's commit landed on the same branch — its description still claims that second issue is "a separate PR next in the chain," which stopped being true the moment that commit was pushed.

**Why it matters:** Before merging any PR, re-diff its actual commits against its stated `Closes` list — don't trust that the list was ever kept in sync after the PR was opened, especially on a branch that keeps receiving commits for follow-on tasks in the same story.

**Evidence:** [[Harness Integration/Step 3 — Tokenomist + OI Canaries on Hetzner]]

---

### A framework's graceful-shutdown only tears down what it manages

Calling a framework's own `close()`/shutdown method will clean up the providers it manages by its own lifecycle hooks — it has no way to know about a raw client instance a factory provider just handed back, unless that client itself is wired to a lifecycle hook.

**Context:** A NestJS one-shot CLI script called `app.close()` after finishing its work, expecting a clean process exit. It hung indefinitely both times, needing a manual `Ctrl+C`. The cause: the `REDIS`/`ASSET_INDEX_REDIS` tokens were plain factory providers (`new Redis(url)`, no `OnModuleDestroy`) — `app.close()` correctly ran every managed provider's teardown, but had no hook into these raw clients' open sockets, which kept the event loop alive.

**Why it matters:** Any one-shot process built on a framework's DI container should explicitly quit/close every raw external-client instance it holds (Redis, DB pools instantiated outside the framework's own service wrappers, etc.) in a `finally` block after the framework's own shutdown — don't assume `app.close()` reaches everything just because the client was obtained via the DI container.

**Evidence:** [[Harness Integration/Step 3 — Tokenomist + OI Canaries on Hetzner]]

---

### Silence after switching branches on a remote box can mean the wrong code is running, not a hang

A command that "hangs" right after a branch switch on a remote box may not be stuck at all — if the branch in the working tree doesn't actually contain the flag/code being tested, the process can silently fall through to a completely different, valid code path (like the app's normal always-on boot) that looks identical to a hang because it, correctly, never exits.

**Context:** A `git stash pop` done to recover unrelated WIP silently left a Hetzner checkout back on the wrong branch. The one-shot flag being tested (`--tokenomist-once`) didn't exist on that branch at all, so it was ignored, and the process booted the normal long-running server instead — which of course never returns to the shell. A long, otherwise-reasonable debugging session (Redis reachability, auth, ioredis diagnostics) chased this as if it were a real bug before the branch mismatch was noticed.

**Why it matters:** Before debugging an unexplained hang on a remote checkout, confirm `git branch --show-current` and `git log --oneline -3` match what's actually being tested — a silent no-op fallback can be indistinguishable from a genuine freeze until the branch itself is checked.

**Evidence:** [[Harness Integration/Step 3 — Tokenomist + OI Canaries on Hetzner]]

---

### A session-level instruction can silently override a stored preference

A stated personal preference (e.g. "no AI attribution trailer in commits") is not self-enforcing — a session-start instruction that explicitly says it replaces earlier guidance will win by default unless the conflict is checked for and flagged, rather than just assumed away.

**Context:** A standing preference was on file for no `Co-Authored-By: Claude` trailer in commits. A session-start reminder in this session explicitly said it "replaces any earlier attribution guidance" and reasserted the trailer. Two commits got it added before the user noticed and asked for removal. Unpushed, so fixing it was a clean rewrite (`git reset --soft` past both, re-split, recommit) with no shared-history risk — would have been worse if already pushed.

**Why it matters:** At the start of any session that will create commits/PRs, check whether a session-level instruction conflicts with a known standing preference, and surface the conflict instead of silently picking whichever one is more recent.

---

### A provider redeclared in each module's own `providers` array creates one instance per module, not a shared singleton

In NestJS, importing the same *module class* from several places gives you one true app-wide singleton of that module and its providers — but repeating the same *provider literal* directly inside multiple different modules' own `providers` arrays does not get deduplicated. Each module silently gets its own separate instance of that token.

**Context:** A prior fix ([[Engineering Lessons#A framework's graceful-shutdown only tears down what it manages]]) added explicit `redis.quit()` calls for the two raw-Redis tokens known at the time, and manual validation looked clean. It wasn't: `TwitterModule`/`SentimentModule`/`StatsModule` (and, in a sibling app, `AssetIndexModule`/`CoinIndexModule`/`TokenomistIndexModule`) each declared their own `RedisProvider` locally, so 4 (and 3) separate ioredis clients existed per app, not the two anyone knew about. `quit()` only ever closed one; real systemd runs hung 28 minutes past "one-shot complete" before being killed on timeout. A code review's mocked-container reproduction caught the true instance count; the fix was importing a shared `RedisModule` (a genuine module-class singleton) instead of redeclaring the provider per module.

**Why it matters:** when auditing "does every raw external client get closed," missing tokens aren't the only risk — the same token can have multiple silent *instances* because it was declared as a provider literal in more than one module. Grep the provider's name across module files: if it appears in more than one module's own `providers` array (not via a shared imported module), that's N instances, not one.

**Evidence:** [[Harness Integration/Step 3 — Tokenomist + OI Canaries on Hetzner#Update — 2026-09-16 (code review found the #748/#753 Redis fix was incomplete; root-caused and fixed)]]

**Evidence:** [[Harness Integration/Step 4 — Watched-Accounts + Image-Retention Batch on Hetzner]]

---

### An LLM-quality win can be an illusion if the pipeline rewrites the output downstream

A model that scores better in a blind quality judge can still produce no better end result, if a later pipeline stage discards or rewrites exactly the part the model did better.

**Context:** In the interest-distiller Claude-vs-Cerebras spike, a neutral judge rated Opus's watch triggers better (more tailored, pair/basket-aware). But `trigger-synthesizer` templates any pair/ratio statement (`isPairStatStatement`) before arming Elfa, which is exactly where Opus differed most, so its edge mostly never reached the fired query.

**Why it matters:** Evaluate the artifact the system actually acts on (the fired trigger), not the raw model output. Judge scores on pre-pipeline text can overstate a model's real-world value when a downstream stage normalizes it away.

**Evidence:** [[Harness Integration/Step 7 — Claude CLI Feasibility Research for interest-distiller#Update — 2026-09-16 (spike run + results + decision)]]
