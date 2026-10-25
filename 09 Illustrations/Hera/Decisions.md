# Decisions

dated, newest at the bottom. "confirmed" means i saw it or approved it. "not checked" means an agent chose it.

## 2026-10-26

### A cat instead of Clawd
- i wanted my own character instead of the Clawd cartoon. first i sent a blocky orange voxel cat, then switched to a slim dark flat-vector cat before anything was built. the second one is her
- she's for storytelling videos in general, not for Still Compiling
- status: confirmed

### Same paint, same expression system as Clawd
- drawn with the kit's own watercolour + ink tools so she belongs with the rest of the kit (the kit's guide asks for that)
- she reuses Clawd's emotion system instead of a new one: same options, same 31 emotions, eyes, mouths, emotes. i asked for her eyes and mouth to be "expressive like current clawds"
- `clawd.js` left untouched. she lives in her own files
- status: confirmed

### Her own eyes
- round amber eyes with a slit pupil, like the reference, instead of Clawd's black slits. moods use eyelids (fur colour painted over the top of the eye) instead of Clawd's slanted shapes, because lids read as cat
- Clawd's line-drawn eyes (happy arcs, hearts, x, swirl...) are reused as they are
- status: confirmed (approved after the first sheet)

### Tail and ears act
- Clawd's arm angles drive her tail (lift and wag). ears fold back and tail droops from the mood
- why: a cat has no arms, and tail + ears are how cats show mood
- status: agent's call, not checked by me in detail

### Mood tints at 60%
- why: full strength turned her dark fur into another colour
- status: agent's call, not checked

### `catEmotions()` instead of `emotions()`
- why: Clawd's version fades from his orange. hers fades from her fur
- status: required fix, not a style choice

### Five drawn key views
- front, q (default), side, qback, back + flip, matching Clawd's views so `turn()`, `spinView()` and `move('spin')` work unchanged
- default view is q (her signature pose from the reference), not front
- status: confirmed (i asked for front, back and turn views)

### She's the kit's default character
- the empty-scene placeholder draws her, and `ANIMATION_GUIDE.md` has a note at the top telling the model to use `cat()` unless Clawd is asked for
- status: confirmed ("instead of that clawd cute cartoon id like it to be this")

### Not committed
- her files and the kit edits are local, not committed to the cloned repo
- status: open. decide whether to commit them on a branch in that clone or keep them as local changes
- **Update (2026-10-26):** decided, see "Committed on a branch" below

### Her name is Hera
- i named her Hera
- the code keeps `cat()` / `catEmotions()` / `CAT` as names, so nothing that already uses her breaks. only comments and docs say Hera
- status: confirmed

### Committed on a branch
- her files and the three kit edits are committed on a local branch `hera` in `ClaudeAnimationBase`, branched from `main`
- why: safe against `git pull` on main, and the history of her changes is kept. not pushed: the remote is JohnHeibel's repo
- status: confirmed (i said go ahead and commit)
