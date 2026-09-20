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

**On-screen controls.** A virtual stick (bottom-left) and ATTACK / DASH buttons
(bottom-right) drive the same actions on touch devices only. Desktop never shows them:
without a gamepad you play on the keyboard. Portrait phones get a full-screen "turn your
phone sideways" prompt instead of the controls until rotated.

Dash cancels the *recovery* of an attack, never its startup or active frames. A short
input buffer (~0.12 s) accepts a press slightly before it can be honoured.

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
