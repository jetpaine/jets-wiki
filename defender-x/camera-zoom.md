---
layout: dx
title: Zoom and moving camera
parent: Camera and coordinates
parent_url: /defender-x/camera.html
summary: The plan for a camera that can pull back and move away from your ship, and the decisions that had to be made before any of it is built.
status: Decided · not built yet
checked: 2026-10-27
---

<div class="glossary" markdown="1">

## Glossary

Read the [Camera and coordinates](camera.html) page first; this one builds on it.

| Term | What it means |
| --- | --- |
| Zoom out / pull back | Show more of the world at once, so everything looks smaller. "3× out" means three times as wide a view. |
| Zoom in | Show less of the world, bigger. |
| Macro-pixel (cell) | One of the big square blocks the game draws with: 4.5 × 4.5 virtual pixels in gameplay. See [Rendering layers](rendering.html). |
| Grid | The fixed pattern of cells covering the screen. Anything drawn "in cells" snaps to it. |
| Sprite | A picture of one object, such as a ship, stored as a small grid of colored cells. |
| Sampling | Deciding a cell's color by looking at the art underneath its center. |
| Downsample | Make a smaller version of a picture by averaging, so detail is merged rather than dropped. |
| Gameplay frame | The fixed, unzoomed view around your ship that the rules (who can be hit, what's "on screen") are measured in. |
| Dolly | A film term: the camera physically moves backwards, rather than the lens zooming. Near things shrink a lot, far things barely. |
| Look test | Building a quick version of two or more options and choosing by eye. |

</div>

## Why zoom at all

Zoom is **cinematic**: the game, not the player, uses it to frame a moment. The first shot planned for the demo: when an enemy holds a firing position on your tail for a moment, the camera eases back to show both of you, holds, and then returns.[^d2] The player doesn't control zoom, so it only has to look good in the moments the game chooses to use it.

For the demo the range is capped at **3–4× out**. Bigger pull-outs (10× or more) would be lovely and are planned for after the demo.

## The central question: what happens to the pixels?

The game's look depends on everything being built from same-size square cells, lined up on one screen grid. Zooming threatens that. There are two basic answers:

- **A. The cells change size.** Zoom out and every cell shrinks. It's like zooming into a screenshot of an old console game. Simple, but at 3× out a cell would be smaller than a real pixel and the retro look would vanish exactly when the screen is busiest.
- **B. The cells stay the same size; objects use fewer of them.** Zoom out and a 40-cell ship is redrawn 13 cells long, on the same screen grid. The look survives at every zoom. This is also how the game already handles rotation, and how the [cutscenes](cinema.html) already zoom.

**Decided: B.** To avoid shimmering when a ship is drawn with fewer cells, the game will keep a few ready-made smaller versions of each ship sprite (made by carefully downsampling the full-size one) and pick the nearest.[^b2]

## The other decisions

Each decision has a short code (D1, D2…) so the design doc and conversations can refer to it.

### D3. Does the world still turn around your ship?

**Yes.** The camera will slide to frame a shot, but the world keeps rotating with your ship's angle, pivoting on your ship. A camera with its own independent angle was dropped as not worth the cost.

### D4. Does gameplay follow the camera?

**No.** Gameplay keeps its own fixed, unzoomed frame around your ship. Otherwise a 3× pull-out would show nine times the area, and every ship in it would suddenly be hittable. The cost: zoomed out, you may see a bullet pass through a ship near the edge without hitting it. That's accepted.

### D5. Split "world cell" from "screen cell"

Today one number, 4.5, means both "how big a sprite cell is in the world" (used by physics and collisions) and "how big a cell is on screen". Under zoom those must separate, or zooming would change the physics. This split is the **first thing to build**, and it changes nothing visible.[^d5]

### D6. Snap the camera to the grid?

If the camera slides by fractions of a cell, objects cross cell lines at slightly different moments and the world seems to shimmer. **Both options will be built**, behind a switch (`CAMERA_SNAP`), and chosen by playtest. The default snaps to whole cells, matching how camera shake already works.

### D7. What about the background?

**The camera dollies.** As it pulls back, far things shrink less than near things, just as they would if a real camera backed away. Background battles at depth 0.5, 0.2 and 0.1 shrink by less than the main fight; the distant stars and nebula don't shrink at all.[^d7]

### D8. The HUD and indicators

The HUD stays unzoomed. Arrows pointing to off-screen enemies follow what's visible, but measure distance from your ship, not from the camera.

### D9 and D10. Speed and precision

More on screen means more to draw, so zoom will be benchmarked the same way the screen effects were. Precision rules from the main camera page still apply, and matter more: zoom magnifies rounding errors.

## From cutscene to gameplay without a cut

A bonus of decision B: a cutscene can hand over to gameplay **with no cut or fade**. Picture the pilot climbing in, the canopy closing, and the player simply flying. It works because:

1. **The grids nest.** Cutscene cells are 9 pixels, gameplay cells 4.5: exactly half. Every cutscene cell is a perfect 2 × 2 block of gameplay cells, so a cutscene frame is already a valid gameplay frame.
2. **The cutscene camera ends where the gameplay camera begins.** Its last move eases to exactly your ship's position, angle and anchor.
3. **The content is literally the same.** The background is drawn live by the game behind the footage; the ship in the footage is rendered from the same art as the game's sprite; engine effects are started by the game.
4. **The motion carries over.** The footage comes with a small track of the ship's position, angle and speed, so physics starts from where the footage left off.

Then the screen sharpens from 9-pixel cells to 4.5-pixel cells, either at once or as a quick dissolve, and the HUD fades in. The reverse (gameplay into a cutscene, such as landing on the carrier) is the same steps backwards.

## Suggested build order

All of these leave the game looking exactly as it does now:

1. Split the world cell from the screen cell (D5).
2. Teach the view transform to zoom.
3. Remove the shortcuts that assume your ship is always at the anchor.
4. Give gameplay its own view (D4).
5. Add a debug camera (mouse to move, scroll wheel to zoom) as a test bench.

[^d2]: The full decision log lives in the "Defender X Camera and Zoom" design page, decisions D1 to D10. It was written against the code at commit `9c651a2` on `main` (October 27, 2026).

[^b2]: In the design doc this is option **B2** (precomputed zoom steps) with **B1** (plain sampling of the full-size sprite) as the fallback. Plain sampling at zoom-out skips cells, so thin details flicker from frame to frame. The downsampling code already exists: `render/ShrunkArt.kt`, built for the background battles, makes a smaller copy of a sprite, keeping a cell solid where the art covers at least half of it and never letting a ship shrink to nothing. Zoom-in (more detail than the 1× sprite) waits until after the demo.

[^d5]: The number is `GameConfig.MACRO_PIXEL_SIZE` (4.5). As a world length it's used by `Pose.cell` (where each hull cell sits in the world, for hits and fires), by hull radii in collisions, projectiles, rocks and AI sensing, and by effects spacing. As a screen length it's used by every cell-snapping drawing routine. The plan names the world one `WORLD_CELL`.

[^d7]: A layer at depth `s` under a pull-out of `k`× is drawn at scale `s / (1 + s(k − 1))`. For the foreground (`s` = 1) that's `1/k`. For the 0.1 layer at 3× out it's 0.1 / 1.2 ≈ 0.083, barely smaller than its normal 0.1. The nebula image is drawn at 1.5× the play area today, so it will need to be bigger (or tile) to cover a 4× pull-out.
