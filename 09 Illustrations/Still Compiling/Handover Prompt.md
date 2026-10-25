# Handover Prompt

paste the block below into a new Claude Code session started in `C:\Users\prakh\Downloads\Prakhar`. written 2026-10-26.

```
/movie Still Compiling

You're picking up my short animated film "Still Compiling": a 98 s, 13-scene coming-of-age film about my
college-to-engineer journey (2023 college entry → friends → someone → absence → building → Pear Protocol → leaving).
Everything is drawn and scored in code. No video generation models.

Read first, in this order, before doing anything:
1. C:\Users\prakh\.claude\skills\movie\SKILL.md            (logistics: tools, paths, commands, Windows gotchas, logging rules)
2. C:\Users\prakh\Downloads\Prakhar\09 Illustrations\Illustrations.md
3. 09 Illustrations\Still Compiling\00 Index.md, then Next Steps.md, Decisions.md, the latest Progress Log.md entries
4. 09 Illustrations\Still Compiling\01 Storyboard.md     (all 13 scenes + sections A–H: character, art direction,
                                                           avoid list, sound timeline, assets, build order, continuity rules)
5. C:\Users\prakh\still-compiling\TREATMENT.md
6. C:\Users\prakh\lemo-opuscar\styles\urban-sketch\STYLE.md and DEMO.md (the locked style's rules and engine API),
   then C:\Users\prakh\lemo-opuscar\DIRECTOR.md, TECHNIQUE.md, core\README.md

Then run: sh ~/.claude/skills/movie/scripts/check.sh
Only voice tier, music tier and instrument samples should be MISSING (skipped on purpose).

Where things stand:
- Style is locked: Urban Sketch · Pen & Wash from the Lemo-Opuscar library. Its STYLE.md rules apply
  (no dissolves, line before colour, colour arrives as an event, figures boil at 12 fps, handwriting on the page).
- Colour grammar: amber wash = people/college, cyan = code/systems, pear green = the Pear era.
- Built so far: a 7.5 s opening test (scenes 01→02) with sound.
    picture:  C:\Users\prakh\still-compiling\clip\opening.js   (timeline table T at the top drives everything)
    sound:    C:\Users\prakh\still-compiling\clip\mix.py       (reads events.json exported from window.EV)
    output:   C:\Users\prakh\still-compiling\clip\out\still-compiling-opening-test.mp4
  and a scene 02 style still: C:\Users\prakh\still-compiling\sample\gate.js → scene02-gate-sample.jpg
- The Urban Sketch engine (engine.js, poses.js) and OFL fonts (Caveat, Reenie Beanie, JetBrains Mono) are already
  copied into the project folders. Copy what you need from there, never build inside $LIB/styles/.
- The full film is NOT started.

How to work:
- Next Steps.md is the plan. Start at its first unchecked item (character sheet for P23 / P26: ask me for my real
  look first, the storyboard's section A traits are placeholders).
- Creative decisions (story, shots, look, Hera placement, voiceover) are mine. Propose, don't decide.
- Render commands, the page contract, loudness targets and the Windows rules are in the /movie skill. In short:
  run tools from ~/lemo-opuscar, never run the library's setup.sh here, Python is ~/lemo-opuscar/.venv/bin/python
  (a shim), OneTake runs via `sh ~/.claude/skills/onetake/py`, not python3.
- Tool roles: Lemo-Opuscar (Urban Sketch) for the painted scenes and the look, Hyperframes for UI/type/collage
  layers, OneTake for the continuous interface shots 08 and 11.
- Check your own work: contact sheets before full renders, ffprobe durations, mux.sh at −14 LUFS / −1.2 dB peak.
  Say plainly what you only measured and didn't watch or hear.
- Easter egg: hide my cat character Hera in the film (candidate spots in Next Steps.md, I pick). Redraw her in
  Urban Sketch style with her colours (fur #4B3F55, amber eyes #E9A93A, pink ears #E8948A). Her character bible
  is 09 Illustrations\Hera\Character Bible.md. Don't import ClaudeAnimationBase/src/cat.js, it's a different engine.
- After each session: append to Progress Log.md, update Next Steps.md / Decisions.md / 00 Index.md status, in my
  writing style (no em dashes). Don't commit or push without asking me.
- C:\Users\prakh\still-compiling\ is not under git yet. Suggest making it a local repo before big changes.
```
