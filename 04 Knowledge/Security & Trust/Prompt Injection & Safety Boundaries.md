---
type: knowledge
domain: Security & Trust
confidence: applied
last_reviewed: 2026-08-30
---

# Prompt Injection & Safety Boundaries

## In Simple Terms

LLM safety is a boundary around what the model may receive, reveal, or claim. A useful safety check must block actual unsafe behavior without treating ordinary public text as an attack merely because it contains a sensitive-looking word.

## What I Understand

CynoShield uses a fast regex layer for obvious injection/jailbreak/harmful requests, then an Ollama classifier returning strict JSON before main generation. Its own notes acknowledge weaker behavior for subtle multi-turn or obfuscated attacks and classifier-error fail-open behavior.

At Pear, an internal-leak guard evolved from an unconditional deny-list to a sentence-scoped disclosure check for model/provider names. A same-day review caught an over-correction, and the final change was tested against both benign reporting and real first-person disclosure frames.

## Why It Matters

Safety logic that only optimizes one error direction is brittle. False positives destroy useful product behavior; false negatives leak information or allow unsafe action. The boundary and its tests must reflect both.

## How I Learned This

- [[03 Projects/Real Projects/LLM_Firewall-main]]
- [[02 Work/Pear Protocol/Incidents & Investigations/Internal-Leak Guard Blocked Legitimate AI-Industry Reporting]]
- [[02 Work/Pear Protocol/Decisions/Internal-Leak Guard — Contextual Frame vs Unconditional Deny-List]]
- [[02 Work/Pear Protocol/Engineering Lessons#Tuning a safety check in one direction can silently break the other]]

## Real Failure Modes I've Seen

- An unconditional substring match blocked a legitimate Z.AI/Zhipu news summary because it mentioned GPT-5.6 and OpenRouter.
- The first contextual rewrite became too narrow and missed legitimate self-disclosure phrases such as “based on X” and “running on X.”
- CynoShield's project note identifies fail-open classifier errors and subtle multi-turn/obfuscated attacks as weaknesses, but those are project limitations rather than production incidents.

## Engineering Tradeoffs

- Regex is fast and explainable, but brittle against paraphrase and obfuscation.
- LLM classification handles harder cases, but adds latency, cost, and its own failure modes.
- Contextual matching preserves benign language, but requires a carefully maintained frame vocabulary.
- Blocking whole responses is simple, but can hide useful partial content and make false positives more severe.

## Patterns / Rules I Currently Believe

**Strong evidence:** Tune safety checks in both directions and keep adversarial plus benign near-miss cases in the same evaluation set.

**Current belief:** A safety boundary should distinguish internal operational disclosure from public discussion of the same names.

**Open question:** I have not yet measured attack success rates or false-positive rates across a broad, evolving prompt distribution.

## Examples From My Work

- Pear's final leak-guard suite: 60/60 cases.
- CynoShield's pipeline: `input → firewall → llm → output`, with the main generation blocked when the classifier marks the input unsafe.

## Things I Still Don't Fully Understand

- Robust multi-turn prompt-injection defenses.
- How to combine deterministic policy, model classifiers, and human review under uncertainty.

## Related

- [[Authentication, Authorization & Trust Boundaries]]
- [[Structured Outputs & Validation]]
- [[LLM Reliability & Grounding]]
- [[LLM Observability & Evaluation]]

