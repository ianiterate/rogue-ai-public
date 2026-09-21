# rogue-ai — design

**What the game is right now.** Present tense, no history. Run `/open-questions` to
see everything currently waiting on a human call.

> This file is published verbatim to the public play repository on every web deploy
> (see the deploy pattern in `SETUP.md`). Write it knowing players may read it.

---

## Premise

A fast, top-down action-roguelike. Combat flow in the family of Hades and Cult of the
Lamb: short commitments, dash as the answer to everything, readable enemy telegraphs.

You are an android — a tall, poised humanoid unit in Bayonetta's silhouette, drawn in
the Hades manner — sent down by an AI to gather resources from a strange world and to find
out what is there. The world is not empty. Tone, the AI's character, and what "down there"
turns out to mean are the next things to write — see World and fiction.

## Core loop (first cut)

Move, dash, slash. Enemies close distance, telegraph, strike; the player reads the
telegraph and answers with position or a dash. Everything else — rooms, rewards,
runs — is built on this loop holding up, so it is being made first and judged with a
controller in hand before anything is stacked on it.

## Controls

Controller-centric. Left stick moves and aims (aim is the last non-zero move
direction, resolved at full 360°). Face buttons: **Dash** (south) and **Attack** (west).
Keyboard is the secondary scheme: WASD / arrows, Space to dash, J to attack. The mouse
does nothing in play — it is reserved for the on-screen controls and UI.

**On-screen controls.** A floating stick — it appears wherever the left thumb lands on
the left half of the screen — and ATTACK / DASH buttons (bottom-right) drive the same
actions on touch devices only. Desktop never shows them:
without a gamepad you play on the keyboard. Portrait phones get a full-screen "turn your
phone sideways" prompt instead of the controls until rotated.

The flow is Hades': you are never locked. Dash cancels any phase of an attack. Moving
cancels an attack's recovery, so recovery is only felt standing still. Attacks are a
three-hit chain — each swing steps toward the aim, the third hits harder with a longer
recovery — and a press during a swing's wind-up is queued, not dropped. Attacking during
a dash produces a quick dash-strike that carries the dash's momentum: dash → strike → dash
is the core rhythm.

Movement is planted: the robot runs at 5.5 u/s (the 24 u room in about 4.4 s), reaches that
speed and stops within a frame or two, and reverses without an arc — Hades' instant turns.
The camera follows with a 0.12 s soft lag, enough to cushion a dash without making a stopped
robot look like it slides. Decided from play-testing (2026-09-20): 7 u/s was too fast for
the room and the 0.3 s camera lag read as drift.

The character is animated from a rendered sprite sheet: idle, run, dash, hit, death and
one distinct swing per hit of the chain. The swing *is* the timing — each attack clip's
wind-up, strike and follow-through frames are stretched over the attack's startup, active
and recovery, so retuning a profile can never desync the art from the hitbox.

**Assumed** — the player renders one side view mirrored for left/right (as Hades does);
up/down aims read through the arc and the pointer. Four directions would triple the sheet
and needs smaller frames on iOS. **Assumed** — customisation (hair, outfit, weapon, colours)
is authored as architecture only: the character is modular on one rig and the runtime
stacks per-part layers with tints, but v1 ships one look. Adding a variant is a render
flag and a layer, not a rebuild.

**Assumed** — the post-dash dash-strike window (attack pressed just *after* a dash still
counts as a dash-strike) ships disabled, and the third hit's short uncancellable tail is off.
Not reviewed by you. Both are single fields to turn on after a play-test.

## The run

A run is **four rooms** (decided 2026-09-21). Each room is a fight: waves of Breakers and
Surveyors, two waves per room, growing from three Breakers in the first to five Breakers and
three Surveyors in the last. When the last wave falls, two things happen: a **reward** appears
at the room's centre, and the **exit** — a door in the middle of the north wall — sinks into the
floor. Walk through it and the next room is built on the spot: a fresh layout of the same kit
(pillars, rock clusters, hull slabs, crystal spires) laid out by a seeded generator and checked
by rule — every gap walkable, no spawn inside lunge reach of a blocker, the entry, every spawn
and the exit provably connected — so a room can never be unplayable. You enter the new room
from the south, where you came in. Health carries over; nothing else does yet.

The first room is hand-authored — it is the tutorial room and stays the same every run.

Rewards alternate: an **HP crystal** (mint; heals four of ten) in rooms one and three, a **Core
Sample** (violet) in rooms two and four. Samples are counted on the HUD; they are the thing the
run is for. Picking either up is optional; the door opens regardless.

After the fourth room the exit leads to the surface: the run ends with a tally of rooms cleared
and samples collected, and any press starts a new run. **Death ends the run** — the arena
reloads as room one with a new seed. There is no checkpoint and nothing persists between runs
yet.

