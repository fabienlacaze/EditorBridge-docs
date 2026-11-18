# EditorBridge

Editor plugin for Unreal Engine 5.7 (Win64). It lets an external program control the editor by writing a JSON file in
`Saved/EditorBridge/` and reading the JSON answer the editor writes back.

We wrote it to let scripts and coding agents compile, test and inspect our own game without anyone clicking in the editor.

## What it does

- Live Coding: start a compile, get the result and the compiler errors.
- Saving: save all modified packages without dialogs, list unsaved packages, fix up redirectors.
- Logs: read the Output Log, or only the lines added since the last call.
- Play In Editor: start, stop, screenshot, move the player pawn, list what the camera sees.
- Input: mouse clicks (left, right, double, drag, wheel) and keyboard input sent to the game viewport.
- Screenshots: editor or PIE viewport, timed sequences with an optional contact sheet, asset thumbnails.
- Import: FBX, textures, sounds; create a user-defined enum.
- Console commands, CVars, project settings (.ini), Python scripts.
- Actors: list, read and change properties, select.

46 commands in total. Documentation: [Docs/README.md](Docs/README.md) (getting started, protocol, permissions, every command with an example, troubleshooting).

## Example

Request, written to `Saved/EditorBridge/editor_request.json`:

```json
{ "id": "1", "command": "trigger_live_coding", "args": {} }
```

Answer, written by the editor to `Saved/EditorBridge/editor_response.json`:

```json
{ "id": "1", "status": "ok", "message": "Live Coding succeeded", "data": { "result": "succeeded", "duration_seconds": 14.2, "errors": "" } }
```

## Permissions

Commands that run Python, change project settings or close the editor can be turned off one family at a time in
Window > Dev Tools > Editor Bridge, or in `Saved/EditorBridge/permissions.json`.

## Limits

- Windows only. Popup handling uses Win32.
- One request at a time; the editor checks for a new request every 1.5 seconds.
- Live Coding cannot apply header changes such as new class members. Those still need a full build.

## Requirements

Unreal Engine 5.7, Windows 64-bit, Python Editor Script Plugin (part of the engine, enabled automatically).

Copyright (c) 2026 Fabien Lacaze. See `LICENSE.md`.
