# Next Steps

in order. the first unchecked item is what to do next.

- [x] **move the two misplaced clones** out of the vault: `Prakhar\~\.claude\skills\onetake` → `C:\Users\prakh\.claude\skills\onetake`, and `Prakhar\~\ClaudeAnimationBase` → `C:\Users\prakh\ClaudeAnimationBase`, then delete the empty `Prakhar\~\` folder. done 2026-10-26
- [x] **design P23 / P26 properly.** a character sheet in the Urban Sketch figure rig: same face, glasses, black analog watch. P23 slightly longer messy hair, slouchier, new navy backpack. P26 shorter hair, upright, faded backpack. her: face never shown, mustard scarf. replace the placeholder trait list in [[01 Storyboard]] section A with my real look first
  - 2026-10-26: real look in section A, sheet at `C:\Users\prakh\still-compiling\chars\character-sheet-v1.jpg`. approved by me 2026-10-26, colours included. two small rig flaws left (P26 sleeve shoulder caps, side-view feet), fix when the figures go into scenes
- [x] **build the master timeline** (`timeline.js`): all 13 scene windows from [[01 Storyboard]], tempo and hit points, one source for picture, sound and checks. done 2026-10-26: `C:\Users\prakh\still-compiling\timeline.js`, check with `node check-timeline.mjs` (writes `timeline.json` for Python)
- [x] **title hold**: the storyboard gives the end title 1 s (97 to 98 s), the library asks for at least 4 s. my call: make the film about 101 s, or start the title earlier in scene 13. done 2026-10-26: film is 101 s, scene 13 runs to 101
- [x] **decide the page layout**: one giant continuous page (e.g. 5120×2880 or bigger) that the camera travels across, versus a few spreads joined by page turns. treatment says continuous page. decided 2026-10-26: one continuous page
- [ ] **build scenes in the order from storyboard section F**: gate pair 02 + 12 first, then 05 + 06, then 13, then 04 → 07, then 01 (reuse the opening test), 03, 08 → 10, 09, 11
- [ ] **Hyperframes layers** for the UI scenes (terminal, rejection windows, Slack, Pear dashboard, candlesticks, order book, agent graph). decide how they composite onto the page (render separately and overlay in ffmpeg, or draw inside the same page)
- [ ] **OneTake** for scenes 08 and 11
- [ ] **score**: extend the felt piano motif across the film. decide whether to download Salamander piano samples (218 MB) for a real piano
- [ ] **Hera easter eggs**: hide [[Hera/00 Index|Hera]] (my cat character) somewhere in the film. she must be redrawn in Urban Sketch (pen + wash, with the library's `figure`-style ink), not imported from `ClaudeAnimationBase/src/cat.js` (different engine and look). keep her colours: fur `#4B3F55`, amber eyes `#E9A93A`, pink ears `#E8948A`. small and easy to miss, never a plot point. candidate spots, i pick:
  - a Hera sticker on the laptop (fits the "stickers only increase" rule: appears at 04, still there at 07 and 13)
  - a doodle of her on the corkboard (03 and 13)
  - a sketched cat on the hostel windowsill at night (04 or 07)
  - asleep on the suitcase in the last room (13)
  - walking past the gate in the background (02 and/or 12)
  - a tiny cat doodle in the lecture notebook in the split screen (11)
- [x] **put `C:\Users\prakh\still-compiling\` under git** (a local repo is enough). done 2026-10-26, `git init` only, no commits yet
- [x] **transition test**: one join between tools before building scenes, e.g. 03 → 04 (Lemo push into the laptop screen handing over to the Hyperframes terminal), all at 24 fps, checked frame by frame across the cut. done 2026-10-26, `C:\Users\prakh\still-compiling\tests\t0304\` (`sh build.sh` rebuilds it). i watched it and liked it
- [ ] **lossless intermediates**: before the real film, render every tool's clip to lossless RGB (PNG frames or similar) and encode once at the end. the test still shows about 2 levels of blue shift at the cut on the near-black screen from two lossy encodes
- [ ] check Pear Protocol branding with Pear before posting anything that shows their real logo or UI
- [ ] full review loop per the library: contact sheets every 1 to 2 s, frame strips on key actions, loudness and black-frame checks, one full watch with sound

## Open questions
- ~~college name~~ IIIT Bhopal (decided 2026-10-26)
- ~~voiceover~~ none for now (decided 2026-10-26)
