---
type: knowledge
domain: AI & LLM Systems
confidence: deep
last_reviewed: 2026-08-30
---

# LLM Observability & Evaluation

## In Simple Terms

LLM systems need evidence about what happened, not just a final error string or a green HTTP status. Observability makes failures diagnosable; evaluation checks that a change improved the intended behavior without breaking the opposite case.

## What I Understand

At Pear I built a shared best-effort `formatLlmError()` that extracts provider status, body/data, URL, retryability, and request IDs without throwing inside a catch path. I used Langfuse traces to find silent context and schema failures, added telemetry to a previously dark basket cron, and used focused real-LLM evaluations for retry promises, leak detection, planner recovery, and retrieval.

I also learned that telemetry itself is part of the product: a capped preview needs its length and a truncation flag, and a detailed diagnostic must come from the same evaluator traversal as the boolean verdict.

## Why It Matters

Without useful traces, debugging becomes inference from symptoms. Without regression evaluation in both directions, a fix for false positives can introduce false negatives or a fix for one call site can leave the same failure alive elsewhere.

## How I Learned This

- [[02 Work/Pear Protocol/Incidents & Investigations/Chasing the Corrupted Thought Signature Failure]]
- [[02 Work/Pear Protocol/Incidents & Investigations/Telegram Outbound Delivery Ordering — Unresolved Investigation]]
- [[02 Work/Pear Protocol/Engineering Lessons#A capped telemetry field can manufacture a false incident]]
- [[02 Work/Pear Protocol/Engineering Lessons#Observability helpers must never throw]]
- [[02 Work/Pear Protocol/Engineering Lessons#Tuning a safety check in one direction can silently break the other]]
- [[02 Work/Pear Protocol/My Work at Pear Protocol#Recurring Engineering Themes]]

## Real Failure Modes I've Seen

- A Langfuse field named `finalText` was capped at 500 characters, making a complete 753-character response look genuinely truncated.
- OpenRouter returned an HTTP 200 containing a body-level error, but the diagnostic payload was in `APICallError.data` rather than `responseBody`; the old formatter lost the provider name.
- A loose request-ID header match recorded the literal `MISS` as if it were a real identifier.
- A chart fix covered the first observed call site while the same error recurred 13 times in three days through sibling DAG entry points.
- An open Telegram delivery investigation had no ordering evidence until send-start/complete/error instrumentation was added.

## Engineering Tradeoffs

- Logging more fields improves diagnosis but requires redaction and bounded output.
- Best-effort observability is safer than a formatter that can throw while handling another failure.
- Real-LLM evaluation is more representative than only mocks, but it is slower and provider-dependent.
- Instrumentation can establish facts for the next occurrence without proving the current root cause.

## Patterns / Rules I Currently Believe

**Strong evidence:** Telemetry fields should distinguish display truncation from data loss, and diagnostics should be generated from the same path that produces the verdict.

**Current belief:** A production AI change is not complete until there is a way to identify the failure class and a focused evaluation for the likely regression direction.

**Open question:** I have not yet defined a durable taxonomy for all LLM quality metrics or connected these evaluations to a long-term dashboard.

## Examples From My Work

- `formatLlmError()` evolved from 6/6 to 9/9 focused tests after a live Gemini failure exposed the `.data` path.
- Pear's KB evaluation held retrieval @1/@3/@5 at 97/100/100 after Lighter changes.
- Leak-guard tuning finished with 60/60 adversarial and benign cases; signal-evidence work had 40/40 tests including boolean/detail parity.

## Things I Still Don't Fully Understand

- How to choose representative evaluation sets for changing real-user distributions.
- How to quantify the operational value of a new trace field after deployment.

## Related

- [[LLM Reliability & Grounding]]
- [[RAG & Retrieval]]
- [[Retry Semantics & Failure Recovery]]
- [[Real-Time Communication & Delivery Ordering]]
- [[Model Routing Under Provider Instability]]

