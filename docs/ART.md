# Art direction

Read `GAME.md` first; this file holds the specifics.

## The method

Hades is isometric 2.5D: painted, tilted rooms in which props and characters stand up. We do
the same over a flat 2D physics plane. **The game camera is orthographic and tilted 35° from
vertical.** Anything that lies on the floor — the floor itself, decals, blob shadows, the enemy
danger wedge, the slash arc, the aim pointer — is a flat sprite or mesh that the camera
foreshortens. Anything that stands — walls, props, the robot, the drones, health bars — is an
upright billboard rotated to face the camera, with its pivot at the bottom centre of its
footprint so the feet sit exactly on the collider. Colliders are authored shapes, never traced
from a silhouette. Depth order comes from Y: every upright in-world sprite shares one sorting
order and the renderer sorts them along the Y axis, so a robot behind a pillar is drawn behind
it and in front of it when in front. There are no Z offsets anywhere: under the tilt, Z is
screen rise.

The camera is orthographic rather than perspective on purpose. Hades' read comes from
foreshortening and upright walls, not from vanishing points, and orthographic keeps billboards
stable, pivots on colliders and floor effects position-independent.

## Geometry first

The room's structure is real geometry, not sprites: perimeter walls are three-metre boxes,
pillars are columns, and placeholder obstacles are boxes and prisms, all built by the scene
builder and coloured in the palette (lit top face, front face, shadowed sides). Under the tilted
camera every one shows its top edge and its front face, so height reads without any art at all.
Height runs along −Z, toward the camera; the physics plane stays XY. Drawn sprites — props,
characters, later wall dressing — sit on top of that structure as billboards.

## Rendering

Two Blender rigs, selected per sprite:

| | flat | billboard |
|---|---|---|
| camera | straight down | tilted 35°, no pre-scale |
| used for | floor plates, decals, blob shadow | walls, props, characters |
| pixels per metre | 64 | 128 |
| pivot | centre | bottom-centre of the footprint: `y = (frame/2 − H·sin35/2) / frame` with `frame = (D·cos35 + H·sin35)` padded to a multiple of 4 px |

Shading is a node graph, not lamps — normal against a fixed key direction through a stepped
ramp — so renders are deterministic and identical in EEVEE and Cycles. Outlines are an inverted
hull 0.018 m thick. Upright sprites carry no baked shadow; one shared flat ellipse under each
does that job, because a baked shadow would stand up with the billboard. Every render is seeded
and reproducible; the manifest in `Tools/blender/biome1.py` is the source of truth, and
`Tools/build_assets.sh` installs a kit only when the whole set rendered.

The floor is an 8×8 m sheet of four distinct 4 m plates, tiled over the room so the visible
4 m grid is continuous while the ornament repeats only every 8 m, plus a few hero inlays — the
grid Hades has, at a fraction of the texture memory of a single painting.

All scenery and characters are our own work; nothing here needs attribution.

## Biome 1 — Rustwater Shelf

Tartarus transposed to sci-fi.

| Role | Colour |
|---|---|
| Alloy ruin lit / base / shadow | `#3F8F72` / `#2E6B58` / `#17392F` |
| Floor plate / alternate / seam | `#274C41` / `#1E3B34` / `#12241F` |
| Copper machinery / highlight | `#C8762E` / `#F2B15C` (vertical surfaces only) |
| Energy violet / bright | `#C23BD6` / `#E45BFF` (glowing seams, at most 4% of floor pixels) |
| Mint glow | `#7BE8A4` (crystals, all 2D lights) |
| Hazard red | `#FF3A2E` — **the enemy danger wedge only** |
| Player blue | `#7ED0FF` — the slash arc, the aim pointer, the robot's rim |
| UI-safe | `#EDE7F2` |

## The contrast rule

Saturation is not capped. Instead:

- Actors carry the outline and a rim light 18 L\* above their own base, and sit at least 25 L\*
  from the floor behind them.
- Flat surfaces stay within L\* 18–34; ornament brighter than L\* 45 covers less than 8% of the
  floor. Vertical surfaces may reach L\* 55 and 60% saturation — they are at the frame's edge,
  not under the fight.
- Hues 0–25° and 200–230° are forbidden to scenery above 25% saturation: red means a strike is
  coming, blue means you.
- The wedge at 0.55 alpha is checked against both floor plates; the arc against the brightest
  plate and against copper. Per biome.

## What blocks

**What stands up blocks; what lies flat is walkable.** Every obstacle is an upright sprite with
an outline and a shadow on the floor under it; every walkable element — lichen, grates, cables,
cracks, inlays — is painted flat into the floor with no outline and no shadow. A player never
has to test a prop to learn whether it is solid. The shadow is the tell: only solids cast one.

## Layering

