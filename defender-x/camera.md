---
layout: dx
title: Camera and coordinates
summary: How a position somewhere in space becomes a spot on your screen, and why in Defender X it's the universe that turns, not your ship.
status: Built · zoom planned
checked: 2026-10-27
---

<div class="glossary" markdown="1">

## Glossary

| Term | What it means |
| --- | --- |
| Coordinates | A pair of numbers, **x** (across) and **y** (up), that pin down a position, like a grid reference on a map. |
| World coordinates | Where something is in the game's universe, in **world units**. They don't care what's on screen. |
| Screen coordinates | Where something appears in the picture, in pixels from the bottom-left corner. |
| Virtual canvas | A pretend screen exactly 1920 × 1080 pixels that the game always draws on. It is then stretched to fit your real window, so the game looks the same on any monitor.[^canvas] |
| Camera | The imaginary viewpoint the picture is taken from: where it is, which way it's turned, and how far it's zoomed. |
| Transform | A recipe that converts one kind of coordinates into another. The game's main one is called `ViewTransform`. |
| Anchor | The fixed spot on the screen where the camera's center lands. Today that's where your ship is drawn. |
| Rotation | Turning around a point. Measured as an **angle**, in degrees or radians.[^radians] |
| Play area | The part of the screen where the action is drawn (the whole screen today, unless letterbox bars are switched on). |
| Camera shake | Small jolts to the picture when something hits nearby, to make impacts feel physical. |

</div>

## The idea in one picture

In most 2D games, the ship moves around a fixed map, like a toy car on a tabletop. Defender X does the opposite. **Your ship stays in one place on the screen, pointing the same way, and the whole universe moves and turns around it.** When you press "nose up", the ship doesn't visibly rotate; the stars, rocks and enemies swing the other way.

<figure>
<svg viewBox="0 0 640 230" role="img" aria-label="Left: a top-down map of the world with the ship pointing up and to the right, an enemy ahead and a rock above. Right: the same scene on screen, turned so the ship points right; the enemy and rock have swung around it.">
  <g font-family="JetBrains Mono, monospace" font-size="11" style="fill:var(--muted)">
    <text x="10" y="16">WORLD (the map)</text>
    <text x="340" y="16">SCREEN (what you see)</text>
  </g>
  <rect x="10" y="24" width="290" height="196" rx="6" style="fill:none;stroke:var(--rule)"/>
  <rect x="340" y="24" width="290" height="196" rx="6" style="fill:none;stroke:var(--rule)"/>
  <!-- world panel -->
  <g transform="translate(10 24)">
    <g transform="translate(260 40)"><line x1="0" y1="14" x2="0" y2="-14" style="stroke:var(--muted)" stroke-width="1.5"/><path d="M0 -18 L-4 -10 L4 -10 Z" style="fill:var(--muted)"/><text x="6" y="4" font-size="10" font-family="JetBrains Mono, monospace" style="fill:var(--muted)">N</text></g>
    <g transform="translate(100 130) rotate(-45)"><path d="M16 0 L-10 -9 L-6 0 L-10 9 Z" style="fill:var(--cell-c)"/></g>
    <g transform="translate(200 70)"><path d="M-10 0 L8 -7 L4 0 L8 7 Z" style="fill:var(--cell-d)"/></g>
    <rect x="52" y="34" width="20" height="16" rx="3" style="fill:var(--cell-b)"/>
    <text x="72" y="160" font-size="11" font-family="JetBrains Mono, monospace" style="fill:var(--fg)">you</text>
    <text x="192" y="96" font-size="11" font-family="JetBrains Mono, monospace" style="fill:var(--fg)">enemy</text>
    <text x="78" y="46" font-size="11" font-family="JetBrains Mono, monospace" style="fill:var(--fg)">rock</text>
  </g>
  <!-- arrow -->
  <g><line x1="306" y1="122" x2="332" y2="122" style="stroke:var(--accent)" stroke-width="2"/><path d="M336 122 L328 117 L328 127 Z" style="fill:var(--accent)"/></g>
  <!-- screen panel -->
  <g transform="translate(340 24)">
    <g transform="translate(260 40) rotate(45)"><line x1="0" y1="14" x2="0" y2="-14" style="stroke:var(--muted)" stroke-width="1.5"/><path d="M0 -18 L-4 -10 L4 -10 Z" style="fill:var(--muted)"/><text x="6" y="4" font-size="10" font-family="JetBrains Mono, monospace" style="fill:var(--muted)">N</text></g>
    <line x1="60" y1="24" x2="60" y2="196" style="stroke:var(--accent)" stroke-dasharray="3 4" opacity=".6"/>
    <g transform="translate(60 110)"><path d="M16 0 L-10 -9 L-6 0 L-10 9 Z" style="fill:var(--cell-c)"/></g>
    <g transform="translate(173 138) rotate(45)"><path d="M-10 0 L8 -7 L4 0 L8 7 Z" style="fill:var(--cell-d)"/></g>
    <g transform="translate(95 23) rotate(45)"><rect x="-10" y="-8" width="20" height="16" rx="3" style="fill:var(--cell-b)"/></g>
    <text x="36" y="136" font-size="11" font-family="JetBrains Mono, monospace" style="fill:var(--fg)">you</text>
    <text x="165" y="166" font-size="11" font-family="JetBrains Mono, monospace" style="fill:var(--fg)">enemy</text>
    <text x="112" y="30" font-size="11" font-family="JetBrains Mono, monospace" style="fill:var(--fg)">rock</text>
    <text x="66" y="190" font-size="10" font-family="JetBrains Mono, monospace" style="fill:var(--accent)">anchor</text>
  </g>
