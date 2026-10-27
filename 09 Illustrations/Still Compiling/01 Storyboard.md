# Storyboard

written 2026-10-26. 13 scenes, 98 s (101 s since 2026-10-26, scene 13 runs 3 s longer so the end title holds 4 s). P23 = 2023 me, P26 = present me.

the storyboard was written before the style was locked, so the "brief" lines were first meant as image-model prompts. they now work as scene briefs for code. where the storyboard says "cinematic 3D" or similar, the locked style wins: Urban Sketch · Pen & Wash (see [[Decisions]]). skill fields below use the corrected tool roles.

## Scenes

### 01 · Boot · 0:00 to 0:05
- **purpose:** curiosity, something is about to begin
- **opening frame:** pure black, `> start` with a warm white block cursor blinking every 530 ms
- **action:** cursor blinks, then stretches vertically into a tall pillar that gains stone texture and becomes the gate's left pillar. twin pillar appears
- **camera:** locked, then a slow push in during the stretch
- **light:** black to cold blue pre-dawn
- **text:** `> start`
- **sound:** room tone, one soft tick per blink (the film's sonic motif), low felt piano note on the stretch
- **transition:** the pillar is the cursor, it resolves into scene 02's gate
- **tools:** Hyperframes (type + cursor), Lemo-Opuscar (morph)
- **built:** yes, in the opening test (`clip/opening.js`), as a black screen opening into the page instead of pre-dawn blue

### 02 · Arrival · 0:05 to 0:12
- **purpose:** freedom and being a little lost
- **opening frame:** wide shot of the gate at sunrise, P23 small in centre with a new backpack, adjusting a strap
- **action:** P23 hesitates, looks up at the name arch, walks in away from camera
- **environment:** gate arch, guard booth, freshers' week banners, scooters, blurred students
- **camera:** one continuous follow through the gate, rising into a crane over campus
- **motion:** leaves and banners stir, a sketch line overlay traces the buildings briefly
- **light:** low gold sunrise, long shadows. amber = human palette
- **text:** small corner note `2023`
- **sound:** birds, distant chatter, piano theme begins (4 notes)
- **transition:** crane tops out on the football ground, a ball is kicked toward the lens
- **tools:** Lemo-Opuscar, OneTake
- **built:** partly. gate, sunrise colour arrival, P23 walking, corner note are in the opening test. crane to football ground not built

### 03 · Everything at once · 0:12 to 0:21
- **purpose:** freedom, friendship, wasted time that wasn't wasted
- **opening frame:** football hits the lens, freeze, it becomes a polaroid pinned to a corkboard
- **action:** a burst of memory cards: football with friends, fest stage, canteen chai, midnight walk to the gate, group photo with eyes closed, LAN night, P23 laughing
- **environment:** scrapbook of tape, ticket stubs, fest wristbands, doodles. each card plays as a 1 to 2 s live clip
- **camera:** slow drift across the board, cards animate as the camera passes
- **motion:** cards flip, slide, get taped down, hand drawn arrows animate on. last card: a dark hostel window with one lit laptop screen
- **light:** mixed warm: floodlights, fest LEDs, sodium streetlights
- **text:** handwritten scribbles ("sem 1", "why are we awake")
- **VO (optional):** "For a while, I tried everything."
- **sound:** ball thud, crowd, fest bass bleed, laughter, piano gets a light beat
- **transition:** push into the lit laptop screen on the last card
- **tools:** Hyperframes (corkboard and cards), Lemo-Opuscar (the clips inside the polaroids)

### 04 · Hello, World · 0:21 to 0:28
- **purpose:** curiosity becomes a hook
- **opening frame:** full screen terminal, `> ` and the same cursor typing `index.html`
- **action:** P23 cross-legged on a hostel bed builds an ugly first webpage, clicks it, it works, grins. then a JS alert, a GitHub first commit, a weather API returning JSON
- **environment:** messy hostel room, roommate asleep, fan, chai cup. the worn football rolls in and stops under the desk
- **camera:** starts inside the screen, pulls back through the bezel, slow orbit to 3/4
- **motion:** browser tabs spill out of the screen as floating panes and fill the room
- **light:** cool screen light on the face vs warm desk lamp. first time cyan meets amber
- **text:** `<h1>hello world</h1>`, `git commit -m "first"`
- **sound:** keys, fan, a pop per new tab, cursor tick returns
- **transition:** floating tabs drift to the window, through the glass the library steps at golden hour
- **tools:** Hyperframes (UI), Lemo-Opuscar (character), OneTake (screen to room pull)

### 05 · Two Cups · 0:28 to 0:36
- **purpose:** falling for someone. warm, human, specific
- **opening frame:** library steps at golden hour, two chai cups on the step between two people
- **action:** small moments, never a big kiss: one shared earphone, her laughing at his terrible webpage on his phone, walking the corridor in step, her mustard scarf in the wind. her face always soft focus or turned away
- **environment:** library steps, corridor, rooftop at dusk
- **camera:** long lens feel, shallow depth, gentle handheld sway, lateral track along the corridor
- **motion:** pencil sketch overlays bloom at the frame edges, like a memory being drawn. the two cups anchor the scene
- **light:** warmest of the film. backlit hair, dust in the air
- **sound:** fingerpicked guitar joins the piano, muffled laughter, wind
- **transition:** hold on the two cups, hard cut to the exact same composition (scene 06)
- **tools:** Lemo-Opuscar (characters), Hyperframes (sketch overlay)

### 06 · One Cup · 0:36 to 0:42
- **purpose:** disappointment and emptiness, no melodrama
- **opening frame:** same locked composition as the end of 05, now overcast. one cup, empty space on the step
- **action:** P23 alone, one earphone in, the other hanging loose. phone shows a chat with the cursor blinking in an empty input, nothing typed. locks the phone
- **camera:** locked, then a very slow pull back leaving P23 small
- **motion:** the sketch overlays from 05 erase themselves line by line. in this style: colour drains back to ink
- **light:** flat grey, all amber drained
- **sound:** music drops out completely. rain, a far bike, the cursor tick alone, slower
- **transition:** phone locks to black, its reflection becomes the 3 AM terminal
- **tools:** Lemo-Opuscar, Hyperframes (chat UI)

### 07 · 3:17 AM · 0:42 to 0:51
- **purpose:** inward focus, failures, rejections. building as a place to put everything
- **opening frame:** black reflection, then red `Build failed. 3:17 AM`
- **action:** rewrite, rerun. time lapse over many nights: hair messier, cups stacking. rejection emails pile up in space like glass sheets. deploys red, green, red. rubs eyes, keeps typing
- **environment:** same hostel room, crowded with floating windows (stack traces, Stack Overflow, Docker errors). the football still under the desk, dusty
- **camera:** slow 360 orbit around the desk, day/night flickering through the window
- **text:** `Build failed`, `403`, `Application status: Not selected` ×4, `ECONNREFUSED`, `deploy: success`
- **VO (optional):** "So I built things. Most of them broke."
- **sound:** synth arpeggio speeding up, error beeps tuned to the key, steady cursor tick
- **transition:** orbit ends behind P23's head, push into the terminal, green `deploy: success` fills frame
- **tools:** Hyperframes (all UI), Lemo-Opuscar, OneTake (orbit)

### 08 · Toys become systems · 0:51 to 0:59
- **purpose:** the work matures into real engineering
- **opening frame:** inside the terminal, `deploy: success`, camera flies through the text
- **action:** no character (only a reflection on a server rack). terminal windows rise into buildings, GitHub contribution squares light up as their windows, a night city. the college corridor turns panel by panel into a server aisle. API diagrams → database cylinders → log streams → an LLM node branching into an agent graph
- **camera:** long gliding forward track at constant speed
- **motion:** architecture builds itself, code streams like water through wall cracks
- **light:** amber tube lights hand off to cyan rack LEDs
- **text:** `GET /api/v1`, `postgres`, `redis`, `agent.plan()`, `p99: 41ms`
- **sound:** arpeggio blooms into a wider pulse, server fans, whoosh as the corridor flips
- **transition:** the aisle ends at a single dark laptop on a hostel desk, it lights up with a notification
- **tools:** OneTake, Hyperframes (labels)

### 09 · A notification · 0:59 to 1:04
- **purpose:** the climax, played quiet
- **opening frame:** close up on P23's hands frozen over the keyboard, a toast slides in top right
- **action:** "Pear Protocol · production access granted". no movement for 2 s, a small exhale, no celebration. hands go back to the keys
- **camera:** macro on hands, rack focus to the notification, then the cursor under it
- **light:** neutral desk lamp, first balanced amber + cyan
- **sound:** total silence except one soft chime, then the cursor tick
- **transition:** Enter is pressed, a ripple of candlestick data becomes scene 10
- **tools:** Hyperframes (notification), Lemo-Opuscar (hands)

### 10 · The market breathes · 1:04 to 1:13
- **purpose:** what the work actually is
- **opening frame:** BTC/ETH candlesticks fill the frame, ticking live
- **action:** P26 (tidier hair, calmer, same face) walks through a spatial data world. order book depth bars, data streams, an agent graph (planner, signal, risk, execution) lighting node by node. P26 pulls a production log panel closer. a PR card: `merged ✓`
- **environment:** abstract trading space, floor grid, candlesticks rise into the scene 08 skyline
- **camera:** smooth lateral dolly alongside P26, then a slow rise
- **light:** deep navy, pear green accents, cyan data. restrained
- **text:** `BTC/ETH`, `bid / ask`, `agent: risk → execution`, `PR merged`, `INFO fill confirmed`
- **VO (optional):** "Now the things I build move real things."
- **sound:** full theme, piano + pulse, data hum, order flow ticks
- **transition:** P26 turns, a projector beam cuts across and splits the screen
- **tools:** Hyperframes (charts, order book, logs, PR), Lemo-Opuscar (P26)

### 11 · Two tabs open · 1:13 to 1:21
- **purpose:** the strange overlap of student and engineer
- **opening frame:** vertical split. left: lecture hall, professor at the whiteboard, P26 in the back row with a notebook. right: same moment, Slack pings, GitHub, trading dashboard, agent logs
- **action:** mirrored movement. notes on the left = code on the right. attendance called on the left, Slack ping on the right. then the divider dissolves: notes become code comments, the whiteboard formula becomes a log line, notebook and laptop on one desk
- **camera:** locked symmetric, slow push as the divider melts
- **motion:** the divider line is the cursor stretched tall, it blinks out to merge the halves
- **light:** warm fluorescent left, cyan right, blending
- **text:** left `Attendance: present`, right `@prakhar can you look at prod?`
- **sound:** muffled professor in the left channel, Slack knocks in the right, merging to centre
- **transition:** the merged desk is by a window, through it the gate at dusk
- **tools:** Hyperframes (split + UI), OneTake (merge), Lemo-Opuscar

### 12 · Passing · 1:21 to 1:28
- **purpose:** realising life has changed
- **opening frame:** exact composition of scene 02, at dusk
- **action:** P23 (new backpack, unsure) walks in, away from camera. P26 (worn backpack, calm) walks out, toward camera. they pass shoulder to shoulder in the centre of the gate. P26 glances back, P23 doesn't notice. P26 smiles a little, keeps walking
- **environment:** same gate, banners now for a different year's fest
- **camera:** locked wide mirroring 02, then a slow push on P26
- **motion:** ~70% speed at the pass. faint ghost trail on P23, like a memory
- **light:** dusk amber, the mirror of 02's sunrise
- **sound:** the opening 4 piano notes in reverse, birds settling
- **transition:** P26 walks out of frame, cut to the room where the same backpack is set down
- **tools:** Lemo-Opuscar, OneTake

### 13 · `> next` · 1:28 to 1:41
- **purpose:** this is only the beginning
- **opening frame:** still life of the hostel room, emptier. laptop closed, old football boots, the football, corkboard of memories (polaroids from 03, fest wristband, a torn ticket), a suitcase by the door
- **action:** P26 picks up the suitcase, looks at the room one beat, switches off the light, leaves. door closes. camera stays. long hold. the closed laptop's LED glows and wakes
- **camera:** slow drift across the still life, locked when P26 leaves, tiny push to the laptop
- **motion:** dust motes, the door shadow sweeps the floor, laptop wakes to black then `> next`
- **light:** warm lamp → moonlight → one cyan glow from the laptop
- **text:** `> next ▍` → black → **STILL COMPILING** (small, centred, same warm white as the cursor)
- **sound:** footsteps, door click, silence, one cursor tick, a final unresolved piano chord on the title
- **transition:** cut to black, title, cursor blinks twice and stops
- **tools:** Lemo-Opuscar, Hyperframes (end text)

## A. Character consistency

- **protagonist, same person throughout.** my real look (given 2026-10-26): black medium wavy hair, half-rim glasses, 6 ft and lean, fair wheatish skin, no beard, normal shirts and trousers, black analog watch. it's a cartoon, it doesn't have to be exact. at 6 ft he stands about a head over most people in a crowd
- **P23:** hair longer and messy, slouchier, fidgety hands, oversized new navy backpack, wide eyed
- **P26:** hair shorter and kept, upright and relaxed, slower movements, same backpack now faded with patches, sometimes a lanyard, calmer eyes
- shirt colours on the sheet (P23 white half-sleeve shirt + beige trousers, P26 muted navy full-sleeve shirt + charcoal trousers) are an agent proposal, not confirmed. sheet: `C:\Users\prakh\still-compiling\chars\character-sheet-v1.jpg`
- face, glasses and watch identical between P23 and P26
- **her:** face never fully shown, shoulder length dark hair, mustard yellow scarf or cardigan. never shown after scene 05
- **friends:** 3 to 4 recurring silhouettes, one signature item each (red cap, headphones round the neck, jersey #10, tote bag)

## B. Art direction (updated for the locked style)

Urban Sketch · Pen & Wash. sepia fineliner `#2b2520` that wobbles and overshoots, loose transparent wash off the lines, cream paper `#f5eede`, margins left blank, edges fraying. colour grammar: **amber = human / college / memory, cyan = code / systems, pear green accent = the Pear era.** colour can be withheld and arrive as an event. interfaces sit in the world, not as flat overlays. one monospace (JetBrains Mono) for terminal and UI text in warm white `#F2EBDD`, handwriting (Caveat, Reenie Beanie) for everything on the page.

## C. Avoid

cartoonish or chibi, supercars, money, coins, rockets, SUCCESS text, trophies, confetti, fist pumps, hooded hackers, Matrix rain, grindset imagery, motivational quotes, linkedin stock look, crying close ups, rain on window melodrama, her face, changing face or glasses, gibberish UI text, oversaturated neon, logos other than specified.

## D. Sound and music timeline

| Time | Music | Key sounds |
|---|---|---|
| 0:00 to 0:05 | silence, one felt piano note | cursor tick ×4 |
| 0:05 to 0:12 | 4 note piano motif | birds, distant chatter |
| 0:12 to 0:21 | motif + light brushed beat | ball thud, crowd, fest bass, laughter |
| 0:21 to 0:28 | piano, sparser | keys, fan, tab pops, cursor tick |
| 0:28 to 0:36 | fingerpicked guitar joins (warmest cue) | wind, muffled laughter |
| 0:36 to 0:42 | **music cuts out** | drizzle, far bike, slow cursor tick |
| 0:42 to 0:51 | synth arpeggio, accelerating | error beeps in key, typing |
| 0:51 to 0:59 | arpeggio into a wide pulse | server fans, corridor whoosh |
| 0:59 to 1:04 | **silence** | one soft chime, cursor tick |
| 1:04 to 1:13 | full theme: piano + pulse + low strings | data hum, order flow ticks |
| 1:13 to 1:21 | theme, stereo split left/right merging to centre | professor (L), Slack (R) |
| 1:21 to 1:28 | 4 note motif reversed | birds settling, footsteps |
| 1:28 to 1:41 | fade, silence, one unresolved chord on the title | door click, one cursor tick |

note: the Urban Sketch style's own sound palette suggests jazz waltz or solo nylon guitar families. the felt piano motif used in the opening test is our choice, it works, keep it unless i say otherwise.

## E. Reusable assets (make once)

1. cursor glyph + blink cycle (530 ms, `#F2EBDD`) + tick sound
2. P23 / P26 character sheets (turnaround, 5 expressions, hands)
3. her silhouette + mustard scarf
4. friends ×4 silhouettes
5. college gate set in 3 lights: pre-dawn, sunrise, dusk
6. hostel room in 3 states: 2023 messy, obsession era cluttered, final half packed
7. laptop with a sticker set that only grows (2 at scene 04, ~6 at 07, ~9 by 13)
8. football (one panel pattern) + football boots
9. navy backpack, new and faded
10. chai cups (pair)
11. corkboard + polaroid frames (scenes 03 and 13)
12. UI kit: terminal, browser tab, GitHub commit / contribution grid / PR card, rejection email, build fail panel, Slack, Pear notification toast, candlestick chart, order book, agent graph, log panel
13. terminal city / candlestick skyline (one city, two skins)
14. corridor → server aisle transform
15. music motif stems: piano, guitar, arpeggio, pulse

## F. Build order

1. asset sheets: characters (lock P23 / P26 first), cursor, gate, room, laptop, football, UI kit
2. scenes 02 + 12 together (mirrored gate shots set continuity)
3. scenes 05 + 06 together (same composition)
4. scene 13 (the final room shows what earlier rooms must plant)
5. scene 04, then 07 (room progression)
6. scene 01 (already prototyped in the opening test)
7. scene 03 (reuses the corkboard from 13)
8. scene 08, then 10 (the city must match the candlestick skyline)
9. scenes 09, 11
10. score to picture last, then a sound pass, then the cursor tick everywhere

## G. Continuity rules

- cursor: same glyph, colour, font and blink rate every time. appears at each phase change: 01, 04, 06, 07, 09, 11, 13
- face, glasses and watch never change between P23 and P26. only hair, posture and backpack wear show time
- screen direction: entering a phase = moving away from camera, leaving = moving toward it (02 vs 12)
- palette: amber and cyan first meet in 04, first balance in 09, merge in 11. no cyan in 05, no amber in 08's server half
- football visible in 03, 04, 07 (dusty), 13. never moves on its own after 04
- laptop stickers only increase
- her: never seen after 05. absence in 06 shown only by the single cup, the empty space and the loose earphone
- locked compositions repeat exactly: 02 ↔ 12, 05 ↔ 06
- every cut has a link (object, shape, motion or light). only plain cuts are 05 → 06 and 13 → title
- UI text must be legible and real
- no celebration beats. the biggest reaction in the film is an exhale

## H. Tool to scene map (corrected)

| Tool | Scenes | Job |
|---|---|---|
| Hyperframes | 01, 03, 04, 05, 06, 07, 09, 10, 11, 13 | UI and type, plus the scrapbook corkboard in 03 and sketch overlays in 05 |
| Lemo-Opuscar (Urban Sketch) | 02, 05, 06, 12, 13 + the overall look | painted character scenes, one visual language for the film |
| OneTake | 08, 11 | continuous camera through interfaces |
| Claude Animation Base | optional | overlaps Lemo-Opuscar, two looks would clash. probably not used |

pipeline: Lemo-Opuscar for painted scenes, Hyperframes for UI and collage layers and the final edit, OneTake for 08 and 11.
