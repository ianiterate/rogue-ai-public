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
**J / K / L**; Space and Shift also dash. Bindings are by physical key position (the US
names above); the on-screen labels follow your keyboard layout, so AZERTY shows ZQSD and W
for attack (read from the browser's layout map in Chrome and Edge, guessed from the browser
language elsewhere, `?layout=azerty|qwertz|qwerty` to force it). Aim is the last direction you moved, resolved at
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
throws it back hard (knockback ×3; the chain's first two hits shove at ×0.5), and costs a long recovery — the
"get off me" button for when the Breakers close in. It has no cooldown; the recovery is the
price. Dash cancels it like anything else; Attack pressed during its recovery starts the
chain. With the lance the Special throws it, and it comes back on its own.

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
three-hit chain — each swing steps ~0.6 m toward the aim with a blade that reaches ~1.95 m (playtesters found
the original 1.5 m lunge dragged them forward, and its removal made the game much harder — the
lunge had been carrying the blade onto the target — so reach went up and the step came back at
half, 2026-09-23), the third hits harder
with a longer recovery — and a press during a swing's wind-up is queued, not dropped. Attacking during
a dash produces a quick dash-strike that carries the dash's momentum: dash → strike → dash
is the core rhythm.

**The slash finds its target** (2026-09-27, from play: dash + Nova beat the chain, because the
chain needed perfect aim and Surveyors outran it; the ruling was to improve the slash and leave
Nova alone). Every chain hit, the dash-strike and the lance's thrusts use **aim assist**: the
swing bends onto the best enemy within 3.2 m and 55° of the stick — never one behind her — and
goes down the stick if there is none. Inside 22.5° of the stick every target counts as straight
ahead and the nearer wins; outside it, angle decides. The lance's **throw** has its own, narrower
assist: 35° of the stick and the throw's full range, so a line of enemies is threaded through the
nearest. Nova and the spin are rings and go down the stick. Thrown things now **point where they
fly** on screen, foreshortened by the camera's tilt, so a spear thrown south reads as a spear.
**Assumed** — that foreshortening is exaggerated past the true projection (length ratio cubed:
straight down the screen 0.55 long rather than 0.82, diagonals 0.76) so the south throw reads
strongly; one constant, `IsoView.ForeshortenExponent` = 3, and 1 restores the honest view. Hits two and three **track** the enemy the
chain last struck while it is within 3.5 m and not behind the stick. A chain hit with a target
takes a **closing step** instead of its short step: fast enough to bring the target to the middle
of the blade by the end of the swing, leading where it is moving, never more than 1.2 m (1.6 m
for the finisher); a target already there gets a planted swing. Without a target the step is
the old one, and **swinging no longer brakes a chase**: hits one and two thrown at a run keep the
run, never faster than it. Hits one and two **shove half as far** (knockback ×0.5 and ×0.55, about
0.7 m), so the target is still in reach for the next; the finisher still throws. The sword's blade
is **taller**, 1.6 × 1.4 m reaching 0.35–1.95 m (was 1.5 × 1.1), because the 35° camera squashes
up and down on screen; the lance's is 2.0 × 1.4, as wide as the sword's. The dash itself still goes where
the stick says. Numbers and the chase model: `docs/design/slash.md`; in code `AimAssist`,
`ChainStep`, `MoveSets`.

**Assumed** — aim assist's 22.5° dead band applies to every control scheme, not only the
keyboard's eight directions; cheap to split per scheme. **Assumed** — tracking gives up at 90° off
the stick, not the plan's 100° (100° could swing her at something behind her). **Assumed** —
closing steps come up to speed at her run's acceleration (220 u/s², not the lunge's 90) and lead
the target's velocity; without them hits land at the tip of the blade rather than its middle.
Not reviewed by you; all cheap.

**Undecided** — whether Nova keeps its 0.42 s recovery instead of cancelling into another Nova on
its first recovery frame (which loops it every 0.26 s). Kept as it is by your ruling; it blocks
nothing, and it sets the ceiling on Nova spam.

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

A run is **four rooms and a boss** (rooms decided 2026-09-21, the boss 2026-09-22). Each room is a fight: two waves of
Breakers and Surveyors, and, once the beacon is lit, Welders in rooms three and four, growing from
two Breakers in the first room to eight machines a wave in the last. When the last wave falls, a reward appears and the **doors** in
the north wall sink into the floor. Walk through one and the next room is built on the spot: a fresh layout of the same kit
(pillars, rock clusters, hull slabs, crystal spires) laid out by a seeded generator and checked
by rule — every gap walkable, no spawn inside lunge reach of a blocker, the entry, every spawn
and the exit provably connected — so a room can never be unplayable. You enter the new room
from the south, where you came in. Health carries over; nothing else does yet.

