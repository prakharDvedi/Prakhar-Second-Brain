---
type: knowledge
domain: Data & Databases
confidence: applied
last_reviewed: 2026-08-30
---

# PostgreSQL & Data Modeling

## In Simple Terms

Data modeling is deciding what the system must preserve, constrain, query, and explain later. The database design is part of the product contract, not just storage behind the API.

## What I Understand

I have used PostgreSQL with Prisma in Pear and the Finance API, Supabase Postgres with RLS and full-text search in MedBud, and PostgreSQL with Drizzle/JSONB in the outreach dashboard. The clearest practical decisions include using Postgres `numeric`/Prisma Decimal for money, soft deletion for financial records, versioned reference observations, page-aware policy chunks, and GIN-backed full-text search.

## Why It Matters

Incorrect types, weak constraints, or missing provenance can make a correct-looking application lose money, hide history, return untraceable answers, or become impossible to reproduce after reference data changes.

## How I Learned This

- [[03 Projects/Real Projects/Finance-Data-Processing-and-Access-Control-Backend]]
- [[MediBuddy]]
- [[Medbud/Status]]
- [[Personalized-Outreach-Dashboard]]
- [[05 Career/Companies/Pear Protocol]]

## Real Failure Modes I've Seen

- JavaScript floating-point arithmetic was unsuitable for money, so Finance uses Decimal mapped to Postgres `numeric`.
- MedBud's first full-text query semantics were too strict for multi-topic questions and needed an OR-based query construction.
- MedBud's migration workflow lacked a proper installed database client in the container, leading to a documented throwaway-script process and an explicit infrastructure gap.
- A client/schema change in Pear could break all reads if the external database migration had not landed first.

## Engineering Tradeoffs

- JSONB is flexible for variable prospect inputs and whole conversation threads, but relational columns are clearer for queryable facts and constraints.
- Soft delete preserves audit history but requires every read path to filter deleted rows.
- Full-text search is simple and explainable, but semantic paraphrases may require a measured retrieval upgrade.
- RLS and private storage improve tenant isolation, but add setup and debugging complexity.

## Patterns / Rules I Currently Believe

**Strong evidence:** Choose database types and constraints based on the invariant the business needs, not the convenience of the application language.

**Current belief:** Preserve version, source, and timestamp metadata whenever external reference data can change.

**Open question:** I have not yet developed deep knowledge of PostgreSQL query planning, locking, indexing internals, or operational migrations.

## Examples From My Work

- Finance: `numeric` money, soft deletion, role-aware queries.
- MedBud: `tsvector` + GIN index, RLS-scoped tables, versioned official references.
- Outreach: Drizzle schema with JSONB for variable inputs and server-side database reads.

## Things I Still Don't Fully Understand

- Query plans, isolation levels, lock behavior, and index maintenance.
- Safe zero-downtime schema migration and rollback strategies.

## Related

- [[RAG & Retrieval]]
- [[Redis & Caching]]
- [[APIs & Layered Backend Architecture]]
- [[Authentication, Authorization & Trust Boundaries]]