Floor −100 · decals −90 · wall base shade −85 · cast shadows −80 · contact shadows −79 · danger
wedge −70 · slash arc −60 · aim pointer −50 · wall geometry −10 · drawn wall faces −9 · **props,
player, enemies all at 0, sorted by Y** · spores 22 · sparks 25 · health bars 30 · HUD 50 · touch
controls 100. Tall props still fade to 45% while the player stands behind them.

## Shadows

A shadow centred exactly under its caster is invisible: under the 35° tilt the caster's front
face or billboard covers precisely its own footprint. So every blocker and actor carries two flat
ellipses — a **cast** shadow (1.35 W × 1.0 D, near-black green at 0.55, pushed +0.22 W right and
−0.18 D down, away from the upper-left key light) that pokes out from under the caster, and a
tighter **contact** core (0.9 W × 0.7 D at 0.35) that shows in the gaps around legs and under
the drone. Each wall run has a 0.35 m shade strip along its inside base so the walls sit on the
floor. Never re-centre a shadow on its footprint.

## Perimeter

The wall ring encloses the 26 × 16 plate field exactly: north and south runs span x ±13, the
side runs y ±7.5, corners are the end segments of the side walls — full 3 m at the north corners,
lip height (0.4 m) at the south so nothing occludes the player. Only void (`#100D14`) lies
beyond. The artist's north faces and the south lip faces are billboarded over the geometry; the
side walls are geometry only until side faces exist at the 15 m length.

## Characters

The player is an animated humanoid android on the Quaternius universal rig (UAL2 Female
Mannequin, CC0, `Tools/blender/src/Quaternius/UAL2/`), re-proportioned in Blender to a
Bayonetta silhouette — long legs, narrow waist, long neck, heeled stance, helm-hair swept up,
a visor slit, an energy blade — and rendered through the same rig as everything else: 35°
billboard camera, toon steps, 0.018 m inverted-hull outline, `actor_rim` per frame. Attitude
comes from silhouette and pose, never from anatomy; it has to read at 128 PPU.

**Sheet contract.** One PNG per part under `Sheets/`, uniform frames packed row-major, and a
sidecar with `sheet: true`, `frame_px`, `columns`, `frames`, `part`, and `clips` — each clip
`{start, count}` plus either `fps` + `loop`, `stretch: true` (the clip spans the state's
duration: dash), or `phase_frames: [startup, active, recovery]` (attacks; each phase's frames
are spread over that phase of the `AttackProfile`). Every frame has the same footprint pivot,
computed by `pivot_of` from the character's 1.0 × 0.8 × 1.8 m box, and the camera never moves
between frames so the feet never swim. v1: 56 frames — idle 8 · run 12 · dash 5 · hit 3 ·
death 8 · attack1 6 · attack2 6 · attack3 8 — one direction (+X = screen-right), mirrored
with `flipX`. The importer slices the sheet from the sidecar; the builder wires the frames
and the clip table onto `SheetAnimator`; `ClipSampler` (Sim, tested) picks the frame.

**The frame is bigger than the footprint.** A prop's frame is its footprint; a character's is
not, because a 1.1 m blade leaves the collider in every direction. v1 frames are **256 × 288
px** against a footprint that would derive 128 × 216 — measured, not chosen: across the 56
frames the art spans 2.00 m sideways and 2.19 m up-screen. `cam_lift` (0.105 m) re-centres the
camera on the art rather than on the collider and `pivot_of` reads the same number, so the
pivot stays exact (0.5, 0.2239). 8 columns × 7 rows puts the sheet at **2048 × 2016**, inside
the WebGL cap on both axes — and it is that cap, not the art, that fixes the frame width at
256, so a blade at *full* extension does not fit. The strike frames are sampled a frame and a
half off peak extension for that reason; widening the frame would cost a column and blow the
height cap. Margins are 2.8 px sideways, 4.4 px below and 11.6 px above: re-pose the character, or
resize the helm, and re-measure before assuming there is room.

**Three of the eight clips are constructed.** The UAL2 pack in the repo is a 43-action subset,
not the full 120+. Every one of the 43 was measured for stride amplitude, leg antiphase, ground
contact and looping: exactly two are forward locomotion cycles, `Walk_Carry_Loop` (antiphase
−0.96) and `Zombie_Walk_Fwd_Loop` (−0.68), and **both are walks**. There is no sprint, no jog
and no death in the pack at all.

- **run** — `Walk_Carry_Loop`'s lower body, stride amplified, the carry's backward pelvis pitch
  cancelled, the lean taken at the waist so the back stays straight, arms driven off the posed
  thigh angle so they stay opposed by construction, and the blade angled back at the wrist. The
  part that makes it a run rather than a stretched walk is **knee drive**: the knee is folded in
  proportion to how far the thigh is from vertical, which gives knee-up in front and heel-up
  behind. Amplifying the hip swing alone exaggerates a walk's *reach*, which is the opposite of
  what a run does, and it looked like diving.