The first room is hand-authored — it is the tutorial room and stays the same every run.

**Doors** (decided 2026-09-27). The first room pays an HP crystal (mint; heals four of ten),
then a module. Rooms one to three open **two doors**, each marked with what the room behind it
pays: a **module** (blue-white chip), a **Core Sample** (violet crystal), an **HP crystal**
(mint), a **Refit** (copper chip), or a **marked room** (the Foreman's red seam: more machines,
pays a module and a sample; marked rooms appear only once the beacon is lit, because the Foreman
marks a room only after it has heard the beacon). The HUD names both, left door first: `MODULE OR SAMPLE`. Room four
has one door, the Foreman's. A sample always waits behind a door into rooms two and four, so
three samples are reachable on every run without a marked room; a fourth is only ever behind
one. Samples are counted on the HUD; they are the thing the run is for. Picking a reward up is
optional; the doors open regardless. The plan is the run seed's, never the player's.

**The plates stay up** (2026-09-27, from play: they read as under the door and vanished while
deciding). Each door's plate is 0.9 m, hung 0.35 m up the panel's face so it sits in the middle
of the strip of face the camera shows, and it is drawn over everything standing in the room —
a Breaker at the door cannot cover it. When the panel sinks the plate stays where it hung, lit
over the open doorway, until you walk through: the choice is made with the doors open, so the
sign has to be there then. **Assumed** — the size, the mount and drawing over the actors (under
the station labels, enemy bars and HUD). Not reviewed by you; three constants.

**Assumed** — the door rules (`docs/design/doors_and_depth.md` §1): the two doors never match;
room one's doors are always a sample against a module or a Refit (70/30); room two's never hold
a plain sample; room three's always hold one; exactly one HP door a run, at room two's or room
three's doors, even odds; no marked room behind room one's doors, so the first marked room can
be room three; the free slots weigh module 50, Refit 20, marked 30, or module 70, Refit 30 and no
marked room while the beacon is dark. A Refit door pays a module
offer instead when nothing held can be raised — nothing held, or everything Epic. A **marked
room** adds one Breaker and one Surveyor to each wave at the next room's difficulty, with no
step past room four (a marked room four is harder by head count only); once the beacon is tuned,
its last wave fields an **elite** in place of that extra Surveyor (see Enemies and pressure). Doors at ±6 m on the
26 m wall; a two-door room keeps its blockers and spawns 1 m off each door's lane rather than
the single door's 3.5 m, which covered the whole north half. "Marked room" is the name; HARD is
the fallback. Not reviewed by you. Each is a constant, a weight or a table entry.

After the boss, the exit leads up. The world holds on the **ascent tally**: rooms cleared,
marked rooms, the samples she carries and where they came from, the modules she held, and the
beacon's count with this run's samples added. Any press takes her up to the Landing.
**Assumed** — the tally's lines and their order, and that a press counts only after half a second
and only if it began after the card appeared, so an attack still held from the fight cannot skip
it (`docs/design/beacon_and_unlocks.md` §5). Not reviewed by you; view-side and cheap.
**Death ends the run** and sends her up: see The surface. There is no checkpoint inside a run.

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
clear; no module offer after the boss. Not reviewed by you. The boss arena is Biome 1.

**Assumed** — rooms are all 24 × 14 (variable sizes need per-room camera framing); two waves per
room with the table Breakers·Surveyors·Welders 2·0·0/3·0·0 and 3·1·0/4·1·0, then, while the beacon
is dark, 4·2·0/5·2·0 and 5·2·0/5·3·0, and once it is lit, 4·2·1/5·2·1 and 5·2·1/4·2·2. Room four's
last wave is capped at eight so that a marked one still puts at most two machines on each of the
five spawn points (`docs/design/welder.md` §2). HP crystal heals 4;
each door is a 3 u geometry gap in the north wall (two at ±6 in rooms one to three, one at the
centre in room four) with a 0.9 m reward plate on its face; entry at (0, −5);
generator picks 6–10 blockers and 8–12 decals per room. Not reviewed by you; all constants in
`RunPlan` and `RoomGenerator`.

