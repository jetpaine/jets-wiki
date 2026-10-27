---
layout: dx
title: Cinema engine
summary: How Defender X makes its cutscenes, from AI-generated video to a chunky pixel picture filmed by a camera you can hold.
status: Built · opening scene in progress
checked: 2026-10-27
---

<div class="glossary" markdown="1">

## Glossary

| Term | What it means |
| --- | --- |
| Cinematic / cutscene | A short film inside the game, where the player watches instead of plays. |
| Footage | A video file, such as `.mov` or `.mp4`. Here it's made with AI video tools. |
| AI image / video generator | A tool that makes pictures or video from a written description or a reference picture. The game uses **Leonardo** (images) and **Kling** (video). |
| Character sheet | A reference picture of a character from several angles, used to keep them looking the same from shot to shot. |
| Frame rate (fps) | How many still pictures per second make up a video. Films are 24; the game's cutscenes are 12. |
| Clip | One piece of footage, converted into the game's format. |
| Shot | A stretch of one clip, with its own camera movement. |
| Scene | A run of shots played one after another: a whole cutscene. |
| Cut / fade / flash | How one shot ends into the next: an instant switch, a dip through black (or another color), or a white flash. |
| Keyframe | A camera position set at a particular moment; the camera glides smoothly between keyframes. |
| Easing | How a movement speeds up and slows down between keyframes, so it doesn't start or stop with a jolt. |
| Take | One recorded attempt at a camera move, performed live with the mouse. |
| Handheld | A slight natural wobble added to the camera, as if a person were holding it. |
| Tripod | No wobble: the camera is perfectly steady. |
| Pan / tilt / roll / zoom | Camera moves: slide sideways, tip up or down, turn around the lens, and move closer or further. |
| ffmpeg | A free command-line tool for converting video. The game uses it to read footage. |
| Gradle task | A named command in the project's build tool, run as `./gradlew <name>` in a terminal. |

</div>

## The approach in one paragraph

Cutscenes are **AI-generated footage, shrunk down to the game's chunky pixel grid, and played through a live camera** that pans, zooms and tilts while the pixels stay square to the screen. The camera is what makes it feel like it's happening inside the game rather than like a video playing on top of it.[^decided]

## Why this approach

It was chosen after look tests in October 2026:

- Plain AI video, played as is, looks like low-resolution video, not like a game.
- Footage with strong silhouetted characters, in an anime style, reads as animated sprites once it's turned into cells.
- A live camera, with a handheld wobble, slow leans and zooms, is what sells it as in-engine.
- One footage pixel per cell, looked up exactly, rotates with the same stair-stepped edges as the game's ships. It looked better than a smoothed, higher-resolution version, and its zooms hold up.
- Cutscene cells are 9 pixels, twice the size of gameplay's 4.5. That suits film better, and it's easy to change later, since the original footage is kept.

## From idea to cutscene

1. **Make character sheets.** Leonardo draws the characters, ship and sets.
2. **Generate footage.** Kling animates the character sheets into video.
3. **Convert.** One command turns each video into a clip on the cell grid.
4. **Write a scene.** A short text file listing shots, transitions, sound and dialogue.
5. **Record camera takes.** Perform the camera live with the mouse, and keep the best take.
6. **Commit the scene.** The scene file and its kept takes go into the repository.

Everything before the scene file can be regenerated, so it stays **out** of the repository (video files are large). Scene files and kept takes are the first things committed.[^pipeline]

### 1. Footage becomes clips

Footage files go in a `footage` folder next to the game's folder. One command converts them:

```
./gradlew convertCinematic
```

Each file becomes a **clip**: shrunk to the cutscene grid (plus a margin so the camera can move around), one pixel per cell, at 12 frames per second, with its sound pulled out. Optionally its colors can be locked to a small palette, like an old console's. A file named `Wake Up.mov` becomes the clip `wake-up`. Clips that are already up to date are skipped.[^convert]

### 2. Clips become a scene

A **scene** is a small text file listing its shots. The simplest possible one:

```
{
  name: wake-up
  shots: [
    { clip: wake-up, in: 0.0, out: 4.0 }
  ]
}
```

`in` and `out` are seconds into the clip. A scene can also give each shot:

