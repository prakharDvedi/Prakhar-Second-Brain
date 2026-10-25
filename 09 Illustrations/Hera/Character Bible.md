# Character Bible

Hera, the cat. everything needed to put her in a video without breaking her. in code she's `cat()`. source of truth is `C:\Users\prakh\ClaudeAnimationBase\src\cat.js`. if this note and the code disagree, the code wins, fix the note.

## Look

- slim, long legged, dark purple-grey fur, no outline-free vector look: she's watercolour wash + sepia-black ink like everything else in the kit
- back dips slightly behind the shoulders, then the neck rises into the head. rounded rump and chest
- round amber eyes with a slit pupil and a cream glint. pink inner ears. little 3-stroke tuft on top of the head. small dark nose, "ω" mouth at rest, two whiskers per side
- tail up with a curl at the tip by default
- reference: a stock flat-vector illustration of a dark cat i picked. she's an original drawing in code inspired by it, not a copy. don't put the reference image itself in anything published

### Colours (`CAT` in cat.js)

| Part | Hex |
|---|---|
| fur | `#4B3F55` |
| fur dark (belly, far legs) | `#2C2433` |
| fur light (back highlight) | `#6E5F7C` |
| inner ear / nose tint | `#E8948A` |
| iris | `#E9A93A` (shading `#B9741C`) |
| furious iris | `#FF3B4E` (shading `#A8162B`) + red glow |
| ink | the kit's `PAL.ink` `#2B2233` |

### Proportions

- unit `u` = body unit. in profile she's about 12u nose to rump (body x from -6.7u to 7.3u), about 13u tall at the ears, about 19u with the tail up
- (x, y) passed to `cat()` = the ground point between her feet. negative y is up
- good sizes: u 10 to 13 for grids and small figures, 17 to 26 for sheets, 30 to 40 for a hero shot at 1080p
- facing: every view faces screen-right. `flip: true` mirrors

## Calling her

```js
cat(x, y, u, feel('happy', t))                                   // one mood, alive at time t
cat(x, y, u, { ...catEmotions(t, keys), view: 'front' })          // acted mood changes
cat(x, y, u, { ...feel('neutral', t), ...turn(t, 2, 2.4, 0, .25) }) // drawn turn from front to facing right
cat(x, y, u, { ...feel('happy', t), view: 'side', walk: t * 1.7 })  // walking
cat(x, y, u, { ...feel('excited', t), ...move('spin', t) })       // beat-locked dance
```

keys for `catEmotions` are Clawd's format: `[[0, 'neutral'], [2.2, 'surprised'], [3.3, 'happy', { emote: 'music' }]]`.

**always `catEmotions()`, never Clawd's `emotions()`**. Clawd's cross-fades the fur from his orange, so she flashes orange on every change.

## Options

all of clawd()'s options work. the ones that matter for her:

| Option | What it does on her |
|---|---|
| `view` | `front`, `q` (default), `side`, `qback`, `back` |
| `flip` | mirror (face left) |
| `eyes`, `mouth` | any of Clawd's names (lists below). `eyes` can be a pair `['narrow', 'wide']` |
| `lookX`, `lookY` | -1..1, moves the pupils |
| `squint` | 0..1, squashes the eyes (above .8 they become a line) |
| `blush` | 0..1, rose patches under the eyes |
| `gloom` | 0..1, dark wash over the forehead |
| `tint`, `tintK` | mood colour. on her it's applied at 60% strength automatically |
| `emote`, `emoteK`, `emoteAge` | Clawd's emotes, placed over or beside her head per view |
| `dx`, `dy`, `sq`, `rot`, `sx`, `sy` | position offset (in u), squash (negative stretches), rotation at the feet, scale |
| `walk` | phase. profile views swing the paws, front / back views lift them alternately |
| `aL`, `aR` | Clawd's arm angles. on her: their mean lifts the tail, their difference wags it |
| `tail` | extra wag angle in radians |
| `tailDown` | 0..1 droop. default comes from the mood (sad 0.75, scared 0.3) |
| `earBack` | 0..1 folds the ears out and down. default from the mood (scared 0.9, cry 0.8, sad 0.7, red 0.7, angry 0.55, x 0.6, or 0.5 with gloom) |
| `whiskers: false` | hide whiskers |
| `seed` | blink timing, so two cats don't blink together |
| `boilKey` | stable id for the 12 fps line boil if characters come and go mid shot |
| `draw(u, sw)` | hook to draw a prop in her local space |

