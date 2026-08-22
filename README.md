# Cinematic Systems Demo

Unreal Engine 5.8, Blueprint only. Greybox only — no third-party assets.

An interactive cinematic built around state integrity: control handoffs, camera blending, skip and replay, save/load, and streaming boundaries.

Clone, open, press Play. Nothing to install or download.

## Status

**0.2.1** — the first act plays end to end and hands off to the cinematic. Emergency lighting and an alarm come up on start, the AI reports the damage in on-screen text, the panel by the bay door refuses, the maintenance lever restores power, and the powered panel raises the door.

Crossing the trigger by the gate console gives the scene to the director: input is suppressed, momentum cleared, the character set on the marked spot, and the view blends into a twelve-second sequence cut across three cameras. Control comes back at the end of it. Every state transition prints to the screen log.

## Controls

Move and look as in the Third Person template. `E` acts on whatever the on-screen prompt names.

## Levels

| Level | Type | Holds |
|---|---|---|
| `Content/Maps/L_Main` | Persistent | The ship: interior spaces and outer hull |
| `Content/Maps/L_Outside` | Streaming sublevel | The surface: ground and terrain, loaded on demand |

Greybox colour tracks the state an object is in and changes when that state does: orange is usable now, purple is dead until powered, green responds to the player, grey grid is shell and scenery.

Audio is generated at runtime by MetaSounds. The repository carries no sound files.
