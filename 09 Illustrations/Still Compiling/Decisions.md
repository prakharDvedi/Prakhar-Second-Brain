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

### Hera as an easter egg
- my cat character [[Hera/00 Index|Hera]] gets hidden cameos in the film. where exactly is still open (candidates in [[Next Steps]])
- she gets redrawn in the film's Urban Sketch style, same colours, so she doesn't look pasted in from another kit
- status: idea confirmed by me, placement not decided

### Code lives outside the vault
- project code and renders are in `C:\Users\prakh\still-compiling\`, not in the vault
- why: the vault is a git repo, renders and fonts are binary
- status: agent's call, not checked by me

### Protagonist look (P23 / P26)
- black medium wavy hair, half-rim glasses, 6 ft and lean, fair wheatish skin, no beard, normal shirts and trousers, black analog watch. cartoon, doesn't need to be exact
- status: confirmed (my answer, 2026-10-26)
- drawn with the Urban Sketch figure rig, face blank like the rest of the style, glasses inked on the blank face so they're the one facial feature
- shirt colours: P23 white half-sleeve shirt + beige trousers, P26 muted navy full-sleeve shirt + charcoal trousers. picked to stay out of the film's colour grammar (no amber, cyan or pear green on him). status: confirmed, i approved the sheet with these colours (2026-10-26)
- character work uses its own copy of the engine in `chars\` with two small fixes (shorter neck tube, shirt over trouser tops). status: agent's call, not checked

### Tempo grid: 120 bpm, 4/4
- every whole second is a beat, a bar is 2 s, slow sections felt in half time (60). keeps every storyboard scene time exactly on the grid, no retiming needed
- the opening test's motif notes were 0.65 s apart. on this grid they'd move to 0.5 or 0.75 s when the score is redone
- rejected: around 90 bpm to match the opening test's motif (half the scene boundaries would fall between beats)
- status: agent's call, not checked

### Film length 101 s
- scene 13 runs to 101 s so the end title holds 4 s (library rule). status: confirmed, my call 2026-10-26

### How tools join (from the 03 → 04 test)
- every tool renders at 24 fps. Lemo renders at 96 fps and is blended down for motion blur
- a seam is a matched frame: the outgoing shot ends exactly on the incoming shot's first frame (same layout, font, colours, cursor phase), and the camera comes to rest 0.1 s before the cut
- the cursor blinks on the film clock everywhere, so its phase matches across any cut
- fonts are loaded from the project's own files in every tool, never a bundled or system font
- for now clips are joined through RGB with each clip's own colour matrix. lossless intermediates are the planned fix
- status: confirmed, i watched the test and liked it (2026-10-26)

### Creative calls (2026-10-26, mine)
- gate arch reads **IIIT Bhopal** instead of the "COLLEGE OF ENGINEERING" placeholder
- music: keep the synthesised score (felt piano and the rest made in code), no sample download
- **no voiceover** for now
- page layout: follow the treatment, **one continuous page** the camera travels across
- Hera: the agent places her, small and easy to miss, in a few of the candidate spots
- i liked the 03 → 04 transition and the design, so the seam method in "How tools join" is approved
- status: confirmed
