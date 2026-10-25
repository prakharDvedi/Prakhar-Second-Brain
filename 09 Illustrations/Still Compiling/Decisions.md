# Decisions

dated, newest at the bottom. "confirmed" means i saw it or approved it. "not checked" means an agent chose it and i haven't reviewed it yet.

## 2026-10-26

### Film concept and arc
- **title:** Still Compiling
- **arc:** curiosity → freedom → exploration → friendship → falling for someone → disappointment → emptiness → obsession with building → failures and rejections → serious engineering → Pear Protocol (AI + crypto trading infra) → life has changed → only the beginning
- **tone:** coming of age film. not a resume, not motivational, not a success montage
- **the relationship:** subtle. two people spending time together, then absence. never "she rejected me so i became successful"
- **the climax is understated:** a quiet "Pear Protocol · production access granted" notification. no SUCCESS screen
- **recurring motif:** a blinking terminal cursor. opens on `> start`, becomes the college gate, comes back at every phase change, ends on `> next`
- **ending:** 2023 me walks into the gate while present me walks out, they pass. then my room, i leave, camera stays, laptop wakes, `> next`, black, STILL COMPILING
- **avoid:** supercars, money, SUCCESS screens, sigma grindset, hacker hoodies, exaggerated heartbreak, linkedin aesthetics
- status: confirmed (my brief)

### Everything is drawn in code, no video generation
- frames are drawn by JS in a browser page, captured frame by frame by headless Chrome, joined with ffmpeg. sound is synthesised in Python
- why: i asked for it. it also keeps the character and style consistent across shots, which video models are bad at
- status: confirmed

### Art style: Urban Sketch · Pen & Wash (Lemo-Opuscar)
- a sketchbook page drawn on location: wobbly sepia fineliner, loose watercolour off the lines, cream cold-press paper, handwritten margin notes
- why it fits:
  - **colour can be withheld and arrive as an event.** the world is ink only, colour shows up where something mattered. this gives the film its emotional colour logic: amber wash = people and college, cyan wash = code and systems. colour drains out in the absence scene. her mustard scarf can be "the one coloured thing"
  - **one continuous page** suits the one film, not separate clips idea. camera travel across the page replaces cuts
  - handwritten corner notes give dates for free ("Aug. 2023 · 6:10 am · first day")
- rejected: Watercolor Brush (no ink line, softer, less "observed"), Impasto Oil (heavier, less room for UI overlays)
- the style's own rules apply (in `C:\Users\prakh\lemo-opuscar\styles\urban-sketch\STYLE.md`): no dissolves, figures boil at 12 fps, colour never snaps, leave margins, text is handwriting on the page
- status: **confirmed**, i said "i like the style" after the opening test

### Film structure (treatment)
- one continuous page, colour withheld and arriving as the emotional signal (candidates 2 + 3 in `TREATMENT.md`)
- the three candidates were: a sketchbook page per phase, one continuous page, colour as memory
- status: chosen by the agent, not checked by me in detail

### Tool roles
- **Lemo-Opuscar** (Urban Sketch): the painted, character scenes and the overall look
- **Hyperframes** (HeyGen): UI and typography layers (terminal, Slack, Pear dashboard, charts, split screen) composited over the page. also does the scrapbook/collage scenes 03 and 05
- **OneTake**: the continuous camera shots through interfaces, scenes 08 and 11
- **Claude Animation Base**: optional. it overlaps Lemo-Opuscar and mixing both would give two clashing looks. leaning towards not using it
- **Collage View**: no such skill exists. dropped, Hyperframes covers it (my call)
- "Limo Opus Car" in my brief was actually Lemo-Opuscar. the first storyboard wrongly treated it as a camera move, corrected later
- status: confirmed

### Piano is synthesised for now
- felt piano built from additive partials in numpy, no samples
- why: real Salamander grand piano samples are a 218 MB download, the agent didn't download without asking
- the motif: low D2 alone in the dark when the cursor stretches, then A4 → F#4 → D5 → C#5, ending on C#5 on purpose so it feels unfinished
- status: works, i said "sound works". upgrading to real piano is still open

### Code lives outside the vault
- project code and renders are in `C:\Users\prakh\still-compiling\`, not in the vault
- why: the vault is a git repo, renders and fonts are binary
- status: agent's call, not checked by me
