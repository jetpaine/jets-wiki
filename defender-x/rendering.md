---
layout: dx
title: Rendering layers
summary: How the game paints one picture, back to front, mixing chunky retro blocks with smooth modern light.
status: Built
checked: 2026-10-27
---

<div class="glossary" markdown="1">

## Glossary

| Term | What it means |
| --- | --- |
| Rendering | Drawing: turning the game's state (where everything is) into a picture. |
| Frame | One finished picture. The game draws as many per second as your screen shows, often 60 or more. |
| Pixel | One dot of a screen. A 1920 × 1080 screen has about 2 million of them. |
| Macro-pixel (cell) | A big square of one flat color that stands in for a pixel on an old console's TV. In gameplay a cell is 4.5 × 4.5 virtual pixels, so about 102,000 fit on screen. |
| Virtual canvas | The pretend 1920 × 1080 screen the game always draws on, then stretches to fit your window. See [Camera](camera.html). |
| Layer | One pass of drawing. Later layers are painted over earlier ones, like coats of paint. |
| Sprite | The picture of one object, such as a ship, stored as a grid of colored cells. |
| Hidden image (buffer) | A picture drawn off-screen, in memory, so it can be processed before (or instead of) being shown. Programmers call it a **frame buffer** or **texture**. |
| Capture | Drawing several layers into one hidden image so an effect can work on all of them at once. |
| Shader | A tiny program that runs on the graphics card, once for every pixel, to change its color. Fast, because the card runs thousands at once. |
| Screen effect | A filter run over a finished picture: glow, blur, color changes, film grain. |
| Bloom / glow | Light that seems to spill out around bright things, as it does in a camera lens or on a CRT screen. |
| HUD | "Heads-up display": the text and warnings drawn over the action. |
| gdx-vfx | A free add-on library of screen effects for libGDX that the game uses for some of its filters. |

</div>

## Two kinds of pixel

Everything you see ends up in one of two styles, and which one decides how it looks:

| | Macro-pixel (cell) | Screen pixel |
| --- | --- | --- |
| What it is | One big square of flat color | One real dot of the display |
| Size | 4.5 × 4.5 virtual pixels in gameplay; 9 × 9 in cutscenes | 1 pixel |
| How many on screen | About 427 × 240 in gameplay | Millions |
| Looks | Blocky, flat colors, stair-stepped edges: the Super Nintendo look | Smooth gradients and soft edges: modern polish |

The cells form a fixed grid over the canvas, starting at its bottom-left corner. **The rule that keeps the retro look:** anything meant to look like it's "on the console" must land on that grid, as whole cells of one flat color with edges on the grid lines. Anything drawn without that discipline comes out smooth, in screen pixels, whether it's a rotated image, a blur or a soft glow.[^grid]

Most of the game is cells. A few things are deliberately smooth: **text** (so it stays readable), the **g-force greyout**, and the **screen effects** at the end.

**The rule for glow.** The cells stand in for a TV's pixels, and the screen effects stand in for what happens when that light hits the glass in front of them. So glowing shapes stay blocky, but the light spilling off them is smooth, the way it would be on a real set.

## Three ways to get onto the grid

| Technique | How it works | Used for |
| --- | --- | --- |
| **Cell by cell** | For each cell the object covers, look at which bit of the art lies under the cell's center, and paint the cell that color. However the art turns, its edges stair-step. | Ships, enemies, rocks |
| **A small picture, enlarged** | Draw into a hidden image with exactly one pixel per cell, then stretch it over the screen with hard edges, so each pixel becomes one cell. | Stars and nebula, vapor trails, cutscene footage, the glow layer |
| **Squares placed on the grid** | Code draws shapes out of squares, each snapped to the nearest cell. | Dust specks, engine flames, thruster jets, bullet streaks, muzzle flash, sparks, explosions, enemy indicators |

The second technique matters most for effects: once something is a one-pixel-per-cell image, any processing done to it (a blur, a glow, a color change) also comes out in cells when it's enlarged.[^techniques]

## A frame of gameplay, back to front

Here is the order the game paints one gameplay frame. The list reads bottom (painted first, furthest back) to top (painted last, in front). Smooth-pixel layers are shaded.

<ol class="stack">
  <li><span>Stars and nebula, the far background</span> <span class="kind">cells</span></li>
  <li><span>Space dust specks, which drift past at different speeds to show you're moving</span> <span class="kind">cells</span></li>
  <li><span>Background battles at three depths, far to near, each through its own haze</span> <span class="kind">cells</span></li>
  <li><span>Vapor trails behind every ship</span> <span class="kind">cells</span></li>
  <li><span>Engine flames, under the hulls</span> <span class="kind">cells</span></li>
  <li><span>Bullets passing <em>under</em> ships</span> <span class="kind">cells</span></li>
  <li><span>The hulls: your ship, rocks, enemies</span> <span class="kind">cells</span></li>
  <li><span>Black smoke from wrecks and burning hulls</span> <span class="kind">cells</span></li>
  <li><span>Thruster jets, then smoke, debris and broken-off hull pieces</span> <span class="kind">cells</span></li>
  <li><span>Bullets passing <em>over</em> ships, then flashes, sparks and fire</span> <span class="kind">cells</span></li>
  <li><span>Arrows pointing to off-screen enemies</span> <span class="kind">cells</span></li>
  <li class="px"><span>HUD text and the G-LIMIT warning</span> <span class="kind">screen px</span></li>
  <li class="px"><span>Screen effects over everything above: glow, heat shimmer, color fringing, vignette, grain, scanlines, greyout</span> <span class="kind">screen px</span></li>
  <li class="px"><span>Pause menu and TUNING screen, never filtered</span> <span class="kind">screen px</span></li>
