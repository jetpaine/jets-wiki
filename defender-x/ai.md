---
layout: dx
title: AI layers
summary: How every enemy pilot notices, decides and flies, why it can't cheat, and where the planned Level Conductor sits above them all.
status: Built · rework planned
checked: 2026-10-27
---

<div class="glossary" markdown="1">

## Glossary

Terms are in reading order. The [Every enemy state](ai-states.html) sub-article uses the same ones.

| Term | What it means |
| --- | --- |
| AI | "Artificial intelligence". In games it just means code that controls a character. It isn't the same thing as chatbots; there's no learning involved, only rules. |
| Agent | One thing the AI controls: here, one enemy ship and its pilot. |
| Layer | One part of the AI with one job, which only talks to the layers next to it. Layering keeps each part simple. |
| Tick | One step of the game clock, 1/120 of a second. The AI thinks once per tick. |
| Percepts | The short list of facts the sensing layer writes down each tick, such as "the player is on radar" or "I'm in their sights". The other layers read only this list, never the game world itself. |
| State | One thing an enemy can be doing, like *patrolling* or *strafing run*. It's in exactly one at a time. |
| Transition | A rule for leaving a state: "if *this* becomes true, switch to *that*". |
| State machine | A set of states and the transitions between them. Picture a flowchart the enemy is always standing on one box of. |
| Hierarchical state machine (HSM) | A state machine whose states are nested inside bigger states, like folders. A rule on an outer folder covers everything inside it. |
| Behavior | The flying routine for one state, e.g. "turn onto the player, thrust when lined up, fire in bursts". |
| Behavior tree (BT) | Another way to write a behavior: a tree of small steps read top to bottom each tick, trying alternatives until one works. Good for step-by-step routines with fallbacks. |
| Virtual keys | The enemy's "keyboard": the same turn, thrust and fire controls you have. |
| Blackboard | A small notepad each enemy keeps for things that last across ticks, like which side it's flanking on. |
| Nose cone | A narrow wedge (about 5°) straight ahead of a ship, out to about a screen's distance. A ship inside another's nose cone is "in its sights". |
| Zero | Firing while an enemy is in your nose cone. Each one costs the enemy some nerve. |
| Morale / nerve | An enemy's will to fight. Its nerve drops with each zero; when it runs out (3 by default), its morale breaks and it stops attacking. |
| Fighter / quitter | The two enemy personalities. When caught after breaking, a **fighter** turns for a last stand; a **quitter** flies straight and accepts its fate. |
| Conductor | A planned AI layer *above* the pilots that paces the whole fight like a film director. See [Level Conductor](conductor.html). |

</div>

## The big picture

The AI in Defender X is a stack of layers. Each layer answers a different question, at a different scale of time, and passes its answer down.

<ol class="stack plain">
  <li><span><strong>Acting</strong>: <em>How do I fly this?</em> Presses the virtual keys, many times a second</span> <span class="kind">built</span></li>
  <li><span><strong>Deciding</strong>: <em>What am I trying to do?</em> Picks the current goal, changes every few seconds</span> <span class="kind">built</span></li>
  <li><span><strong>Sensing</strong>: <em>What do I notice?</em> Writes down the facts, every tick</span> <span class="kind">built</span></li>
  <li><span><strong>Level Conductor</strong>: <em>What should this fight feel like right now?</em> Paces threats and beats across minutes</span> <span class="kind">planned</span></li>
  <li><span><strong>Level script</strong>: <em>What is this level about?</em> Written by the designer: phases, objectives, timers, waves</span> <span class="kind">planned</span></li>
</ol>

Reading from the top down: the designer's **level script** sets the goals and the options; the **Conductor** watches play and picks when and which; and each enemy's three-part **pilot** plays its part. Information flows down. The pilots don't know the Conductor exists, and the Conductor doesn't steer any ship directly.[^layers]

The rest of this page is about the pilot, which exists today. The Conductor has [its own page](conductor.html).

## The pilot: three parts

Every enemy is flown by the same three-part pilot. Think of a fighter pilot's head.[^files]

### 1. Sensing: what the pilot notices

Every tick, each enemy takes stock. Is its target on radar? Close enough to see? Am I in its sights? Am I about to hit a rock? It doesn't act on any of this; it just writes down the **percepts**.

