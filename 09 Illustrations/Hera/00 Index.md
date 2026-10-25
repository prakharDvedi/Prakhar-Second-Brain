# Hera

my main cartoon character, made to replace Clawd in Claude Animation Base. a slim dark purple-grey cat with round amber eyes, pink ears, whiskers and a tail that's usually up. painted in code (watercolour + ink), not a video model. meant to be reused across storytelling videos, not tied to one project.

her name is Hera (named 2026-10-26). the code still calls her `cat()`.

## Status (2026-10-26)

- design: done, approved ("purrr fect")
- expressions: all 31 of Clawd's emotions work on her, plus her own cat eyes with eyelids
- views: front, q (3/4, default), side, qback, back + flip. turns and spins work
- walk: works in every view
- showreel: done (35 s)
- not done: sitting / lying / jumping poses, head tilt, sound. see [[Next Steps]]
- code committed on a local branch `hera` in the kit's repo (not pushed, the remote is JohnHeibel's)

## Where things live

| What | Path |
|---|---|
| the kit (Claude Animation Base) | `C:\Users\prakh\ClaudeAnimationBase\` |
| her code | `src\cat.js` (drawing, views, eyes, `catEmotions()`) |
| her model sheets (loops) | `src\cat-sheets.js`: `cat`, `catViews`, `catEmotions`, `catActing`, `catReel` |
| renders | `out\cat_hero.jpg`, `out\cat_views.jpg`, `out\cat_emotions.jpg`, `out\cat_acting.mp4`, `out\cat_turn.mp4`, `out\cat_showreel.mp4` |
| kit files edited for her | `studio.html` (loads her scripts), `src\timeline.js` (placeholder draws her), `ANIMATION_GUIDE.md` (note at the top: she's the default) |
| live preview | open `studio.html?loop=catReel` (or `cat`, `catViews`, `catEmotions`, `catActing`) in Chrome |

## Notes in this folder

- [[Character Bible]] : how she looks, her colours, proportions, views, every option and how each emotion lands on her. read this before using her in a video
- [[Learnings]] : what broke while building her and why, kit nuances, things to remember
- [[Decisions]] : what was chosen and why
- [[Progress Log]] : what was done, step by step
- [[Next Steps]] : what's left, in order

## Related
[[Illustrations]] · [[Still Compiling/00 Index|Still Compiling]] (not used there, that film is Urban Sketch style)
