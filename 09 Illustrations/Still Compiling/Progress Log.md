# Progress Log

dated entries, newest at the bottom. don't rewrite old entries, add a correction below instead.

## 2026-10-26

### 1. Storyboard
- gave the brief: director / storyboard / cinematographer for a 75 to 100 s film, 10 to 14 scenes, 15 fields per scene, plus global prompts and continuity rules
- agent wrote 13 scenes, 98 s total, plus sections A to H. full text in [[01 Storyboard]]
- at that point none of the named skills were installed, so the agent used their names as technique labels and guessed "Limo Opus Car" was a camera move (wrong, see step 2)

### 2. Finding and installing the skills
- searched online and on GitHub, vetted each repo (stars, licence, manifests, no hooks) before installing
- found:
  - Hyperframes: `heygen-com/hyperframes` (HeyGen official, Apache 2.0, ~72k stars). **installed** as user plugin `hyperframes@hyperframes`
  - Lemo-Opuscar (= "Limo Opus Car"): `lemomo-ai/lemo-opuscar` (MIT, ~7.3k stars). **installed** as user plugin `lemo-opuscar@lemolab`
  - OneTake: `feitangyuan/onetake` (~3.2k stars). a 0-star copy `JackChen1220/onetake` exists, it's not the real one
  - Claude Animation Base: `JohnHeibel/ClaudeAnimationBase` (MIT, ~1.8k stars). a starter kit you clone and open, not a skill
  - Collage View: does not exist. closest were small community collage skills (2 to 4 stars), skipped
- auto mode blocked the agent from cloning OneTake and Claude Animation Base, so i ran the clones myself
- **problem found later:** i ran them in PowerShell from the vault, `~` didn't expand, so both landed inside the vault at `Prakhar\~\.claude\skills\onetake` and `Prakhar\~\ClaudeAnimationBase` instead of my home folder. still there, not moved yet (see [[Next Steps]])
- decided: skip a separate collage skill, do scrapbook scenes with Hyperframes

