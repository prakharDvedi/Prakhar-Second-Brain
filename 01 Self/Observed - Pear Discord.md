# Observed - Pear Discord

> **Scope:** this establishes my Pear/work-Discord register only — a ~2-week window (roughly 8/17–8/29 2026), one channel, 164 messages after cleaning. It is not a claim about my universal writing style. [[My Writing Style]] is the source of truth for that; this note is narrower and more mechanical, useful for another AI writing *in this specific context*.

Confidence key: **High-confidence stable behavior** / **Context-specific behavior** / **Insufficient evidence**.

---

## Global Writing Patterns

### Capitalization
- First-person "i" is lowercase in the large majority of instances. **High-confidence.**
- Sentence-start capitalization is genuinely mixed (~90% lowercase, ~10% capitalized), clustering on short greeting/ack words ("Thanks ser", "Gm ser.", "Got it. Will do") — reads like mobile auto-capitalization, not a deliberate choice. **Context-specific.**
- Formality (full words, correct capitalization, structure) rises for broadcast-style updates aimed at multiple people or less-familiar contacts. **High-confidence — clearly evidenced counterexample exists.**
- **Two distinct drivers of this rise, both evidenced separately:** (1) *many recipients* — the "Quick Lighter update" broadcast to the team; (2) *an unfamiliar/external contact* — "Ah its the same as Hl, first navigate to pro.pear.garden. There in the accounts menu, click add trade account and select venue as lighter. Once that is set up, please resync and it should be up and running." (addressed to `@Caroulemapoule`, a less-familiar contact) is the single most grammatically correct message in the whole corpus — full capitals, proper commas, complete sentences, no dropped apostrophes. Audience unfamiliarity alone raises formality, independent of recipient count. **Context-specific, n=1 for this specific driver, but clean and unambiguous.**
- Technical acronyms are inconsistently capitalized ("HL" vs "Hl" vs lowercase "hyperliquid"); names are only correctly cased when Discord autocompletes an `@mention`. **Context-specific.**

### Punctuation
- Zero em dashes found across the entire sample. **High-confidence.**
- Zero semicolons found, even in long run-on technical messages. **High-confidence.**
- A two-dot `..` used as an informal trailing-off pause, not a proper three-dot ellipsis. **High-confidence (6+ instances, no contradicting "..." use).**
- Exclamation marks are rare (1–2 across 164 messages). **High-confidence.**
- Commas used, but longer messages read as comma/"and"-joined run-ons rather than short clean sentences. **Context-specific.**

### Grammar
- "I'll" → "ill" and "I'm" → "im" drop the apostrophe consistently (**12 / 5 exact instances** in the corpus, effectively no counterexamples in casual messages). **High-confidence.**
- "didn't" → "didnt" (**3 instances**), "aren't" → "arent" also drop it. **Context-specific (fewer instances).**
- Other contractions ("it's", "don't", "we'll", "isn't", "hasn't", "haven't") usually *keep* the apostrophe, especially in slower/technical writing. **High-confidence — this is specific to the first-person modals above, not a blanket rule.**
- One genuine misspelling ("doesent" for "doesn't") — **Insufficient evidence**, single instance, not a pattern.
- Sentence fragments and run-ons are normal, not accidental. **High-confidence.**

