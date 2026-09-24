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

Three schemes, picked by whatever you touched last. **Keyboard**: two hand positions are
bound at once — arrows to move with **Z / X / C** for attack / Special / dash, or WASD with
**J / K / L**; Space and Shift also dash. Aim is the last direction you moved, resolved at
full 360°, and the pointer on the character shows it. **Gamepad**: left stick moves and
aims, X attacks, Y Special, A dashes. **Touch**: the floating stick plus ATTACK, SPECIAL and
DASH buttons. The mouse does nothing: trackpad aiming on laptops is miserable, so the desktop
scheme is keyboard-only by decision (2026-09-21).

Nobody should have to guess: a hint line sits at the bottom of the screen from the start of
a run naming the buttons for your scheme; each item disappears once you have used it, and
the line goes away two seconds after all three have been. It comes back, shortened, on the
death card.

**Two attacks.** The **Attack** is the three-hit chain (below). The **Special** is the
weapon's (see Weapons). With the sword it is *Nova*: a
full circle of blade around the character that hits everything within about two units once,
throws it back hard (three times the chain's knockback), and costs a long recovery — the
"get off me" button for when the Breakers close in. It has no cooldown; the recovery is the
price. Dash cancels it like anything else; Attack pressed during its recovery starts the
chain. With the lance the Special throws it, and a second press calls it back.

**Assumed** — the two keyboard hand positions (arrows + Z X C, WASD + J K L) are both live
rather than a chooser; Nova as the Special rather than a thrown blade: the chain already
covers reach, and the pressure problem in the room is being surrounded. Its numbers (startup
0.16 s, active 0.10 s, recovery 0.42 s, damage ×1.5, knockback ×3, radius 2.2) are builder
consts. Not reviewed by you.

**On-screen controls.** A floating stick — it appears wherever the left thumb lands on
the left half of the screen — and ATTACK / SPECIAL / DASH buttons (bottom-right) drive the same
actions on touch devices only. Desktop never shows them:
without a gamepad you play on the keyboard. Portrait phones get a full-screen "turn your
phone sideways" prompt instead of the controls until rotated.

The flow is Hades': you are never locked. Dash cancels any phase of an attack. Moving
cancels an attack's recovery, so recovery is only felt standing still. Attacks are a
three-hit chain — each swing steps ~0.6 m toward the aim with a blade that reaches ~1.8 m (playtesters found
the original 1.5 m lunge dragged them forward, and its removal made the game much harder — the
lunge had been carrying the blade onto the target — so reach went up and the step came back at
half, 2026-09-23), the third hits harder
with a longer recovery — and a press during a swing's wind-up is queued, not dropped. Attacking during
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

A run is **four rooms and a boss** (rooms decided 2026-09-21, the boss 2026-09-22). Each room is a fight: waves of Breakers and
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

After the boss the exit leads to the surface: the run ends with a tally of rooms cleared
and samples collected, and any press starts a new run. **Death ends the run** and sends her
up: see The surface. There is no checkpoint inside a run.

**The fifth section is the boss.** The Foreman waits in a
sparse arena at the top of the shelf, and the fight is Hades' first boss transposed: it never
stands still; up close a three-hit cleaver combo, each hit announced with a wedge; at mid range
a fan of five bolts along five thin lanes; when crowded it blinks away; at two-thirds and at
one-third health it calls two Breakers; below a third it lashes three telegraphed lines across
the arena. Thirty health against our one to three a hit — about forty-five seconds without
modules, for either weapon (halved from sixty, 2026-09-24). It never staggers, only
flinches, so stun-locking is not a plan; reading it is. Its health is a big bar along the
bottom of the screen with its name over it. The room clears when the Foreman and its Breakers
are all down; the exit leads to the surface.

**Assumed** — the numbers: 30 HP (phases at 19 and 9); combo wind-ups 0.35/0.25/0.25 s with 2 damage a hit; bolt fan
after a 0.7 s tell, five bolts at 0°/±12°/±24° at 8 u/s; blink 6 u away after 1.5 s of being
crowded, 0.3 s invulnerable; lash lines after a 1.0 s tell, 3 damage; phases at 66 % and 33 %
with cooldowns ×0.85 and ×0.7; the boss bar at the bottom of the screen; adds must die for the
clear; no module offer after the boss. Not reviewed by you. The boss arena is Biome 1 —
whether the boss lives in a different biome is still **Undecided**.

**Assumed** — rooms are all 24 × 14 (variable sizes need per-room camera framing); two waves per
room with the table Breakers·Surveyors 3·0/3·1, 4·1/4·2, 4·2/5·2, 5·2/5·3; HP crystal heals 4;
the door is a 3 u geometry gap at the north centre with no dressing yet; entry at (0, −5);
generator picks 6–10 blockers and 8–12 decals per room. Not reviewed by you; all constants in
`RunPlan` and `RoomGenerator`.

**Undecided** — what Core Samples buy on the surface (the meta-goal the run feeds), and whether
the boss room is a different biome. Affects: economy, the end-of-run screen, the second kit.

## The surface

Above the shelf is **the Landing**: the fabrication dome the expedition left behind, run by
the AI that stayed. It is one room in the same cold light as the shelf but in the AI's colour
space — pale alloy and the player's blue, no violet — and it holds five things: the
**assembly line** along the west wall with its gantry arms, where she comes together; the
**Lens** on the north wall, the AI's eye, an iris that breathes; the **Fabricator's bench**
where Revisions are bought; the **Armoury**, the rack of frames on the east wall, where the
weapons are; the **Beacon** mast with its three stage lamps; and the **shaft** in the floor,
the way back down. Around them the dome is dressed as a workshop: parts crates by the line (the
spares are unfinished cold alloy — gold is hers alone once she is poured), a cable spool, blue
wall lamps, cable trays along the south — on a floor of its own, cold blue-grey plates with a
single guide line.

Arriving is the same whether she died or walked up: fade, the line, her parts converging into
the standing pose over two seconds (the blue lines come on last), the Lens opening, and the
Fabricator speaking — two or three lines that know what happened: which room, what parted her,
what she brought. Then she has the room. Walk into the bench, the rack or the shaft and the
prompt names the key; the bench opens the Revisions; the rack opens the Armoury; the shaft asks
once and drops her into a new run. The status line at the top names what she will carry down.

The Fabricator speaks — a typewriter line, attack to continue, dash to skip — and is **voiced**:
every line without a run-specific number has a clip, generated offline with Piper TTS from a
CC0 voice and put through a deterministic machine treatment (pitch down two semitones, an old
PA's band, a short metallic reverb). Lines that name a number stay text. Every line is under
fourteen words so the voice reads clean. **Assumed** — the voice (a neutral US male carried
toward the machine by the treatment) and the treatment itself; both are one script to re-run.

## Progression

**Between runs: the surface** (decided 2026-09-23). Every run ends above ground, in the
Foundry, whether she died or walked up past the Foreman. The AI that made her — the
Fabricator — rebuilds her on the assembly line,
her memory of the run feeds the next Unit, and everything she carried is **banked in full**:
Core Samples always come home, even from a death (Hades' rule for Darkness). Death costs only
the run's modules.

Samples are spent at the Fabricator's bench on **Revisions** — permanent: *Gauge* (+2 max
health, 1), *Gauge II* (+2 more, 2), *Reserve* (start every run holding one module, 2),
*Lattice* (dash cooldown −25 %, 2), *Retention* (keep your first module between runs, 3). Each
once. They are folded into the next run before its own modules.

### Weapons

She carries one weapon down the shaft. The Fabricator made her for the **sword**. The
**Coring Lance** was the expedition's tool for pulling cores from the shelf; she buys it at
the **Armoury**, the rack of frames on the Landing's east wall. Its thrusts reach twice as
far as the sword's. Its Special throws it through every enemy in a line, and she fights
unarmed until she recalls it. She can switch weapons at the rack before any descent. Once
bought, the lance is hers for good.

The numbers are in `docs/design/weapons.md` and in code in `MoveSets`: the lance's chain is
the sword's with +0.04 s startup and +0.05 s recovery a hit, a 2.0 × 0.8 m box reaching 0.6 to
2.6 m, a 1/1/2 chain like the sword's; the throw deals 2 to everything on a 9 m line (12 with
Reach) and sticks in a wall; the recall deals 1 on the way back and pulls what it hits toward
her; while it is out she has two quick jabs of 1. A thrust draws a blue line on the floor out to
its tip, since the lance itself stops a metre short of where it can land. The run takes the weapon the save says she
carries at the descent; nothing mid-run can change it. Every run module works with both.

**Assumed** — the lance costs 4 Core Samples, and buying it equips it. Not reviewed by you.
Each is one constant. **Assumed** — the Armoury is the rack as a station of its own, not a row
on the Fabricator's bench. **Assumed** — the recall is a second press of Special (latched if
pressed in the 0.35 s after the throw), it ignores walls on the way back, and it pulls rather
than pushes; the thrown lance sticks where a wall stops it. **Assumed** — the lance's box is
2.0 × 0.8 at 1.6 m rather than the brief's 1.2 × 0.9 at 2.0, which left a blind spot a Breaker
in contact stands in; unarmed jabs deal 1. Not reviewed by you; each is a constant or a flag.

