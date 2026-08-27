# Cinematic Systems Demo

Unreal Engine 5.8, Blueprint only. Greybox only — no third-party assets.

An interactive cinematic built around state integrity: control handoffs, camera blending, a branching choice, skip and replay, and streaming boundaries.

Clone, open, press Play. Nothing to install or download.

## Status

**0.2.2** — the scene plays end to end, both branches included.

The first act runs under full control: emergency lighting and an alarm come up on start, the AI reports the damage in on-screen text, the panel by the bay door refuses, the maintenance lever restores power, and the powered panel raises the door.

Crossing the trigger by the gate console gives the scene to the director — input suppressed, momentum cleared, character set on the marked spot, view blended into a twelve-second sequence cut across three cameras. When it ends, control returns and the two buttons on the console come alive: blue runs the standard gate cycle, red blows the seals. Each starts its own sequence and leaves its own mark on the world — the plate either stands open overhead or lies torn outside the gate, and the bay stays red with the alarm running.

`Enter` skips one sequence at a time, never the scene: from part A it hands over to the choice, from part B it forces the final state of the branch already picked. Every state transition prints to the screen log.

## Controls

Move and look as in the Third Person template. `E` acts on whatever the on-screen prompt names. `Enter` skips the running sequence.

## Levels

| Level | Type | Holds |
|---|---|---|
| `Content/Maps/L_Main` | Persistent | The ship: interior spaces and outer hull |
| `Content/Maps/L_Outside` | Streaming sublevel | The surface: ground and terrain, loaded while the player decides |

Greybox colour tracks the state an object is in and changes when that state does: orange is usable now, purple is dead until powered, green responds to the player, grey grid is shell and spent mechanisms. The two console buttons carry the branch colours instead — blue and red — because they offer a choice, not access.

Audio is generated at runtime by MetaSounds. The repository carries no sound files.
