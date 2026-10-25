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
