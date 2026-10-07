# Writing PRs, Issues and Commits

How to write pull requests, issues and commit messages for me. For humans and AI both: someone with no chat context should be able to read it and act.

Order of authority:
1. The repo's own rules (CLAUDE.md, CONTRIBUTING, PR templates, commit conventions, commitlint)
2. This note
3. [[My Writing Style]] for voice and word choice

If the repo and this note disagree, the repo wins. Never add AI attribution anywhere (no Co-Authored-By, no "generated with", no mention of Claude or AI). See [[AI Vault Rules]].

## For every piece of writing

- Simple words. No jargon, no buzzwords, no corporate phrases. If a plain word exists, use it.
- Direct and precise. Say what happened, with the real name of the file, function, error or number.
- Mark what is confirmed, what is suspected and what is unknown. Never state a guess as a fact.
- Say what was NOT verified. An honest "not run" beats a vague "tested".
- No em dashes, no semicolons.
- No names of teammates and no quotes of what they said. State the decision neutrally.
- No trace IDs, session IDs or dated incident stories in code comments or prompts. A trace ID is fine in a commit message or PR description when the repo asks for it (it is the audit trail).
- Short beats long. Cut anything the reader does not need to act.

## Commit messages

- Follow the repo's format. In agent-pear-v2 that is `type(scope): summary`, for example `fix(signals): name the cause when an Elfa network error has an empty message`.
- Subject: lowercase after the prefix, present tense, says what the change does, no trailing period.
- Body only when the why is not obvious. One short paragraph: the cause, then the change.
- Include a Langfuse trace ID when the fix came from a trace, if the repo asks for it.
- No AI trailer. Never amend an earlier commit, make a new one.

## Pull requests

Voice: professional and well written, in my style. Proper capitalization, full sentences, correct grammar, no slang, no typos, no casual lowercase. Still plain and direct, not stiff or formal.

Structure that fits the repo (see PR #1087 as a model). Keep sections that apply, drop the ones that do not:

- **What**: one or two sentences. What this changes and what was seen.
- **Root cause**: the real cause, with the exact code path. Only for fixes.
- **Change**: bullets, one per file or behavior. Say what changed, not how the diff looks.
- **Verification**: what was run (typecheck, tests, replay) and the result. State clearly what was not run or not reproduced.
- **Out of scope**: related things noticed but not changed.

Also:
- Link the issue (`Closes #N`).
- Title follows the commit format.
- Say plainly when behavior does not change ("Log text only, no behavior change").
- Base branch is `development` unless the repo says otherwise.
- Draft it, show it to me, and wait for a yes before opening.

## Issues

Voice: plain and clear. Lowercase headings and a short tl;dr are fine, as in my earlier issues, but sentences stay clean and readable (adjust here if I want issues as formal as PRs).

Structure:
- **tl;dr**: what is wrong or needed and what we will do, in two or three lines.
- **Evidence**: what was checked and where. Numbers, links, trace links. Mark confirmed vs not checked.
- **Deliverables**: a checklist of small, concrete items. Each should be doable and checkable on its own.
- **How we check**: the commands or replay that prove it works.
- **Not in this issue**: related work left out on purpose.
- If there is a workaround for users meanwhile, list its steps.

Write it so anyone can pick it up without the chat.

## Before posting anything

- Re-read for em dashes, semicolons, names, quotes and AI attribution.
- Check every number and file name against the source.
- Ask me before posting, committing or pushing, unless I already gave approval for that exact action.

## Related
[[My Writing Style]] · [[Desired AI Voice]] · [[AI Vault Rules]]
