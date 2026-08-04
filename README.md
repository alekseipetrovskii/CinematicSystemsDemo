# Cinematic Systems Demo

Unreal Engine 5.8, Blueprint only. Greybox only — no third-party assets.

An interactive cinematic built around state integrity: control handoffs, camera blending, skip and replay, save/load, and streaming boundaries.

Clone, open, press Play. Nothing to install or download.

## Status

**0.0.2** — blockout complete. Three interior spaces, ship exterior, terrain, doors, panels, lighting, player and exit markers. No logic yet.

## Levels

| Level | Type | Holds |
|---|---|---|
| `Content/Maps/L_Main` | Persistent | The ship: interior spaces and outer hull |
| `Content/Maps/L_Outside` | Streaming sublevel | The surface: ground and terrain |

Greybox colour marks the state an object is in when the scene starts: orange is usable now, purple is dead until powered, green responds to the player, grey grid is shell.
