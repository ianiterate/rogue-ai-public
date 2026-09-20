# rogue-ai — design

**What the game is right now.** Present tense, no history. Run `/open-questions` to
see everything currently waiting on a human call.

> This file is published verbatim to the public play repository on every web deploy
> (see the deploy pattern in `SETUP.md`). Write it knowing players may read it.

---

## Premise

A fast, top-down action-roguelike. Combat flow in the family of Hades and Cult of the
Lamb: short commitments, dash as the answer to everything, readable enemy telegraphs.

**Undecided** — setting, tone, factions, who the player is and why.
The working title suggests AI as subject matter; nothing is established.

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

**Assumed** — the post-dash dash-strike window (attack pressed just *after* a dash still
counts as a dash-strike) ships disabled, and the third hit's short uncancellable tail is off.
Not reviewed by you. Both are single fields to turn on after a play-test.

## The run

**Undecided** — what a "run" is, how long it lasts, how it ends.
Affects: save format, difficulty pacing, session length, UI.

**Undecided** — permadeath, or resumable from a checkpoint.
Affects: save format and versioning, tuning, how punishing failure can be.

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

Chasers run just under the player's speed, so distance is earned, not free. An attack is a
committed lunge: the enemy locks its aim, paints its landing lane on the floor, and in the
last part of a short wind-up launches along it. Walking straight away does not escape it;
stepping out of the lane early does, and a dash through the strike does.

At most two enemies attack at once; the rest hold at arm's length and circle, so there is
always one tell to read. Stagger is rationed: a few quick hits stun, then the enemy shrugs
the next ones off and finishes its swing through them.

Death holds the frame for a beat, then the arena reloads fresh. Clearing the room brings the
next wave after a short pause.

**Assumed** — the numbers: chase 6 vs run 7, wind-up 0.28 s with the lunge in its last 0.14 s
at 18 u/s, active 0.10 s, damage 2 of 10, two concurrent attackers, stun budget 0.9 s per
2.5 s, five enemies per wave. Not reviewed by you. All inspector fields; the wedge's reach
follows the lunge automatically.

## World and fiction

**Undecided** — see Premise.

## Art direction

2D, top-down, Universal Render Pipeline (2D renderer). Placeholder solid-colour shapes
until a direction exists.

**Undecided** — visual style, palette, whether sprites rotate or flip.
Affects: how much of the art can be sourced (`asset-scout`) versus authored
(`blender-artist`), and animation approach.

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
