# AI Vault Rules

Operational rules for any AI maintaining this vault. This note itself follows its own rule: it requires my approval to change, same as Goals or My Identity.

## AI may automatically

- append verified work-history information
- add backlinks
- update indexes
- process Inbox items conservatively
- generate weekly reviews from existing evidence
- update Current Focus from explicit recent information
- identify stale/open items
- add evidence links

## AI requires my approval before

- deleting notes
- merging notes
- rewriting historical prose
- changing My Identity
- changing My Writing Style
- changing Goals
- changing an engineering decision after the fact
- moving large groups of notes
- changing vault structure

## AI must never

- turn hypotheses into facts
- fabricate missing context
- silently rewrite my personal writing
- overwrite historical records because newer information differs
- treat generated AI text as evidence about me
- duplicate information just to fill a template

Prefer linking to existing information over reproducing it.

## Maintenance Loop

### During the day

Capture freely into:
- Daily Work Log
- Inbox

Do not require categorization during capture.

### After meaningful work

When strong evidence exists:
- update [[My Work at Pear Protocol]]
- link relevant feature/system
- update an existing incident or decision when appropriate
- create a new incident/decision only when it genuinely deserves independent history

Do not generate a new permanent note for every PR.

### Weekly

Compile evidence from:
- Daily Work Logs
- Inbox
- My Work at Pear Protocol
- Incidents
- Decisions
- Projects

Create/update the current Weekly Review. Then:
1. update Current Focus
2. move resolved open loops out of Current Focus
3. append genuinely reusable Engineering Lessons
4. update Resume Evidence only when meaningful new evidence appeared
5. identify stale Inbox items
6. surface unresolved questions

Do not rewrite Identity, Writing Style, Goals, or historical notes.

### Monthly

Create a Monthly Review from weekly reviews and durable notes. Look for:
- recurring themes
- changes in priorities
- work with disproportionate impact
- repeated mistakes
- abandoned work
- changing opinions
- measurable progress

Do not turn ordinary activity into achievements.

## Source-of-truth hierarchy

When information conflicts, prefer evidence in approximately this order:
1. original logs / concrete evidence
2. PRs, commits, tests, metrics
3. Daily Work Logs
4. incident / decision notes
5. My Work at Pear Protocol
6. weekly/monthly summaries
7. AI-generated synthesis

Never use an AI-generated summary to override stronger original evidence. When newer evidence contradicts older evidence, preserve the historical claim and add the correction rather than silently rewriting history.

## When should a new note exist?

Create a standalone note when at least one is true:
- I will likely need to reference it independently later
- it represents a meaningful incident
- it captures a durable engineering decision
- it contains a reusable concept or lesson
- it has enough information to stand on its own
- multiple other notes naturally need to link to it

Otherwise prefer adding information to an existing note.