Samples also count, lifetime, toward the **Ascent Beacon** in three stages — 3 it lights, 6 it
tunes, 12 it fires — each a visible change to the mast in the Foundry and new lines from the
Fabricator. At 12 the beacon fires and the loop continues with it lit. The Foreman drops a
sample, so a full run is worth three.

**Inside a run: modules** (unchanged). When you walk through a cleared room's exit, the world
holds and you are offered two of eight upgrades — take one or skip — before the next room fades
in. The offer sits at the exit rather than on the last hit because the take button is also the
attack button. There is no offer after the boss. Each card is labelled with the button it
changes — CHAIN, SPECIAL, DASH, HULL — and reads true for either weapon: Momentum works on the
last hit of whichever chain is in hand, Reach and Shock on whichever Special.

**Assumed** — Edge multiplies chain damage by 1.6, not a quarter: damage rounds to the nearest
even whole number at .5, so ×1.25 turned every 1 back into 1 and did nothing; ×1.6 makes both
weapons' chains 2/2/3, and its card says so: "Chain hits deal 2, the last one 3". The
categories were BLADE and NOVA. Not reviewed by you.

The goal is stated at the start: **the beacon needs twelve Core Samples; the shelf holds
three a run — four rooms and a Foreman down.**

**Assumed** — the death card shows the room reached and the samples carried (the spoken lines
no longer name numbers); the draft count is not shown anywhere now. **Assumed** — the
Fabricator's speaking rules: on the very first arrival all four opening lines
play before the death or ascent lines; a death plays a killer line then a haul line, and a
beacon stage line replaces the haul line when a stage is crossed; a kill with no known attacker
counts as a Breaker's, and any kill in the Foreman's room as the Foreman's; the shop's
introduction plays once ever; Retention brings back the same first module every run once owned;
Reserve draws with the run seed. **Assumed** — the save is JSON in PlayerPrefs (IndexedDB on the web) with a version field for
migration; the Revision list and costs; beacon stages 3/6/12; the Foreman's sample; two-of-eight
modules per room. Not reviewed by you. **Undecided** — whether the beacon does what the
Fabricator says it does (the next arc), and what fires after it fires.

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
above them from their first hit. The player's health is a ring at her feet — blue, then
orange under 60 %, then a pulsing red-orange under 30 % — because in the fight there is no
time to look at a screen edge (playtest, 2026-09-21); the top-left bar stays for the number.

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