- **a camera**: keyframes of zoom, where it's looking and roll, with easing, or a recorded take;
- **handheld or tripod**: wobble by default, or `tripod: true` for none;
- **a transition** into the next shot: cut, fade (through black or any color) or flash;
- **sound**: the clip's own audio (louder, quieter or off), a music track running across cuts, and sound effects at set moments;
- **text**: captions ("SOMEWHERE IN THE KUIPER BELT") and dialogue boxes with a speaker's name, at the top or bottom.

### 3. The scene editor and camera rig

```
./gradlew runScene -Pscene=wake-up
```

This opens the scene in an editor that reloads whenever the file is saved, so shot lengths can be adjusted while watching. Then the camera is performed live, like a camera operator:

- **Tab** rehearses: the shot loops while you aim with the mouse, zoom with the scroll wheel, roll with the arrow keys.
- **Enter** records a take: the shot plays once while you work the camera.
- **[ and ]** move between takes; **K** keeps the one playing and writes it into the scene file.

Kept takes are saved beside the scene and committed; the rest stay on your own machine.[^takes]

## In the game today

The title screen loops a scene (a slow-motion burning Earth) behind its text, darkened so the text stands out. Other scenes play only in the scene editor until the opening cutscene is hooked into the game's flow.

## What's next

1. **Proof of concept** for the opening scene: turn rough character sheets into video in Kling, try rendering characters separately and combining them, try a green-screen background, and decide whether a second workflow is needed.
2. **The real opening scene**: the pilot is rocked awake by the alarm and rushes to the fighter.
3. **Sound effects, then music.**
4. **Screen effects in scenes** (lighting, distortion, bloom), sharing the gameplay effects described in [Rendering layers](rendering.html).
5. **A seamless handoff** from the last shot into live flying, with no cut. See [Zoom and moving camera](camera-zoom.html#from-cutscene-to-gameplay-without-a-cut).

## Open questions

- How to keep the pilot, fighter and hangar consistent across shots.
- What frame rate and resolution to generate at, given the 12 fps grid and the demo's download size.
- Whether to use the generated clips' own audio, or replace it all.
- The tools' commercial-use terms, and the wording of Steam's required disclosure of AI-generated content.

Decided: dialogue is **text only**, not voiced, for now.

[^decided]: Decided on October 13, 2026, after look tests; recorded in the Episode 1 Demo Roadmap and the "Cinematics Generation Approach" notebook. The full design is `design/scene-player.md` in the separate defender-x-docs repository.

[^pipeline]: Character sheets are saved in `assets/character-sheets` and raw Kling output in `assets/raw-footage`; neither is committed. Converted clips go in `assets/cinematics` and also aren't committed, because they rebuild from the footage. Scenes are `assets/scenes/*.scene` (libGDX's relaxed JSON format, read by `cinema/SceneFile.kt`); music and effects are `.ogg`, `.wav` or `.mp3` in `assets/audio`. A fresh copy of the repository needs the same footage to rebuild the clips; where that footage lives long-term is still open.

[^convert]: `cinema/CinematicConverter.kt`, which runs ffmpeg (`cinema/Ffmpeg.kt`). Options: `-Pclip=wake-up` (just one clip), `-Pfps=24` (each clip remembers its own frame rate), `-Ppalette=16` (lock to 2–256 colors; 0 is full color), `-Pforce` (reconvert anyway), and `-PfootageDir=<dir>` for a different footage folder. `./gradlew runCinematic` previews the clips through a random "documentary" camera (`cinema/DocCamera.kt`). At play time `cinema/ClipRenderer.kt` draws the current frame through the camera into a 213 × 120 image, one pixel per cell, and enlarges it with hard edges. Zoom is limited to 1.0–2.0: below 1.0 one footage pixel would be smaller than a cell and detail would be lost (issue #17).

[^takes]: The editor is `cinema/SceneEditorApp.kt`; the live camera is `CameraRig.kt`; takes are `Take.kt` and `TakeStore.kt`; timing across shots is `SceneTimeline.kt`; sound is `SceneAudio.kt`; text is `SceneTextRenderer.kt`; the handheld wobble is `Handheld.kt`. Kept takes are written to `assets/scenes/<scene>.takes`; the others stay in `assets/scenes/takes-local/`, which git ignores. A take is recorded against the shot's length at the time, so settle `in` and `out` before recording; the editor warns when a take is shorter than its shot.
