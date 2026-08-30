---
type: knowledge
domain: AI & LLM Systems
confidence: deep
last_reviewed: 2026-08-30
---

# LLM Reliability & Grounding

## In Simple Terms

An LLM is useful only when its answer stays tied to facts the system actually supplied or verified. Reliability is not just preventing crashes; it is preventing confident answers, promises, defaults, and explanations that the system cannot support.

## What I Understand

I have learned to separate model-generated explanation from deterministic system truth. In Pear, the composer was prevented from inventing a clock, future retry capability, leverage defaults, and hidden margin changes. In MedBud, the model understands documents and explains findings, while code performs money math, matching, duplicate checks, and insurance calculations. Token-unlock amounts are withheld when the source contains only a projected ceiling rather than a committed claim.

Grounding also includes preserving the user's actual context. A streaming DAG path that passed empty history silently replaced an explicit 5% TP instruction with a 100% default.

## Why It Matters

The most dangerous AI failures often look polished: a fabricated time, an unsupported root cause, a silent default, or a promise of background work. In financial, legal, medical, and operational systems, a truthful limitation is safer than a fluent guess.

## How I Learned This

- [[02 Work/Pear Protocol/Incidents & Investigations/Composer Fabricated the Current Time]]
- [[02 Work/Pear Protocol/Incidents & Investigations/Composer Promised a Retry Capability That Didn't Exist]]
- [[02 Work/Pear Protocol/Incidents & Investigations/Silent TP-SL Default on the Streaming Chat Path]]
- [[02 Work/Pear Protocol/Incidents & Investigations/Partial Fill Alert Blamed the Wrong Cause]]
- [[02 Work/Pear Protocol/Incidents & Investigations/Token-Unlock Amounts Shown as Confirmed Facts]]
- [[MediBuddy]]
- [[02 Work/Pear Protocol/Engineering Lessons#A hardcoded root-cause string in an error message is a guess wearing a lab coat]]

## Real Failure Modes I've Seen

- The composer fabricated current time because no deterministic clock was present at its boundary.
- A failed trade step led the composer to promise an invisible retry loop and future notification that did not exist.
- A hardcoded partial-fill message blamed a notional floor even though the incident's numbers disproved that cause.
- A leverage clamp roughly doubled stated margin, but the ticket disclosed only the leverage adjustment.
- An LLM-generated document answer could have performed arithmetic or pricing reasoning that MedBud deliberately keeps in deterministic code.

## Engineering Tradeoffs

- More deterministic checks can add friction or reject an operation, but they make the trust boundary explicit.
- Prompt-level disclosure fixes are appropriate when the missing truth is in LLM-composed prose; changing the engine would be the wrong layer.
- Recovery is useful only when it preserves the original intent and has a bounded capability.
- Preserving uncertainty or returning `null` is sometimes preferable to filling a field with a plausible estimate.

## Patterns / Rules I Currently Believe

**Strong evidence:** The system should provide deterministic facts, source evidence, and actual capabilities to the model; the model should explain them without adding unsupported claims.

**Current belief:** Every user-visible claim should be traceable either to source evidence, a deterministic calculation, or an explicitly labeled inference.

**Open question:** I have not yet built a systematic claim-level grounding evaluator beyond focused regression cases and retrieval evaluation.

## Examples From My Work

- MedBud's insurance Q&A keeps the original English policy clause and page citation while answering in Hindi or Hinglish.
- Pear's `formatLlmError()` and trace review exposed when a model/provider failure was real rather than merely a truncated display field.
- Pear's fail-closed confirm-time margin gate checks fresh `availableToTrade` before money-moving writes.

## Things I Still Don't Fully Understand

- How to measure unsupported-claim rates across open-ended conversations.
- Which grounding failures should be blocked, caveated, or surfaced as an explicit unknown.

## Related

- [[RAG & Retrieval]]
- [[Structured Outputs & Validation]]
- [[LLM Observability & Evaluation]]
- [[Trading Execution Safety]]
- [[Retry Semantics & Failure Recovery]]