**Decided 2026-09-28** — the boss room is **the Foreman's yard**, a variant kit on the Biome 1
pipeline rather than a second biome: its own floor (darker ash scored with copper cut lines and
marked hexes), plating walls, stacked cut plates and a cleaver rack for blockers, a crane hook in
the foreground, and the Foreman's red-violet as its only emissive. The shelf's palette rules hold
there except that violet and mint are absent. Details: `docs/ART.md`, "the Foreman's yard".
**Assumed** — a variant kit, not a folder of its own; a second biome would need the catalogue,
the generator and `build_assets.sh` made per-biome (listed in ART.md).

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

The Landing points the way. Once she has control, a card says what the visit is for:
spend, or go down for more. A station she has never opened, or can now buy from, carries a
floating label (REVISIONS NEW, LANCE AFFORDABLE). If she passes one she can afford, the
Fabricator points her to it; with nothing to buy, it points at the shaft. **Assumed:** each
nudge plays once a visit, after eight or twenty seconds, bench before rack, as a caption she
keeps walking through rather than a line that stops her. **Assumed:** the card holds five
seconds. The shaft carries one label only, HEAT  NEW, from the arrival that first puts the dial
on its card until she opens the card with the dial on it; otherwise it has none of its own. The
strings and the rules are in `docs/design/landing_guidance.md` and `docs/design/pressure_text.md`
§5; each timing is one constant.

**Heat.** Once she has walked up past the Foreman, the shaft card carries a dial, HEAT 0 to 5.
The Fabricator runs the shaft hotter to reach further down, and the shelf pushes back. Each step
adds a condition on top of those below it: **Tempered** (every machine tougher), **Hot shelf**
(each room fights like the next), **Thin repair** (crystals and Salvage heal less), **Marked
waves** (a marked machine in every wave past room one), **Foreman's temper** (more health, less
patience). An ascent at heat *n* pays *n* more samples, as one more pickup where the Foreman
falls. The dial offers one level past the best heat she has cleared, and a run keeps the heat it
went down at. The status line names the dialled heat (HEAT 3) before the weapon; the ascent tally
adds HEAT *n*  +*m* SAMPLES, and FIRST CLEAR AT HEAT *n* when it sets the record. **Assumed** — the
name, conditions and payout; numbers in `docs/design/pressure.md`, words in
`docs/design/pressure_text.md`.

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

Samples are spent at the Fabricator's bench on **Revisions**, which are permanent. Tier I is
there from the start: *Gauge* (+2 max health, 1), *Gauge II* (+2 more, 2), *Reserve* (start every run
holding one module, 2), *Lattice* (dash cooldown −25 %, 2), *Retention* (keep your first module
between runs, 3). Tier II sits below it, greyed with NEEDS THE BEACON TUNED until the beacon is tuned: *Capacitor* (the Special recovers a
quarter sooner, 2), *Cladding* (the first hit she takes in each room deals nothing, 4), *Failsafe*
(once a run, a hit that would destroy her leaves her at 1 health, 3), *Patchwork* (the Foreman drops
two samples, 4). Each is bought once. They are folded into the next run before its own modules.
The bench and the rack together cost 27 samples, before tier III.

**Tier III** opens one Revision at a time: the first ascent at each heat opens one more, and the
bench says REVISIONS NEW. The heat taught them to the Fabricator, so they are named for what heat
does to metal: *Flux* (the dash stays untouchable 0.05 s longer, and the dash's shortest cooldown
rises with it, 5), *Anneal* (+2 max health, each cleared room repairs 1, 8), *Preheat* (Reserve's
module comes up at least Rare; needs Reserve, 6), *Quench* (the first hit each room that gets past
Cladding deals half, 6), *Braze* (more doors lead to a Refit, 5). Thirty samples in all, opened in
that order at heats 1 to 5. The row is off the bench until the shaft carries the dial; after that
each card shows greyed, CLEAR HEAT *n* in its corner, until its heat is cleared. **Assumed** — the
five, costs and order, and the row's visibility. A name is permanent once a build sells it.

### Weapons

She carries one weapon down the shaft. The Fabricator made her for the **sword**. The
**Coring Lance**, the expedition's tool, is bought at the **Armoury** on the Landing's east
wall. It thrusts quick and long. Hold Attack to charge; release to spin it full circle. Its
Special throws it through everything in a line; it comes back on its own, sooner if
she presses Attack. She is never unarmed. She can switch weapons at the rack before any
descent. Once bought, the lance is hers for good.

