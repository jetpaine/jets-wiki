---
layout: dx
title: Mission statement
summary: What Defender X is trying to be, and the few rules every design decision is checked against.
status: Draft · still being shaped
checked: 2026-10-27
---

<div class="note" markdown="1">
<span class="note-label">Draft</span>
The game's purpose is still being worked out. This page gathers what has been decided so far, mostly from the project notes and design docs. Expect it to be rewritten as the vision firms up.
</div>

<div class="glossary" markdown="1">

## Glossary

| Term | What it means |
| --- | --- |
| Dogfight | A close fight between fighter aircraft (here, spacecraft), each trying to get behind the other and shoot. |
| Newtonian | Following Isaac Newton's laws of motion. In space there's no air, so a ship that is moving keeps moving until something pushes it the other way. See [Physics](physics.html). |
| Cinematic | Looking and feeling like a film: dramatic timing, framing and moments, even when reality would be duller. |
| Pillar | One of a few core rules a game is designed around. When two ideas clash, the pillars decide. |
| Exception | A deliberate, written-down place where the game breaks one of its own rules (usually physics) for the sake of a pillar. |
| 16-bit | The generation of game consoles from about 1988 to 1996, such as the Super Nintendo (SNES). Shorthand for their chunky, colorful pixel-art look. |
| Vertical slice | A short piece of a game finished to release quality, so people can judge the whole from it. |

</div>

## In one sentence

**Defender X is a story-first 2D space dogfighting game with a 16-bit look, where your ship stays fixed on screen and the universe turns around it, so that flat pixel art can feel like a cinematic, Newtonian dogfight.**

## What makes it different

- **The ship is fixed; the world rotates.** In most 2D shooters your ship moves around a still background. Here your ship sits in one place on screen and the whole universe turns and slides around it. Pulling the nose up swings the world down. This is what gives the game its sense of flying. See [Camera and coordinates](camera.html).
- **Free movement in 2D space.** The original *Defender* only scrolled left and right. Defender X lets you fly in any direction across a 2D level.
- **Fast, deadly weapons.** Bullets are meant to feel supersonic: streaks, not slow glowing dots.
- **A retro look with modern feel.** Ships and effects are drawn in big square pixels like an early-90s console, but flown with modern controls, physics and lighting.

## The pillars

These are the rules every design decision is checked against.

1. **Newtonian wherever possible; cinematic when there can be no compromise.** Ships obey real physics: they drift, keep their momentum, and have to burn their engines to stop. When physics and a dramatic, film-like feel genuinely conflict and no design can satisfy both, the cinematic choice wins.[^tenet]
2. **Exceptions are narrow and written down.** Every place the game bends physics is recorded with its reason, so it's a deliberate choice rather than a slow drift away from the rules. See the exceptions list on the [Physics](physics.html#exceptions-where-the-game-bends-physics) page.
3. **Weapons should feel fast and lethal.**
4. **World rotation creates an immersive dogfighting feel.**
5. **Force dogfights.** Circling another ship under g-forces should beat simply flipping around and firing backwards. The game rewards turning fights.[^starbuck]
6. **16-bit look, modern gameplay.** The overall mood is the 1980s to early 1990s.

## Inspirations

| Source | What it gives the game |
| --- | --- |
| *Defender* (Atari, 1981) | The name, the arcade energy, fast projectiles |
| Super Nintendo era games | The visual style: chunky pixels, bold color |
| *Battlestar Galactica* (2004) | The feel of the dogfights: momentum, flips, handheld-camera drama |
| *Ace Combat* series | Fighters circling each other, turning to get on the enemy's tail |

## Setting

A mix of space and planets. The first release focuses on space; planet environments come in later updates.

## The first target: the demo

The near-term goal is a **vertical slice of Level 1**, released as a demo during *Steam Next Fest* (February 22 to March 1, 2027). It is story-first: a real player's first experience has to tell a story, not just be an arcade loop.[^demo]

The bare minimum for the demo, ahead of everything else:

- the basic dogfighting loop,
- the opening cutscene,
- the AI for enemies and for the player's own side (wingmen),
- the art, sound and music all of that needs (a silent demo isn't an option).

Deliberately pushed until after the demo: the [Level Conductor](conductor.html), visual polish, and anything else not on that list.

## Open questions

- What exactly is the story, and how much of it is told in cutscenes versus during play?
- How long is a level, and how many are there in an episode?
- Who is the game for: arcade players, story players, simulation fans, or a mix?

[^tenet]: The full wording, from the g-forces design doc: "Newtonian wherever possible; cinematic when there can be no compromise. When physics and the cinematic feel genuinely conflict and no design satisfies both, cinematic wins." The same tenet is repeated in the cinematics notebook, the enemy AI docs and the Level Conductor research, so it really is the top rule.

[^starbuck]: A "Starbuck" is the move made famous in *Battlestar Galactica*: cut the engines, spin the ship around while it keeps drifting forward, and shoot at whoever is chasing you. Because the game is Newtonian, that move works. The g-force system (see [Physics](physics.html)) is designed so that circling is usually the stronger choice, and a "boost combo" rewards pilots who turn and then burn their engines.

[^demo]: The demo's beats, from the Episode 1 roadmap: a solemn title screen over a scorched Earth; a cutscene where the pilot is rocked awake and rushes to the fighter; launch from the carrier; holding the line against waves with wingmen while a jump timer counts down; dodging the carrier's own covering fire; racing back to land before the fleet jumps; and an ending, either the fleet escaping or, if you miss it, being left behind.
