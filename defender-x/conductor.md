---
layout: dx
title: Level Conductor
summary: A planned "meta AI" that watches a level as it plays and shapes its key moments, so each run feels shot like a film.
status: Research · after the demo
checked: 2026-10-27
---

<div class="note" markdown="1">
<span class="note-label">Not built yet</span>
The Conductor is at the research stage, and it's deliberately pushed until after the February 2027 demo. "Level Conductor" is a working name. This page describes the idea, where it came from, and what's been decided so far.
</div>

<div class="glossary" markdown="1">

## Glossary

| Term | What it means |
| --- | --- |
| Conductor / director | An AI that controls the *level*, not any one character: when enemies arrive, how many, what happens next. Valve's *Left 4 Dead* (2008) called its version the **AI Director**. |
| Pacing | The rhythm of a level: when it's intense, when it's quiet, and how those alternate. |
| Intensity | A rough number for how stressed the player is right now. A director watches it to decide whether to push or back off. |
| Level scripting | The hand-written rules that make a level's events happen: "when the player reaches here, spawn a squad". |
| Trigger | A condition that fires an action: the player enters an area, a timer runs out, a target dies. |
| Phase | One stretch of a mission with its own goal, such as "hold the line until the jump". |
| Wave | A group of enemies sent in together. |
| Set piece | A big, hand-designed moment, like a bridge collapsing or a capital ship arriving. |
| Rubber-banding | Quietly helping whoever is behind and holding back whoever is ahead, to keep a race or fight close. |
| Off screen / on screen | Here, a rule of honesty: what the player can see must obey the game's physics; what they can't see is the Conductor's to arrange. |

</div>

## Why a Conductor

Today, each enemy pilot runs its own loop (see [AI layers](ai.html)). Nothing shapes a fight *as a whole*: nobody decides that now is the moment for an enemy to swoop across your nose, for a shot to land just off your bow, for a wingman's radio call, or for the pressure to let up so you can breathe. Fights can be busy without being dramatic.