The numbers are in `docs/design/weapons.md`, `docs/design/spear.md` and in code in `MoveSets`:
the lance's thrusts have the sword's cadence (the finisher's tail 0.34 s against the sword's
0.38) in a 2.0 × 1.4 m box reaching 0.6 to 2.6 m, a 1/1/2 chain like the sword's. Attack held
0.3 s after a thrust's press is a charge she can walk in at 60 %; let go — or held to 2 s — it
is a spin that deals 2 to everything within 2.6 m of her and stuns for 0.4 s, and the next
press is thrust one again. The throw deals 2 to everything on a 9 m line (a quarter further
with Reach) and sticks in a wall; it turns for home on its own 0.6 s after the throw (longer
with Reach, as far as the range grows) or 0.15 s after it stops, whichever is first, and an
Attack or Special press from 0.15 s after the throw calls it sooner. The return deals 1 on the
way back and pulls what it hits toward her. While it is out she moves and dashes as ever, and
Attack is the recall rather than a thrust. A thrust draws a blue line on the floor out to its
tip; the spin draws a ring. The run takes the weapon the save says she carries at the
descent; nothing mid-run can change it. Every run module works with both.

**Assumed** — the lance costs 4 Core Samples, and buying it equips it. Not reviewed by you.
Each is one constant. **Assumed** — the Armoury is the rack as a station of its own, not a row
on the Fabricator's bench. **Assumed** — the returning lance ignores walls and pulls rather
than pushes; the thrown lance sticks where a wall stops it. Each is a flag.

**Assumed** — the spear's numbers: thrusts at the sword's cadence, box 2.0 × 1.4; spin radius
2.6, 2 damage, stun 0.4. Cheap: `MoveSets` constants. **Assumed** — hold 0.30 s to charge, walk
at 60 %, and the spin fires by itself at 2.0 s. Cheap. **Assumed** — the throw returns on its
own at 0.6 s (scaled by Reach) or 0.15 s after it stops; Attack or Special recalls it from
0.15 s; the catch beat is 0.2 s. Cheap. **Assumed** — Reach does not grow the spin, and Shock
does not stun with it; Edge and Tempo do apply. One line each in `Effective`. **Assumed** — an
Attack pressed while the lance flies home becomes thrust one if she catches it within the
input buffer. Cheap. None of these reviewed by you.

Samples also count, lifetime, toward the **Ascent Beacon** in three stages: 3 lights it, 6
tunes it, 12 fires it. Each stage is a visible change to the mast on the Landing, a new line from
the Fabricator, and **new content below** (decided):

- **Dark** (under 3): the shelf as it first is, with no marked rooms and no Welders.
- **Lit** (3): marked rooms enter the doors, because the Foreman marks a room only once it has
  heard the beacon. Welders join the waves of rooms three and four.
- **Tuned** (6): tier II of the Revisions opens, and each marked room's last wave fields an elite.
- **Fired** (12): the ending arc plays once, then the loop continues with everything open. The
  status line reads BEACON FIRED, and samples keep buying Revisions.

A run keeps the stage it had at the descent. The Foreman drops a sample, two with Patchwork, so a
full run is worth three, or four with Patchwork.

**Assumed** — which stage opens what. While the beacon is dark, the door slots that marked rooms
would take go to module 70 and Refit 30. The tier II effects and prices are Capacitor 2, Cladding 4,
Failsafe 3 and Patchwork 4; the brief had 3 for the first two, and they moved because Cladding is
worth more health per sample than anything else on the bench. Capacitor is read as a shorter Special
tail, because no Special has a charge. Failsafe is the once-a-run save rather than a faster lance
return. Tier II is greyed, not hidden, until the beacon is tuned, so the player sees what the beacon buys. A player who wins every run fires
the beacon in four to six runs and owns everything in seven to twelve
(`docs/design/beacon_and_unlocks.md` §3.5). Not reviewed by you. Each is a constant or one
comparison, but a Revision's name is permanent once a build that sells it ships.

Past the first 27, samples buy tier III, and heat opens it: each new level cleared puts one more
Revision on the bench, and the Foreman pays one more sample per level. **Undecided** — what samples
buy once tier III is spent too, at about 57.

**Assumed** (the heat, `docs/design/pressure.md`; not reviewed by you):
- The dial is gated at one past the record, and the status line shows the dialled level. Free now;
  once shipped without the gate, adding it strands records.
- Tempered: every machine ×1.3 health, the Foreman too, from heat 1 (Breakers and Surveyors 4,
  Welders 5, marked machines 8 and 10, the Foreman 39). One constant.
