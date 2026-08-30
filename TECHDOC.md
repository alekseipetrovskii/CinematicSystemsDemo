# Technical Overview

Blueprint-only demo of an interactive cinematic.

![The director's event graph: every entry point, coloured by what fires it](Media/director_state_flow.jpg)

Blue is the world calling in, red is a Sequencer event track, tan is a timer or a delegate the director set for itself.

## Shape of the scene

The scene runs in three beats:

1. **Part A** — a twelve-second sequence, control suppressed.
2. **The choice** — a gameplay phase. Control returns in full, two buttons on the gate console come alive, and the scene waits as long as the player needs.
3. **Part B** — one of two sequences, control suppressed again.

Parts A and B are separate assets, so nothing is paused between them. The middle beat is ordinary gameplay rather than a sequence with partial input: no half-suppressed state exists anywhere in the scene.

## State machine

`E_CinematicState`: `Idle → Arming → Blending_In → Playing_A → Interactive_Beat → Playing_B → Blending_Out → Completed`.

`BP_CinematicDirector` owns it. `SetState` is the only writer, and every transition prints to the screen log: any recording carries its own trace.

`Interactive_Beat` is the only state where the player holds control. `Completed` is terminal within a session: the scene starts only from `Idle`, so a second overlap after it does nothing.

## Director

| Function | Does |
|---|---|
| `TakeControl` | suppresses input, clears momentum, places the character on the entry mark, freezes movement, gives the director its own input |
| `ReleaseControl` | blends the view back to the character, restores movement and returns input |
| `PlaySequence` | records the active sequence actor, binds the finish delegate, starts playback |
| `GoToBeat` | enters the choice phase and wakes the console after the view settles |
| `ApplyFinalState` | forces the end state of the picked branch: plate position, bay lighting, alarm |
| `FinishCinematic` | starts the exit blend and hands off to a timer |

Blend phases use timers rather than `Delay`. A `Delay` cannot be cancelled: a skip arriving mid-blend cannot stop it, so part A would start on top of the choice phase. Timer handles are cleared inside the branch that knows what it is cancelling.

## Skip

`Enter`, not `Space`: the choice phase has full control, and `Space` is jump there.

A skip drops one sequence, never the scene. From part A it hands over to the choice; from part B it stops the player, forces the branch end state and completes. In the choice phase it does nothing: the scene is waiting for a decision, and a skip would make it for the player.

The end state is applied, not fast-forwarded: event tracks fire unevenly when a sequence is scrubbed, so the same skip would leave a different world each time.

## On-screen text

Lines come from three places: two world triggers in the first act, the lever at the moment it restores power, and the director for anything inside the scene. All of them call one widget, which shows one line at a time. The director clears the line when it takes the scene, so a line fired a step earlier cannot survive into the first shot.

## Sequencer

**Cameras** are spawnable and live inside the sequences; the level holds only the entry rig the view blends to.

**The character** is a replaceable binding — a preview object in the editor, `Resolve to Player Pawn` at runtime. No stand-in is left in the level and the director takes no part in the substitution.

**The gate** is bound by tag. The sequence carries a tagged binding, and on `BeginPlay` the director finds the actor by the same tag and calls `Set Binding By Tag` on both part B players. The sequence describes a movement without knowing which door performs it. A hard reference would resolve to nothing the moment the gate is replaced.

**Event tracks** reach the director through the sequence's Director Blueprint, which caches exactly one reference — the director itself. Events ask the director for everything else, so lighting and doors are addressed in one place. The alarm on the emergency branch is one such event: the sequence owns the timing, the director owns the behaviour.

Part A's lines go further: the endpoint takes the text and its duration as parameters, so each key carries its own payload. Rewording a line or moving it a second later happens in Sequencer, next to the shot it belongs to, and the director is not touched.

Bay lighting is not switched when the gate opens. Lumen carries the light from the opening inward on its own, and a preset on top would be a second definition of a lit bay.

## Two interfaces

`BPI_Interactable` answers "what happens when the player acts on this" and is implemented by the lever, the panel and both console buttons. `BPI_ShipDoor` describes a door — locked, unlocked, open, forced open — and is used by the player, by the sequences and by the skip path. Merging them would put two unrelated questions into one interface.

The jammed cockpit door implements neither: it is scenery, not a mechanism.

## Control handoff

`SetCinematicMode`, not `UnPossess`. The pawn stays active: movement state and physics survive the scene, and nothing has to be re-possessed at the end. Re-possession is where the character arrives in a T-pose or through the floor.

`SetCinematicMode` suppresses input, not simulation: the movement component keeps running, and its floor adjustment will pull the capsule off the mark it was just placed on. Movement is switched off for the length of the scene and restored with control, so the placement has one owner.

The blend target is `BP_CineCameraRig`, a camera in the level whose transform matches the first frame of part A: the view arrives where the sequence starts, so the cut has nothing to jump over. Blends are 0.5 s in and 0.4 s out, 0.25 s when a skip ends part B.

## Streaming

`L_Outside` is a streaming sublevel holding the surface. Loading starts when the choice phase begins: the player decides while the level loads. Part B waits for the sublevel to be visible, not merely loaded — a loaded level sits in memory but is not yet in the world. While it waits the scene stays in the choice phase, so a skip cannot force the ending onto a world that has not arrived.

The persistent level holds the ship, including its outer hull. An exterior object placed there by mistake would stay visible after the sublevel unloads, and the streaming boundary would mean nothing.

## Edge cases

| Situation | Behaviour | What guarantees it |
|---|---|---|
| Entering the trigger mid-air | Character ends on the entry mark, no T-pose, no fall through the floor | `TakeControl` clears velocity, sets the transform and freezes the movement component |
| Skip during part A | Part A stops, control returns, the scene waits at the choice | `RequestSkip` hands over to `GoToBeat` |
| Skip during part B | The end state of the picked branch is forced, the scene completes | `RequestSkip` calls `ApplyFinalState`, then `FinishCinematic` |
| Re-entering the trigger after the scene | Nothing happens | The trigger's guard flag, and `Completed` reached only from `Idle` |
| Two overlaps in one frame | Handled once | The guard is set before the director is called, not after |
| Leaving the trigger while the scene arms | The scene commits | There is no `EndOverlap` path to unwind |
| `L_Outside` not ready when part B is due | The scene holds at the choice until the sublevel is visible | `IsOutsideReady` polled every 0.2 s before the state advances |

## Colour code

Greybox colour encodes state, not type — the legend is in `README.md`.

The console buttons are the exception: blue and red mark two exclusive outcomes rather than availability. Before the scene the console is purple; when the trigger hands the scene to the director it turns grey and the buttons take their colours.

## Rendering

Lumen in software mode. Ray tracing is off (`r.RayTracing=False`, `r.Lumen.HardwareRayTracing=False`): on greybox it changes nothing visible, while slowing shader compilation and narrowing the set of machines that can run the project. Auto-exposure is off as well — brightness is authored, not measured. The cine cameras constrain their aspect ratio to 16:9, so any shot they own is letterboxed while gameplay fills the viewport.
