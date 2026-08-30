# Features I Built at Pear Protocol

*Feature work only — `feat()`-tagged commits/PRs and other net-new product capability, filtered out of fixes/hardening. Author: `prakharDvedi`, repo `pear-protocol/agent-pear-v2`, 2026-07-07 → 2026-08-30.*

## Contents

- [[Features Owned#Multi-venue execution: Lighter integration|Multi-venue execution: Lighter integration]]
- [[Features Owned#Basket generation|Basket generation]]
- [[Features Owned#Trade execution|Trade execution]]
- [[Features Owned#Telegram / product surface|Telegram / product surface]]
- [[Features Owned#Observability / agent infrastructure|Observability / agent infrastructure]]
- [[Features Owned#Model routing|Model routing]]

---

## Multi-venue execution: Lighter integration

The flagship feature body of work — bringing Lighter online as a second execution venue alongside Hyperliquid (Epic #521, Story #524). Spans both PRs opened solo and authored commits inside teammate (GPSxtreme)-opened umbrella PRs on the shared `lighter-integration` branch.

- **Lighter asset index** — ingestion/resolution for Lighter-listed instruments. [[My Work at Pear Protocol#2026-08-18 – 08-20 — Bringing up Lighter as a second execution venue|#548]] *(collab PR, own commits)*
- **Venue-aware price / market-overview reads** — read tools stop assuming Hyperliquid. [[My Work at Pear Protocol#2026-08-18 – 08-20 — Bringing up Lighter as a second execution venue|#547]] *(collab PR, own commits)*
- **`execution_venue_price` tool** — conditional TP/SL/trigger/ladder levels now anchor to the price on the *resolved account's actual venue*, keyed off `context.tradeAccount.connector`, never model-guessed. [[My Work at Pear Protocol#2026-08-18 – 08-20 — Bringing up Lighter as a second execution venue|#557]] *(own)*
- **Venue disambiguation + ticket venue binding** — cross-venue account ambiguity asks instead of silently defaulting; resolved venue bound onto the pending trade ticket. [[My Work at Pear Protocol#2026-08-18 – 08-20 — Bringing up Lighter as a second execution venue|#545]] *(collab PR, own commits)*
- **Venue-honest trade-exec prompts** — prompt/hot-facts keyed to the account's actual connector instead of hardcoded HL language. [[My Work at Pear Protocol#2026-08-18 – 08-20 — Bringing up Lighter as a second execution venue|#546]] *(collab PR, own commits)*
- **Invocation-scoped trade-account binding** — pins the resolved account for the lifetime of one agent invocation so reads/writes can't diverge across venues. [[My Work at Pear Protocol#2026-08-18 – 08-20 — Bringing up Lighter as a second execution venue|#558]] *(own)*
- **Telegram Lighter readiness** — inline multi-account chooser, connector-aware copy across `/start /help /resync /trade /notifications`, test-tap harness fixes for the new venue. [[My Work at Pear Protocol#2026-08-18 – 08-20 — Bringing up Lighter as a second execution venue|#564]] *(own)*
- **Inline venue picker for `/linkwallet`** — replaced a typed-arg (`hl`/`lighter`) with buttons; both venues now first-class and discoverable. [[My Work at Pear Protocol#2026-08-24 — A confirm-time margin gate, a recurring ambiguous-symbol crash, and a hidden 2x-budget confirm|#594]] *(own)*
- **Multi-account signal-CTA chooser** — signal tickets ask which account when a user has more than one. [[My Work at Pear Protocol#2026-08-21 — Surface copy still said "Hyperliquid" regardless of the account's actual venue|#574]] *(own)*
- **Lighter margin-mode contract, pinned** — verified live (227/227 instruments) that `cross` is the safe default today; test-locked so a future `marginMode` change is a deliberate, visible break. [[My Work at Pear Protocol#2026-08-18 – 08-20 — Bringing up Lighter as a second execution venue|#556]] *(own)*
- **KB made Lighter-aware** — execution-venue docs stop implying Hyperliquid-only. [[My Work at Pear Protocol#2026-08-18 – 08-20 — Bringing up Lighter as a second execution venue|#559]] *(own)*

**Related:** [[Ambiguous HIP-3 Symbol Crashed the Whole Reply, Repeatedly]] · [[TRIGGER-LADDER JSON Crash Silently Substituted the User's Trade]] · [[TWAP Close-All — Explicit Fallback Required]] · [[Elfa Webhook Trust Model]] · [[Resume Evidence#Breadth across a full new execution venue (Lighter integration)]]

## Basket generation

- **Semantic tags + long-only / short-only basket generation** — closed-tag taxonomy on structured output, plus a directional generation path that bypasses two-sided-only pair-stat enrichment. [[My Work at Pear Protocol#2026-07-28 – 08-01 — Basket generation could only produce two-sided baskets, and had no way to tag them|#351]]
- **Prod cherry-pick of the above.** [[My Work at Pear Protocol#2026-07-28 – 08-01 — Basket generation could only produce two-sided baskets, and had no way to tag them|#352]]
- **Single-tag simplification** — collapsed the two-tier asset-class+themes model down to one tag after real usage. [[My Work at Pear Protocol#2026-07-28 – 08-01 — Basket generation could only produce two-sided baskets, and had no way to tag them|#367]]
- **Basket-stats scoped to the surface venue** — the Lighter-vs-Hyperliquid listed/unsupported check follows the same surface binding as asset validation. [[My Work at Pear Protocol#2026-08-24 — A single day of hardening the Lighter surface: wiring gaps, a doc that contradicted itself, and a crash that silently substituted the user's request|#588]]
- **`runtimeContext`: the agent can see the live Basket Builder draft** — request-scoped, never-persisted context (legs/side/weights/size) threaded SDK → controller → DAG, so "review my basket" reasons over the real draft instead of an invented one. [[My Work at Pear Protocol#2026-08-03 – 08-06 — The agent couldn't see the Basket Builder draft a user was actively editing|#379]]

**Related:** [[Directional Baskets — Skip Pair-Stat Enrichment]]

## Trade execution

- **Confirm-time margin check for basket opens** — fail-closed gate against fresh `availableToTrade` with a configurable buffer, positioned before any leverage/order write; reduce-only adjustments exempted. [[My Work at Pear Protocol#2026-08-24 — A confirm-time margin gate, a recurring ambiguous-symbol crash, and a hidden 2x-budget confirm|#569]]

**Related:** [[Confirm-Time Margin Gate]] · [[Resume Evidence#Designed a confirm-time margin safety gate]]

## Telegram / product surface

- **Slash-command usage counters** — Redis-backed `tg:cmd:count:<command>`, counted before the wallet gate so it measures intent, not just successful execution. [[My Work at Pear Protocol#2026-08-03 – 08-06 — The agent couldn't see the Basket Builder draft a user was actively editing|#380]] / [[My Work at Pear Protocol#2026-08-03 – 08-06 — The agent couldn't see the Basket Builder draft a user was actively editing|#390]] (cherry-pick)
- **Group-aware suggested actions + atomic callback consumption** — group trade pitches get conversational follow-ups instead of dead execution buttons; suggested-action callbacks made single-consume via a new atomic Redis `getDel`. *(feat commits inside* [[My Work at Pear Protocol#2026-07-30 – 07-31 — Group trade pitches showed execution buttons that only worked in DM|#362]]*)*

**Related:** [[Linkwallet Poll Crash Paged On-Call for an Expected Telegram Event]] · [[Telegram Outbound Delivery Ordering — Unresolved Investigation]]

## Observability / agent infrastructure

- **Full `APICallError` detail on LLM failures** — shared `formatLlmError()` surfacing status/body/provider/request-id instead of a bare message. [[My Work at Pear Protocol#2026-07-15 — LLM failures went dark past "the model errored"; the trade-execution prompt contradicted itself on leverage|#258]]
- **Basket-cron provider-error + step telemetry** — the daily basket-generation cron gets the same class of instrumentation plus a read-only replay harness. [[My Work at Pear Protocol#2026-07-20 — Chasing a recurring Gemini failure meant enriching the error pipeline twice|#298]]
- **Self-diagnosing signal predicate failures** *(open)* — distinguishes "re-measurement missed the crossing" from "crossing met but no authenticated execution proof" in the terminal alert instead of one undifferentiated bucket. [[My Work at Pear Protocol#2026-08-28 — The internal-leak guard blocked legitimate AI-industry reporting; signal failures collapsed two mechanisms into one alert; delivery ordering had no visibility (open)|#640]]
- **Telegram outbound delivery-ordering observability** *(open)* — per-turn send_start/complete/error logging via `AsyncLocalStorage`, to tell a real ordering bug from a client-rendering quirk. [[My Work at Pear Protocol#2026-08-28 — The internal-leak guard blocked legitimate AI-industry reporting; signal failures collapsed two mechanisms into one alert; delivery ordering had no visibility (open)|#641]]

**Related:** [[Chasing the Corrupted Thought Signature Failure]] · [[Telegram Outbound Delivery Ordering — Unresolved Investigation]] · [[Internal-Leak Guard Blocked Legitimate AI-Industry Reporting]] · [[Engineering Lessons#Observability helpers must never throw]]

## Model routing

- **Migrated remaining Gemini nodes to `gpt-5.6-luna`** — 10 node configs across `apps/agent` + `apps/context`, validated live before merge. [[My Work at Pear Protocol#2026-08-05 – 08-13 — Model rate-limit whack-a-mole: Gemini → Luna → Flash Lite, and MiMo → Gemini → Luna|#386]] / [[My Work at Pear Protocol#2026-08-05 – 08-13 — Model rate-limit whack-a-mole: Gemini → Luna → Flash Lite, and MiMo → Gemini → Luna|#387]] (cherry-pick)

**Related:** [[Model Routing Under Provider Instability]] · [[Resume Evidence#Live-verified every emergency model swap before merging]]

---

*Scope note: excludes bug fixes, hardening, and reliability work even where filed under a `feat()` commit prefix on a mixed PR — see the fuller "My Work at Pear Protocol" record for that side. Excludes two locally-seen commits (`9ae90601` wallet resolution, `286149662` notification scripts) with no corresponding PR found on GitHub — unverifiable, so left out rather than guessed at.*