### Vocabulary
- Confirmed present: `hmm`, `ahh`/`ah`, `makes sense`, `checked this`, `probably`, `fs`, `atp`, `smth`, `tho`/`though` (both forms), `prblm`, `wip`, `obv`, `kinda`, `def`, `cc @name`, `bullish` (used as general enthusiasm, not just trading). **High-confidence for all listed.**
- Also confirmed: `alr` / `alrdy` (already — 3 instances combined), `outta` (out of — "lighter came outta no where"), `nw` (now — "seeing nw"), `les go` (let's go — "Yes les go"), `pheww` and `holy` as standalone interjections of relief/surprise. **Context-specific — fewer instances each, but real.**
- **Letter-elongation as an emphasis/casualness device**: doubling a letter beyond normal spelling — "annd" (and), "yeahh" (yeah). Two independent instances of the same device, not just a one-off typo. **Context-specific.**
- Not found in this sample: `ig`, `imo`, `rn`, `btw`, `yea` (as opposed to `yeah`, which did appear). **Insufficient evidence** — absence here doesn't mean these are wrong elsewhere, just unconfirmed by this corpus.
- Address terms: "ser" (**13 instances**) and "boss" (often "gm ser ser"). Greeting: "gm" (**11 instances**, near-default opener; "gn" not observed). **High-confidence.**
- Signature ack emoji `:🫡:` — **14 exact instances**, functions like a nod/"got it". Secondary emoji `:pepenoted:` (custom Discord emoji) appears **2 times**, both specifically paired with "checking" — a narrower, task-specific ack emoji distinct from the general-purpose `:🫡:`. **High-confidence for both counts.**
- The phrase "root cause" never appears in Discord despite constant use in written PR descriptions — a real register split between chat voice and written technical-doc voice. **High-confidence.**

### Message length
- Short reply: 1–6 words. Normal coordination message: ~10–25 words. Technical/status message: 40–200+ words as one paragraph. **High-confidence.**

### Message splitting
- In live back-and-forth, a single train of thought gets fired as several short separate messages rather than composed into one (e.g. 5 separate one-liners answering different things within the same minute; a pricing discussion spread across ~8 tiny consecutive messages). **High-confidence.**
- For async status/technical reports, the opposite holds: one long, dense, unbroken paragraph. **High-confidence — this is a real mode split, not noise.**

---

## Conversation Modes

### Casual
**Context-specific** — linguistically indistinguishable from Work Discord in this sample; differs by topic (social/personal), not register. No purely non-work chat was sampled, so this can't be separated from Work Discord with confidence.
Tone: reactive, low-effort, quick. Structure: reaction, optionally + one clause. Vocabulary: "lol", "ahh", boss/ser, emoji shortcodes.
Counterexample: "Thanks ser" capitalized despite being casual — capitalization isn't reliably tied to mode.

### Work Discord
**High-confidence.**
Tone: direct, unceremonious, no pleasantries. Structure: answer → optional one-clause reason. Short, 1–20 words typically.
Counterexample: "Got it. Will do" — capitalized start, same content family as lowercase "got it."

### Technical Explanation
**High-confidence.** Deeper architecture/reasoning narratives, the "why" not just the "what."
Tone: methodical, hedged, willing to call something genuinely hard rather than oversimplify. Structure: what I checked → current state → feasible now vs. later → the actual constraint → conclusion. One dense paragraph, 40–200+ words.
Counterexample (formality ceiling): the single longest message in the sample used a bolded header, full capitalization, listed design decisions and issue numbers — reserved for genuine multi-recipient project updates.

### Technical Status Update
**High-confidence.** Shorter, more routine than Technical Explanation.
Tone: brisk, honest about uncertainty and misses ("my bad", "haven't been able to trigger it myself yet"). Structure: greeting/address → what I checked → finding → next action. 15–80 words.
Counterexample: a numbered multi-point reply ("1 - ... 2 - ... 3 - ...") used specifically when answering several distinct questions at once — more structured than the norm for this mode.

### Disagreement / Pushback
**Context-specific** — fewer examples than other modes, but the pattern held consistently across all of them.
Tone: never flatly declarative; works through the tension out loud, then hands the actual call back to the other person. Structure: state the tension → list concerns → counter own concerns → conditional recommendation → ask. 40–120 words.
No strong counterexample found — even the most pushback-heavy message ends with "what do you say?" rather than a flat verdict.

---

## Things That Sound Unlike Me (confirmed by absence)
- Em dashes — 0 instances. **High-confidence.**
- Semicolons — 0 instances, even where conventional. **High-confidence.**
- Corporate/formal vocabulary ("please find attached", "kindly", "as per") — none found. **High-confidence.**
- Excessive headings — only one bolded label in the whole sample, in the one broadcast-style update. **High-confidence.**
- Exclamation-mark enthusiasm — 1–2 instances total. **High-confidence.**
- Long formal introductions, formal closings ("Best,", "Regards") — none found. **High-confidence.**
- "Perfect grammar" in casual messages — directly contradicted by "ill"/"im"/"didnt" and run-on structure. **High-confidence.**

---

Full representative quotes live in [[Examples - Pear Discord]]. Practical writing instructions derived from this note live in [[Desired AI Voice]].