- **death** — `LayToIdle`, a get-up from supine, sampled backwards.
- **hit** — `Idle_Shield_Break`, not `Hit_Knockback`: that clip has no flinch in it, being
  doubled over on its first frame and airborne by its third, so every sampling of it gave a
  knockdown identical to the death.

The **idle** is half constructed too. `Idle_No_Loop` measures 3° of stride and 7 mm of hip
travel over its whole length — it is a held pose, so a breath cycle is authored on top of it:
chest, shoulder and weight-shift terms on one period that closes over the eight frames, and
never on the pelvis, because the legs hang off the pelvis and the feet have to stay welded to
the footprint pivot. It contributes +2.2 px of chest rise on top of the action's 1.4.

If a future pack adds a real run, death, hit reaction or idle, these are the entries to replace.

**Enemies use the same contract**, smaller: 32 frames, 8 × 4 — idle 6 · move 8 · attack 10
(`phase_frames` brute [4, 2, 4], sentry [6, 1, 3]) · hit 2 · death 6 — one per type
(`biome1_actor_brute_sheet`, `biome1_actor_sentry_sheet`). Each enemy follows the player's
process, not the player's rig: a CC0 model that already *is* the thing — rigged and shipped
with its own clips — restyled and re-proportioned in Blender and rendered through the
same 35° toon rig, in the hostile palette: alloy, violet emissives, `RIM_HOSTILE`; never the player's blue or the wedge's
red. The brute's lunge is the last wind-up frames; the sentry's bolt spawns on its release
frame. The bolt itself is a one-frame billboard (`biome1_fx_bolt`, 0.6 × 0.3 m), flat-projected
at 128 PPU so its pivot is the centre — a projectile turns about its middle.

**Both are cast from the OGA Robot Enemy Pack** (CC0, `Tools/blender/src/OGA/RobotEnemyPack/`),
surveyed headless into `Tools/blender/out/oga_survey.txt` before anything was built. The
casting does not follow the source files' names, because two of the six are not what their
names suggest: `rocket.blend` is a 158-triangle cylinder with no armature — the *projectile*,
not a robot — and `roller.blend` is a featureless ball whose attack keys two bones.

- **Breaker** ← `lobber`. The heaviest, most grounded chassis in the pack: six legs, 39 bones,
  and the only three-segment articulated arm, which is what can carry a guard plate and swing it.
  Its gun tube is cut down — the fiction says it no longer carries the tool.
- **Surveyor** ← `blaster`. The only legless body in the pack, so it hover-drifts and has no
  foot contact to slide. Its attack already *is* charge → release: the arm extends and holds
  (speed 0.000 at source frame 16), then snaps at frame 22.

`oga_rig.py` holds everything true of the pack — import by append, the +90° facing turn (every
robot in it faces −Y, measured from the `eyetarget` bones), the holder Empty that carries scale
and seating where no action can reach it, the IK helpers, and the outline. `oga_breaker_rig.py`
and `oga_surveyor_rig.py` hold the characters.

**Three things the pack forced, all of them stated rather than hidden.**

- **The outline is 0.011–0.013 m on these two, not 0.018.** An inverted hull cannot be thicker
  than half the thinnest thing it wraps: offset two faces of a plate by more than half its
  thickness and they swap sides, the shell turns inside out, and only its backfaces are drawn,
  so the sheet fills with black slivers thrown clear of the silhouette. A 2019 hard-surface
  model has shins, antennae and struts between 2 and 4 cm at this scale. The thin parts are
  thickened where that also helps the read — a 3 cm leg is 4 px and reads as wire — and the
  hull is matched to the model for the rest.
- **Both carry a depth squash** (0.70 brute, 0.62 sentry) on the axis the camera never sees
  end-on. Under the 35° tilt depth is screen height, and the footprint pivot pins the frame's
  bottom edge 0.504 m below the standing line for *every* actor; a wide-hipped walker at 1.4 m
  does not fit that. Growing the frame to 320 px was measured and rejected — with the pivot
  held at 0.224 it sends 78% of the new pixels to the top, where 0.7 m is already going spare.
  The cost is an outline thinner by that factor on camera-facing edges only.
- **The scale is solved from the standing frames alone**, and the lift from every frame.
  Coupling the two makes the character's size depend on its death: tucking the Breaker's
  collapse in raised the global minimum, shrank the denominator, scaled the whole robot up, and
  pushed the *idle* back out through the edge the tuck had just pulled it inside.