</svg>
<figcaption>The same moment twice. On the map (left) your ship points up and to the right. On screen (right) the world has been turned so your ship always points right from the same spot, the anchor. Everything else, even "north", has swung around you.</figcaption>
</figure>

This is the game's signature. It makes the player feel like they're in the cockpit: the horizon tilts, not the plane. It's also the reason the game has its own engine instead of using an off-the-shelf one.

## From world to screen in three steps

Every object has a position in the world. To draw it, the game converts that position into screen coordinates. It does this with one small recipe, the **view transform**, rebuilt once per frame (every picture) because the camera moves every frame.[^vt]

The recipe takes four ingredients:

- **the camera's position** in the world (today: exactly where your ship is),
- **the world angle**, how far to turn everything (today: your ship's own angle),
- **the anchor**, the spot on screen where the camera's center lands,
- **the scale**, how many screen pixels one world unit takes up (1 for the main fight).

And it applies them in three steps:

1. **Move relative to the camera.** Subtract the camera's position. Now the ship is at (0, 0), and everything else is described as "this far from me".
2. **Turn.** Rotate every point around the ship by the world angle, so the ship's nose ends up pointing right on screen. Points ahead of you end up to your right; points above your canopy end up above you.
3. **Place on the screen.** Add the anchor's position, so (0, 0) lands at the anchor.

Running the recipe backwards (screen to world) answers questions like "what's under the mouse pointer?".[^inverse]

## Where your ship sits

The anchor isn't the middle of the screen. It's **600 pixels left of center** (on the 1920-wide canvas). Your ship is drawn there, nose pointing right, so most of the screen shows what's **ahead** of you. That's where enemies, rocks and incoming fire matter most.[^offset]

## Things the camera also does

**Camera shake.** When something hits nearby, the picture jolts. The game does this by nudging the anchor, always by whole [macro-pixels](rendering.html), so the chunky pixel look never smears. Only the world shakes; the text and gauges on top (the HUD) stay put.[^shake]

**Depth layers.** Some ships are far away in the background, part of a separate battle you can watch but not touch. The same recipe draws them with a smaller scale (0.5, 0.2 or 0.1), so they look smaller and move more slowly as you turn, like distant scenery seen from a car window. This effect is called **parallax**.[^depth]

**"What you see is what you can hit."** Gameplay uses the same view. Ships can only be hit by bullets when they're well inside the frame (at least 150 pixels in from every edge), so every kill happens where you can see it. Rocks can be hit anywhere on screen.[^hitzone]

## Why the numbers need care

The universe is large, and a computer stores numbers with limited precision. If a big world position is turned into a screen position carelessly, rounding errors can push a point across the line between two macro-pixels, so a detail like the engine flame flickers between two blocks as you turn. The game avoids this by keeping positions in high-precision numbers until after step 1 (when they've become small "distance from me" values) and by using exact trigonometry rather than a faster approximate table.[^precision]

## What's next: zoom and a moving camera

Today the camera is always locked on your ship and never zooms. Plans are under way for a camera that can **pull back** (zoom out) to show, for example, an enemy sitting on your tail, and then return. That raises interesting questions, chiefly: when you zoom out, do the chunky pixels get smaller, or do objects just use fewer of them? Those decisions are explained in the sub-article **[Zoom and moving camera](camera-zoom.html)**.

[^canvas]: `GameConfig.VIRTUAL_W` × `VIRTUAL_H` (1920 × 1080). libGDX's `FitViewport` scales that canvas to the window and adds black bars if the shapes don't match ("letterboxing"). On a 3584 × 1982 laptop screen, one virtual pixel is about 1.87 real pixels.

[^radians]: Programmers usually measure angles in **radians** rather than degrees: a full circle is 2π radians (about 6.28) instead of 360°. One radian is about 57°. It's the natural unit for the trigonometry (sine and cosine) that rotation uses.

[^vt]: The code is `util/ViewTransform.kt`. It replaced an older `CoordinateTransformer`, which is why that name still turns up in old notes. It precomputes the rotation's sine and cosine (multiplied by the scale) once per frame, so converting each point afterwards is just a few multiplications and additions: `screenX = (x − camX)·cos − (y − camY)·sin + anchorX`, and the matching formula for y.

[^inverse]: `ViewTransform.screenToWorld`. There's also `directionToScreen`, which turns a direction or offset (rather than a position) onto the screen. It skips step 1, so huge world coordinates never enter the sum at all.

[^offset]: `GameConfig.PLAYER_SCREEN_OFFSET_X` = −600 and `PLAYER_SCREEN_OFFSET_Y` = 0, measured from the center of the play area. `ViewTransform.forPlayer` builds the main view: camera on the player, world angle from the player's angle, anchor at the play area's center plus that offset.

[^shake]: `ViewTransform.shifted(dx, dy)` makes a copy of the view with the anchor moved. The renderer (`game/WorldRenderer.render`) builds a **steady** view and a **shaken** view each frame: the world is drawn with the shaken one, the HUD and off-screen enemy indicators with the steady one. Shake strength is set per cause, for example `NEAR_MISS_PASS_SHAKE_CELLS`.

[^depth]: `ViewTransform.atDepth(s)` keeps the camera, angle and anchor but multiplies the scale by `s`. The depth values are `GameConfig.DEPTH_LAYER_SCALE` (0.1, 0.2, 0.5). The drawing side is `render/DepthLayer.kt`; the background fight itself is `game/BackgroundBattle.kt`, which is a separate little world that never touches yours.

[^hitzone]: `GameConfig.SHIP_HIT_ZONE_INSET` = 150 virtual pixels. `World.processProjectileHits` builds its own `ViewTransform.forPlayer` for this check, which is important for the planned zoom: if gameplay used the zoomed-out view instead, zooming out would make far-off ships hittable. See [Zoom and moving camera](camera-zoom.html#d4-does-gameplay-follow-the-camera).

[^precision]: Issue #32 on GitHub. World positions are stored as `Double` (about 16 significant digits) and only turned into `Float` (about 7) after the camera has been subtracted. The rotation uses Kotlin's exact `cos`/`sin` rather than libGDX's `MathUtils` lookup table, whose small error changes with the angle and made a point sitting on a cell line hop between cells.