**Pacing ramps.** Playtesters said room one went from nothing to everything at once, and that
the pace would suit a later room — so it does. Room one is two then three Breakers at 4.5 u/s
with one attacker at a time and a slow attack cooldown; the Surveyor arrives in room two with
two concurrent attackers; by room four Breakers run at 5.5 with the fastest cooldown. All of it
is one table (`RunPlan.Difficulty`).

**Assumed** — the numbers: Breaker chase 4.5 → 5.0 → 5.0 → 5.5 across the rooms, attack cooldown
1.3 → 1.0 → 0.9 → 0.8 s, concurrent attackers 1 then 2, wave grace 1.2 → 0.8 → 0.8 → 0.6 s;
waves 2·0/3·0, 3·1/4·1, 4·2/5·2, 5·2/5·3 (Breakers·Surveyors); base Breaker chase 5 vs run 5.5, wind-up 0.28 s with the lunge in its
last 0.14 s at 18 u/s, active 0.10 s; Surveyor band 6–9 u (retreat under 5, approach over 10),
charge 0.6 s, bolt 9 u/s with 11 u of reach, cooldown 2.2 s; damage 2 of 10 for both; two
concurrent attackers; stun budget 0.9 s per 2.5 s; first-gesture grace 1.5 s. Not reviewed by you. All builder consts in one
table in `ArenaSceneBuilder`; the lanes' reach follows the lunge and the bolt automatically.

