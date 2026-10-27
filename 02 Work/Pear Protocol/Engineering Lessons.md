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

**Update — 2026-09-19:** A third confirmed instance, `DiscordListenerService` opened a persistent `discord.js` gateway `Client` in `onModuleInit()` with no teardown, hanging `--watched-accounts-once`/`--twitter-ingestion-once` after logging completion. This one had a cleaner fix than the raw-Redis cases: since it was already a real `@Injectable()` managed by Nest's DI (not a bare factory-provided client), the fix was implementing `OnModuleDestroy` on the class itself and letting Nest's own shutdown sequence call it — no manual `finally`-block quit needed in the entrypoint. Refines the lesson: check whether the leaking resource lives on a real DI-managed provider first (give it a lifecycle hook) before reaching for the manual-cleanup-in-main.ts pattern, which should be the fallback for genuinely raw/external clients only. See [[Incidents & Investigations/Hetzner One-Shot's Full AppModule Boot Fired an Unrelated Cron, Colliding With Railway#Update — 2026-09-19 (root cause #1 fixed and merged; root cause #2 still open)]].

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

**Update — 2026-09-24:** the lesson holds, but the cause in this case was upstream too. A follow-up run showed the production prompt's examples (including a ratio example) were what steered Opus into ratio wording in the first place: with a loose prompt, only 2% of Opus's triggers were templated, against 46% before. So check whether the pipeline discards the model's edge, and also whether your own prompt is pushing the model into the form the pipeline discards. See [[Harness Integration/Step 7 — Claude CLI Feasibility Research for interest-distiller#Update — 2026-09-23 (trigger-only neutral judge + Notion write-up)]].

---

### Tool-emitted safety metadata must survive LLM synthesis boundaries

When an upstream tool emits a safety or reliability label, downstream LLM summaries need an explicit rule to preserve it verbatim in every output format, especially when compressing multiple candidates into one recommendation.

**Context:** CoinGecko tools correctly emitted Pear tradability tags, but the market scanner could drop a tag while compressing a multi-asset scan into one decisive pick. The composer then built an executable trade card for an asset that was not tradeable on Pear. The fix was a prompt rule requiring the tag to remain attached through tables, prose, and single-asset recommendations; the evidence did not support a deterministic composer guard.

**Why it matters:** A safety-critical label can be correct at the source and still be absent at the action boundary. Review the full transformation path and explicitly carry decision-critical metadata across model rewrites; a prompt-level rule can be the right scoped fix when the failure is a rare synthesis omission rather than a deterministic pipeline defect.

**Evidence:** PR #794, commit `3691e06a`, `apps/agent/src/ai/agents/market-scanner/prompt.ts`.

---

### A "done on purpose" comment is a claim about the code when it was written, not a fact about the code now

A comment justifying a design choice (sequential execution, a lock, a cap) can quietly outlive the reason it gave. Later code changes can remove the real constraint while the comment — and the behavior it justifies — stay untouched, because nothing forces a comment to be re-verified when the code around it changes.

**Context:** `MadeBasketsCronService.refreshDaily` ran 3 platforms sequentially with the comment "Sequential on purpose — the runs share the LLM + stats API budget." Git blame put that comment at June 2026, before `categoryAnalysisCache` existed. By September the cache had been rebuilt specifically to be concurrency-safe (memoizes the in-flight Promise, not the resolved value) and the budget cap had become a per-subprocess CLI flag, not an in-process tracker — the comment's stated reason was gone, but the sequential loop (and the ~3-hour full run it produced) remained.

**Why it matters:** Before trusting a "this must stay X because Y" comment as a hard constraint, check whether Y still holds against the *current* code — grep the actual shared resource it names, check who else touches it, check git blame for when the comment was written vs. when the resource last changed shape. Treat the comment as a historical claim to verify, not a fact to inherit.

**Evidence:** [[Harness Integration/Step 2 — Claude Basket Generation on Hetzner#Update — 2026-09-18 (part 2): #786 conflict resolved + merged, #796 concurrency fix shipped, first real Hetzner install|Step 2 §2026-09-18 update]], PR #796.

---

### A one-shot that boots the full app isn't scoped to one job — and an unlocked write path turns a coincidence into a collision

A CLI one-shot that reuses the app's full module (`NestFactory.createApplicationContext(AppModule, ...)`) instead of a dedicated minimal module doesn't just need the whole app's env — it's also subject to every other scheduled job that module registers, for as long as the process stays alive. If a downstream write path has no per-item claim/lock, two callers hitting it in the same window (an accidental extra invocation + the regular scheduled one) can double-process the same rows.

**Context:** A code review had already named this exact risk in writing three days before the affected PRs merged anyway ("can trigger unrelated scheduled writers outside the one job's own flock"). It manifested on the first real dry run against the merged code: an OI one-shot's full-`AppModule` boot let an unrelated Twitter-labeling cron fire mid-run, and because the Twitter pending-queue read (`getPendingContent()`) had no row-level lock, Railway's own regularly-scheduled instance of the same cron collided with it a minute later — real LLM calls, real DB writes, real Discord posts, none of them the job that was actually requested.

**Why it matters:** A one-shot's isolation claim is only as strong as its module boundary. If it boots the full app module, "I only asked for job X" is not true — check what else that module schedules before treating a manual/delayed invocation as safe. Separately, any queue-style read-then-write (`findMany` on an unprocessed-flag, then mark-as-processed later) needs an atomic claim if more than one process could ever call it concurrently — "this only runs once, on a schedule" is an assumption, not a guarantee enforced by the code.

---

### Pinning a script's own tool invocation by absolute path doesn't pin the environment its nested subprocesses inherit

A deploy/automation script that calls its main tool (`$PNPM_BIN`, `$NODE_BIN`) by absolute path looks environment-independent, but if that tool's own scripts shell out again internally (a `package.json` script running a bare `pnpm`), those nested calls resolve via whatever `PATH` the process actually has — which a non-interactive SSH session never gets populated for, since it skips the profile that normally sets it up (e.g. `nvm`'s shell init).

**Context:** The new Hetzner CI deploy workflow's `scripts/hetzner/deploy.sh` invoked `$PNPM_BIN run railway:prisma:generate` by absolute nvm path — but that script runs `pnpm -C packages/database run generate:all` internally as a bare `pnpm`. First real CI run failed with `sh: 1: pnpm: not found`, because the SSH session's plain `PATH` had a stale system Node v18 and no `pnpm` at all. A near-identical bug hit one step earlier in the same script: `mkdir -p /run/agent-pear` as plain `prakh` failed with `Permission denied` because `/run` is root-owned tmpfs and the script never used the `sudo` that was actually available.

**Why it matters:** Before trusting a script "because it uses absolute paths," check every place it calls back into itself indirectly (nested npm/pnpm scripts, subshells) — those calls don't inherit the absolute-path discipline unless the directory holding those binaries is explicitly exported onto `PATH`. And "works when I run it manually" isn't evidence it'll work under CI/SSH — an interactive shell sources profile files a non-interactive one skips; test the actual invocation context, not a hand-run approximation of it.

**Evidence:** PR #809, PR #810, [[Daily Work Logs/19th Sep 2026#Update — same day, third session: PR #804 finished, CI deploy actually ran, two real bugs found and fixed|19th Sep 2026 update]].

---

### A write-only secret can't be debugged by re-reading it — only by a live functional test

Secret stores (GitHub Actions secrets, most others) intentionally never let you read back a stored value — only overwrite it. If the value got corrupted between the source and the store (a copy-paste through a browser converting line endings, a partial paste), the only signal is the consumer failing downstream, and there's no way to inspect what's actually stored to confirm the fix worked before trying again.

**Context:** A dedicated SSH deploy key's private half was generated cleanly, but the value that ended up in the GitHub environment secret failed to parse (`error in libcrypto`) — almost certainly mangled during a Windows copy-paste into the browser. Because the secret is write-only, there was no way to confirm or deny corruption by looking at what GitHub had stored; the only real fix was regenerating the key and verifying the *new* file with a live `ssh` auth test before handing it off to be pasted again.

**Why it matters:** Treat any write-only credential store the same way: verify the source file parses/works *before* it goes in (`ssh-keygen -y -f key`, a live auth attempt), and if a downstream consumer fails after a paste, don't assume the paste was faithful — regenerate and re-verify from a clean source rather than trying to diagnose a value you can't see.

**Evidence:** [[Daily Work Logs/19th Sep 2026#Update — same day, third session: PR #804 finished, CI deploy actually ran, two real bugs found and fixed|19th Sep 2026 update]].

**Evidence:** [[Incidents & Investigations/Hetzner One-Shot's Full AppModule Boot Fired an Unrelated Cron, Colliding With Railway]]

---

### Output deduplication cannot create diversity after upstream inputs become identical

An output-level deduplication guard can prevent exact repeats without producing genuinely different results. If an upstream research or selection stage receives the same prompt and state for every attempt, it can keep returning the same anchors; later stages can only reshuffle them.

**Context:** The Claude two-stage basket runner stripped `avoidBaskets` from the research prompt because the naming/distinctness block was considered noise for survivor generation. After category analysis was memoized, attempts in a run received identical research inputs and repeatedly surfaced the same anchors (`DOGE`/`AMD`/`GALA`), producing near-identical basket themes. The OpenRouter path retained `avoidBaskets` and did not show the same collapse. PR #811 restores the signal and adds a research-scoped diversity instruction; the behavioral improvement still needs a live multi-basket run.

**Why it matters:** When a system promises variety, inspect where variation is introduced. A final deduplication check is not a substitute for upstream state that changes the candidate set. Preserve novelty signals at the stage where candidates are generated, then keep downstream deduplication as a safety backstop.

**Evidence:** PR #811, commit `b00eeb45`, [[Daily Work Logs/19th Sep 2026#Basket research diversity regression — PR #811 (open, not live-verified)]], [[Harness Integration/Step 2 — Claude Basket Generation on Hetzner#Update — 2026-09-19 (PR #811: research-stage diversity regression)]].

---

### A strict, example-heavy prompt can help a small model and cap a strong one

The same prompt change can move two models in opposite directions. A detailed recipe with Good/Bad examples keeps a small model in line, but a stronger model copies the template instead of using its judgment.

**Context:** interest-distiller watch triggers, same 25 users and evidence, prompt and model varied separately. Removing the recipe and examples (keeping only the constraints the pipeline enforces) made Opus's triggers win 24-0 with two neutral judges, and cut its boilerplate from 39% to 5%. The same loose prompt made Cerebras `gpt-oss-120b` worse: trading-style tag leaks went from 1% to 14%, and its triggers shrank to bare one-liners.

**Why it matters:** When you compare models on one shared prompt, you're partly measuring how well each model fits that prompt. Before concluding a stronger model "isn't much better", test it on a prompt written for it. And don't loosen a prompt across the board: keep the scaffolding for the small model, and give the strong model only the hard constraints plus a goal.

**Evidence:** [[Harness Integration/Step 7 — Claude CLI Feasibility Research for interest-distiller#Update — 2026-09-23 (trigger-only neutral judge + Notion write-up)]], [[Daily Work Logs/24th Sep 2026]], branch `spike/interest-distiller-loose-prompt`.

**Update — 2026-09-24:** the loose prompt wasn't the end state. Adding back four targeted rules — one per measured weak spot (one subject per statement, explicit numbers, follow stated preferences, no absolute levels), still no examples or recipe — beat the loose prompt 21-3 / 22-2 and cut multi-asset triggers 34% → 4%, without re-templating (0% discarded before Elfa). So the pattern for a strong model is: hard constraints + goal, run it, then add a rule for each concrete failure you measured — rules, not examples. Cost: some variety (distinct shapes 100% → 82%). See [[Harness Integration/Step 7 — Claude CLI Feasibility Research for interest-distiller#Update — 2026-09-24 (middle-ground prompt results)]].

---

### Hand-check a regex metric's hits before reporting it

A deterministic check is only as good as its pattern, and a pattern written for one arm's output can misfire on another's.

**Context:** the interest-distiller A/B counted "absolute price level" triggers with a regex. The middle-ground prompt explicitly banned absolute levels, yet the metric reported 61.5% for it. Reading the hits showed it matched dollar liquidation sizes ("$2M of LIT long liquidations") and backtracked into funding thresholds ("above 0.04%") — conditions the new prompt produced far more of. Hand count: 2.8%. The same bug had inflated the previous run's 18% figure for the loose prompt, which was already in the write-up.

**Why it matters:** when a metric contradicts what the prompt says (or moves sharply between arms), read the matches before interpreting it. Arms that change *what kind* of text is produced also change what a regex catches, so a comparison can flip on a false-positive rate rather than on the real property.

**Evidence:** [[Harness Integration/Step 7 — Claude CLI Feasibility Research for interest-distiller#Update — 2026-09-24 (middle-ground prompt results)]], `ABS_LEVEL` in `apps/agent/src/scripts/interest-distiller-quality-ab.ts` on branch `spike/interest-distiller-loose-prompt`.

**Also (2026-09-25):** the advanced-mode grounding check's number regex in #991 read the JSON array `[3100,3200]` as one number and skipped the figure after `"64,000, "`. The spec passed because its fixtures didn't contain either shape. Found by feeding it real tool-output shapes. See [[25th Sep 2026]].

---

### Test output-shrinking code against each tool's real return shape, not a hand-built object

Code that compacts or summarizes tool output (truncation, digests, dedupe) encodes an assumption about what the output looks like. If the specs only use neat object fixtures, a tool that returns a formatted string or `JSON.stringify(...)` gets treated as one opaque string and silently loses data.

**Context:** #987's advanced-mode compaction applied a per-field 1500-char string cap to the whole result. Its digest dropped any long single-line string. Every account read returns a JSON string and TA returns multi-line text, so `get_account`'s balance vanished from the digest and a 10-asset TA call kept only 3 assets. The specs all passed because they only used object rows with an `asset` key.

**Why it matters:** the failure is silent. The model still gets a well-formed result, just with the numbers it needs for sizing missing, and it writes the answer anyway. Fixtures for this kind of code should be copied from what the real tools actually return.

**Evidence:** [PR #987 review comment](https://github.com/pear-protocol/agent-pear-v2/pull/987#issuecomment-5820172069), fixes in [#991](https://github.com/pear-protocol/agent-pear-v2/pull/991) (open as of 2026-09-25), [[25th Sep 2026]].

---

### An eval harness with a silent model fallback measures a different model when the primary is overloaded

If the code under test falls back to another model when the primary call fails, an eval run during a provider brownout quietly mixes two models' behaviour into one score. Nothing shows up as an error, because the fallback succeeded.

**Context:** comparing two planner prompt wordings, 216 calls each, 12 in flight. Cerebras started failing and the planner fell back to `claude-haiku-4.5` through OpenRouter, which routes differently. One wording looked clearly worse (the Dutch probe went from 4/6 to 0/18, with identical answers repeated). Rerun with the model pinned (`modelOverride`, which disables the fallback), failures became visible errors (about half the calls). Among the successful calls the two wordings were statistically the same (90.3% vs 88.6%).

**Why it matters:** a fallback that keeps production resilient makes an eval dishonest. For A/B runs, pin the model so failures show up as errors, count those errors, and compare only calls that succeeded. A sudden big swing on one variant during a noisy provider period is a sign to check which model actually answered. The fallback also costs money on a different provider.

**Evidence:** [[Generic US-Stock Picks Refused as Crypto-Only, Then Reported as a Data Outage]], [[24th Sep 2026]].

### A streaming fallback must trigger on "nothing emitted", not only on a thrown error

With `ai@6` `streamText`, an error on a later step of a multi-step tool loop doesn't throw out of `textStream`. The SDK turns it into a stream error part, `textStream` just ends, and `usage` resolves from the earlier successful step. Code that only falls back inside a `catch` never sees the failure.

**Context:** PR #923 moved the composer's fallback to a different provider and dropped primary retries. A Cerebras 429 on step 2 (after a tool call) still ended as "composer prose stream was empty", and the OpenRouter fallback was never called. Found by reading the SDK source. Confirmed by a unit test that fails on the old commit, and by a live trial with an injected 429 (failed 2/2 before the fix, recovered after).

**Why it matters:** a fallback that looks right in a first-step failure test can be dead for the failures that actually happen under load. Rate limits are more likely on later steps, because each step resends the full prompt. When wrapping a streaming call with a fallback, key it on "no output was produced" and test a failure after a tool call, not just an initial rejection. Separately, a fallback on the same provider key shares the rate-limit bucket that just rejected the primary, so it can't help with a 429.

**Evidence:** [[25th Sep 2026]], PR #923 commits `ae33a54c` (fix), `0faadd94` (test).

### A packaged desktop app's writes to AppData can land in a private virtualized copy

On Windows, an MSIX-packaged app (the Claude desktop app is one) gets its writes under `%LOCALAPPDATA%` / `%APPDATA%` redirected into `AppData\Local\Packages\<app>_…\LocalCache\…`. Processes launched from inside the app see the merged view, so everything looks fine — but a normal terminal can't see those files at all.

**Context:** building the external `codectx` index from a Claude Code session, the Graphify output went to `%LOCALAPPDATA%\CodeIntelligence\…`. `graphify query` printed its real path as `AppData/Local/Packages/Claude_pzs8sxrjxfjjc/LocalCache/Local/CodeIntelligence/…`; `Test-Path` from outside the package said the normal path didn't exist. Moved the store to `~/.local/share/code-intelligence` (not virtualized, same as `~/.local/bin`).

**Why it matters:** any tool, cache, or config an agent session creates "for later use in a terminal" silently won't exist for the terminal. Put user-level tooling data outside AppData (e.g. under the home directory), or verify from a process outside the app before relying on it.

**Evidence:** [[Codebase Retrieval with codectx (Graphify + Embeddings)]], [[27th Sep 2026]].

### A validation error that doesn't name the field just moves the model's guessing loop

Validating LLM tool input only helps if the error says *which* field is wrong. With a zod discriminated union, `issue.message` alone ("expected string, received undefined") gives the model nothing to act on, so it guesses field names and retries. Include `issue.path` in the message, and name each variant's fields in the tool description, not only in one node's prompt.

**Context:** the 2026-08-24 fix validated `open_position`'s `condition` but joined only `issue.message`. Ratio / weighted-ratio opens then failed on every attempt, with up to 11 retries per turn, and the model ended by inventing platform limits to explain the failures.

**Why it matters:** an unactionable error can be worse than a crash. It burns retries and pushes the model to make up an explanation for the user.

**Evidence:** [[TRIGGER-LADDER JSON Crash Silently Substituted the User's Trade#Follow-up — 2026-09-28 (the validation error wasn't actionable)]], PR #1017 (`a78b6bd7`).


### Moving a job off the database's private network can break code that was only ever fast enough

Code that makes one DB round trip per row inside a time-boxed transaction works on the database's private network (~1 ms per trip) and can fail once the same job runs across a public proxy (~30 ms per trip). Prisma's interactive `$transaction` defaults to a 5 s timeout, so ~175 row-by-row writes overran it, and the failure rolled back every row the run had already paid an LLM for.

**Context:** moving the weekly sentiment job from Railway to a Hetzner systemd one-shot. The daily variants (~15 rows) passed and the weekly (~175 rows) failed with "Transaction not found". Fixed with an explicit timeout (PR #1034). Batching the writes into fewer round trips is the upgrade if row counts grow.

**Why it matters:** "it's worked on Railway for months" is no evidence for a relocated job. Before cutover, run the largest variant of each job from the new host, not just the daily one, and look for per-row loops inside transactions, locks or timeouts.

**Evidence:** [[28th Sep 2026#Weekly sentiment failed from Hetzner → fixed in PR #1034]], PR #1034 (`64ae34f7`).

### A prediction-market probability isn't a direction unless you know where the price is

"Will X be above $K on date?" at 95% Yes usually just means spot is already far above K. A "drop below $K" market with high Yes is bearish. So a rule like "high Yes = bullish" reads strike ladders wrong about half the time. The odds only say something about direction next to the current price and the strike (e.g. the implied chance of a +Y% move by the date).

**Context:** asset sentiment kept only Polymarket *price* markets (above/below/hit/drop) and told the model "high Yes odds (>65%) bullish". Spot was never in the prompt. Removed in #1054 rather than fixed, since a proper reading needs price data that phase 1 of #1052 only now adds.

**Why it matters:** any place that feeds prediction-market odds to an LLM as a sentiment signal has the same trap. The chat agent's prediction-odds tool was not checked for this.

**Evidence:** [[Sentiment Data Ingestion — Improvement Plan]] (noise #6), PR #1054 (`3deb6f0d`).

### A retry that resends the identical request can't escape a failure an exact-match cache has stored

If a response cache keys on the exact request, retrying the same request inside the TTL returns the same cached answer. A "successful" bad answer (HTTP 200 with empty content) gets cached like any other, so the retry loop just replays it. Retries only help if the retried request can land somewhere else: bypass the cache, or change something the cache keys on.

**Context:** the asset-sentiment synth retries up to 3 times on empty prose. The shared OpenRouter factory sends an always-on 5-minute response cache header (exact messages + params). When deepseek was routed to Relace and came back empty, attempts 2 and 3 returned 0 tokens, $0, in under a second. That fits a cache hit, but it was not confirmed from a cache header. Relace also showed ~99.5% "uptime" on OpenRouter while doing this, because an empty 200 counts as up.

**Why it matters:** any retry-on-bad-content loop (empty output, failed parse, schema mismatch) behind this factory has the same blind spot, not just sentiment. Retry on HTTP errors isn't affected, since errors aren't cached. The fix (bypass cache on retry) is not built yet.

**Evidence:** [[Sentiment Data Ingestion — Improvement Plan]] (Update "why the new variant failed"), `packages/shared/src/llm/openrouter.constants.ts` (`CACHE_TTL_SECONDS = 300`), PR #1060.

### Find out which branch actually runs in prod from the deploy trigger and the host, not the runbook

A runbook line like "the box tracks `development`" is a claim that stays true only until someone changes the deploy workflow. What decides where code runs is the workflow's trigger plus what's actually checked out on the host. Check both before planning anything that depends on "when does this reach users".

**Context:** two Hetzner runbooks said the box's checkout tracks `development`. `deploy-hetzner.yml` triggers only on pushes to `main` and checks out the exact pushed SHA, and on the box the repo was detached at the `origin/main` tip. Railway prod had every cron kill-switch on, so Hetzner was the only prod runner. The release plan for the familiar-asset project had assumed "merge to `development` → watch in prod", which would have measured nothing.

**Why it matters:** release sequencing, A/B measurement windows and "is this live yet?" answers all depend on the real deploy source. A stale branch claim doesn't fail loudly; it just makes you wait for data that never arrives.

**Evidence:** [[30th Sep 2026#Familiar-asset pair/basket: structure-neutral plan, issue tree, Phase 0 (late)]], `.github/workflows/deploy-hetzner.yml` (`branches: [main]`), runbook fix `f5cb1a14`.



### Before fixing "the filter dropped my output", recompute which rule actually fired

When a chain of filters drops results and the drop reasons aren't logged at the level you captured, the obvious-looking rule is a guess. Re-run each rule against the saved inputs and outputs before designing the fix. A fix aimed at the wrong rule passes its specs and changes nothing.

**Context:** in the #1068 replay, Lighter stored 3 of 8 valid ideas. The ideas reused SOL/SUI/PUMP, so the per-asset cap (max 3 per run) looked like the cause, and the proposed fix was "drop capped assets from the pool". Recomputing the cron's checks on the saved selections showed most drops tripped the composition-overlap rule (>50% of legs shared with an accepted idea); PUMP was only on its 2nd use. The cron logs skip reasons at debug, which the replay didn't capture. The fix became "unused assets first each attempt" instead.

**Why it matters:** the cap-based fix would have shipped, passed tests and left the shortfall in place.

**Evidence:** [[1st Oct 2026#Post-replay: #1077, Lighter root cause + fix, epic checklists]], `made-baskets-cron.service.ts` (`isCompositionDuplicate`, `hitsFrequencyCap`), `eb6353fd`.

---

### Repeat runs of an LLM experiment are not independent samples if a response cache sits in front of the model

Running the same experiment several times only measures noise if each run actually reaches the model.

**Context:** the sentiment playground's repeat runs went through OpenRouter, whose response cache is always on with a 5 minute TTL. Three runs started about 2 minutes apart cost $0.5929, $0.0486 and $0.0513, and the later two returned the same calls as the first (`new` 15/2/0 in all three).

**Why it matters:** a noise estimate or an arm comparison built on cache replays understates the variance and double-counts calls when scored. The cost line is the tell: a repeat that is about 10 times cheaper was served from the cache.

**Evidence:** [[5th Oct 2026#Noise measurement]], `requests.jsonl` costs per run.

---

### A dry-run count hides what a filter throws away; read the dropped items

A before/after count said a stricter tweet matcher was removing collisions, but the counts could not show that it was also removing on-topic items.

**Context:** the first strict rule (capitals-only whole word) looked good by count (`PUMP` 6→0, `HYPE` 15→6). Reading the dropped tweets showed it had lost lowercase "btc" / "sol" and 8 of 9 tweets about Hyperliquid, which the old substring search had matched only by accident.

**Why it matters:** a filter's precision gain is only visible next to its recall loss, and only the dropped items show the recall loss.

**Evidence:** [[5th Oct 2026#A2: strict tweet matching (PR #1083, merged)]], `tweet-match-diff.ts` (`DUMP=`).

---

### A stacked PR must be retargeted before its parent merges, or the child lands on a dead branch

Merging a PR whose base is another feature branch only updates that feature branch.

**Context:** #1081 was based on #1080's branch. After #1080 merged into `sentiment-enhancement`, #1081 was merged into `feat/sentiment-measurement`, so the strict-tweet commit never reached `sentiment-enhancement` and a later revision was in no merged PR. It took a replacement PR (#1083) to land it.

**Why it matters:** the PR shows as merged, so the miss is invisible unless the target branch is checked.

**Evidence:** [[5th Oct 2026#PR / merge mix-up (corrected)]], `git merge-base --is-ancestor` against `origin/sentiment-enhancement`.

---

### A thrown tool error is not a tool result, so a summary built from results omits failures

A ground-truth block for the composer was built only from `toolResults`, so a read that threw never appeared in it and the model's "nothing found" prose won.

**Context:** in the AI SDK a thrown tool is recorded as a `tool-error` content part, not a `toolResults` entry. #1148 found `summarizeToolCalls` reading only `toolResults`, so after every account read errored the user was told there was no recorded HYPE short. Empty feeds had the same shape (an empty unlock feed became "no significant unlocks").

**Why it matters:** an absent result and an empty result look the same to a reader, and the agent states either as a negative fact. Any code that summarizes tool activity has to list failures and empties as unavailable explicitly.

**Evidence:** [[20th Oct 2026]], PR #1148 (a replay of the original trace was not run).

---

### In zod v4 an object key typed z.unknown() or z.custom() is required unless marked optional

A schema meant to pass a field through untouched rejected every request that left the field out.

**Context:** for #1159 the `surface` and partner `query` fields were left as `z.unknown()` / `z.custom<Surface>()` so the controller's own checks kept their error messages. zod v4 treats those keys as required, so a missing `surface` on `/chat/execute` and a missing body on the partner route came back as a 400 ("expected nonoptional, received undefined"). The validation specs caught it before commit. The fix was `.optional()`, the same as `runtimeContext` in #1154.

**Why it matters:** a pass-through field reads as "accept anything", so the new 400 looks impossible and would only show up when a real caller omits the field. A spec that sends a missing body or a body without that field is what catches it.

**Evidence:** [[21st Oct 2026]], PR #1159 (failing jest output in that session).

---

### An audit row that says unchecked input reaches X is a lead, not a bug, until the path is traced

Of the 19 rows in the #1158 input audit, several described a real gap that turned out to have no failure: the malformed value already ended in a correct refusal.

**Context:** a repeated query key on `verifyCode` and the x-link routes is joined into a Redis key that does not exist, giving the existing invalid answer. A non-string tg-grant `tradeAccountId` fails an `===` match and returns the existing `unknown_account` 400. The trade-ticket Redis parses (rated money-path risk) already return null on a bad parse, and the single-use plan token bound on the server means a malformed ticket can only produce an error, not a different trade. A schema copy of `TradeTicket` there would have added a failure mode (valid tickets rejected if the schema lagged the type). The rows that did crash (onboarding `value.trim()`, the Elfa webhook `title.slice`) were confirmed by a spec failing on the old code.

**Why it matters:** fixing every row adds 400s that can break sloppy but working clients, plus schemas that need dual maintenance, for no reliability gain. Trace each row to where the value is used and fix only the ones that crash, persist bad data, or change an action.

**Evidence:** [[21st Oct 2026]], #1158 Decision column, PRs #1159 and #1160.