## Views

| View | What you see | Face |
|---|---|---|
| `front` | facing camera. chest, two front legs, hind legs just peeking behind, tail rising behind the right shoulder | both eyes, symmetric |
| `q` | her signature pose. body in profile facing right, head turned to camera | both eyes, 3/4 |
| `side` | pure profile. muzzle to the right, far ear tucked behind | one eye, narrowed to 70%. mouth squashed to 65% |
| `qback` | profile body, head turned away | none (back of head) |
| `back` | rump to camera, ear backs in fur colour (no pink), tail up and off to one side | none |

turns step through drawn views, one per eighth of a turn, like a cartoon model sheet: front, q, side, qback, back, then the same mirrored. `spinView(a)` gives the view for a heading (0 front, .25 facing right, .5 back, .75 facing left).

## Expressions

### Her own round eyes (amber iris + pupil)

| Eye | Look |
|---|---|
| normal, look | slit pupil, glint, blinks every ~3.3 s (offset by `seed`) |
| wide | bigger iris, round pupil, blinks |
| scared | bigger iris, tiny trembling pupil |
| shine | big pupil, two glints |
| dot | small round pupil |
| blank | cream iris, no pupil |
| red | red iris, red glow, angry lid (furious) |
| angry | lid slants down toward the middle, strongly |
| determined | same, softer |
| sad, teary | lid slants up toward the middle (worried). teary adds a tear pool |
| narrow | flat upper lid + lower lid |
| sleepy | lid almost closed |
| shades | dark lens (cool) |

eyelids = the part of the iris above the lid line painted in fur colour, then an ink lid line. that's what makes them read as cat eyelids and not Clawd's slanted black slits.

### Borrowed from Clawd (line drawn)

`happy` (arcs), `closed`, `squeeze` (> <), `cry` (streams), `heart`, `x`, `swirl`, `spark`. `wink` = happy + normal.

### Mouths

all 16 of Clawd's at 32% scale under her nose: `o`, `O`, `smile`, `grin`, `flat`, `wobble`, `cat`, `frown`, `smirk`, `laugh`, `open`, `wail`, `teeth`, `tongue`, `pout`, `yawn`. no mouth given = `cat` (ω). pass `mouth: null` for none.

### The 31 emotions on her

neutral, happy, excited, laugh, love, shy, proud, smug, relieved, sad, cry, angry, furious, scared, surprised, confused, thinking, idea, determined, sleepy, bored, nervous, suspicious, disgusted, dizzy, cool, starstruck, ko, playful, mischief, hopeful. each is a face + colour + a way of moving to the beat (`PROJECT.bpm` in `src/config.js`). sheet: `out\cat_emotions.jpg`.

what reads best on her: love (heart eyes + pink), sad (worried lids, drooped tail, flat ears, rain cloud), angry / furious (lids + red), scared (wide, trembling pupil, ears back), surprised (wide + "!"), sleepy (heavy lids + zzz). weakest: proud and disgusted, where the tint turns her brownish / greenish grey.

## Rendering her

from `C:\Users\prakh\ClaudeAnimationBase` in Git Bash:

```bash
node render.mjs --loop=catReel --sheet=1,5,12 --cols=3 --w=640 --out=out/check.jpg     # quick look at chosen times
node render.mjs --loop=catEmotions --sheet=1 --cols=1 --w=1920 --out=out/cat_emotions.jpg
node render.mjs --loop=catReel --clip --range=0:35.25 --out=out/cat_showreel.mp4      # a loop to MP4
node render.mjs --clip --out=out/video.mp4                                            # the actual video (src/scenes/)
```

for a real story video: write scenes in `src/scenes/`, add each as a `<script>` in `studio.html` after `cat-sheets.js`, register them with `shots([[t0, fn], ...])`, set `PROJECT.duration` and `bpm` in `src/config.js`. the kit's rules are in `ANIMATION_GUIDE.md` (no on-screen text in videos, transitions always, something happens in every shot).
