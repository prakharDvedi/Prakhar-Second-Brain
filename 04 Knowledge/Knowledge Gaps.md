---
type: knowledge-gaps
domain: Technical Knowledge Base
last_reviewed: 2026-08-30
---

# Knowledge Gaps

These are gaps exposed by the work record, not a generic computer-science syllabus.

## PostgreSQL Internals and Operational Migrations

### Why I know it's a gap

I have used Postgres, Prisma, Supabase, FTS, RLS, and migrations, but the notes explicitly leave query planning, locking/isolation, indexing internals, and a proper migration toolchain as unfinished depth. MedBud required a throwaway `pg` script because the container lacked a durable database client workflow.

### What I already understand

I can model money with Decimal/numeric, use relational constraints, version reference data, use GIN-backed FTS, and scope records with RLS.

### What I need more depth in

Query plans, lock behavior, transaction isolation, zero-downtime migrations, and rollback strategy.

### Priority

High

## Queue Semantics, Idempotency, and Distributed Ordering

### Why I know it's a gap

Pear has retries, polling, atomic callback consumption, and an open Telegram delivery-ordering investigation, but no documented general idempotency-key strategy or proven ordering model across workers. The engineering notes explicitly leave queue retry policy and multi-leg compensation unresolved.

### What I already understand

I understand bounded retries, fresh-vs-original evidence, atomic single-consume operations, and the need to correlate asynchronous sends before naming a root cause.

### What I need more depth in

At-least-once delivery, deduplication, idempotency keys, dead-letter queues, ordering guarantees, and failure recovery across processes.

### Priority

High

## Production Infrastructure and Runtime Operations

### Why I know it's a gap

The vault shows real deployments and CI workflows, but also records no meaningful production infrastructure at scale, no autoscaling or infrastructure-as-code, incomplete deploy hardening in MedBud, and no monitoring in the ChatApp project.

### What I already understand

I can containerize multi-service applications, configure hosted deployment, use CI workflows, run smoke checks, and distinguish deployment from operational maturity.

### What I need more depth in

Networking, service discovery, secrets rotation, autoscaling, rollback/canary releases, and infrastructure observability.

### Priority

Medium

## LLM Quality Evaluation at Scale

### Why I know it's a gap

I have focused trace investigations and bounded evals—Pear retrieval, leak guard, planner recovery, and retry-promise tests—but no durable claim-level grounding metric, broad attack distribution, or long-term quality dashboard is documented.

### What I already understand

I can turn a concrete failure into a regression case, evaluate benign and adversarial directions, and pair retrieval changes with measured recall/MRR checks.

### What I need more depth in

Evaluation-set design, annotation, hallucination/grounding measurement, drift detection, and comparing providers without changing the task.

### Priority

Medium

## Multi-Leg Trade Compensation and Chain/Exchange Internals

### Why I know it's a gap

Pear documents partial fills, stale reduce-only orders, margin modes, and venue-specific execution, but explicitly leaves rollback/compensation for non-atomic multi-leg execution unresolved. Authentica demonstrates wallet/on-chain integration, while production nonce/reorg/key-management experience is not documented.

### What I already understand

I can place safety gates before writes, verify fresh state, preserve explicit execution mode, and report only venue-proven facts.

### What I need more depth in

Compensation design, liquidation/settlement, nonce and confirmation lifecycles, reorgs, and production custody/key management.

### Priority

Medium