- Hot shelf: every room one ramp step later, with a fifth step past room 4's for room 4 and the
  yard (chase 6.0, cooldown 0.7, two attackers, 0.4 s breath). One row.
- Thin repair: a crystal heals 2, and SALVAGE repairs once a room. Two constants.
- Marked waves: the marked machine takes a Breaker's place in every wave of rooms 2 to 4, at every
  beacon stage. A rule change to undo; adding one instead would need a sixth spawn point.
- Foreman's temper: 45 health (not 58), and every cooldown ×0.85 on top of the phases'. Two
  constants.
- The payout: +*n* as one pickup, paid on the kill, tallied on its own row.
- Tier III: the five effects, their order and costs 5 / 8 / 6 / 6 / 5. The save names Flux, Anneal,
  Preheat, Quench and Braze are permanent once a build sells them. Preheat raises Reserve's module
  rather than adding one; Quench acts after Cladding; Braze takes its 15 from Module.
- The win rates and hit counts in `pressure.md` §2 and §3.6 are illustrative, not measured.

**Inside a run: modules**. When you walk through the exit of the first room, and of every
module or marked room after it, the world holds and you are offered two of eight upgrades —
take one or skip — before the next room fades in. The offer sits at the exit rather than on the last hit because the take button is also the
attack button. There is no offer after the boss. Each card is labelled with the button it
changes — CHAIN, SPECIAL, DASH, HULL — and reads true for either weapon: Momentum works on the
last hit of whichever chain is in hand, Reach and Shock on whichever Special. Edge and Tempo
also reach the lance's spin; Momentum, Reach and Shock do not (it is neither the last hit nor
a Special).

**Assumed** — Edge multiplies chain damage by 1.6, not a quarter: damage rounds to the nearest
even whole number at .5, so ×1.25 turned every 1 back into 1 and did nothing; ×1.6 makes both
weapons' chains 2/2/3, and its card says so: "Chain hits deal 2, the last one 3". The
categories were BLADE and NOVA. Not reviewed by you.

**Rarity, Refit and synergies** (decided 2026-09-27). Modules come in three **rarities**:
Common, Rare and Epic. The card shows the rarity by colour (copper, mint, gold) and by the word
RARE or EPIC before its button. Edge's chain goes from 2/2/3 at Common to 3/3/4 at Epic on the
sword (3/3/5 on the lance). The first room never deals an Epic, and marked rooms deal rarer
cards. A **Refit** — a copper kit on a Refit room's floor — stops the world and raises one held
module a step; Plating heals the extra points. Four pairs are **synergies**, live while both are
held at any rarity: **Aftershock** (Momentum + Shock: the finisher's stun reaches every enemy
within 1.5 m of what it hit), **Ram** (Blink + Strike: the strike out of a dash stuns for half a
second; its damage is unchanged), **Cadence** (Tempo + Blink: a dash mid-chain keeps your place
in it) and **Salvage** (Reach + Plating: a Special that hits repairs 1, up to twice a room — once from heat 3). A card
that would complete one says so on a third line, `+ AFTERSHOCK with MOMENTUM`, and the HUD's
module row names the live ones after the modules. The Fabricator remarks, once ever, on the
first two doors, the first marked room cleared, the first pair and the first Refit.

**Assumed** — rarity weights 75/25/0 at room one's offer and for Reserve, 70/25/5 in module
rooms, 40/45/15 in marked rooms; every tier's numbers (`docs/design/doors_and_depth.md` §3):
Edge ×1.6/×2.2/×2.6, Momentum knockback ×1.5/×2/×2.5 and stun 0.5/0.7/0.9 s, Tempo
×0.8/×0.7/×0.6, Reach ×1.25/×1.4/×1.6, Shock 0.6/0.75/0.9 s, Strike 0.12/0.2/0.3 s, Blink
0.25/0.22/0.19 s with Lattice multiplying it afterwards and a 0.16 s floor, Plating +3/+5/+8.
Reach is now a scale, so its Common ring shrinks from 2.8 to exactly a quarter further (2.75 m).
A dash-strike through Strike's window takes no chain slot (the chain goes on from where it
was). Ram stuns rather than deals 2, which was faster and safer than the chain; Salvage is
capped at two a room, which uncapped out-healed an enemy's hits. Edge never touches the
dash-strike. Retention keeps the rarity the first module was **taken** at, never what a Refit
made of it. The Refit is a choose-one panel. The tier colours; Common prints no rarity word.
Not reviewed by you. The numbers are the
sim-designer's; the wording is in `docs/design/depth_text.md`. **Undecided** — whether weapons
get their own module pools. Blocks nothing yet.

