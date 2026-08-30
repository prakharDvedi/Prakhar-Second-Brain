---
type: knowledge
domain: AI & LLM Systems
confidence: applied
last_reviewed: 2026-08-30
---

# Model Routing Under Provider Instability

## In Simple Terms

When a model stops working, the first job is to identify what stopped working. A rate limit, deprecation, capacity wall, malformed provider response, and tool-loop latency problem are different operational events, even if the user only sees “the model failed.”

## What I Understand

At Pear, several models required emergency swaps in one production window. I distinguished 429-style rate limiting from flat 502/deprecation behavior, selected replacements per node, and live-verified each replacement with a bakeoff, concurrent burst, or tool-loop latency check before merging.

I have also used provider fallbacks in personal projects: Groq/Ollama in Astrology-AI, Groq/OpenAI planning in SupraThon, and OpenRouter in outreach and Pear. The repeated production-grade evidence is concentrated in Pear; the personal projects establish exposure to provider abstraction and fallback, not the same operational depth.

## Why It Matters

Routing is part of reliability. A swap that fixes capacity but breaks tool use, latency, output shape, or cost can turn one visible incident into a less diagnosable regression.

## How I Learned This

- [[02 Work/Pear Protocol/Decisions/Model Routing Under Provider Instability]]
- [[02 Work/Pear Protocol/My Work at Pear Protocol#2026-08-05 – 08-13 — Model rate-limit whack-a-mole: Gemini → Luna → Flash Lite, and MiMo → Gemini → Luna]]
- [[02 Work/Pear Protocol/Incidents & Investigations/Chasing the Corrupted Thought Signature Failure]]
- [[Astrology-AI]]
- [[SupraThon]]
- [[Personalized-Outreach-Dashboard]]

## Real Failure Modes I've Seen

- Multiple upstream models hit rate limits or capacity walls close together.
- An HTTP-200 provider response contained an error object, so transport status alone was misleading.
- A replacement needs to be checked on the actual node behavior, not just by proving that a simple completion works.

## Engineering Tradeoffs

- Repeated swaps can create churn, but waiting for a “final” provider answer is worse when production needs a working route.
- A model abstraction makes provider changes cheaper, but it does not erase differences in tool calling, latency, context, or failure payloads.
- A fallback may preserve availability while reducing quality or observability; that tradeoff needs an explicit check.

## Patterns / Rules I Currently Believe

**Strong evidence:** Classify the failure before swapping, then validate the replacement in the workload shape that matters.

**Current belief:** Provider choice should be treated as runtime infrastructure, not as a one-time configuration detail.

**Open question:** The vault does not establish why the three Pear providers degraded in the same window or what long-term routing policy replaced emergency swaps.

## Examples From My Work

- Flash Lite bakeoff: 1/1.
- Luna concurrent burst: 8/8 without 429s.
- Gemini advanced-mode smoke test: about 5.4 seconds with streamed reasoning.

## Things I Still Don't Fully Understand

- How to build a durable routing policy from historical latency, error, and quality data.
- How model fallback affects evaluation comparability and user-visible behavior.

## Related

- [[LLM Observability & Evaluation]]
- [[LLM Reliability & Grounding]]
- [[Retry Semantics & Failure Recovery]]
- [[Containers, CI & Deployment]]

