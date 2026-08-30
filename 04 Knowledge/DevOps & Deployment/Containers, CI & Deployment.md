---
type: knowledge
domain: DevOps & Deployment
confidence: applied
last_reviewed: 2026-08-30
---

# Containers, CI & Deployment

## In Simple Terms

Deployment engineering is making the path from source code to a running environment repeatable, inspectable, and close enough to production that environment differences do not become the main source of bugs.

## What I Understand

I have used Docker and Docker Compose in ChatApp, Redis-chatApp, eVakeel2.0, Astrology-AI, MedBud, Finance, and the automatedTesting exercise. I have used hosted deployment paths including Vercel, Render, Streamlit Cloud, Azure Static Web Apps, and a hosted Supabase project. Brinavv has real GitHub Actions staging/production workflows, while Pear uses protected-branch GitFlow and live checks before merging provider changes.

The evidence is strongest for building and deploying applications, not for operating large infrastructure. I therefore classify this as applied rather than deep.

## Why It Matters

Environment drift, missing migrations, inconsistent ports, stale containers, and untested deployment assumptions can make a correct application fail before users ever reach its logic.

## How I Learned This

- [[03 Projects/Study and Practice/automatedTesting]]
- [[ChatApp]]
- [[eVakeel2.0]]
- [[Astrology-AI]]
- [[MediBuddy]]
- [[Medbud/Status]]
- [[05 Career/Companies/Brinavv]]
- [[05 Career/Career Journey]]

## Real Failure Modes I've Seen

- eVakeel's Docker Compose layout and port references do not fully match the current repository.
- MedBud's container did not have `psql`, Supabase CLI, or `pg` installed, leading to a documented throwaway migration-script workflow; deploy hardening remains incomplete.
- ChatApp documents manual deployment and no monitoring/CI in its known limitations despite a Render configuration.
- Brinavv's backend test script is a stub, so its deployment workflow does not imply meaningful automated test coverage.

## Engineering Tradeoffs

- Compose makes local multi-service setup repeatable, but the local topology can still diverge from hosted production.
- CI and protected branches improve confidence, but they do not replace runtime smoke tests or observability.
- A one-deployable-app architecture reduces operational surface, but places more responsibility on the application process and platform.

## Patterns / Rules I Currently Believe

**Strong evidence:** A deployment claim should be separated from a test/monitoring claim; hosting something does not prove it is operationally mature.

**Current belief:** The most valuable deployment check is the smallest end-to-end smoke test that exercises the actual hosted dependencies.

**Open question:** I have not yet operated production infrastructure with incident response, autoscaling, or infrastructure-as-code at meaningful scale.

## Examples From My Work

- MedBud workflows were verified live against hosted Groq and Supabase for both end-to-end demos.
- Astrology-AI reduced its Docker image by removing unnecessary GPU dependencies and measured cold/warm behavior.
- AutomatedTesting models unit, integration, API/E2E, and optional load tests across Dockerized services.

## Things I Still Don't Fully Understand

- Kubernetes, service discovery, secret rotation, and production networking.
- Release rollback, canary deployment, and infrastructure observability.

## Related

- [[PostgreSQL & Data Modeling]]
- [[Model Routing Under Provider Instability]]
- [[LLM Observability & Evaluation]]
- [[Real-Time Communication & Delivery Ordering]]