**What is constructed.** The Breaker's move is a **bound**, not the pack's walk: the source
tripod gait moves its feet 1.53 units, a 0.193 m stride, and the contract's 8 frames at 12 fps
under a 5 u/s chase asks for 3.33 m of ground per cycle — seventeen times that. A walk cannot
be amplified across that gap, so the legs gather, the body surges, and the feet leave the floor
for most of the cycle, which is both the only gait that covers multiple body lengths and the
only one whose feet cannot be *seen* to slide. It does not close the gap; see the open question
in GAME.md. Its idle is a weight-shift authored over a held pose (the source Body bone moves
0.00 units vertically across 91 frames), and its strike is an authored downward arc, because
the source swing is a pure horizontal thrust — 3.2 units forward, 0.02 of height. The
Surveyor's move is the source drift **levelled from 45–50° to about 15°**; at the source angle
the sensor head points at the floor. Its death is `die3` recentred on its own artwork, because
that clip pivots the body 83° about its base and a 1.4 m body laid over sweeps its top out of a
2.0 m frame — rotation, which cancelling drift cannot touch.

**Measured on the shipped sheets**, against the L\* 26.7 plate: Breaker median 58.7 (**+32.0**),
rim band +19.6; Surveyor median 58.6 (**+31.9**), rim band +18.2. Violet is 1.2% and 3.8% of
opaque pixels against an 8% budget. Both sheets are 2048 × 1152, 32 frames, no empty frames, and
their pivot is 0.22390276 — the same float the lady's is, because `_actor_lift` re-solves each
`cam_lift` from her camera height rather than restating it as a constant.

**Modular by construction.** In Blender the character is separate objects on one rig —
`body`, `hair`, `outfit_top`, `outfit_bottom`, `weapon`. `--parts` renders each part alone
with the others as holdouts, so per-part sheets composite correctly in any stacking order;
the runtime stacks one `SpriteRenderer` layer per part with a tint. v1 renders all parts
into the single `body` layer. Non-loop clips hold their last frame; death runs on unscaled
time because `GameFlow` freezes the clock.

## Light and post

One global light at 1.0, nothing else: painted values stay exact and every glow is paint in the
sprite. Additive point lights and bloom were tried and removed — soft pools pulsing over
painted art read as noise and added nothing to the Hades look, which is painted glow, not
real-time glow. Vignette stays at 0.22. The camera is clamped so the visible floor never
leaves the 26×16 plate field.

## WebGL caveats

iOS Safari has no DXT, so textures decompress to RGBA32 there: every texture is capped at
2048 px and the biome budget is **8 MB** of PNG. It was 6 MB and the player sheet moved it: one
2048 × 1960 sheet is 1.9 MB of the kit's 4.7 MB on its own, and a second character will be
another. The tiling floor is the reason there is still headroom. Headless EEVEE needs a GPU
context; the render script falls back to Cycles CPU, which the emission-only shading makes
equivalent.

## The player's surface — and why it is not black

The android was gloss black for one pass and it did not survive contact with the game's own
scale. A 1.8 m figure is about 130 px tall under this camera; a hull whose three toon steps
all landed between L\* 7 and L\* 27 had no readable interior, and what reached the screen was
a silhouette holding a sword. The hull is now a **dark gloss slate** (`#444A58`) whose steps
are solved backwards from the three values the surface has to hit — base L\* 22 on the flank,
L\* 40 camera-facing, L\* 55 where the key catches shoulders, thighs and the helm crown. The
torso shell is a further step lighter (`#646C7E`, ~L\* 45) so the torso never merges with the
limbs. Measured on the shipped sheet, hull and panel pixels run p05 19 · p25 37 · p50 49 ·
p75 52, which is those steps within a few points — `KEY_COL` is warm and pulls them slightly
under target.

**The contrast rule passes.** Against the L\* 26.7 plate the character measures mean 49.9
(**+23.2**), median 51.5 (**+24.8**), p75 57.8 (**+31.1**), and the black outline sits 26.7
below the floor. The rim band is L\* 69.0, **+19.0** over the surface behind it, against the
18 the rule asks for.

One trap is worth writing down, because it cost a render to find: **`actor_rim`'s strength is
not portable between surfaces.** The lift is added in linear light, so the brighter the
surface the more of it is needed to move the same distance in L\*. The same rim that measured
+22 L\* over the black hull measured **+5.5** over the slate one and had to be re-solved from
0.15 to 0.42. Anything that changes an actor's base tone invalidates its `rim_amount`; the
audit prints the delta, so check it.

Player blue at hue ≈ 200 is about 17% of her pixels and above 25% saturation. The hue ban in
the contrast rule is on **scenery**; blue means the player, and this is the player. The energy
*lines* — limbs, spine, visor — are 5.1% of her opaque pixels against an 8% budget, measured
with the rim band excluded, because the rim is player blue too but is a separate thing the
rule requires of every actor rather than a marking on this one.
