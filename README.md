# Cinematic Systems Demo

[![The gate console at the moment of the choice](Media/header.png)](https://www.youtube.com/watch?v=j-VVgqHQgfM)

[Watch the demo](https://www.youtube.com/watch?v=j-VVgqHQgfM) — the run, the other branch, and the edge cases.

Unreal Engine 5.8, Blueprint only. Greybox geometry and the Third Person template: the work is in the systems.

An interactive cinematic wired end to end: trigger, control handoff, camera blending, a branching choice, skip, and a streaming boundary.

## The scene

The first act runs under full control aboard the landed ship. Emergency lighting and an alarm come up on start, the AI reports the damage in on-screen text, and the panel by the bay door refuses. The maintenance lever restores power, and the powered panel raises the door.

The second act begins when the player crosses the trigger by the gate console: the scene passes to the director, and the view blends into a twelve-second sequence cut across three cameras. When it ends, control returns and the two buttons on the console come alive: blue runs the standard gate cycle, red blows the seals. The AI names what each costs before the choice is made. Each starts its own sequence and leaves its own mark on the world. The plate either stands open overhead, or lies torn outside the gate, where the bay stays red with the alarm running.

Control comes back when the branch has played. Every state transition prints to the screen log.

## Where to look

| What | Where |
|---|---|
| State machine, control handoff, skip | `Content/Blueprints/BP_CinematicDirector` |
| The three sequences and their event tracks | `Content/Cinematics/` |
| Interaction: lever, panel, console buttons | `Content/Blueprints/`, through `BPI_Interactable` |
| Subtitles, hints and the action prompt | `Content/UI/WBP_HUD` |

Architecture, the director's graph in full, and the reasoning behind the choices that have obvious alternatives: [TECHDOC.md](TECHDOC.md).

## Controls

| Action | Key |
|---|---|
| Move | `W` `A` `S` `D` |
| Look | mouse |
| Jump | `Space` |
| Interact | `E` |
| Skip the running sequence | `Enter` |

`E` acts on whatever the on-screen prompt names. A skip drops the sequence, not the scene: the run still ends where the branch would have left it.

## Colour code

Greybox colour tracks state, not type, and changes when the state does:

| Colour | Means |
|---|---|
| Orange | usable now |
| Green | opens once the control is used |
| Purple | dead until powered |
| Grey grid | shell, and mechanisms already spent |
| Blue and red | the two branches at the console |

Nothing stays orange once it can no longer be used.

## Levels

| Level | Type | Holds |
|---|---|---|
| `Content/Maps/L_Main` | Persistent | The ship: interior spaces and outer hull |
| `Content/Maps/L_Outside` | Streaming sublevel | The surface: ground and terrain |

Audio is generated at runtime by MetaSounds. The repository carries no sound files. Assets are stored with Git LFS.

Aleksei Petrovskii — alexeypetrowski@gmail.com