</ol>

Some things in this list are painted **twice**: once onto the screen, and once into a hidden **glow layer** that only holds what should glow (engine flames, vapor trails, flashes and sparks). The screen effects later blur that layer and add it back as light. That way a white muzzle flash can glow while a white highlight on a hull doesn't, even though both are just "white" by the time the picture is finished.[^order]

### Why some things are drawn twice: masks

A screen-effect library only sees finished pixels, not objects. By the time it runs, a spark and a white hull highlight are indistinguishable. To make an effect apply only to some things, those things are first drawn into a picture of their own, a **mask**, which says pixel by pixel how strongly the effect applies. The glow layer is one mask; the **heat shimmer** around near misses (the ripple when a bullet or ship passes very close) is another.[^masks]

There are two ways to use a mask:

- **Effects that only change color** (vignette, grain, scanlines, color grading, glow) can be applied to the whole picture and then blended back with the original according to the mask.
- **Effects that move pixels** (color fringing, heat shimmer) can't be blended like that; halfway between a shifted and unshifted picture is a ghostly double image, not a smaller shift. Those have to read the mask inside their own shader, so the game writes them itself.

## A frame of a cutscene

Cutscenes are simpler, because the whole picture starts life as one small cell image:[^cutscene]

1. The current frame of footage is drawn through the live camera (pan, zoom, tilt) into a hidden 213 × 120 image, one pixel per 9-pixel cell.
2. Optionally, its colors are adjusted once per cell.
3. It's enlarged to the screen with hard edges.
4. Dialogue, captions, fades and flashes go on top, as smooth text.
5. The same screen effects as gameplay can run last.

See [Cinema engine](cinema.html) for how the footage gets there.

## What's ours and what's borrowed

The game uses the **gdx-vfx** library for its plumbing (passing a picture from one effect to the next) and for film grain, scanlines and the vignette. Bloom, the glow layer, the cell color grade, the greyout and the mask steps are the game's own. The decision (October 2026): keep the library while it's cheap, and replace it the moment it starts fighting the game's pipeline; replacing it is roughly a day's work.[^vfx]

[^grid]: The canvas is 1920 virtual pixels wide and `MACRO_PIXEL_SIZE` is 4.5, so 1920 / 4.5 = 426.7: the grid is 427 cells wide and its last column hangs slightly off the right edge. A cell lands on a whole number of real pixels only on some screens; on a 1080p display 4.5 pixels can't be exact, which is one reason zoom has to be handled carefully (see [Zoom and moving camera](camera-zoom.html)).

[^techniques]: Cell by cell: `render/MacroPixelRenderer.kt`, using "inverse mapping": for each screen cell in the sprite's rotated bounding box, rotate the cell's center back into the art and paint one square if that art pixel is solid. Sprites are imported from 1024 × 1024 source art at a set length in cells (40 for ships) by `render/SpriteImport.kt`. Small pictures enlarged: `render/CellBackground.kt` and `render/GlowLayer.kt` (427 × 240 frame buffers), `render/VaporGrid.kt` and `VaporGpu.kt` (vapor trails), `cinema/ClipRenderer.kt` (213 × 120). Squares on the grid: libGDX `ShapeRenderer` rectangles, their positions rounded down to the grid with `floor(x / cell) × cell`, in `SpeckField`, `WorldRenderer.drawEngineFlame` and `drawJet`, `ProjectileRenderer`, `EffectRenderer` and `IndicatorShape`.

[^order]: The order is in `game/WorldRenderer.kt`: `render()` places and captures the background and depth layers, then `renderSprites()` paints the passes in the order above and finishes with `drawGlowLayer()` and `drawShimmerMask()`; then the off-screen indicators and HUD. The whole thing runs between `screenEffects.begin()` and `end()` in `DefenderXGame.renderGameplay`, which is the capture the screen effects work on. Effects are applied in this priority order (`fx/ScreenEffects.kt`): bloom, shimmer, chromatic aberration (color fringing), masked vignette, grain, scanlines, greyout.

[^masks]: `fx/MaskBlendEffect.kt` blends a color-only effect by a mask; `fx/ShimmerEffect.kt` and `fx/ChromaticSplitEffect.kt` read their masks inside their own shaders; `fx/GlowBloomEffect.kt` is the bloom that mixes in the glow layer before blurring. The glow blur runs on the full-size picture (at half resolution, for speed), so its haze is smooth and hugs the cell edges. An older "bloom in cells" option, which blurs on the one-pixel-per-cell image so the glow itself comes out blocky, is still there as a switch.

[^cutscene]: `cinema/ClipRenderer.render` (camera into the grid), then `grade`, then `drawToScreen`; after that `SceneTextRenderer` (dialogue and captions) and the scene editor's fades and overlay. Cutscene cells are `MACRO_PIXEL_SIZE × CINEMATIC_CELL_SCALE` = 4.5 × 2 = 9 virtual pixels.

[^vfx]: gdx-vfx 0.5.1 is the last release built for the libGDX version the game uses (1.11), so a future libGDX upgrade is one of the triggers for replacing it. One libGDX quirk worth knowing: finishing a frame buffer always switches drawing back to the screen, so anything that draws into its own buffer *inside* a capture has to remember the outer buffer and switch back to it afterwards (`render/GlBuffers.kt`).
