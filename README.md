# Cinematic Systems Demo

Unreal Engine 5.8, Blueprint only. Greybox only — no third-party assets.

An interactive cinematic wired end to end: trigger, control handoff, camera blending, a branching choice, skip, and a streaming boundary.

Clone, open, press Play.

## Status

**1.0.0** — the scene is complete: both branches, skip and replay, the streaming boundary.

The first act runs under full control. Emergency lighting and an alarm come up on start, the AI reports the damage in on-screen text, and the panel by the bay door refuses. The maintenance lever restores power, and the powered panel raises the door.

Crossing the trigger by the gate console hands the scene to the director: the view blends into a twelve-second sequence cut across three cameras. When it ends, control returns and the two buttons on the console come alive: blue runs the standard gate cycle, red blows the seals. The AI names what each costs before you choose. Each starts its own sequence and leaves its own mark on the world. The plate either stands open overhead, or lies torn outside the gate, where the bay stays red with the alarm running.

Every state transition prints to the screen log.

## Controls

Move and look as in the Third Person template. `E` acts on whatever the on-screen prompt names. `Enter` skips the running sequence.

## Levels

| Level | Type | Holds |
|---|---|---|
| `Content/Maps/L_Main` | Persistent | The ship: interior spaces and outer hull |
| `Content/Maps/L_Outside` | Streaming sublevel | The surface: ground and terrain |

Greybox colour tracks state, not type: orange is usable now, purple is dead until powered, green responds to the player, grey grid is shell and spent mechanisms. The two console buttons carry the branch colours instead — blue and red.

Audio is generated at runtime by MetaSounds. The repository carries no sound files.
