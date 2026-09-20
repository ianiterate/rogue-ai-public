# Art direction

Read `GAME.md` first; this file holds the specifics.

## The method

Hades is 2.5D: three-dimensional props and characters rendered from a fixed, tilted camera
over painted floors, with gameplay on the flat plane. We do the same, rendered ahead of time.
Everything is modelled in Blender, toon-shaded, rendered from **one orthographic camera
tilted 35° from vertical**, and placed in Unity as a sprite. Objects are pre-scaled by
1/cos 35° in depth before rendering so the ground plane maps exactly 1:1 to world units; a
sprite's pivot sits at the centre of its footprint, so where the sprite is placed is where
its collider is. Colliders are authored shapes, never traced from the sprite's silhouette —
the tilted silhouette of a tall prop would otherwise stop attacks on thin air.

Sprites are 128 pixels per metre (the floor 64). Shading comes from a node graph rather than
lamps — normal against a fixed key direction through a stepped ramp — so renders are
deterministic and identical between EEVEE and Cycles. Outlines are an inverted hull 0.018 m
thick. A contact shadow is rendered as a second pass and composited under each prop; that
shadow is what makes things sit *in* the floor rather than on it.

All scenery is our own work; nothing here needs attribution.

## Biome 1 — Rustwater Shelf

| Role | Colour | Rule |
|---|---|---|
| Floor base | `#2A2431` | most of the screen; L\* 14–26 |
| Floor accent | `#3E3446` | drifts, terraces, worn paths; never more than a quarter of the floor |
| Wall | `#574652` | inner shadow falls to `#1A151F` |
| Glow | `#7BE8A4` | crystals, live machinery, all 2D lights |
| Hazard | `#FF5A3C` | **the enemy danger wedge only** |
| UI-safe | `#EDE7F2` | HUD text, bar highlights |

Key light `#FFF2D8` from the upper left (azimuth 135°, elevation 55°); fill `#4A5A96` at 0.3
from the lower right; rim `#7BE8A4` at 0.22 from up-screen; contact shadow `#14101A` at 0.55,
offset 0.10 m down-right, blurred 0.12 m.

## The legibility contract

The dressing must never cost a combat read.

- Scenery saturation stays at or below 35%. Actors sit at 60–80%. Saturation means "this
  moves and can hurt you".
- Nothing on the floor wider than 3 m is brighter than L\* 40. Only crystals exceed the
  UI-safe brightness, and they are small and static.
- Red is reserved for the danger wedge; blue for the player and the slash arc. Glow is green.
- The wedge at 0.55 alpha must be unmistakable over the darkest floor accent; the arc must pop
  over the brightest one. Check both on every new biome.

## Layering

Floor −100 · floor decals −90 · walls −70 · ground props −50 · spores −40 · danger wedge 2 ·
enemies 5 · player 10 · aim pointer 15 · slash arc 20 · sparks 25 · health bars 30 · HUD 50 ·
touch controls 100. Tall props draw at 12, above the player, and fade to 45% while the player
stands behind them. There is no Y-sorting: it cannot coexist with the fixed actor ladder, and
the fade is what Cult of the Lamb does visually.

## Light and post

A global multiply light at 1.0 leaves the painted floor exactly as rendered. Every other light
is additive — crystals, the vent, a small lamp on the robot — so lights can only add glow.
No 2D shadow casters: the baked contact shadow does that job for a fraction of the cost.
Bloom and vignette come from the URP volume; bloom is the single largest frame cost on mobile
and is switched off by the low-FX setting. The camera is confined to the 30×18 painting.

## WebGL caveats

iOS Safari has no DXT, so textures decompress to RGBA32 there: the floor is capped at 2048 px
and the biome budget is 6 MB of PNG, which becomes roughly 22 MB of texture memory in the
worst case. Headless EEVEE needs a GPU context; the render script falls back to Cycles CPU,
which the emission-only shading makes equivalent.