The goal is stated at the start: **the beacon needs twelve Core Samples; the shelf holds
three a run, a fourth behind a marked room once the beacon is lit — four rooms and a Foreman down.**

**Assumed** — the death card shows the room reached and the samples carried (the spoken lines
no longer name numbers); the draft count is not shown anywhere now. **Assumed** — the
Fabricator's speaking rules: on the very first arrival all four opening lines
play before the death or ascent lines; a death plays a killer line then a haul line, and a
beacon stage line replaces the haul line when a stage is crossed; a kill with no known attacker
counts as a Breaker's, and any kill in the Foreman's room as the Foreman's; the shop's
introduction plays once ever; Retention brings back the same first module every run once owned;
Reserve draws with the run seed. **Assumed** — the save is JSON in PlayerPrefs (IndexedDB on the web) with a version field for
migration; the Revision list and costs; beacon stages 3/6/12; the Foreman's sample; two-of-eight
modules at room one's exit and at every module and marked room's. Not reviewed by you. **Undecided** — whether the beacon does what the
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

A hit on her reddens the rim of the screen (the enemy red-orange), which drains in about a
third of a second; while she is critical the rim holds a faint red that breathes with the
ring. A Breaker's swing — a marked one's too — and each hit of the Foreman's combo draw the
same arc hers do, in the hostile violet, over the wedge that warned of it and under hers;
the lanes (Welder, Surveyor, the Foreman's fan and lash) are their own picture and get none.
Going through a door wipes the screen black from that door's side — the west door left to
right, the east right to left — and the next room is revealed carrying on the same way; a
room with one door, the climb, a death and the Landing keep the plain fade.

**Assumed** — the hurt rim is a screen overlay rather than the post-processing vignette
(cheaper on the web build); its peak 0.55, 0.35 s drain, and a muted 0.09 ± 0.03 critical idle breathing at 1.2 Hz (was 0.18 ± 0.08 at 3 Hz; the user found the flash too constant);
the enemy arc's violet `#E45BFF`; the wipe on room-to-room transitions only, 0.25 s each way.
Not reviewed by you. All are numbers or single call sites — cheap to change.

Still deferred: a directional wipe *along* the swing arc itself — it needs per-vertex alpha, which
the WebGL material path drops (the arc is tinted through `_Color` for that reason).

## Enemies and pressure

Three kinds: a chaser, a caster and, once the beacon is lit, a denier. The room works because
of the mix: the chasers make you move, the caster punishes moving badly, and the Welder takes
away floor to move on.

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

**Welder** — the denier, in rooms three and four once the beacon is lit. It is slow and never
chases: it holds four to seven units away. Its tell is the longest of the three. For 0.8 s it paints
a lane 4 m long from its own feet toward you, then walks that lane with its torch down and leaves a
burning **seam** that lasts three seconds. At its working distance the lane stops short of you,
because it is not aiming at you: it is cutting the floor between you and itself. Get closer than
four metres and the lane goes through you. Touching the seam costs 1, so walking across it costs 1
and dashing across it costs nothing. It blocks nothing, and other machines cross it unharmed. The
Welder's body hurts (1) only while it walks. It has four health: one full chain.

At most two enemies attack at once, of any kind; the rest hold at arm's length and
circle, so there is always one tell to read. A Welder's turn lasts from its tell to the end of its
walk. Stagger is rationed: a few quick hits stun, then the enemy shrugs the next ones off and
finishes its swing through them. A stunned Surveyor drops its charge. A stunned Welder drops its
tell, or stops its walk where it is, and what it has already laid keeps burning. Seams go out when
the room is cleared.

**The opening is fair by rule.** Nothing moves until the player has pressed something, and
then there is a beat (1.5 s) before any enemy leaves its spot; each new wave gets the same
beat as it materialises. An enemy's first swing always comes after a chase, never off the
spawn. On a phone held upright the "turn sideways" prompt holds the world. The first wave
is two Breakers and the second three; the Surveyor arrives in room two, and the Welder, once the
beacon is lit, in room three.

Death holds the frame for a beat, then the arena reloads fresh. Clearing the room brings the
next wave after a short pause.

The enemies are animated from rendered sheets like the player: idle, move, one attack whose
wind-up/strike/follow-through frames are stretched over the attack's real timing, hit, death.
The Welder's attack is 16 frames, split 6 / 6 / 4 over tell, walk and recovery, so the walk (the
part that moves) runs at 10 fps. **Assumed**; one re-render.