Its target is **the nearest ship of another team**, so the same AI can fly wingmen on your side, and two AI sides can fight each other (that's how the background battles work).

Sensing is deliberately imperfect, to make enemies feel like people:

- **Reaction delay.** Enemies see their target where it was a moment ago (a tenth to a fifth of a second), so a sharp juke can fool them. Reflexes, like noticing they're in your sights, stay instant.
- **Dizzy.** After spinning hard, their sense of where you are wobbles for a while.
- **Fixated.** While watching you, they can ignore threats from behind.
- **Distracted.** After dodging a rock, they can lose track of you for a few seconds.

### 2. Deciding: what the pilot is trying to do

The deciding part reads the percepts and keeps **one current goal**, such as "chase the player", "line up a strafing run" or "run away". It's a **hierarchical state machine**: goals nested inside bigger goals. "Strafing run" sits inside "engaging", inside "on the offensive", inside "morale high". That nesting means one rule on an outer goal, such as "morale broke → retreat", pulls the enemy out of every attacking goal at once, without repeating the rule on each one.

Two safeguards keep decisions steady:

- **Dwell time.** A goal is held for at least a quarter of a second before its own rules can switch it, to stop flickering. Urgent goals like evading skip this.
- **Interrupts.** *Avoiding obstacle* can cut in over everything when a crash is predicted, then hand back to whatever the enemy was doing.

The full list of goals and what switches them is in **[Every enemy state](ai-states.html)**.

### 3. Acting: how the pilot flies

Each goal has a flying routine, a **behavior**, that presses the ship's **virtual keys**: turn left, turn right, thrust, fire. **Enemies press the same keys you do, so they obey exactly the same physics**, g-forces included. They can't cheat by teleporting or turning faster than their ship allows.

Most behaviors are short pieces of Kotlin code. The first one has now been rebuilt as a **behavior tree** that's loaded from a file and can be edited in a visual drag-and-drop tool, as a trial for building more behaviors that way.[^bt]

## Morale and personality

On top of the three parts sit two ideas that give fights a shape:

- **Morale.** Each time you catch an enemy squarely in your sights and fire, it loses nerve. With nerve gone, it stops attacking and runs.
- **Personality.** Half of enemies are **fighters** and half are **quitters**. Caught after breaking, a fighter turns to face you and fights to the death; a quitter flies straight at full thrust and accepts its fate. Fighters also react twice as fast and turn faster.

The guiding rule for every behavior: **choose the cinematic move over the realistic one**. A dramatic swoop away the instant you fire beats a cleverer but duller dodge.

## What's being reworked

Playtesting showed problems with the current pilots:[^problems]

- **Little actual fighting.** An enemy on screen almost always ends up in your nose cone, which flips it to *evading*, so fights rarely develop unless you're swarmed.
- **Fleeing enemies cut their engines.** Physically sensible (they're already at top speed) but it leaves no vapor trail to chase.
- **Twitchy decisions.** With nothing locking in a plan, every new fact causes an immediate change of mind.
- **Nothing shapes the fight as a whole.** Each enemy runs its own loop; nobody paces the threat into build-up, peak and relief. That gap is what the **[Level Conductor](conductor.html)** is for.

The gameplay itself is also changing, with the **turn-and-burn** boost and **near misses** (shots and ships passing very close) becoming central, so the enemy AI is being rethought from the ground up around them. The three-part pilot, morale, personalities, "cinematic over realistic" and "same keys as the player" all stay.

## Tools for watching the AI

- **Debug overlay** (F3): draws each enemy's vision cone, predicted collisions, current state, nerve, held keys and g-force breakdown over the game.
- **AI log**: set `AI_LOG` to 1 in TUNING and every decision is written to a file for later reading.
- **Automatic diagrams**: a build task reads the AI's code and draws its state map, highlighting what changed against the main version, so the diagram can never fall out of date.[^diagrams]

[^layers]: The layering comes from the Level Conductor research: in every example studied (Left 4 Dead, Alien: Isolation, Halo), the director sits between the hand-made level and the actors, and "fills in what the script leaves open". The level script layer is the planned **M1 framework**: phases, triggers, objectives, timers and waves, probably written as a small Kotlin "domain-specific language" (DSL), a mini-language built inside Kotlin.

[^files]: All under `src/main/kotlin/defenderx/ai/`. Sensing: `sensing/Sensor.kt`, `Percepts.kt`, `WorldSnapshot.kt`; the faults are in `decision/PerceptFilters.kt`. Deciding: `decision/EnemyBrain.kt` runs the state machine defined in `decision/States.kt`, with `decision/Avoidance.kt` for crash prediction. Acting: `behavior/Behaviors.kt`, with shared steering math (`turnToward` and friends) in `Steering.kt`. Timings in `GameConfig`: `QUITTER_REACTION_TICKS` 24 and `FIGHTER_REACTION_TICKS` 12 (0.2 s and 0.1 s), `MIN_DWELL_TICKS` 30, `DIZZY_TICKS` 90, `DISTRACTION_TICKS` 180, `LOST_TARGET_TICKS` 240, `ENEMY_FIGHTER_RATIO` 0.5.

[^bt]: `ai/bt/`: a small behavior-tree runtime that reads trees in the XML format saved by **Groot2**, a free visual editor for behavior trees (format: BehaviorTree.CPP v4). Trees live in `assets/ai/trees/`; so far there's one, `strafe_run.xml`, which flies the *strafing run* state, with the original Kotlin version kept as its reference. The editor only arranges nodes: each condition and action ("turn toward target", "if lined up, fire") is still Kotlin. `./gradlew btPalette` writes the list of available nodes for Groot2. A trap to watch: trees that re-check every condition each tick react fast but can bring back the twitchiness, unless branches lock in for a while.

[^problems]: From the "Enemy AI: How It Works and What Changes" design doc, Problems section, and the follow-up "Enemy AI: Rework for the New Gameplay".

[^diagrams]: `./gradlew aiDiagrams`, code in `src/aitools/`. It reads the state definitions straight from the Kotlin source and renders them with Graphviz (`tools/graphviz/`). It only runs one way, code → diagram: editing a diagram doesn't change the game. `GameConfig.AI_LOG` = 1 writes `ai-log.jsonl`.