### 3. Picking the style
- confirmed with me: everything drawn in code, no video gen
- Lemo-Opuscar library cloned to `C:\Users\prakh\lemo-opuscar\` (59 styles)
- shortlisted Urban Sketch, Watercolor Brush, Impasto. picked Urban Sketch, reasons in [[Decisions]]
- wrote `TREATMENT.md` first (the library requires a treatment before reading its demo code)

### 4. Scene 02 style still
- copied the style's engine (`engine.js`, `poses.js`) and OFL fonts (Caveat, Reenie Beanie) into `C:\Users\prakh\still-compiling\sample\`
- wrote `gate.js`: college gate at sunrise, campus block behind, trees, path, carrot people, P23 walking in with a navy backpack, freshers banner, "COLLEGE OF ENGINEERING" on the arch, sun as reserved paper, colour arrived from the right with the left still ink, corner note, title with an inked cursor
- first render issues: sun hidden behind a tree, building ink showing through the pillars, left tree outlines looked like birds. fixed (moved sun, knock out far ink inside pillars, scallops hug the crown)
- output: `sample\scene02-gate-sample.jpg`

### 5. Explained how scenes become a film
- each scene is `render(t)`, the renderer calls it 24 times a second. transitions are camera travel across one big page, line before colour, colour arriving, page turns, match cuts, pushing into screens. no dissolves (style rule)

### 6. Opening test clip (scenes 01 to 02, 7.5 s)
- `clip\opening.js`. timeline table `T` at the top drives everything
  - 0.0 to 1.3 s: black, `> start` types, cursor blinks
  - 1.3 to 2.1 s: cursor stretches into the left pillar
  - 1.9 to 3.0 s: black recedes from the pillar with a ragged edge, paper revealed
  - 2.0 to 3.6 s: gate draws itself stroke by stroke (pillars, arch + name, gate leaves, building, trees, ground, people)
  - 3.4 s on: P23 walks up the path, boiling at 12 fps
  - 3.6 to 5.2 s: sun outlined, colour spreads from it, stops before the left trees
  - 4.7 to 7.5 s: corner note, title, blinking inked cursor
  - slow 4% push-in across the whole shot
- checked 8 key frames on a contact sheet before rendering. fixed: cursor lagging behind the typed text, page reveal looked like smoke (made the edge sharper and raggeder), paper mottling too strong
- rendered 180 frames in 12 s with 3 workers

### 7. Fixing Python audio on Windows
- the library's setup made a venv but its Python lives in `.venv\Scripts\` on Windows, the library expects `.venv/bin/python`
- fix: installed `numpy scipy soundfile soxr pillow` with `uv`, and added a small shim `C:\Users\prakh\lemo-opuscar\.venv\bin\python` that execs `Scripts/python.exe`
- **gotcha:** the library's `setup.sh` mangles Windows paths in Git Bash (sparse checkout patterns became `C:/Program Files/Git/...`, and a re-run tried to clone into a broken path). it didn't delete anything. sparse checkout was fixed by running `git sparse-checkout set` by hand with `MSYS_NO_PATHCONV=1`. don't re-run `setup.sh` on this machine

### 8. Sound for the opening test
- the page exports sound cues (`window.EV`): 7 key presses, cursor blinks, the stretch, the page opening, 243 pen strokes with their exact times and lengths, 3 handwriting passes, the colour wash, birds start
- `clip\mix.py` synthesises everything from `events.json`: cursor ticks and keys, pen on cold-press paper (one burst per stroke), wet brush sweeping right to left, paper room tone, birds (J-cut 0.3 s before colour), felt piano (D2 then A4 F#4 D5 C#5 over D3 + A3), small room reverb
- measured and fixed: low end was +1.5 dB vs mids (target about -3), now -4.1. typing was ~20 dB under the rest, raised. peaks clipped the master once, lowered the limiter ceiling
- final master: -14.0 LUFS, true peak -1.2 dB. video and audio both 7.5 s
- the agent checked levels by numbers only, i listened and said "sound works"

### 9. Stopped before the full film
- i asked to start the full 98 s film, then stopped it and asked for this vault space first. nothing of the full film was built

### 10. Vault space and the misplaced clones
- created `09 Illustrations/` (root note [[Illustrations]]) and this project folder, linked from [[Home]] and the vault README
- moved OneTake to `C:\Users\prakh\.claude\skills\onetake` and Claude Animation Base to `C:\Users\prakh\ClaudeAnimationBase`, removed the empty `Prakhar\~\` folder. both checked: files present, Claude Animation Base git tree clean
- OneTake shows up as a skill only in a new Claude Code session

### 11. /movie skill, OneTake and Claude Animation Base set up
- made the global `/movie` skill (`~/.claude/skills/movie/`): logistics only, plus `check.sh` (dependency check) and `new.sh` (project scaffold). both tested
- rule i set: nothing may break existing Python or other tools on the PC
- OneTake: own venv `~/.claude/skills/onetake/.venv` from uv's existing Python 3.12 (no new system Python), packages playwright numpy scipy Pillow matplotlib fonttools brotli opencv-python. its own Chromium inside `.venv/ms-playwright`, so Playwright's browser cleanup can't touch Lemo-Opuscar's headless shell. launcher `onetake/py` replaces `python3`. checked: imports work, Chromium launches and renders a page. OneTake's `test_motion.js` self-test was blocked by auto mode, not run
- Claude Animation Base: `npm ci --ignore-scripts`, 46 packages, 72 MB, local to its folder. test render of the demo at 3 s and 6 s worked (Clawd, moon, hills)
- verified unchanged after: system Python 3.10 path, `python3` still the Store stub, same 6 uv Pythons, Lemo venv imports, shared browser cache folders, Lemo-Opuscar still renders
- still not installed on purpose: voice tier, music tier, instrument samples

### 12. Handover prep
- checked that every path these notes reference exists (code, renders, library docs, engine, venv shim, skills): all present
- added Hera easter eggs as an idea with candidate spots, wrote [[Handover Prompt]] for the next session

### 13. Character sheet v1 (P23 / P26)
- picked the project up with /movie. check.sh: everything OK, only voice tier, music tier and instrument samples MISSING (skipped on purpose)
- gave my real look: black medium wavy hair, half-rim glasses, 6 ft lean, fair wheatish skin, normal shirts and trousers, black analog watch, no beard. updated [[01 Storyboard]] section A
- new folder `C:\Users\prakh\still-compiling\chars\` (own copy of `engine.js`, `poses.js`, fonts). `sheet.js` draws a 2560×1440 sheet with the real Urban Sketch figure rig: P23 and P26 each in front, 3/4, side, back and one walking pose (P23 walking in, P26 walking out), plus palette swatches and handwritten notes
- added on top of the rig: half-rim glasses (heavy top bar, hairline bottom, temple in profile, hidden from behind), wavy hair marks (P23 messier, bigger hair mass, a lock on the forehead), black watch on the left wrist, backpack (full from behind, slab in profile, straps from the front, P26 faded with a patch)
- two small changes in the chars copy of `engine.js` (not in clip/ or sample/): the neck tube is shorter so its ink no longer draws over the chin and the back of the head, and `look.shirtOut` draws the shirt over the trouser tops so the hips stop showing two round caps
- fixed after the first render: row labels overlapped the first figures (moved figures right), P26 navy read as grey (more saturated), profile backpack floated off the back
- output: `chars\character-sheet-v1.jpg`. i checked it by eye at full size plus an enlarged crop of all heads (glasses read as half-rim in front, 3/4 and side)
- known flaws still in v1: P26's long sleeves show rounded shoulder caps that look a bit like pads, and the side-view feet draw as a stack of short lines (both come from the rig)
- shirt colours are the agent's proposal, not confirmed. not a git repo yet, didn't run git init (no answer on that)

### 14. Character sheet approved, master timeline
- i approved character sheet v1, colours included
- wrote `C:\Users\prakh\still-compiling\timeline.js`, the one source for picture, sound and checks: tempo grid, all 13 scene windows from [[01 Storyboard]] (unchanged), per scene the allowed amount of amber / cyan / pear green, music state, ambience bed and tools, 65 named hit points (`s07_fail`, `s12_pass` and so on), where the cursor appears, and the two music-free windows (36 to 42, 59 to 64)
- `check-timeline.mjs` checks it (scenes join with no gaps and end at 98 s, every hit sits inside its own scene and on the half-beat grid, cursor scenes exist, silences don't overlap scenes with music, plus a small self-test) and writes `timeline.json` for the Python mixer. runs clean
- the check warns that the end title only holds 1 s (97 to 98 s), the library asks for at least 4 s. left for me to decide
- added `package.json` with `"type": "module"` in the project root so Node reads `timeline.js` as a module without a warning
- the render tools only mount the page's own folder, so pages that import `timeline.js` have to live in the project root (fits the one continuous page plan)
- hit times are the agent's placement of the storyboard actions on the grid, not checked by me
- broke and fixed: a sed edit uppercased the Windows paths in these notes (`\U` in GNU sed), fixed by hand

### 15. Local git repo, transition check
- `git init` in `C:\Users\prakh\still-compiling\` (i said yes). `.gitignore` keeps `out/`, `stills/`, `node_modules/`, `*.mp4`, `*.wav` out. nothing committed, nothing pushed
- checked the tools for joining scenes: OneTake's `render.py` takes `--fps 24` (its defaults are 30 for drafts and 60 for finals), Hyperframes renders at `--fps 24`, Lemo-Opuscar renders at 24. so all three can render at the film's frame rate, no conversion needed. not tried yet, only read in their code and docs
- the title hold question is still open

### 16. Film is 101 s, transition test 03 → 04
- i chose to extend the film so the end title holds 4 s. `timeline.js`: scene 13 now runs 88 to 101, `DUR = 101`. the check runs clean with no warning. storyboard and index updated
- built the transition test i asked for in `C:\Users\prakh\still-compiling\tests\t0304\` (film 18.0 to 23.5 s, 5.5 s), one command: `sh build.sh`
  - Lemo half (`lemo/scene.js`, 18.0 to 21.0): tail of scene 03 on the corkboard, two earlier polaroids (football, chai cups), the camera drifts to an empty spot, the last card draws itself in stroke order (frame, photo, hostel block, window grid, the lit window, laptop, P23's head), the night wash blooms after the ink, the lamp-lit window comes on, then a push into the laptop screen from 1.35× to 160× (eased in log space, a slight roll so the screen ends square). card ink is made once in world space and mapped through the camera, so nothing boils while zooming, with a width cap so lines stay fineliner thin up close
  - rendered at 96 fps and blended to 24 fps (tmix of 2, every 4th frame) for a real 180° shutter blur on the push
  - Hyperframes half (`hf/index.html`, 21.0 to 23.5): the same screen, `> index.html` typed, Enter, then `<h1>hello world</h1>` with the tags in cyan (the first cyan in the film, as the palette rule says for 04). cursor blink uses the film clock on both sides so the phase matches across the cut. lint and check clean, one expected warning (the test ends right after the last line)
  - sound (`mix.py`): fest crowd bleed and brushes on the 120 bpm grid under 03, pen strokes and a wet brush for the card, a soft click when the screen lights, an air rush on the push that stops at the cut, the hostel fan comes in 0.4 s before the cut (J-cut), keys, a deeper Enter, cursor ticks on 04's blinks
- broke and fixed on the way:
  - Hyperframes drew the terminal in Times, its bundled JetBrains Mono didn't apply. fixed with an `@font-face` pointing at the project's own TTF. after that the `>` lands on exactly the same pixels on both sides of the cut
  - the push was still moving at the cut (about 10 px jump). now it lands 0.1 s early so both sides are at rest
  - the two clips use different colour matrices and ranges (Lemo full range bt601 from JPEG screenshots, Hyperframes tv range bt709). the first join shifted the HF side 4 to 5 levels toward blue. joining through RGB brought it down to about 2 levels in blue on the near-black screen. fully fixing it needs lossless intermediates (added to [[Next Steps]])
  - first master peaked at −1.0 dB. lowered the limiter ceiling relative to the RMS and the bass bleed: −13.7 LUFS, true peak −1.6 dB, low end −3.9 dB vs mids
- checked: frame strip across the cut (frames 62 to 77), pixel diff of the two frames at the seam, colour measured either side, ffprobe 5.5 s with video (132 frames) and audio, no black frames
- only measured, not watched or heard: i haven't seen it play at full speed or heard it. it's waiting on me

### 17. Decisions batch
- i watched the transition test and liked the transition and the design
- decided: gate says IIIT Bhopal, synthesised music, no voiceover for now, one continuous page, the agent places Hera. logged in [[Decisions]]
