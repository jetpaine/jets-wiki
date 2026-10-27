---
layout: dx
title: Defender X
summary: How a small 2D space-combat game works on the inside, explained for someone who has never seen it and has never written code.
status: Living document
checked: 2026-10-27
---

**Defender X** is a 2D space dogfighting game in development, inspired by the 1981 arcade game *Defender*. Its unusual idea: your ship never moves on the screen. Instead, the whole universe turns and slides around you, so when you pull the nose up, the stars swing down. The goal is to make a flat, old-school-looking game feel like the dogfights in *Battlestar Galactica* (2004).

This sub-wiki explains how the game is built. It assumes you've never played it and don't program. Every technical idea is explained the first time it comes up, and the deeper details sit in **notes** at the bottom of each page (the small raised numbers, like this one[^example], link to them) or in **sub-articles**, which are linked where they fit.

## How to read this wiki

- **Each page opens with a glossary.** It lists the words that page relies on, in the order you'll meet them. Skim it first, or come back to it when a word stops you.
- **Notes hold the deep details.** File names, exact numbers and the reasons behind them. You can skip every note and still follow the page.
- **"Today" means the code as checked.** The date at the top of each page says when it was last compared with the game's code. The game moves fast; a page can fall behind.
- **"Planned" means not built yet.** Pages say clearly what exists and what is still a design.

<div class="glossary" markdown="1">

## Start-here glossary

These words come up on almost every page.

| Term | What it means |
| --- | --- |
| Code | The written instructions that make up a program. Defender X's code is written in a programming language called **Kotlin**. |
| Repository (repo) | The folder, stored online on GitHub, that holds all of the game's code, art and history. This one is [jetpaine/defender-x](https://github.com/jetpaine/defender-x). |
| Engine | The part of a game that does the general work: drawing, timing, reading the keyboard. Defender X has its own engine, built on a free toolkit called **libGDX**.[^engine] |
| Tick | One step of the game's clock. The game updates the world 120 times a second, so one tick is 1/120 of a second. Everything that moves, moves a little each tick. |
| Frame | One finished picture on the screen. Ticks and frames are separate: the game can tick several times between two pictures. |
| World | Everything in the game's universe: ships, rocks, bullets, smoke. Positions in the world are measured in **world units**. |
| Screen | The picture you actually see. Positions on the screen are measured in pixels. Turning world positions into screen positions is the [camera's](camera.html) job. |
| Macro-pixel (cell) | A big square block of one color. The game draws almost everything out of these, to look like an early-1990s console game. See [Rendering layers](rendering.html). |
| Player | You, the person flying the ship. Also used for the ship you fly. |
| AI | "Artificial intelligence": here, the code that flies the enemy ships. See [AI layers](ai.html). |
| GameConfig | One file that holds every adjustable number in the game (speeds, sizes, timings). Most can be changed while the game runs, from a **TUNING** screen in the pause menu. |
| Design doc | A written plan for a part of the game, made before or while building it. Several pages here are drawn from them. |

</div>

## Contents

<ul class="cards">
  <li><a href="mission.html"><strong>Mission statement</strong><span>What the game is trying to be, and the rules it lives by.</span></a></li>
  <li><a href="camera.html"><strong>Camera and coordinates</strong><span>How a position in space becomes a spot on your screen, and why the world turns instead of the ship.</span></a></li>
  <li><a href="physics.html"><strong>Physics and g-forces</strong><span>How ships move, why they drift, and what happens to a pilot who turns too hard.</span></a></li>
  <li><a href="rendering.html"><strong>Rendering layers</strong><span>How one picture is built, back to front, out of chunky retro blocks and smooth modern glow.</span></a></li>
  <li><a href="ai.html"><strong>AI layers</strong><span>How enemy pilots sense, decide and fly, and where the Conductor sits above them.</span></a></li>
  <li><a href="conductor.html"><strong>Level Conductor</strong><span>A planned "director" that paces each fight like a film.</span></a></li>
  <li><a href="cinema.html"><strong>Cinema engine</strong><span>How the cutscenes are made, from AI-generated video to a camera you can hold.</span></a></li>
</ul>

## Where the game is now

As of late October 2026 the core fighting works: flying, shooting, crashing, explosions, enemy pilots with morale, g-forces, and a cutscene player. The near-term goal is a short **demo** of the first level for *Steam Next Fest* in February 2027 (Steam is the largest online store for PC games; Next Fest is a week where it features free demos).[^demo]

## Keeping this wiki up to date

The game changes faster than any document. A few habits keep this wiki useful:

- When a page is checked against the code, update the date in its header (`checked:` at the top of the file).
- Mark anything not built yet as **planned**, and anything decided but not built as **decided**.
- Add new words to the page's glossary when you add them to the page.
- New sections go in `_data/dx_nav.yml` so they show up in the sidebar.

[^example]: Like this. Notes are where the exact file names, numbers and reasoning live. The ↩ arrow takes you back to where you were reading.

[^engine]: Most games are built on a big ready-made engine such as Unity or Unreal. Defender X isn't, because its core trick (the whole world rotating around a fixed ship) would mean fighting such an engine at every step. libGDX is a lower-level toolkit: it opens a window, draws shapes and images, and plays sounds, and leaves the rest to the game. The game runs on desktop computers through **LWJGL3**, the part of libGDX that talks to the computer's graphics card.

[^demo]: The demo plan is a "vertical slice" of Level 1: a small part of the game, finished to release quality, so players can judge the whole from it. Its beats are a title screen over a scorched Earth, a cutscene where the pilot is shaken awake and runs to the fighter, a launch, a fight with wingmen against waves of enemies while a countdown runs, a race back to land on the carrier ship before the fleet jumps away, and an ending.
