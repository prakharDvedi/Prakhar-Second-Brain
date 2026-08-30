---
type: knowledge
domain: AI & LLM Systems
confidence: deep
last_reviewed: 2026-08-30
---

# Structured Outputs & Validation

## In Simple Terms

When a model is asked to produce data that will drive software, the model's output is an untrusted API request. A schema is not documentation around the request; it is the boundary that stops malformed or semantically wrong data from reaching the next layer.

## What I Understand

I have used Zod and JSON-shaped contracts for planner decisions, trade conditions, basket tags, financial API input, MedBud document extraction, and the CynoShield firewall classifier. The important distinction is where validation happens: it must occur before the usecase or side effect, and every adapter reaching that usecase must enforce the same contract.

The schema also expresses what the model can honestly know. Pear's basket taxonomy excluded state labels such as Trending/New/Hot because the model could not know those states at generation time.

## Why It Matters

An invalid model response can do more than crash. It can trigger a retry that changes the request, silently route to a default plan, or produce a result that looks valid while violating user intent.

## How I Learned This

- [[02 Work/Pear Protocol/Incidents & Investigations/TRIGGER-LADDER JSON Crash Silently Substituted the User's Trade]]
- [[02 Work/Pear Protocol/Incidents & Investigations/Ambiguous HIP-3 Symbol Crashed the Whole Reply, Repeatedly]]
- [[02 Work/Pear Protocol/Engineering Lessons#An unvalidated shortcut on one of two parallel adapters reopens a hole the other already closed]]
- [[02 Work/Pear Protocol/Engineering Lessons#A structured-output schema is also a truthfulness boundary]]
- [[03 Projects/Real Projects/Finance-Data-Processing-and-Access-Control-Backend]]
- [[03 Projects/Real Projects/LLM_Firewall-main]]
- [[MediBuddy]]

## Real Failure Modes I've Seen

- A JSON-string `condition` was parsed and spread as `never` in one agent adapter, while the MCP adapter already used the correct discriminated-union schema. The malformed weighted-basket condition crashed and the model retried as a different price condition.
- A planner emitted a phantom tool call instead of the required decision object; the system silently substituted `generalChat` until a schema-focused recovery was added.
- A field that represented model-unknown runtime state would have invited fabrication; the field was removed from the structured contract instead.
- In a finance API, accepting a role during registration would have allowed privilege escalation, so the input contract always creates a viewer and leaves role changes to an admin endpoint.

## Engineering Tradeoffs

- Strict validation surfaces errors early but can create retry storms when the model is consistently near-valid.
- Schema recovery can help when the model understood the request but emitted the wrong shape; it should be narrowly gated and bounded.
- A closed enum makes downstream behavior predictable, but it needs a deliberate normalization policy when the model sends an unknown value.
- Validation at a shared boundary is safer than relying on each caller to remember it, though existing adapter architecture may require incremental hardening.

## Patterns / Rules I Currently Believe

**Strong evidence:** Validate model-produced data before usecase execution, and test every adapter that reaches the shared write path. Do not let a retry change the request type without making that visible.

**Current belief:** A schema should encode both data shape and the epistemic boundary—what the producer can know at that moment.

**Open question:** I have not yet compared provider-native structured-output guarantees with the Zod-after-generation approach used in these projects.

## Examples From My Work

- Pear reused the MCP adapter's `TriggerOpenInput`/`LadderCreateInput` schemas in the agent/Telegram adapter before the trade usecase.
- Pear changed leverage validation to require an integer before confirmation rather than failing after a write path began.
- CynoShield's classifier returns strict JSON with `allowed`, `risk`, `reason`, and `attack_type` before main generation runs.
- MedBud uses structured extraction, then keeps arithmetic and policy calculations in TypeScript.

## Things I Still Don't Fully Understand

- How to design schemas that remain stable across multiple model providers and tool-calling formats.
- How much bounded repair is safe for each class of malformed output.

## Related

- [[LLM Reliability & Grounding]]
- [[Retry Semantics & Failure Recovery]]
- [[APIs & Layered Backend Architecture]]
- [[Authentication, Authorization & Trust Boundaries]]
- [[Trading Execution Safety]]

