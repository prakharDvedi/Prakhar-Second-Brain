# Progress Log

dated entries, newest at the bottom. don't rewrite old entries, add a correction below instead.

## 2026-10-26

### 1. Getting the kit running
- Claude Animation Base cloned to `C:\Users\prakh\ClaudeAnimationBase` (first landed inside the vault at `Prakhar\~\` because PowerShell didn't expand `~`, moved out). packages via `npm ci --ignore-scripts`. demo test render worked
- explained what the kit is and how replacing Clawd works

### 2. Studying Clawd
- read `src/clawd.js` (views, eyes, mouths, hats, emotes, `EMO` emotions, `feel` / `emotions` / `move`), `src/core.js` (paint, ribbon, boil seeds, camera), `src/timeline.js` (shots, loops), `src/sheets.js`, `studio.html`, `render.mjs` flags

### 3. First cat
- i sent a blocky orange voxel cat, then interrupted and sent the slim dark cat instead
- wrote `src/cat.js`: legs as ribbons, tail, profile body, head turned to camera, ears, tuft, round eyes with lids, nose, mouth via Clawd's `mouth()`, whiskers, emotes. `src/cat-sheets.js` with `cat` and `catEmotions` loops. added both to `studio.html`
- first render: boxy body, mouth too big, dark dot on tail tip, tints too strong. fixed over two passes (see [[Learnings]])
- emotion grid: wink and furious fell back to square slits, fixed

### 4. Made her the default
- `src/timeline.js` placeholder draws her, note at the top of `ANIMATION_GUIDE.md`

### 5. Acting test
- `catActing` loop, 9 s: walks in, then surprised → happy → love → sad → angry → relieved
- bug: orange flash on every change (Clawd's `emotions()` fades from his colour). added `catEmotions()`. re-rendered: `out\cat_acting.mp4`
- i approved her ("purrr fect")

### 6. Turn views
- refactored `cat()` into views: `CAT_VIEWS` (front, q, side, qback, back), `catLeg`, `catTail`, `sideBody`, `CAT_HEADS` (face3q, front, profile, back), `catHead`
- `catViews` loop: the five views, a drawn turn-around, the spin dance
- fixed front-view hind legs and the back-view tail. turn strip checked frame by frame: all eight steps in order
- renders: `out\cat_views.jpg`, `out\cat_turn.mp4`
- updated `ANIMATION_GUIDE.md`, the `/movie` skill (new "My characters" section) and [[Illustrations]]

### 7. Showreel
- `catReel` loop, 35.25 s: 20 emotions facing camera with labels, a slow turn through every view, a walk across in profile, a spin
- checked 10 key frames, made her bigger (u 40) for the emotion part
- render: `out\cat_showreel.mp4`, 1080p 24 fps, no sound. only spot-checked, not watched start to end by the agent

### 8. This vault folder
- made `09 Illustrations/The Cat/` with index, character bible, learnings, decisions, this log, next steps

### 9. Named Hera, committed
- i named her Hera. vault folder renamed `The Cat` → `Hera` (so the folder above is now `09 Illustrations/Hera/`), links in [[Illustrations]] and the `/movie` skill updated. code names stay `cat()` etc, comments now say Hera
- kit: committed on a new local branch `hera` in `ClaudeAnimationBase` (not pushed)
- vault: these notes committed too
