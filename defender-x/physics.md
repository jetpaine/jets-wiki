---
layout: dx
title: Physics and g-forces
summary: How ships move in Defender X, why they keep drifting when the engine stops, and what happens to a pilot who turns too hard.
status: Built · blackout planned
checked: 2026-10-27
---

<div class="glossary" markdown="1">

## Glossary

| Term | What it means |
| --- | --- |
| Physics | Here: the code that moves things each [tick](./#start-here-glossary), and works out what happens when they collide. |
| Velocity | Speed *and* direction together: "8 units per tick, heading north-east". |
| Momentum | The tendency of a moving thing to keep moving the way it's going. Heavier or faster means more momentum. |
| Thrust | The push from a ship's main engine. It always pushes the way the nose points. |
| Newtonian | Following Newton's laws: nothing slows down or turns unless something pushes on it. There's no air in space to slow you. |
| Spin / angular velocity | How fast the ship is turning, in radians per tick. |
| Grip | How much a ship's path bends toward where its nose is pointing. **0** is pure space physics (the path ignores the nose); **1** is like a car on a road. Defender X uses 0. |
| Speed cap | The fastest a ship is allowed to go. Real space has none; the game adds one. |
| g / g-force | How hard the pilot is pushed into their seat, measured in multiples of Earth's gravity. 1g is standing still on Earth; fighter pilots black out somewhere around 9g. |
| Greyout | A pilot's vision fading to grey and dimming under sustained g, a real effect of blood draining from the head. |
| Blackout | The next stage: vision goes completely, and the pilot can lose consciousness. |
| Limiter | A safety system that refuses control inputs that would push the g-force past a set limit. |
| Exception | A deliberate, written-down place where the game bends real physics for the sake of feel. |

</div>

## Flying in space

Ships in Defender X fly by **Newtonian** rules. Three controls do almost everything:

- **Thrust** (W or Up turns the engine on, S or Down turns it off). While it's on, the engine pushes the ship forward, the way the nose points, a little more every tick.[^thrust]
- **Nose up / nose down** (A and D, or the arrow keys). Small side thrusters spin the ship. The longer you hold, the faster it turns, up to a limit.
- **Stabilizers** (Z). Hold to fire thrusters against your spin and slow it down.

The important part is what *doesn't* happen. **Turning the nose doesn't change which way you're moving.** If you're flying east and spin to face north, you keep sliding east, now sideways, until the engine has pushed long enough to change your course. Cut the engine and you coast forever at the same speed. This is how real spacecraft behave, and it's what makes the famous "flip and fire backwards" move possible.[^grip]

The game simulates this in fixed steps, 120 per second, each one adding the engine's push to the velocity and then the velocity to the position.[^tick]

### The speed cap

Real space has no speed limit, but the game does: ships can't go faster than their top speed. It's the oldest deliberate break from physics in the game, and it does useful work. It keeps fights at a readable pace, and because thrust can't add speed past the cap, holding the throttle at top speed costs the pilot nothing (see g-forces below).

### Collisions

When two things touch, the game works out exactly where their outlines overlap and pushes them apart like real objects, with the right amount of bounce and spin for their mass and shape.[^collision]

### The boost combo

A move borrowed from fighting games: with the engine on and no turn key held, **start a turn and then press Up within 0.3 seconds**. That lights a boost: more than five times the thrust for up to 3 seconds, and the ship can't be hit while it lasts. It rewards the turn-and-burn style the game wants, and enemies can do it too, because they press the same virtual keys.[^boost]

### Reversals

Enemies whose nerve breaks can flip end-over-end to flee: the ship swings its nose toward the camera and out the other side, over about a second, and is then facing the other way. The physics stays 2D and the ship's motion isn't changed; only its facing and how it's drawn change. The player's own reversal is planned next.[^reversal]

## G-forces

Pure space physics has a problem for a dogfighting game: the maneuvers the game cares about **feel the same** to the pilot. Thrust always presses you into your seat with the same force, and spinning the nose doesn't curve your path, so a hard turn produces no g at all. The g-force system fixes that, carefully, so it can limit how hard ships turn and create the drama of greying out.

### How g is measured

Each tick, for every ship (player and enemies alike), the game adds up two things:[^gmodel]

1. **Thrust g (real).** How much the ship's velocity actually changed this tick. Thrusting from a standstill is about 2g. Coasting is 0g. Thrusting at top speed is about 0g too, because the speed cap absorbs the push. Burning hard **against** your own momentum (the "flip and burn" to stop) counts extra, up to 2.5 times.
2. **Turn g (an exception).** Speed × spin rate. That's the g you would feel if your path really curved at that rate. It only counts **while the engine is on**, so a coasting flip is still free.

The two are combined the way two forces at right angles combine (the square root of the sum of their squares) into one number.

### What g does

**The limiter.** If turning harder would push turn g past the limit, the turn keys simply stop working in that direction, and if you're already over it, an autopilot gently eases the spin back. Turning *less* always works, and so does the engine. A red **G-LIMIT** warning flashes the moment input is refused, so it never feels like broken controls. At top speed this means a full circle takes roughly three times as long as at low speed: fast ships turn wide, which pushes fights into circles.[^limit]

**Greyout.** Hold a turn near the limit for about two seconds and the pilot's vision starts to go: over three seconds the screen dims, drains of color, darkens at the edges and blurs. Ease off and it comes back. Even if you keep holding, the pilot adapts and vision returns, but only once per turn.[^greyout]

**Blackout (planned).** Too much g for too long would black the pilot out: the screen goes dark and time slows, with the pilot still in control, in slow motion. Designed, not built yet.

### The maneuvers, compared

| Maneuver | What the pilot feels |
| --- | --- |
| Coasting flip (engine off) | Nothing. 0g. The classic "Starbuck" move is free. |
| Thrusting straight at top speed | About 0g: the speed cap absorbs it. |
| Thrusting from a standstill | About 2g. |
| Hard turn under power at top speed | Far over the limit: the limiter caps the turn. |
| Flip, then burn against your momentum | Up to about 5g, so it can grey you out. |

## Exceptions: where the game bends physics

Every departure from real physics is written down with its reason, per the [second pillar](mission.html#the-pillars).

| # | Exception | Why | Status |
| --- | --- | --- | --- |
| — | **Speed cap.** Ships have a top speed. | Readable fights; and it makes thrusting at top speed cost no g. | Built |
| E1 | **Turn g.** The pilot feels a turn their path doesn't actually make. | Without it, turning produces no g and the limiter has nothing to act on. | Built |
| E2 | **Reversal multiplier.** Burning against your momentum counts extra g. | So the flip-and-burn can grey you out. | Built |
| E3 | **Enemy swoop exempt.** An enemy's afterburner dodge doesn't count toward its g. | The swoop is about 15g and would black the enemy out every time, but it's the key cinematic dodge. | Decided |
| E4 | **Powered grip.** Thrust bends your path toward the nose. | Tried in playtest and rejected; grip 0 felt right. | Rejected |
| E5 | **Boosting ships can't be hit.** Bullets pass by, drawn over or under the hull. | In-universe, a ship mid turn-and-burn is too hard to hit. | Built |

[^thrust]: Thrust is a **toggle**, not a held key: it starts on, Up switches it on and Down switches it off (`engine/InputHandler.kt`, `engine/ShipControls.kt`). The player's numbers, all in `GameConfig`: thrust `PLAYER_THRUST_POWER` = 0.08 world units per tick per tick, top speed `PLAYER_MAX_SPEED` = 8 world units per tick, turning `PLAYER_TORQUE_POWER` = 0.0009 and top spin `PLAYER_MAX_ANGULAR_SPEED` = 0.03 radians per tick. From a standstill it takes 100 ticks to reach top speed. Enemies are a little slower (`ENEMY_MAX_SPEED` = 6.5).

[^grip]: `PLAYER_GRIP` and `ENEMY_GRIP` are both 0.0. The comment beside them describes the scale: 0.5 halves the sideways slip every second, 0.99 is "snappy, aircraft-like", 1.0 locks the path to the nose. A debug key (G or F5) cycles through grip presets so the feel can be compared in play.

[^tick]: `game/PhysicsState.kt`, `step()`: clamp speed to the cap, ease any spin over the top rate back down, apply grip, advance any reversal, then add velocity to position and spin to angle. `GameConfig.SIM_TICK_SECONDS` is 1/120. `GAME_SPEED` (0.8) slows the whole clock to 80%: every tick behaves the same, there are just fewer per real second. Because every tick is identical, a game started with the same random seed and the same inputs plays out exactly the same way every time. The tests rely on this: "golden replays" fly scripted games and check that the result matches a saved recording exactly.

[^collision]: Collision outlines are each ship's convex hull (the shape a rubber band would make stretched around the sprite). Overlaps are found with the Separating Axis Theorem (SAT) and resolved with impulses that take each body's mass, moment of inertia (resistance to spinning, worked out from the sprite's size) and restitution (bounciness, 0.4 by default) into account. See `engine/CollisionSystem.kt`, `CollisionManifold.kt` and `CollisionResponse.kt`.

[^boost]: `entity/BoostCombo.kt`. The rules: no turn key held; press and hold a turn key; within `BOOST_COMBO_WINDOW_SECONDS` (0.3) press Up, with thrust already on. Thrust is multiplied by `BOOST_COMBO_THRUST` (5.5) for up to `BOOST_COMBO_MAX_SECONDS` (3.0), or until thrust is switched off. Pressing Up on the same tick as the turn, letting go of the turn first, or pressing too late all reset it. When it ends, top speed eases back down rather than snapping.

[^reversal]: Issue #34. `PhysicsState.startReversal` and `advanceReversal`: a constant yaw acceleration for the first half and deceleration for the second. At the halfway point (nose straight at the camera) the body swaps to its mirror image, which can't be seen at that moment. How it's drawn is covered by "reversal animations" in the README.

[^gmodel]: `physics/g/GLoadModel.kt` (pure math), `GLimiter.kt` (filters the controls before physics), `PilotG.kt` (measures after physics and holds each ship's state between ticks) and `GreyoutState.kt`. They plug into `Craft.applyControls`, the one path from keys to physics, so the player and enemies obey identical rules. `G_UNIT` = 0.04 world units per tick² counts as 1g. `G_REVERSE_FACTOR` = 1.5 gives the reversal extra (up to 1 + 1.5 = 2.5×).

[^limit]: `G_TURN_LIMIT` = 2.0 in the model's own units. The HUD multiplies by `G_DISPLAY_SCALE` (4) so the limit reads as **8G**, about the real human limit. At 2.0, a powered turn at top speed is capped at 0.01 radians per tick, a third of the full rate; below speed 2.7 the full rate is allowed. The autopilot (`G_AUTOPILOT_STRENGTH` 1.0) brakes an over-limit spin as hard as the pilot's own thrusters could, getting back to the limit in about a quarter of a second. Enemies are currently exempt from the limiter until the AI is retuned for it. Note that the original design doc spoke of a `G_LIMIT` of 4g with a tolerance of 3.5g; the built version settled on the numbers above.

[^greyout]: `GreyoutState`: holding a turn key with turn g at `GREYOUT_ONSET_FRACTION` (75%) of the limit or more, without a break, for `GREYOUT_DELAY_SECONDS` (2) starts a fade lasting `GREYOUT_FADE_SECONDS` (3); recovery takes `GREYOUT_RECOVER_SECONDS` (3). The look itself (dim, desaturate, vignette, blur) is a screen effect, `fx/GreyoutEffect.kt`; see [Rendering layers](rendering.html).
