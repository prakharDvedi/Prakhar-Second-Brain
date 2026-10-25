# Tools and Setup

**the up to date, shared version of this note is the `/movie` skill** (`~/.claude/skills/movie/SKILL.md`). since 2026-10-26 OneTake has its own venv + launcher (`sh ~/.claude/skills/onetake/py`) and Claude Animation Base has its packages installed. this note is kept as the record of how it was first set up.

state of this machine as of 2026-10-26. Windows 11, Node 24.19, ffmpeg 9.0, Python 3.12 (in the library venv), uv available.

## Installed

| Tool | Where | How it got there |
|---|---|---|
| Hyperframes (HeyGen) | user plugin `hyperframes@hyperframes`, marketplace clone in `C:\Users\prakh\.claude\plugins\marketplaces\hyperframes` | `claude plugin marketplace add heygen-com/hyperframes` then `claude plugin install hyperframes@hyperframes` |
| Lemo-Opuscar plugin | user plugin `lemo-opuscar@lemolab` | `claude plugin marketplace add lemomo-ai/lemo-opuscar` then `claude plugin install lemo-opuscar@lemolab` |
| Lemo-Opuscar style library | `C:\Users\prakh\lemo-opuscar\` | cloned by the plugin's setup script, sparse checkout |
| OneTake | `C:\Users\prakh\.claude\skills\onetake` (user skill) | `git clone https://github.com/feitangyuan/onetake.git`. first landed in the vault because `~` didn't expand in PowerShell, moved here 2026-10-26 |
| Claude Animation Base | `C:\Users\prakh\ClaudeAnimationBase` (starter kit, open it in Claude Code) | `git clone https://github.com/JohnHeibel/ClaudeAnimationBase.git`. same story, moved here 2026-10-26 |
| Playwright headless Chrome | `C:\Users\prakh\AppData\Local\ms-playwright\chromium_headless_shell-1243` | library deps install |

Hyperframes note: 84 Git LFS files came down as pointers only. probably example media. if something is missing: `git -C ~/.claude/plugins/marketplaces/hyperframes lfs pull`. some Hyperframes features (TTS, media gen) need a HeyGen account.

## Commands (run from Git Bash, from the library root)

```bash
cd ~/lemo-opuscar
# review stills at given seconds
node core/render/still.mjs "C:/Users/prakh/still-compiling/clip" 0.6 2.3 4.3
# full video, no audio
node core/render/video.mjs "C:/Users/prakh/still-compiling/clip" --fps 24 --workers 3 --out "C:/Users/prakh/still-compiling/clip/out/video.mp4"
# export sound cues from window.EV
node core/render/events.mjs "C:/Users/prakh/still-compiling/clip"
# synthesise the sound
.venv/bin/python "C:/Users/prakh/still-compiling/clip/mix.py"
# mux + master to -14 LUFS, grain 1
sh core/render/mux.sh "C:/Users/prakh/still-compiling/clip/out/video.mp4" "C:/Users/prakh/still-compiling/clip/out/mix.wav" "C:/Users/prakh/still-compiling/clip/out/still-compiling-opening-test.mp4" 24 1
```

## Page contract (every scene page)

`window.DUR` (seconds), `window.render(t)` (deterministic, no `Math.random` or `Date.now`), `window.READY = true` when fonts are loaded, optional `window.EV` sound cues and `window.TEXTS(t)` for the reading-time check. full reference: `C:\Users\prakh\lemo-opuscar\core\README.md`, `TECHNIQUE.md`, `DIRECTOR.md`.

## Windows gotchas

- **don't re-run the library's `setup.sh`.** under Git Bash it mangles paths (`/styles/...` becomes `C:/Program Files/Git/styles/...`). to add another style's demo source, run by hand:
  `cd ~/lemo-opuscar && MSYS_NO_PATHCONV=1 git sparse-checkout add '/styles/<slug>/demo/' '/styles/<slug>/poster.jpg'`
- the library expects `.venv/bin/python`. on Windows it's `.venv\Scripts\python.exe`, so there is a shim at `.venv/bin/python`. if the venv gets recreated, recreate the shim:
  `printf '#!/bin/sh\nexec "$(dirname "$0")/../Scripts/python.exe" "$@"\n' > ~/lemo-opuscar/.venv/bin/python`
- only the core Python tier is installed (numpy, scipy, soundfile, soxr, pillow). voice (Kokoro TTS) and music (numba) tiers are not
- `sfx.compress` is mono, call it per channel
- in PowerShell `~` doesn't expand inside `git clone` targets. use `$HOME`

## Style engine (Urban Sketch)

`C:\Users\prakh\lemo-opuscar\styles\urban-sketch\demo\engine.js`, copied into each project folder. main calls: `penStroke` + `drawPen` (fineliner with draw-on), `wash`, `reserve`, `foliage`, `figure` + `drawPrims` (sketched people), `sketchShape`, `handwrite`, `rough`, `ellipse`. pose library in `poses.js` (`walk`, `run`, `sit`...). API table in that folder's `DEMO.md`.