The fifth section is reserved for a **boss** (next iteration); the structure already has the slot.

**Assumed** — rooms are all 24 × 14 (variable sizes need per-room camera framing); two waves per
room with the table Breakers·Surveyors 3·0/3·1, 4·1/4·2, 4·2/5·2, 5·2/5·3; HP crystal heals 4;
the door is a 3 u geometry gap at the north centre with no dressing yet; entry at (0, −5);
generator picks 6–10 blockers and 8–12 decals per room. Not reviewed by you; all constants in
`RunPlan` and `RoomGenerator`.

**Undecided** — what Core Samples buy on the surface (the meta-goal the run feeds), and whether
the boss room is a different biome. Affects: economy, the end-of-run screen, the second kit.

## Progression

**Undecided** — whether anything persists between runs, and what.
Affects: economy shape, whether early runs are tutorials, long-term retention.

## Simulation

Physics-driven: dynamic 2D rigidbodies at a fixed 60 Hz step, manual acceleration
and deceleration for tight control. Combat is a decoupled hitbox / hurtbox system —
a slash, an enemy bite, and a floor trap all use the same hitbox, each carrying its own
damage, knockback, hit-stop and screen-shake values.

**Assumed** — procedural generation (rooms, drops, encounters) is seeded and
reproducible; the moment-to-moment simulation is **not** replay-exact.
Not reviewed by you. Seeded generation is cheap and gives daily runs and shareable
seeds; replay-exact physics on top of dynamic rigidbodies is not realistic and is not
being pursued. Dropping seeded generation later is free.

## Combat feedback

Every hit is read four ways. A coloured arc drawn from the weapon's real hit shape shows
each swing in its committed direction and freezes with the hit-stop; a small pointer on the
player always shows where the next swing will go. Whoever is hit flashes white and throws a
spark at the contact point; enemies shrink out rather than vanish. Sound is real: CC0 clips
from Kenney for light and heavy swings, enemy hits, a distinctly different player-hurt hit,
and deaths — nothing plays before the first press in a browser. Enemies show a health bar
above them from their first hit; the player's bar sits top-left.

Deliberately not yet: a directional wipe on the arc, a red hurt vignette, an arc on enemy
swings. Each is a small addition once the base read is judged by hand.

## Enemies and pressure

Two kinds so far, on the Hades pattern of a chaser and a caster, and the room only works
with both: the chasers make you move, the caster punishes moving badly.

**Breaker** — the chaser. Runs just under the player's speed, so distance is
earned, not free. Its attack is a committed lunge: it locks its aim, paints its landing lane
on the floor, and in the last part of a short wind-up launches along it. Walking straight
away does not escape it; stepping out of the lane early does, and a dash through the strike
does.

**Surveyor** — the caster. Holds six to nine units away, backs off when rushed,
drifts closer when left alone. Its tell is longer than the Breaker's: it charges its emitter
for 0.6 s while a thin lane shows where the bolt will go, locks the aim at the end of the
charge, and fires a straight bolt that never turns. A sidestep with a little distance or a
dash through it beats it; standing still does not. It is fragile — closing on it is the
reward for reading the room.

At most two enemies attack at once, of either kind; the rest hold at arm's length and
circle, so there is always one tell to read. Stagger is rationed: a few quick hits stun,
then the enemy shrugs the next ones off and finishes its swing through them. A stunned
Surveyor drops its charge.

**The opening is fair by rule.** Nothing moves until the player has pressed something, and
then there is a beat (1.5 s) before any enemy leaves its spot; each new wave gets the same
beat as it materialises. An enemy's first swing always comes after a chase, never off the
spawn. On a phone held upright the "turn sideways" prompt holds the world. The first wave
is three Breakers; the Surveyor arrives in the second.

Death holds the frame for a beat, then the arena reloads fresh. Clearing the room brings the
next wave after a short pause.

Both enemies are animated from rendered sheets like the player: idle, move, one attack whose
wind-up/strike/follow-through frames are stretched over the attack's real timing, hit, death.

**Undecided** — **the Breaker's chase speed against its gait.** The move clip is 8 frames at
12 fps, so one cycle lasts 0.667 s, and at the assumed 5 u/s chase that cycle has to cover
**3.33 m of ground**. The source walk it is built from moves its feet 0.193 m per cycle, and a
1.4 m six-legged machine cannot be made to stride seventeen times further — amplifying that far
pulls the legs off the body. The shipped clip is a constructed *bound* with the feet clear of
the floor for most of the cycle, which is the only gait whose feet cannot be seen to slip,
and it hides the gap rather than closing it. Three ways out, and this one is a real choice
rather than a rendering detail:

1. **Drop the chase to roughly 1.5 u/s** and let the Breaker be a slow, heavy thing the player
   outruns and has to choose to engage. Cheapest, and it changes what the enemy *is*.