**Undecided** — what the machines were: the wreck's crew, the world's own machinery, or the
AI's earlier attempt. All three fictions below are written to survive either answer.

**Assumed** — the naming rule: the world's machines are named for the work they were built
to do, one or two words, no honorifics (Breaker, Surveyor, Foreman; free slots that already fit:
Cutter, Hauler, Dredger, Rigger, Welder). Not reviewed by you. Cheap to change now, expensive
after a dozen enemies, items and barks are written against it. In code they are still
`Brute`/`Sentry`/`Warden`; only the boss shows its name on screen (bar and name card).

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

**Foreman.** It marked the work and never did it — which plate the Breakers opened, along
which line, and what was fit to go up the way you came down — and it parted whatever was not,
which is what the cleaver is for. It still works to that order: it stands clear and marks from
a distance, steps out of reach when crowded, calls a pair of Breakers down when the room gets
away from it, and when there is nothing left to call it burns three lines across the silt and
makes the cut itself.

**The Fabricator** was the expedition's manufacturing intelligence. Its crew is gone; it
stayed at the Landing, and it wants to go home. It builds Units from what the shelf gives back
— you are the latest, its Tender — and sends each down for Core Samples. A lost Unit's samples
and memory come up into the next one: every loss is a draft. The samples feed the Ascent
Beacon, which lights at three, tunes at six and fires at twelve to call the fleet home. Warm,
dry, patient, a little too fond of you. Nothing the Foreman has marked has gone up in a long
time; the Fabricator sends Tenders down to fetch what the Foreman will not pass.

**Undecided** — whether the beacon does what the Fabricator says. It never lies; it chooses
what to say. Affects: the ending, what answers at twelve. **Assumed** — "the crew went down the
shaft" is the Fabricator's line, not the doc's fact; the wreck on the shelf may or may not be
its ship.

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

Decided 2026-09-23. **Combat sound** is real and reactive: CC0 Kenney clips for light and
heavy swings, enemy hits, a distinct player-hurt hit, the Surveyor's shot, deaths — nothing
plays before the first press in a browser. **The Fabricator is voiced** — every script line has
a clip (Piper TTS, CC0 voice, a machine treatment). **Music** is one loop per place, crossfaded
over about a second as the scene changes: the shelf, the Foreman's room, the Landing; an
**ambience bed** under each (the drained basin's wind and faint machinery below; the dome's hum
and fabrication ticks above). Music ducks under a voice line and comes back after it. All of it
is gesture-gated for the web and preloaded.

**Assumed** — one loop per place rather than layered or adaptive music; the crossfade and duck
times; CC0-only sourcing for the tracks (the specific tracks are recorded in
`Assets/ThirdParty/ATTRIBUTION.md`). Not reviewed by you. **Undecided** — whether the boss gets
its own track or a variation of the shelf's, and a sound for the beacon firing.

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
