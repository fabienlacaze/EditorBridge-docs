# EditorBridge User Guide

EditorBridge lets an external program control the Unreal Editor. The program writes a JSON request in your project's
`Saved/EditorBridge/` folder; the editor runs the command and writes a JSON answer in the same folder. There is no network
connection involved: anything that can write and read a file can use it (Python, PowerShell, a CI job, a coding agent).

- Engine: Unreal Engine 5.7, Windows 64-bit
- Module type: Editor (not included in packaged games)
- Requires: Python Editor Script Plugin (part of the engine)
- Version: 1.1.0

## Contents

1. [Installation](#1-installation)
2. [How it works](#2-how-it-works)
3. [Your first request](#3-your-first-request)
4. [Permissions and safety](#4-permissions-and-safety)
5. [Command reference](#5-command-reference)
6. [Client examples (Python, PowerShell, Bash)](#6-client-examples)
7. [Using EditorBridge with an AI agent](#7-using-editorbridge-with-an-ai-agent)
8. [Troubleshooting](#8-troubleshooting)
9. [FAQ](#9-faq)

---

## 1. Installation

1. Copy the `EditorBridge` folder into your project's `Plugins/` folder (or install it from Fab into the engine).
2. Open the project. If Unreal asks to rebuild the module, answer **Yes**.
3. Check **Edit > Plugins > Editor > EditorBridge** is enabled.
4. The Output Log shows:
   `LogEditorBridge: EditorBridge IPC auto-poll started (1.5s interval)`

The bridge folder is created on first start: `<YourProject>/Saved/EditorBridge/`.

## 2. How it works

| File (in `Saved/EditorBridge/`) | Written by | Meaning |
|---|---|---|
| `editor_request.json` | you | the command to run |
| `editor_ack.json` | EditorBridge | the request was picked up (written immediately) |
| `editor_response.json` | EditorBridge | the result |
| `permissions.json` | you (optional) | what the bridge is allowed to do |

The editor polls the request file every **1.5 s**. When it finds one it deletes it, writes the acknowledgement, runs the command on
the game thread and writes the response. One request at a time: wait for the response before writing the next request.

**Request**

```json
{ "id": "any-unique-id", "command": "get_editor_state", "args": { } }
```

**Response**

```json
{
  "id": "any-unique-id",
  "status": "ok",
  "message": "Editor state",
  "data": { "is_pie_running": false, "dirty_package_count": 0, "current_level": "MyMap", "actor_count": 163, "last_build_result": "unknown" }
}
```

- `status` is `ok` or `error` (`message` explains the error).
- `id` is echoed back so you can match a response to its request.
- Long operations (Live Coding, sequences of screenshots) answer when they are finished.

## 3. Your first request

Python, with the editor open:

```python
import json, os, time, uuid

BRIDGE = r"C:/MyProject/Saved/EditorBridge"

def call(command, args=None, timeout=120):
    for name in ("editor_response.json", "editor_ack.json"):
        try: os.remove(os.path.join(BRIDGE, name))
        except OSError: pass
    req = {"id": str(uuid.uuid4()), "command": command, "args": args or {}}
    with open(os.path.join(BRIDGE, "editor_request.json"), "w", encoding="utf-8") as f:
        json.dump(req, f)
    end = time.time() + timeout
    while time.time() < end:
        time.sleep(0.2)
        try:
            with open(os.path.join(BRIDGE, "editor_response.json"), encoding="utf-8-sig") as f:
                return json.load(f)
        except (OSError, ValueError):
            pass
    raise TimeoutError(command)

print(call("get_editor_state"))
print(call("trigger_live_coding", timeout=600))
```

## 4. Permissions and safety

Some commands change your project or close the editor. Each family can be switched off, in two ways:

- **In the editor**: **Window > Dev Tools > Editor Bridge** opens a panel with one checkbox per permission (saved immediately), and a
  **Reset to Defaults** button.
- **By file**: `Saved/EditorBridge/permissions.json` (read when the editor starts; the panel writes the same file).

Every permission is **on by default**, so that the bridge works out of the box. On a machine where other people or tools can write
into your project folder, switch off what you do not need: typically `allow_python_exec`, `allow_editor_close` and
`allow_editor_restart`.

```json
{
  "allow_live_coding": true,
  "allow_pie": true,
  "allow_console_commands": true,
  "allow_python_exec": false,
  "allow_actor_modify": true,
  "allow_project_settings": true,
  "allow_editor_close": false,
  "allow_editor_restart": false,
  "confirm_before_close": true,
  "launch_popup_helper": true,
  "save_exclude": ["/Game/Sandbox/"]
}
```

| Key | Guards |
|---|---|
| `allow_live_coding` | `trigger_live_coding` |
| `allow_pie` | `start_pie`, `stop_pie` |
| `allow_console_commands` | `exec_console_command` |
| `allow_python_exec` | `run_python` |
| `allow_actor_modify` | `set_actor_property`, `select_actors` |
| `allow_project_settings` | `set_project_setting`, `set_cvar` |
| `allow_editor_close` / `allow_editor_restart` | `request_close` / `request_restart` |
| `confirm_before_close` | asks you in a dialog before `request_close` / `request_restart` go ahead |
| `save_exclude` | package path prefixes that `save_all_dirty` leaves unsaved by default |
| `launch_popup_helper` | starts the Windows helper that records and dismisses modal popups (`get_popup_log`, `dismiss_popup`) |

A refused command answers `status: "permission_denied"` with
`message: "Permission denied: '<command>' is disabled in EditorBridge settings (category: <category>)"`.
`get_permissions` returns the active set.

**Good practice**

- The bridge only reads files inside your project's `Saved/` folder. Keep that folder out of shared network drives.
- Turn `allow_python_exec` on only for tools you trust: Python has full access to the editor.
- `save_all_dirty` never shows a dialog; use `save_exclude` for folders you never want saved automatically.

## 5. Command reference

Arguments marked `?` are optional. Coordinates are Unreal units; screen positions are fractions `0..1` of the viewport unless stated.

### 5.1 Build and compilation

| Command | Arguments | Returns |
|---|---|---|
| `trigger_live_coding` |: | `result` (`succeeded` / `failed`; `status` is `error` on failure), `duration_seconds`, `errors` |
| `get_build_status` |: | `last_result`, `last_timestamp`, `last_errors` |
| `get_compilation_errors` |: | `errors` |
| `unblock_build_rules` |: | Windows: removes the "downloaded from the Internet" mark from the engine's cached build-rules DLLs (fixes rule assemblies blocked by Windows security) |

Example: compile and stop on failure:

```python
r = call("trigger_live_coding", timeout=900)
if r["status"] != "ok":
    print(r["data"]["errors"])
```

### 5.2 Editor state, saving and logs

| Command | Arguments | Returns |
|---|---|---|
| `get_editor_state` |: | `is_pie_running`, `dirty_package_count`, `current_level`, `actor_count`, `last_build_result` |
| `get_dirty_packages` |: | `count`, `packages[]` |
| `list_unsaved_assets` |: | dirty packages seen by the engine and by the Unsaved Assets tracker, files missing on disk |
| `save_all_dirty` | `exclude?` (array of package prefixes), `include_menu_scenes?` | `saved`, `failedPackages`, `stillDirtyAfter`…: saves without any dialog |
| `prompt_save_packages` |: | runs the standard save prompt flow, returns tracker counts before/after |
| `fixup_redirectors` | `path?` (default `/Game`) | `redirectors_found`, `redirectors_fixed` |
| `get_output_log` | `count?` (default 50), `filter?` (substring) | `lines[]`, `total_lines`, `log_path` |
| `watch_output_log` | `filter?`, `reset?` | only the lines written since your previous call (`new_lines`, `read_position`) |
| `get_editor_notifications` |: | recent warnings/errors toasts |
| `get_version` |: | `plugin`, `version`, `engine`, `commands[]` |
| `get_permissions` |: | the active permission set |

### 5.3 Level and actors

| Command | Arguments | Returns |
|---|---|---|
| `get_level_info` |: | `map_name`, `package_name`, `actor_count`, `bounds` (`min_*`, `max_*`) |
| `list_actors` | `class_filter?` | `actors[]` (`name`, `label`, `class`, `location`) |
| `get_actor_properties` | `name` | `class`, `label`, `folder_path`, `location`, `rotation`, `scale`, `is_hidden` |
| `set_actor_property` | `name`, `property`, `value` | sets any editable property from its text form (`"(X=0,Y=0,Z=100)"`, `"true"`, `"12.5"`) |
| `select_actors` | `names[]` | `requested`, `selected` |

### 5.4 Console, CVars, project settings, Python

| Command | Arguments | Returns |
|---|---|---|
| `exec_console_command` | `command` | `output`: any console command (`stat unit`, `r.ScreenPercentage 50`, your own commands) |
| `get_cvar` / `set_cvar` | `name` / `name`, `value` | `value`, `help` |
| `get_project_setting` | `file` (e.g. `DefaultEngine`), `section`, `key` | `value` |
| `set_project_setting` | `file`, `section`, `key`, `value` | writes the `.ini` value |
| `run_python` | `code` **or** `script_path` | `success`, `stdout`, `stderr`, `errorCount`, `warningCount` |
| `create_user_enum` | `path` (`/Game/.../E_MyEnum`), `entries[]` | `internal_names[]`: creates a UserDefinedEnum asset |

### 5.5 Importing assets

| Command | Arguments | Returns |
|---|---|---|
| `import_texture` | `file_path`, `dst_path` (content folder) | `asset`, `class` |
| `import_fbx` | `file_path` **or** `files[]`, `dst_path`, `dst_name?`, `replace_existing?`, `combine_meshes?`, `generate_collision?`, `import_materials?`, `import_textures?`, `material_folder?`, `material_map?` | `assets[]`, `bounds_min/max`, material `slots` |
| `import_sound` | `file_path`, `dst_path` | imported SoundWave |
| `render_thumbnail` | `asset_path`, `output_path`, `size?` | renders an asset thumbnail to PNG |

### 5.6 Play In Editor

| Command | Arguments | Returns |
|---|---|---|
| `start_pie` / `stop_pie` |: | `started` / `stopped` |
| `capture_pie` | `output_path?` | PNG of the PIE viewport (`path`, `width`, `height`) |
| `teleport_pie_pawn` | `x`, `y`, `z`, `pitch?`, `yaw?`, `roll?` | moves the player pawn |
| `world_to_screen` | `x`, `y`, `z?` | screen position of a world point |
| `trace_screen` | `cols?`, `rows?` (default 24×14), `channel?` (`visibility`/`worldstatic`/`camera`) | casts a grid of rays from the PIE camera and groups the hits by mesh and materials: "what is on screen?" |
| `virtual_click` | `x`, `y` (fractions) **or** `widget` (name), `button?` (`left`/`right`/`middle`), `double?`, `hold_ms?`, `to_x?`/`to_y?` (drag), `wheel?`, `hover?`, `shift?`/`ctrl?`/`alt?`, `render_tooltip?` | clicks inside the game viewport through Slate, exactly like a mouse: works while the editor window is in the background |
| `virtual_type` | `text` **or** `key` (e.g. `Escape`, `F5`), `shift?`/`ctrl?`/`alt?`, `hold_ms?`, `to_game?` | types into the focused widget or sends a key to the game |

### 5.7 Screenshots and recordings

| Command | Arguments | Returns |
|---|---|---|
| `capture_viewport` | `output_path?`, `target?` (`editor`/`pie`) | PNG path and size |
| `capture_sequence` | `output_dir`, `frames?` or `fps?` + `duration_s?`, `interval_ms?`, `target?`, `scale?`, `sheet?` (true = also a contact sheet), `cols?`, `exec_each?` (console command run before each frame) | `frames`, `times_s[]`: a burst of screenshots, ideal to check an animation |

### 5.8 Popups, closing and restarting (Windows)

| Command | Arguments | Returns |
|---|---|---|
| `get_popup_log` | `since_seconds?` | modal dialogs that appeared (title, text, buttons) |
| `dismiss_popup` | `title_substring?`, `button?`, `window_class?`, `pid?` | closes a modal dialog that blocks automation |
| `inspect_writable_folders` |: | why the editor refuses to write a folder (allow/deny lists) |
| `request_close` | `reason?` | closes the editor cleanly (needs `allow_editor_close`) |
| `request_restart` | `reason?` | restarts the editor (needs `allow_editor_restart`) |

## 6. Client examples

**PowerShell**

```powershell
$dir = "C:\MyProject\Saved\EditorBridge"
Remove-Item "$dir\editor_response.json" -ErrorAction SilentlyContinue
@{ id = [guid]::NewGuid().ToString(); command = "capture_viewport"; args = @{ output_path = "C:\Temp\shot.png" } } |
    ConvertTo-Json | Set-Content "$dir\editor_request.json" -Encoding utf8
while (-not (Test-Path "$dir\editor_response.json")) { Start-Sleep -Milliseconds 200 }
Get-Content "$dir\editor_response.json" | ConvertFrom-Json
```

**Bash (Git Bash / WSL)**

```bash
D="/c/MyProject/Saved/EditorBridge"
rm -f "$D/editor_response.json"
echo '{"id":"1","command":"exec_console_command","args":{"command":"stat fps"}}' > "$D/editor_request.json"
until [ -f "$D/editor_response.json" ]; do sleep 0.2; done; cat "$D/editor_response.json"
```

**A typical automated check**

```python
call("trigger_live_coding", timeout=900)
call("start_pie")
time.sleep(10)
call("capture_pie", {"output_path": "C:/Temp/check.png"})
print(call("get_output_log", {"count": 200, "filter": "Error"}))
call("stop_pie")
```

## 7. Using EditorBridge with an AI agent

We use EditorBridge with coding agents (Claude Code, Cursor, scripts of our own). A typical cycle is: the agent edits C++,
calls `trigger_live_coding`, reads `errors`, fixes, calls `start_pie`, then `capture_pie`, looks at the picture and calls `stop_pie`.

Give your agent these rules:

1. Write `editor_request.json`, then wait for `editor_response.json` (delete the old response first).
2. Never send a second request before the first answered.
3. Check `get_editor_state` before `start_pie` (do not start a session that is already running).
4. Use `save_all_dirty` after asset changes made through Python: unsaved changes are lost when the editor closes.

## 8. Troubleshooting

| Symptom | Cause / fix |
|---|---|
| No response at all | Editor not running, plugin disabled, or the request was written to another project's `Saved/` folder. Look for `LogEditorBridge` in the Output Log. |
| `permission_denied` | Tick the permission in **Window > Dev Tools > Editor Bridge**, or set the key in `permissions.json` and restart the editor. |
| `trigger_live_coding` returns `failed` with no errors | Live Coding is disabled in Editor Preferences, or a header changed (adding members needs a full build: close the editor and build from your IDE). |
| Response file never appears after a dialog opened | A modal dialog blocks the game thread. Enable `launch_popup_helper` and use `dismiss_popup`. |
| `run_python`: "PythonScriptPlugin not available" | Enable **Python Editor Script Plugin** in Edit > Plugins. |
| JSON parse error | Write the request in UTF-8; the bridge accepts a BOM. |

## 9. FAQ

**Does it ship with my game?** No. It is an Editor module; packaged builds never contain it.

**Is it safe on a shared machine?** It only reads `Saved/EditorBridge/` of the open project and executes nothing from the network.
Dangerous commands are behind permissions.

**macOS / Linux?** The plugin is tested on Windows. Popup handling (`get_popup_log`, `dismiss_popup`) is Windows-only.

**Support**: see the support link on the Fab page.