2. **Raise the move clip's fps** in the sidecar so the cycle is shorter than 0.667 s. Costs
   nothing to render, but above about 20 fps an eight-frame cycle reads as a vibration.
3. **Keep 5 u/s and accept a skate.** Defensible for a machine — it is not an animal, and
   something that heavy moving that fast can read as driven rather than walked.

This blocks nothing today: the sheet ships and plays at any of the three. It decides whether
the Breaker reads as a bruiser you can outrun or a thing that runs you down.

**Assumed** — the numbers: Breaker chase 5 vs run 5.5, wind-up 0.28 s with the lunge in its
last 0.14 s at 18 u/s, active 0.10 s; Surveyor band 6–9 u (retreat under 5, approach over 10),
charge 0.6 s, bolt 9 u/s with 11 u of reach, cooldown 2.2 s; damage 2 of 10 for both; two
concurrent attackers; stun budget 0.9 s per 2.5 s; waves 3·0, 3·1, 4·1, 4·2, 5·2
(Breakers·Surveyors); grace 1.5 s / 0.8 s. Not reviewed by you. All builder consts in one
table in `ArenaSceneBuilder`; the lanes' reach follows the lunge and the bolt automatically.

**Undecided** — what the machines were: the wreck's crew, the world's own machinery, or the
AI's earlier attempt. Both fictions below are written to survive either answer.

**Assumed** — the naming rule: the world's machines are named for the work they were built
to do, one or two words, no honorifics (Breaker, Surveyor; free slots that already fit:
Cutter, Hauler, Dredger, Rigger, Welder). Not reviewed by you. Cheap to change now, expensive
after a dozen enemies, items and barks are written against it. In code the two are still
`Brute`/`Sentry`; nothing on screen shows a type name yet.

## World and fiction

The first place is **Rustwater Shelf**: a drained alkali basin under a dim violet sky.
Mineral silt has set into terraces the colour of cold ash; the wreck of an earlier
expedition lies half-sunk in it, plating peeled back and still faintly powered. The only
real light is bioluminescent crystal blooming from the cracks — a cold mint green that
pools on the silt. The robot's lamp is the second light source, and it is small.

**Breaker.** Built to open hulls, and still wearing the guard plate that braced the tool it
no longer carries. It closes on anything standing and swings the plate instead: shoulder
down, hips first, the same wind-up every time.

**Surveyor.** It works from the edge of the light — emitter up, a reading held for about a
second, then a bolt down the exact line it sighted. The line is fixed the moment the reading
ends, so it fires at where you were measured, not where you have got to.

**Undecided** — the AI that sent the robot (voice, motive, whether it can be trusted),
what the earlier expedition was, and what lives here. `world-builder` owns these next.

## Art direction

Isometric-feeling 2.5D in the manner of Hades. The game camera is tilted 35° over the flat
physics plane: the floor recedes with a visible plate grid, walls stand three metres tall
along the top and sides, and props and characters are upright sprites sorted by depth.
Everything is modelled and toon-rendered in Blender — flat things straight down, upright
things from the same 35° camera — and placed as sprites; gameplay keeps honest 2D footprints.
The palette is saturated, Tartarus transposed to sci-fi: green alloy ruin, copper machinery,
violet energy, mint crystal glow. Red belongs to enemy telegraphs and blue to the player;
scenery is forbidden those hues. Details: `docs/ART.md`.

**Assumed** — the specific palette values and the 35° tilt. Reviewed by you only as
"isometric like Hades"; both are single values in the render script and the camera rig.

## Platform and input

Desktop (macOS first) and browser via WebGL, published to a public GitHub Pages
repository. Controller is the primary input; keyboard/mouse secondary; touch later.
Browser builds start behind a first user gesture (browsers require it before gamepads
and audio become available).

## Audio

**Undecided** — nothing established.

---

## Third-party assets

**Assumed** — commercial-safe licences only: CC0, CC-BY, and permissive (MIT, Apache,
BSD, OFL). CC-BY-NC, CC-BY-SA and CC-BY-ND are rejected, as is anything advertised as
"free to use" without a named licence.

Not reviewed by you. Relaxing this later is one edit to the `asset-licensing` skill;
tightening it later means re-sourcing every asset already in the project, which is why
the default sits at the strict end. Allowing NC would permanently foreclose any paid
release; allowing SA risks share-alike terms propagating into your own content.

Every external asset is recorded in `Assets/ThirdParty/ATTRIBUTION.md` in the same
commit that adds it.

## Tooling

Unity `6000.5.8f1`, URP 17.5, New Input System, Cinemachine 3. Blender `5.0.1`.
Editor automation through MCP for Unity (CoplayDev, MIT) once wired — see `SETUP.md`;
until then, batchmode with the Editor closed.
