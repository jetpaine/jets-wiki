---
layout: dx
title: Every enemy state
parent: AI layers
parent_url: /defender-x/ai.html
summary: A walk through a typical fight from the enemy's side, and a reference table of every goal an enemy can have and what makes it switch.
status: Built · being reworked
checked: 2026-10-27
---

<div class="glossary" markdown="1">

## Glossary

This page uses the terms from [AI layers](ai.html#glossary). The ones that matter most here:

| Term | What it means |
| --- | --- |
| State | One thing an enemy can be doing. It's always in exactly one **leaf** state. |
| Leaf state | A state with nothing nested inside it. Only leaf states actually fly the ship. |
| Outer state | A "folder" of states. It doesn't fly anything; it holds rules shared by everything inside it. |
| Radar range | 6,000 world units, a few screens away. Inside it, the enemy knows where you are. |
| Visual range | 1,500 world units, about a screen ahead of the enemy's nose. Inside it, the enemy can see you. |
| Standoff | Hanging back off to one side of the target, out of its guns, waiting for an opening. |
| Strafing run | Turning onto the target, thrusting in, and firing in bursts. |
| Swoop | A sudden burst on the afterburner, the enemy's dramatic dodge. |

</div>

## A typical fight, from the enemy's side

1. **Patrolling.** The enemy cruises between random points near where it appeared.[^patrol]
2. **Closing distance.** You show up on its radar. It heads for where you're *going* to be, at full speed.
3. **Engaging.** You come into view. It now loops through a three-step attack:
   1. *Holding standoff*: hang back to one side, out of your guns, for a random few seconds.
   2. *Strafing run*: turn in, thrust when lined up, fire in bursts.
   3. *Breaking off*: when it gets too close or the run times out, burn away across your path, then back to standoff.
4. **Evading.** Whenever it finds itself in your sights, it drops everything and flies straight until you fire, then swoops away on the afterburner. This is the cinematic beat: the dramatic dodge.
5. **Morale breaks.** Each zero costs one point of nerve. At zero nerve, whatever it was doing, it switches to retreating:
   - *Gaining distance*: run straight away. Fighters first get a short booster burst you can't match.
   - *Caught, quitter*: if you get it in your sights from behind, it flies straight at full thrust and waits for the end.
   - *Caught, fighter*: from any angle, it turns to face you and fires until one of you dies. It stops dodging you, too.
6. **Lost.** If you've been off its radar for 2 seconds, it goes back to patrolling.

At any moment, a predicted crash into a rock or another ship hands the controls to **avoiding obstacle**: the enemy swerves around it, or turns tail and burns to kill its speed if it's charging a rock head-on, then resumes what it was doing.

## The state map

Indentation shows nesting: each state sits inside the one above it.

```
Root
├── Avoiding obstacle          (interrupt: can cut in over everything)
└── Flying
    ├── Morale high
    │   ├── Patrolling
    │   └── On offensive
    │       ├── Closing distance
    │       ├── Engaging
    │       │   ├── Holding standoff
    │       │   ├── Strafing run
    │       │   └── Breaking off
    │       └── Evading
    └── Morale low
        ├── Patrolling
        ├── On retreat
        │   ├── Gaining distance
        │   └── Evading low       (quitter, caught)
        └── Last stand            (fighter, caught)
```

## Every state

| State | Sits inside | What the enemy does | Leaves when |
| --- | --- | --- | --- |
| Patrolling | Morale high (or low) | Cruises between random points near its spawn | Target on radar |
| Closing distance | On offensive | Heads for where the target will be, full speed | Target in view → engaging; in its sights → evading |
| Holding standoff | Engaging | Hangs back to one side; doesn't fire | Its random timer runs out → strafing run |
| Strafing run | Engaging | Turns onto the target, thrusts and fires in bursts | Too close, or timer out → breaking off |
| Breaking off | Engaging | Burns away across the target's line | Timer out → holding standoff |
| Evading | On offensive | Flies straight; swoops away the instant the target fires | Out of its sights and swoop done → closing distance |
| Gaining distance | On retreat | Runs straight away (fighters boost first) | Caught → its personality's end state |
| Evading low | On retreat | Quitter: flies straight, accepts its fate | Only if it loses the target |
| Last stand | Morale low | Fighter: turns on the target and fires; won't swerve from it | Loses the target → patrolling |
| Avoiding obstacle | Over everything | Swerves around, or turns and brakes | Path clear for a moment → back to what it was doing |

The outer states (morale high, on offensive, engaging, morale low, on retreat) don't fly the ship. They hold the shared rules, such as "morale broke → leave" and "lost the target → patrol".[^timers]

[^patrol]: Patrolling and "lost → patrol" are both marked in the design doc as going away in the rework (see "What's being reworked" on [AI layers](ai.html#whats-being-reworked)).

[^timers]: The timings, all in `GameConfig`, in ticks (120 per second): standoff lasts `STANDOFF_MIN_TICKS`–`STANDOFF_MAX_TICKS` (1–2.5 s); a strafing run at most `STRAFE_MAX_TICKS` (3 s); breaking off `BREAK_OFF_TICKS` (1.25 s); the swoop `SWOOP_TICKS` (0.6 s) at `ENEMY_AFTERBURNER_THRUST` (10× thrust); the fighter's breakaway boost `FIGHTER_BREAKAWAY_TICKS` (1.5 s). Enemies fire every `ENEMY_FIRE_RATE_TICKS` (10) in bursts of `ENEMY_BURST_TICKS` (0.4 s) with `ENEMY_BURST_PAUSE_TICKS` (0.5 s) between. The states themselves are objects in `ai/decision/States.kt`, one per box above.
