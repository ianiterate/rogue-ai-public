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

## Light and post

One global light at 1.0, nothing else: painted values stay exact and every glow is paint in the
sprite. Additive point lights and bloom were tried and removed — soft pools pulsing over
painted art read as noise and added nothing to the Hades look, which is painted glow, not
real-time glow. Vignette stays at 0.22. The camera is clamped so the visible floor never
leaves the 26×16 plate field.

## WebGL caveats

iOS Safari has no DXT, so textures decompress to RGBA32 there: every texture is capped at
2048 px and the biome budget is 6 MB of PNG. The tiling floor is the reason that budget is
comfortable. Headless EEVEE needs a GPU context; the render script falls back to Cycles CPU,
which the emission-only shading makes equivalent.
