# Learnings

what i learned building her, so the next session doesn't relearn it. dated, newest at the bottom.

## 2026-10-26

### How the kit is built (what made reuse easy)

- Clawd is not baked into the engine. he's one drawing function `clawd(x, y, u, o)` in `src/clawd.js`. everything expressive lives in plain functions next to it: `eye()` (24 eye kinds), `mouth()` (16), `emote()`, `tintCols()`, and the emotion table `EMO` with `feel()` / `emotions()` / `move()` / `turn()` / `spinView()`
- those functions only care about the option object, not about Clawd's shape. so a new character that accepts the same options gets the whole emotion system for free. that's why she got 31 emotions on day one
- `eye()` and `mouth()` draw around a local centre at a size set by their `u` argument. to reuse them on her face: `translate` to her eye / mouth point and pass a smaller `u` (her eye uses 0.78u, the mouth 0.32u). Clawd's mouth expects to sit at y = -4.3u, so translate by `+4.3 * uM` to cancel that
- drawing primitives are in `src/core.js`: `paint(points, { wash, fill, ink, curv, ... })` is one painted shape, `ribbon(path, w0, w1)` is a tapered tube (legs, tail), `through()` smooths points, `ellPts` / `rectPts` / `rrPts` make shapes. `curv: .5` on `paint` makes a smooth closed spline through the points
- p5.brush loses strokes drawn far from the origin under zoom, so `paint` / `inkLine` draw every shape around its own centre. don't bypass them with raw `brush.*` calls
- line boil (12 fps wobble) comes from `boilSeed(key)` before each part. give each part its own key (`rs('leg' + i)`) so moving one part doesn't re-boil the others
- `studio.html` is the page. the render tool loads it, so a new file only exists for renders once it's a `<script>` there, in order (after `clawd.js`, which defines `eye`, `mouth`, `EMO`...)
- model sheets are `LOOPS.<name> = t => {...}` with `.len`. `--loop=<name>` renders them. "no loop named X" = the script isn't in `studio.html`

### What broke and why

- **orange flash on every mood change.** `emotions()` cross-fades body colour with `tintCols(prev)` / `tintCols(cur)`, which default to Clawd's clay orange when no `col` is passed. fix: `catEmotions()` passes her colours into every key's override, so the fade starts from her fur. any future character with its own colours needs the same wrapper
- **mood tints swamped her.** Clawd's tints (flush, blue, gold, pale...) are tuned for light orange. on dark fur at full strength she became a different-coloured cat. fix: 60% strength. careful not to apply it twice: `catEmotions()` pre-scales and sets `catTint: 1` so `cat()` skips its own scaling
- **wink showed square blocks.** Clawd turns `'wink'` into a pair inside his own `eyes()`, which she doesn't call. her `eye()` call got the raw `'wink'` and fell back to Clawd's default slit (a black rectangle). fix: expand `wink` to `['happy', 'normal']` in `cat()`
- **furious had square red eyes.** Clawd's `red` eye is a big red rectangle. fix: made `red` one of her round kinds (red iris, glow, angry lid)
- **boxy body.** a closed spline through a few points still looks like a box if the points sit on straight lines. needed 17+ points that actually curve (rump and chest), and a neck that rises into the head
- **mouth too big.** at 50% Clawd's ω read as teeth / a beard. 32% reads as a cat mouth
- **dark dot on the tail tip.** an extra round cap painted over the ribbon end doubled the wash. the ribbon end alone is enough
- **front view: hind legs looked like a second pair of front legs.** their tops started below the rump, so flat ends showed. fix: start them up inside the rump shape (drawn after them) so only the lower part peeks out
- **back view: tail like a stick through the head.** drawn over the body (correct from behind), but straight up it covered the head. fix: send it up and off to one side

### Nuances worth remembering

- turns are drawn **key views**, never a rotated 3D shape. the kit's guide insists on it and it looks better
- default view is `q`, not `front` (Clawd's default is front). `turn()` / `spinView()` always pass a view explicitly, so they're unaffected
- in profile only one eye shows, and it uses the **right** eye's kind (matters for mismatched pairs like confused)
- ears and tail act on their own from the eye kind, so a sad face automatically gets a drooped tail and flat ears. override with `earBack` / `tailDown` when a shot needs something else
- Clawd's arm angles drive her tail. dances that wave arms (`wave`, `roof`, `shimmy`) become tail wags and lifts, which happens to look right for a cat
- eyelids work by painting fur colour over the top of the eye, so the lid colour must match the current fur, including tints. that's why `catEyes` gets `col` passed in
- the emotion grid at u = 10 still reads, the face is the strongest part of her

### Kit / machine gotchas hit along the way

- ffmpeg crop is `crop=W:H:X:Y`, not `WxH`
- long renders (a 35 s clip at 1080p) go in the background. emotion grids take ~5 s per frame, a hero sheet ~0.6 s
- `render.mjs --crop=x,y,w,h` crops in screen pixels, handy for checking one view up close
- the kit's demo starts dark, frame 0 is just the night background, not a broken render
- `npm ci --ignore-scripts` was enough to install the kit (p5, p5.brush, puppeteer-core). Chrome is the system one at `C:\Program Files\Google\Chrome`
- these are local edits to a cloned repo. a `git pull` in `ClaudeAnimationBase` could conflict with the edited `studio.html`, `src/timeline.js`, `ANIMATION_GUIDE.md`. commit or stash first
