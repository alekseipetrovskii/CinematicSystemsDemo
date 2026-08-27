# Technical Overview

Blueprint-only demo of an interactive cinematic. This document covers the architecture and the reasoning behind the choices that have obvious alternatives.

## Shape of the scene

The scene runs in three beats, not one:

1. **Part A** — a twelve-second sequence, control suppressed.
2. **The choice** — a gameplay phase. Control returns in full, two buttons on the gate console come alive, and the scene waits as long as the player needs.
3. **Part B** — one of two sequences, control suppressed again.

Parts A and B are separate assets, so nothing is paused between them: part A ends, the choice happens, part B starts. This is why the middle beat is ordinary gameplay rather than a sequence with partial input — no half-suppressed state exists anywhere in the scene.

## State machine

`E_CinematicState`: `Idle → Arming → Blending_In → Playing_A → Interactive_Beat → Playing_B → Blending_Out → Completed`.

`BP_CinematicDirector` owns it. `SetState` is the only writer and prints every transition to the screen log, so a recording of any run doubles as a trace.

`Interactive_Beat` is the only state where the player holds control. `Completed` is terminal within a session: entry requires `Idle`, so a second overlap does nothing.

## Director

| Function | Does |
|---|---|
| `TakeControl` | suppresses input, clears momentum, places the character on the entry mark, gives the director its own input |
| `ReleaseControl` | blends the view back to the character and returns input |
| `PlaySequence` | records the active sequence actor, binds the finish delegate, starts playback |
| `GoToBeat` | enters the choice phase and wakes the console after the view settles |
| `ApplyFinalState` | forces the end state of the picked branch: plate position, bay lighting, alarm |
| `FinishCinematic` | starts the exit blend and hands off to a timer |

Blend phases use timers rather than `Delay`. A `Delay` cannot be cancelled: a skip arriving mid-blend would be overtaken by it, and part A would start on top of the choice phase. Timer handles are cleared inside the branch that knows what it is cancelling — clearing before deciding once killed the exit timer and stranded the scene between `Blending_Out` and `Completed`.

## Skip

`Enter`, not `Space`: the choice phase has full control, and `Space` is jump there.

A skip drops one sequence, never the scene. From part A it hands over to the choice; from part B it stops the player, forces the branch end state and completes. In the choice phase it does nothing — deciding for the player would defeat the point of the phase.

The end state is applied, never played out: sequences are not fast-forwarded, because event tracks fire unevenly under scrubbing and the resulting world differs run to run.

## Sequencer

**Cameras** are spawnable and live inside the sequences; the level keeps none of them.

**The character** is a replaceable binding — a preview object in the editor, `Resolve to Player Pawn` at runtime. No stand-in is left in the level and the director takes no part in the substitution.

**The gate** is bound by tag. The sequence carries a tagged binding, and on `BeginPlay` the director finds the actor by the same tag and calls `Set Binding By Tag` on both part B players. The sequence describes a movement without knowing which door performs it: rebuild the level or respawn the gate, and a hard reference would silently resolve to nothing.

**Event tracks** reach the director through the sequence's Director Blueprint, which caches exactly one reference — the director itself. Everything else events ask of the director, so lighting and doors keep a single address each.

Bay lighting is not switched when the gate opens. Lumen carries the light from the opening inward on its own, and a preset on top would be a second source of truth about how a lit bay looks.

## Two interfaces

`BPI_Interactable` answers "what happens when the player acts on this" and is implemented by the lever, the panel and both console buttons. `BPI_ShipDoor` describes a door — locked, unlocked, open, forced open — and is used by the player, by the sequences and by the skip path. Merging them would put two unrelated axes into one contract.

The jammed cockpit door implements neither: it is scenery, not a mechanism.

## Control handoff

`SetCinematicMode`, not `UnPossess`. The pawn stays active, movement state and physics survive the scene, and the return needs no re-possession — which is where the character usually arrives in a T-pose or through the floor.

## Streaming

`L_Outside` is a streaming sublevel holding the surface. Loading starts when the choice phase begins, so the player's decision time doubles as load time. The persistent level holds the ship, including its outer hull: an exterior object created in the persistent level would stay visible after the sublevel unloads and would make the streaming boundary meaningless.

## Colour code

Greybox colour encodes state, not type: orange is usable now, purple is dead until powered, green responds to the player, grey is shell and spent mechanisms. Nothing stays orange once it can no longer be used.

The console buttons are the exception — blue and red mark two exclusive outcomes rather than availability. Before the scene the console itself is purple, a target at the end of the bay; the moment the trigger hands the scene to the director it turns grey and the buttons take their colours.

## Rendering

Lumen in software mode. Ray tracing is off (`r.RayTracing=False`, `r.Lumen.HardwareRayTracing=False`): on greybox it changes nothing visible, while slowing shader compilation and narrowing the set of machines that can run the project. Auto-exposure is off as well — brightness is authored, not measured.
