# Career Journey

How engineering scope changed across Brinavv → Pear Protocol. SuprHire omitted (no repo access as of this writing — see [[Resume#Resume Claims Needing Manual Verification]]).

## Confirmed progression

Directly supported by repo evidence.

**Ownership boundary: feature-add → architectural call.**
At Brinavv, [[Companies/Brinavv#Restructured the employee module out of the HR folder|PR #110]] shows making an unprompted structural call (pulling `employee` out of `hr/` as its own module) rather than just adding features inside existing boundaries. At Pear, this becomes routine: [[02 Work/Pear Protocol/My Work at Pear Protocol#2026-08-24 — A single day of hardening the Lighter surface|#584]] traces a crash to a validation-layer gap and fixes it by reusing an existing Zod schema from a different adapter — recognizing and closing an inconsistency between two adapters of the same system, not just patching a call site.

**Debugging depth: symptom fix → root-cause-from-production-trace.**
Brinavv evidence has no visible incident-investigation pattern — PRs there read as features and refactors, not bug hunts with a stated root cause chain (e.g. PR #124's fix has no recorded "why" beyond "corrected employee route"). Pear evidence is dense with this: [[02 Work/Pear Protocol/Resume Evidence#Root-caused a silent trade-execution defect from a cross-lingual production trace|#248]] traces a bug through a live cross-lingual Telegram session to its exact missing-argument cause, and [[02 Work/Pear Protocol/Resume Evidence#Found and closed a crash that was silently substituting a user's trade request|#584]] was found by manually triaging five days of Langfuse ERROR traces, not from a bug report. This is a category of work with no Brinavv precedent in the evidence available.

**Safety/trust design: absent → a recurring theme.**
No Brinavv PR is framed around a safety gate or trust boundary (financial or otherwise) — the closest is permission-check work in early commits (`0cc13934`), which reads as basic ACL plumbing. At Pear, the "trade-execution safety and disclosure" theme spans 6+ separately-dated PRs — [[02 Work/Pear Protocol/My Work at Pear Protocol#Recurring Engineering Themes|see the recurring-themes list]] — including a margin gate designed *ahead of* a failure mode ([[02 Work/Pear Protocol/Resume Evidence#Designed a confirm-time margin safety gate|#569]]), not just a reactive fix.

**Production operations: none visible → real deploy pipelines on both.**
Both companies show actual CI/deploy evidence — Brinavv has a staging workflow (backend) and 4+ Azure Static Web Apps workflows (frontend); Pear has a documented `development`→`main` GitFlow with protected branches and a mandatory back-merge step. This is present at both, so it is not counted as progression — noted here only to avoid the false impression that Brinavv was "just a school project" with no deploy story.

**Breadth: single-repo full-stack → cross-package, cross-team surface.**
At Brinavv, contribution spans one backend repo and one paired frontend repo, both single-team. At Pear, [[02 Work/Pear Protocol/My Work at Pear Protocol#2026-08-18 – 08-20 — Bringing up Lighter as a second execution venue|the Lighter integration work]] spans `apps/agent`, `packages/execution`, and the KB pipeline, plus authored commits landing inside a teammate's shared umbrella PR — direct evidence of working inside someone else's branch/plan, not just your own PRs.

## Inferred progression

Strong synthesis from the evidence, not directly stated by either source.

- **Independent investigation habit** likely strengthened at Pear specifically because the codebase has an actual observability substrate (Langfuse traces, structured error logging) to investigate *with* — Brinavv's evidence shows no comparable tooling, so the absence of trace-driven debugging there may reflect the environment, not a skill gap at the time.
- **Financial/safety-critical code discipline** (fail-closed gates, disclosure-before-execution) is plausibly Pear-specific domain training rather than a general engineering maturation — a school ERP genuinely has lower stakes per transaction than a trading agent, so the two aren't a clean apples-to-apples growth signal.
- **Working inside teammates' shared branches** (Lighter integration) suggests growing comfort with less self-contained ownership than Brinavv's PRs show, but the repo evidence can't establish whether Brinavv ever offered that kind of shared-branch work to begin with.

## Explicitly not claimed

- Brinavv came first chronologically, but chronology alone is not being read as "less skilled" — the safety/observability work at Pear reflects the domain (trading) and available tooling (Langfuse) as much as it reflects growth. [[Resume#Resume Claims Needing Manual Verification|See open verification items]] before treating this as a settled growth narrative for interviews.
- No claim is made about title, seniority, or team-size progression between the two — that data doesn't exist in either repo.