The Conductor is meant to fill that gap. It serves the first [pillar](mission.html#the-pillars): Newtonian wherever possible, cinematic when there can be no compromise.

## Where it sits

The Conductor is a layer between the designer's level script and the enemy pilots:

- **The designer writes the backbone and the options.** The level's phases, objectives and set pieces, and a menu of possible spawn points, wave types and moments.
- **The Conductor reads the player and sets the timing.** It picks *when* and *which* from those options. It never decides what the level is about.
- **The pilots play it out.** Each enemy still senses, decides and flies for itself, with the same physics as you.

Information only flows down. At the big moments (a finale, a set piece) the level script can retune the Conductor's limits for that stretch rather than switch it off.[^layers]

## Where the idea comes from

### Left 4 Dead's AI Director

The best-known example. Its key insight, in designer Mike Booth's words: it changes **how often** threats come, never **how strong** they are.[^l4d]

- **It reads the players.** Each player has one "intensity" number. It rises when they take damage or see an enemy die close by, and slowly falls back toward zero when they're safe.
- **It runs a four-beat rhythm on that number:**
  1. **Build up:** send threats until intensity crosses a peak.
  2. **Sustain:** keep it up for 3–5 more seconds.
  3. **Fade:** stop adding threats and let the current fight play out.
  4. **Relax:** almost nothing for 30–45 seconds. Then build up again.
- **Bosses sit outside the rhythm**, dealt out along the route at the start, so they can break the pacing on purpose.
- **It only works near the players, out of sight.** Most new enemies arrive from behind, where you've already been.
- **Designers place the options; the Director picks which exist.** A designer puts many possible weapon caches in a map; each run, the Director decides which appear. Booth called the result "structured unpredictability".

### Other examples

- **Halo** split combat into two time scales: code owns the 30-second scale (good reactions); designers own the 3-minute scale (what the battle is about, when to fall back).[^halo]
- **Alien: Isolation** splits knowledge: its director always knows where the player is and nudges the alien toward them; the alien itself has to find the player with its own senses, so it can still be fooled. The result feels lucky, not rigged.[^alien]
- **Left 4 Dead 2's dialogue** uses the same idea: lines are picked from rules matched against facts about the moment, the most specific match winning.

## What it might mean for Defender X

These are early observations, not designs.

- **The goal is different from Valve's.** Left 4 Dead's Director exists for replay value. Defender X is single-player and story-first, so its Conductor is more about making the **authored beats land**: the last wave arriving just as the jump timer runs low, or the race back to the carrier ending close.
- **"Cinematic" means more than intensity.** It might pace how things *look*, too: the swoop across your nose, the near miss, the radio call, the music and screen effects at the peak.
- **The game already measures a lot.** G-load and greyout, hull damage, enemies zeroed and their morale, kills, distance to the carrier, time left. Any of these could feed a Defender X version of "intensity".
- **Several Episode 1 levels already lean on one.** Waves until the jump (Levels 1, 5 and 10); a rubber-banding rival in a race (Level 4); a stealth search much like Alien: Isolation's (Level 7); a phantom ship that has to keep pace and taunt at the right moments (Level 11).

## Decided so far

| Question | Decision |
| --- | --- |
| Can it cheat physics? | **What's on screen stays honest; what's off screen is the Conductor's.** Borrowed from Left 4 Dead's out-of-sight spawning. Anything that bends physics goes in the exceptions log. |
| Should it adjust difficulty, or only pacing? | **It adapts to skill.** Like Level 4's rubber-banding rival, it keeps fights always a little close. |
| How much can it invent? | **It mostly chooses among options the designer placed**, rather than inventing new events. |
| Can it direct cutscene-like moments in gameplay? | **Mostly no.** Those belong to scene scripting; the Conductor may share some of the work. |
| Should players notice it? | **Subtle enough that a new player won't.** Fine if experienced players do. Catch-up that erases a clean, hard-won lead at the last second can feel unfair. |

## Still open

- Is it one system, or several smaller directors (waves, radio chatter, music, effects)?
- What is Defender X's "intensity": one number, or several (danger, spectacle, story progress)?
- Does it live inside each mission phase, the way Left 4 Dead 2's director scripts retune the Director for a finale?

[^layers]: In engines generally, a level is **data plus a runtime**: a file listing the level's things (each with a type, position, settings and connections to other things), plus a small program that watches each trigger and fires the connected actions. Designers fill the file in with an editor. Defender X's planned version is the **M1 framework**: phases, triggers, objectives, timers and waves, most likely written as a Kotlin DSL (a mini-language built inside Kotlin) rather than data files.

[^l4d]: Mike Booth (Valve), *The AI Systems of Left 4 Dead*, AIIDE 2009 keynote ([slides, PDF](https://steamcdn-a.akamaihd.net/apps/valve/2009/ai_systems_of_l4d_mike_booth.pdf)). Technical detail from the slides: spawning is limited to the navigation areas around the team (the "Active Area Set"), so a handful of reused characters can stand in for hundreds; a "flow distance" along the escape route measures progress; mobs arrive every 90–180 seconds on Normal.

[^halo]: Chris Butcher and Jaime Griesemer (Bungie), [*The Illusion of Intelligence*](https://www.jmeiners.com/shamans/papers/ai/the_illusion_of_intelligence.pdf), GDC 2002; and [*Combat Evolved: The Encounter Design of Halo 3*](https://www.gamedeveloper.com/design/combat-evolved-the-encounter-design-of-halo-3). In Halo 3, designers wrote a priority-ordered list of tasks; squads claim the highest open task and fall back down the list as tasks finish.

[^alien]: Tommy Thompson, [*The Perfect Organism: The AI of Alien: Isolation*](https://www.gamedeveloper.com/design/the-perfect-organism-the-ai-of-alien-isolation). Its director tracks a "menace gauge" and sends the alien away into the vents when menace stays high too long; the article calls the effect "psychopathic serendipity".
