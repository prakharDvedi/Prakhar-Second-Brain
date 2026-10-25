# Next Steps

in order. the first unchecked item is what to do next. none of these are urgent, she works as she is.

- [x] **decide about the code**: committed on a local branch `hera` in `ClaudeAnimationBase` (2026-10-26). to update the kit later: `git checkout main && git pull && git checkout hera && git rebase main`. never push, the remote is JohnHeibel's
- [x] **give her a name**: Hera (2026-10-26)
- [ ] **more poses** for stories: sit, lie down / loaf, jump, pounce, stretch, groom. Clawd only stands, so these are new work (the kit has `jump()` / `take()` helpers for the motion, the drawings are missing)
- [ ] **cat-only moves**: tail puff when scared, arched back when angry, slow blink for love, kneading when happy, a head tilt for confused
- [ ] **sound**: a synthesised meow / purr / hiss palette, so emotions can have a voice (the Lemo-Opuscar audio tools or OneTake's sfx could do it)
- [ ] **check the weak emotions**: proud and disgusted tint her brownish / greenish grey. maybe swap tint for a different cue on her
- [ ] **first real story** with her: write scenes in `src/scenes/`, follow `ANIMATION_GUIDE.md`

## Open questions
- does she ever talk (lip sync), or is she always silent / meows only?
- should Clawd stay available in the kit, or be removed entirely? right now he's still there, just not the default