**The Breaker skates** (decided). It keeps its chase speed (5 u/s base, 4.5 to 5.5 by room).
Its move clip, an 8-frame bound at 12 fps, cannot stride the 3.3 m a cycle that speed needs. The
feet are clear of the floor for most of the cycle, which hides the slip rather than closing it. It
is a machine, and something that heavy moving that fast reads as driven rather than walked: a
thing that runs you down, not a bruiser you outrun.

**Pacing ramps.** Playtesters said room one went from nothing to everything at once, and that
the pace would suit a later room — so it does. Room one is two then three Breakers at 4.5 u/s
with one attacker at a time and a slow attack cooldown; the Surveyor arrives in room two with
two concurrent attackers; by room four Breakers run at 5.5 with the fastest cooldown. All of it
is one table (`RunPlan.Difficulty`).

**Assumed** — the numbers: Breaker chase 4.5 → 5.0 → 5.0 → 5.5 across the rooms, attack cooldown
1.3 → 1.0 → 0.9 → 0.8 s, concurrent attackers 1 then 2, wave grace 1.2 → 0.8 → 0.8 → 0.6 s;
waves as in The run; base Breaker chase 5 vs run 5.5, wind-up 0.28 s with the lunge in its
last 0.14 s at 18 u/s, active 0.10 s; Surveyor band 6–9 u (retreat under 5, approach over 10,
backing off at 3.2 × the room's pace),
charge 0.6 s, bolt 9 u/s with 11 u of reach, cooldown 2.2 s; damage 2 of 10 for both; two
concurrent attackers; stun budget 0.9 s per 2.5 s; first-gesture grace 1.5 s. Not reviewed by you. All builder consts in one
table in `ArenaSceneBuilder`; the lanes' reach follows the lunge and the bolt automatically.

**Assumed** — the Surveyor backs off at 3.2 (was 4; 2.9–3.5 across the rooms) and its hurtbox is a
0.6 m circle (was 0.45, the Breaker's), to match the hovering body's footprint. An enemy tweak
made alongside the slash, outside your "improve the slash, leave Nova" ruling: at 4 it outran
the chain's step, and at 0.45 players aimed at the body and missed the circle under it. Its band,
wind-up and bolts are unchanged; incidentally Nova now reaches a Surveyor 0.15 m further. Not
reviewed by you; one builder const each (`docs/design/slash.md` §5).

**Assumed** — the Welder's numbers:

- 3.0 u/s at room three's pace, scaled as the Surveyor's is. Band 4–7, retreating under 3 and
  approaching over 8.
- Tell 0.8 s, with the aim locked at its start.
- Lane 4 × 0.6 m, clipped at the first blocker, with no attack if the lane is under 2 m.
- Walk 0.6 s, recovery 0.4 s, cooldown 3.5 s.
- Seam: 1 damage with a 0.5 s re-hit lock, lasting 3.0 s and dimming over its last half second. It
  hurts only her.
- Body: 1 while walking, and it shoves her sideways out of the lane.
- 4 health. Its turn holds an attack slot for 1.4 s.

Not reviewed by you. They are one table (`docs/design/welder.md`).

**Elites** (decided: in marked rooms, once the beacon is tuned). One machine in a marked room's
last wave is **marked**. It has twice the health, is 15 % larger, telegraphs 20 % faster, glows
bright violet and carries its name over it: MARKED BREAKER, MARKED SURVEYOR or MARKED WELDER. Its
kind comes from the run seed: Breaker, Surveyor or Welder, at even odds. **Assumed** — the numbers;
the last wave rather than the first; the marked Breaker's lunge running faster so that its wedge
still covers the same 3.5 m. Size scales the body and the hurtbox, never the hitbox or its wedge.
Not reviewed by you; `SpawnMods` constants.

**Undecided** — what the machines were: the wreck's crew, the world's own machinery, or the
AI's earlier attempt. All three fictions below are written to survive either answer.

**Assumed** — the naming rule: the world's machines are named for the work they were built
to do, one or two words, no honorifics (Breaker, Surveyor, Foreman; free slots that already fit:
Cutter, Hauler, Dredger, Rigger, Welder). Not reviewed by you. Cheap to change now, expensive
after a dozen enemies, items and barks are written against it. In code they are still
`Brute`/`Sentry`/`Warden`, and the Welder is `Welder`. Only the boss and a marked machine show their
names on screen.

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

**Welder.** It closed what the Breakers opened. Once the Foreman had taken what was fit to go
up, the Welder ran a seam along the plate and sealed the hull behind it. There are no hulls left
to close, so it welds the floor: it paints a line from its feet toward whatever is standing,
walks it with the torch down, and leaves the seam burning behind it. The ground it has worked
is the ground to stay off.

**The Fabricator** was the expedition's manufacturing intelligence. Its crew is gone; it
stayed at the Landing, and it wants to go home. It builds Units from what the shelf gives back
— you are the latest, its Tender — and sends each down for Core Samples. A lost Unit's samples
and memory come up into the next one: every loss is a draft. The samples feed the Ascent
Beacon, which lights at three, tunes at six and fires at twelve to call the fleet home. Warm,
dry, patient, a little too fond of you. Nothing the Foreman has marked has gone up in a long
time; the Fabricator sends Tenders down to fetch what the Foreman will not pass.

**The answer.** The shelf hears the beacon before anything else does. Once it is lit, the
Foreman marks rooms again and sends out a Welder; once it is tuned, it marks machines too.
When it fires it is answered within the minute — the fleet's acknowledgement, on the fleet's
frequency, from under the shelf rather than the sky. The Fabricator reports exactly that, and
says nothing about what it thinks sent it.

**Assumed** — what answers at twelve, and that the answer comes on the firing visit. Not
reviewed by you. Cost to change: seven strings, five of them voiced (`docs/design/beacon_text.md`).
**Undecided** — what sent the reply, and whether the beacon does what the Fabricator says. It
never lies; it chooses what to say. Affects: the next arc, and the objective card after firing.
**Assumed** — "the crew went down the shaft" is the Fabricator's line, not the doc's fact; the
wreck on the shelf may or may not be its ship.

**Undecided** — the AI that sent the robot (voice, motive, whether it can be trusted),
what the earlier expedition was, and what lives here. `world-builder` owns these next.

## Art direction

Isometric-feeling 2.5D in the manner of Hades. The game camera is tilted 35° over the flat
physics plane: the floor recedes with a visible plate grid, walls stand three metres tall
along the top and sides, and props and characters are upright sprites sorted by depth.
Everything is modelled and toon-rendered in Blender — flat things straight down, upright
things from the same 35° camera — and placed as sprites; gameplay keeps honest 2D footprints.
The palette is saturated, Tartarus transposed to sci-fi: a cold ash floor under green alloy
ruin, copper machinery, violet energy, mint crystal glow (2026-09-28: the floor went from alloy
green to ash so that hue itself says what blocks — everything that stands is metal or rock,
everything walkable is ash). A static floor lightmap darkens the walls and corners and pools
light at the centre of every room; each room seeds its own pool and one hero floor seal, and
dresses its side walls. Red belongs to enemy harm (telegraphs, and the Welder's burning seam) and blue to the
player; scenery is forbidden those hues. Details: `docs/ART.md`.

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

**The Foreman's yard has its own** loop and its own bed (a heavier trance loop, a steam-boiler
bed), crossfaded in under the black as the room is built. **The beacon has a voice**: a hum
under each stage change, and a chime as it fires, before SIGNAL SENT, with the music ducked
6 dB for the chime's length. **The small moments have cues**: a sample and a repair crystal
each sound different on pickup (a crystal's heal is not sounded twice), a sting on the frame a
room is cleared, the door panels sinking (one sound for a pair), a heal by any other route, an
enemy arriving (one sound per wave), and a heartbeat that loops quietly while she is critical
and stops the moment she is healed over the line, dies or reaches the Landing. Every cue is
silent rather than broken when its clips are missing.

Clips are found by file-name prefix, so a new or replaced clip needs no code: `pickupSample`,
`pickupRepair`, `roomClear`, `doorOpen`, `heal`, `spawn`, `beaconHum`, `beaconFire` under
`Assets/ThirdParty/Kenney/Audio` or `…/Kenney/AudioPolish`; `heartbeat` there or under
`Assets/ThirdParty/OpenGameArt/Heartbeat` (the first by name is the loop); `music_boss*` and
`ambience_boss*` under `Assets/Audio/Music` and `Assets/Audio/Ambience`.

**Assumed** — one loop per place rather than layered or adaptive music; the crossfade and duck
times; CC0-only sourcing for the tracks and cues (recorded in `Assets/ThirdParty/ATTRIBUTION.md`);
the heartbeat at 0.3 volume (ceiling 0.35); the beacon's 6 dB duck; which moments get a cue.
Not reviewed by you. Each is a clip swap or a number — cheap to change.

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
