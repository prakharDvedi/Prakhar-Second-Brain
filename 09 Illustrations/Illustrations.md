# Illustrations

my creator space. films, animations, illustrations and visual stories i make with AI agents, all drawn and animated in code. this folder is the source of truth for that work: what each project is, what was decided and why, where it stands, and what to do next.

if you are an agent picking up work here, read this note first, then the project's `00 Index`.

## What this space is for

- short animated films (like [[Still Compiling/00 Index|Still Compiling]])
- style frames, samples and visual experiments
- illustration and motion work for my own stories, projects and posts

everything here is made in code: pages that draw each frame, rendered to video. no video generation models unless a project says otherwise.

## Rules for every project

1. **one folder per project, named after the project.** `Still Compiling/`, not `film-1/` or `new project/`.
2. **every project folder starts with these notes:**

| Note | What goes in it |
|---|---|
| `00 Index.md` | what the project is, current status, where the code and renders live, links to the other notes |
| `01 ...` onward | the plan or storyboard, numbered in reading order |
| `Decisions.md` | every real decision: what was chosen, why, what was rejected. confirmed vs not checked |
| `Progress Log.md` | dated entries, newest at the bottom. what was done, what broke, how it was fixed |
| `Next Steps.md` | open items, in order. the first one is what the next agent should do |
| `Tools and Setup.md` | tools used, install paths, commands to render, machine gotchas |

3. **log as you go.** after any meaningful session, append to `Progress Log.md` and update `Next Steps.md`. never rewrite old log entries. if something turned out wrong, add a dated correction below it.
4. **code and renders live outside the vault** unless i say otherwise (the vault is a git repo, big binaries don't belong in it). `00 Index` must give the exact path.
5. follow [[AI Vault Rules]]. no em dashes, write like [[My Writing Style]].

## Projects

| Project | What | Status |
|---|---|---|
| [[Still Compiling/00 Index\|Still Compiling]] | 98 s autobiographical animated film, college to engineer | style locked, opening test done, full film not started |
| [[Hera/00 Index\|Hera]] | my reusable cartoon character (a cat) for storytelling videos (Claude Animation Base) | design, 31 emotions, 5 views, showreel done. code committed on branch `hera` in ClaudeAnimationBase |

## Characters

reusable characters, kept outside any one project.

| Character | What | Lives in |
|---|---|---|
| [[Hera/00 Index\|Hera]] | my main cartoon character instead of Clawd. a slim dark purple-grey cat, round amber eyes, pink ears, whiskers, tail up. watercolour + ink, all of Clawd's 31 emotions, 5 turn views. made 2026-10-26 from a reference image i picked | `C:\Users\prakh\ClaudeAnimationBase\src\cat.js` (model sheets in `out\cat_*.jpg`) |

not used in Still Compiling (that film is Urban Sketch style).

## Shared toolkit

the logistics for every project (tool paths, dependencies, render and sound commands, Windows gotchas, logging rules) live in the global `/movie` skill at `C:\Users\prakh\.claude\skills\movie\SKILL.md`. `/movie` checks the setup, `/movie new <slug> "<Title>"` scaffolds a project here and its code folder. the Still Compiling specifics are in [[Still Compiling/Tools and Setup]].

## Related
[[Home]] · [[Projects]]
