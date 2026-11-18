# Commands

46 commands, grouped by use. Every request has the same shape (see [Protocol](Protocol.md)); only `command` and `args` change.

Commands that need a permission are marked; see [Permissions](Permissions.md).

- [Build](#build)
- [Editor state and saving](#editor-state-and-saving)
- [Logs and information](#logs-and-information)
- [Level and actors](#level-and-actors)
- [Console, settings and Python](#console-settings-and-python)
- [Import](#import)
- [Play In Editor](#play-in-editor)
- [Mouse and keyboard (Play In Editor)](#mouse-and-keyboard-play-in-editor)
- [Screenshots](#screenshots)
- [Popups, closing and restarting (Windows)](#popups-closing-and-restarting-windows)

## Build

### `trigger_live_coding`

Starts a Live Coding compile and answers when it is finished. `status` is `error` when the compile fails.

No arguments.

```json
{"id": "1", "command": "trigger_live_coding", "args": {}}
```

Answer:

```json
{
  "id": "1",
  "status": "ok",
  "message": "...",
  "data": {
    "result": "succeeded",
    "duration_seconds": 14.2,
    "errors": ""
  }
}
```

### `get_build_status`

Result and time of the last Live Coding compile started through EditorBridge.

No arguments.

```json
{"id": "1", "command": "get_build_status", "args": {}}
```

Answer:

```json
{
  "id": "1",
  "status": "ok",
  "message": "...",
  "data": {
    "last_result": "succeeded",
    "last_timestamp": "2026-11-18T17:42:10",
    "last_errors": ""
  }
}
```

### `get_compilation_errors`

Compiler errors of the last compile.

No arguments.

```json
{"id": "1", "command": "get_compilation_errors", "args": {}}
```

Answer:

```json
{
  "id": "1",
  "status": "ok",
  "message": "...",
  "data": {
    "errors": "MyActor.cpp(42): error C2065: 'Foo': undeclared identifier"
  }
}
```

### `unblock_build_rules`

Windows only. Removes the downloaded-from-the-Internet mark from the engine's cached build-rules DLLs, which Windows security can otherwise refuse to load.

No arguments.

```json
{"id": "1", "command": "unblock_build_rules", "args": {}}
```

## Editor state and saving

### `get_editor_state`

Is Play In Editor running, how many unsaved packages, which level is open.

No arguments.

```json
{"id": "1", "command": "get_editor_state", "args": {}}
```

Answer:

```json
{
  "id": "1",
  "status": "ok",
  "message": "...",
  "data": {
    "is_pie_running": false,
    "dirty_package_count": 2,
    "current_level": "MyMap",
    "actor_count": 163,
    "last_build_result": "succeeded"
  }
}
```

### `get_dirty_packages`

Lists the packages with unsaved changes.

No arguments.

```json
{"id": "1", "command": "get_dirty_packages", "args": {}}
```

Answer:

```json
{
  "id": "1",
  "status": "ok",
  "message": "...",
  "data": {
    "count": 1,
    "packages": [
      "/Game/Maps/MyMap"
    ]
  }
}
```

### `list_unsaved_assets`

Compares what the engine considers dirty with what the Unsaved Assets tracker shows, and lists files missing on disk.

No arguments.

```json
{"id": "1", "command": "list_unsaved_assets", "args": {}}
```

### `save_all_dirty`

Saves every modified package without any dialog.

| Argument | Type | Required | Description |
|---|---|---|---|
| `exclude` | array of strings | no | Package path prefixes to leave unsaved, e.g. `["/Game/Sandbox/"]`. Replaces the default list (`save_exclude` in permissions.json). |

```json
{"id": "1", "command": "save_all_dirty", "args": {"exclude": ["/Game/Sandbox/"]}}
```

Answer:

```json
{
  "id": "1",
  "status": "ok",
  "message": "...",
  "data": {
    "saved": 3,
    "failedPackages": [],
    "stillDirtyAfter": 0
  }
}
```

### `prompt_save_packages`

Runs the editor's standard save flow (the same as File > Save All).

No arguments.

```json
{"id": "1", "command": "prompt_save_packages", "args": {}}
```

Answer:

```json
{
  "id": "1",
  "status": "ok",
  "message": "...",
  "data": {
    "result": "ok",
    "trackerCountBefore": 3,
    "trackerCountAfter": 0
  }
}
```

### `fixup_redirectors`

Fixes up redirectors in a content folder.

| Argument | Type | Required | Description |
|---|---|---|---|
| `path` | string | no | Content folder. Default `/Game`. |

```json
{"id": "1", "command": "fixup_redirectors", "args": {"path": "/Game/Characters"}}
```

Answer:

```json
{
  "id": "1",
  "status": "ok",
  "message": "...",
  "data": {
    "path": "/Game/Characters",
    "redirectors_found": 4,
    "redirectors_fixed": 4
  }
}
```

## Logs and information

### `get_output_log`

Last lines of the editor log file.

| Argument | Type | Required | Description |
|---|---|---|---|
| `count` | number | no | Number of lines. Default 50. |
| `filter` | string | no | Keep only lines containing this text. |

```json
{"id": "1", "command": "get_output_log", "args": {"count": 20, "filter": "Error"}}
```

Answer:

```json
{
  "id": "1",
  "status": "ok",
  "message": "...",
  "data": {
    "count": 1,
    "total_lines": 5310,
    "lines": [
      "LogBlueprint: Error: ..."
    ]
  }
}
```

### `watch_output_log`

Only the log lines written since your previous call. Use it to follow a long operation.

| Argument | Type | Required | Description |
|---|---|---|---|
| `filter` | string | no | Keep only lines containing this text. |
| `reset` | bool | no | Start again from the end of the file. |

```json
{"id": "1", "command": "watch_output_log", "args": {"filter": "LogTemp"}}
```

Answer:

```json
{
  "id": "1",
  "status": "ok",
  "message": "...",
  "data": {
    "new_lines": 3,
    "lines": [
      "LogTemp: ..."
    ],
    "read_position": 812345
  }
}
```

### `get_editor_notifications`

Recent warning and error notifications shown by the editor.

No arguments.

```json
{"id": "1", "command": "get_editor_notifications", "args": {}}
```

### `get_version`

Plugin version, engine version and the list of available commands.

No arguments.

```json
{"id": "1", "command": "get_version", "args": {}}
```

Answer:

```json
{
  "id": "1",
  "status": "ok",
  "message": "...",
  "data": {
    "plugin": "EditorBridge",
    "version": "1.1.0",
    "engine": "5.7.0",
    "commands": [
      "..."
    ]
  }
}
```

### `get_permissions`

The permissions currently in force (see Permissions).

No arguments.

```json
{"id": "1", "command": "get_permissions", "args": {}}
```

Answer:

```json
{
  "id": "1",
  "status": "ok",
  "message": "...",
  "data": {
    "allow_python_exec": false,
    "allow_pie": true
  }
}
```

## Level and actors

### `get_level_info`

Name and bounds of the open level.

No arguments.

```json
{"id": "1", "command": "get_level_info", "args": {}}
```

Answer:

```json
{
  "id": "1",
  "status": "ok",
  "message": "...",
  "data": {
    "map_name": "MyMap",
    "actor_count": 163,
    "min_x": -51200,
    "max_x": 51200
  }
}
```

### `list_actors`

Actors of the open level.

| Argument | Type | Required | Description |
|---|---|---|---|
| `class_filter` | string | no | Keep only actors whose class name contains this text. |

```json
{"id": "1", "command": "list_actors", "args": {"class_filter": "PointLight"}}
```

Answer:

```json
{
  "id": "1",
  "status": "ok",
  "message": "...",
  "data": {
    "count": 2,
    "actors": [
      {
        "name": "PointLight_1",
        "label": "Lamp",
        "class": "PointLight",
        "location": "X=120 Y=40 Z=300"
      }
    ]
  }
}
```

### `get_actor_properties`

Class, label, folder, transform and visibility of an actor.

| Argument | Type | Required | Description |
|---|---|---|---|
| `name` | string | yes | Actor name or label. |

```json
{"id": "1", "command": "get_actor_properties", "args": {"name": "Lamp"}}
```

Answer:

```json
{
  "id": "1",
  "status": "ok",
  "message": "...",
  "data": {
    "class": "PointLight",
    "label": "Lamp",
    "location": "X=120 Y=40 Z=300",
    "rotation": "P=0 Y=0 R=0",
    "scale": "X=1 Y=1 Z=1",
    "is_hidden": false
  }
}
```

### `set_actor_property`

Sets an editable property from its text form. Needs `allow_actor_modify`.

| Argument | Type | Required | Description |
|---|---|---|---|
| `name` | string | yes | Actor name or label. |
| `property` | string | yes | Property name, e.g. `Intensity`. |
| `value` | string | yes | Value as Unreal text, e.g. `5000`, `true`, `(X=0,Y=0,Z=100)`. |

```json
{"id": "1", "command": "set_actor_property", "args": {"name": "Lamp", "property": "bHidden", "value": "true"}}
```

### `select_actors`

Selects actors in the editor. Needs `allow_actor_modify`.

| Argument | Type | Required | Description |
|---|---|---|---|
| `names` | array of strings | yes | Actor names or labels. |

```json
{"id": "1", "command": "select_actors", "args": {"names": ["Lamp", "Floor"]}}
```

Answer:

```json
{
  "id": "1",
  "status": "ok",
  "message": "...",
  "data": {
    "requested": 2,
    "selected": 2
  }
}
```

## Console, settings and Python

### `exec_console_command`

Runs a console command in the editor world. Needs `allow_console_commands`.

| Argument | Type | Required | Description |
|---|---|---|---|
| `command` | string | yes | The command, e.g. `stat unit`. |

```json
{"id": "1", "command": "exec_console_command", "args": {"command": "r.ScreenPercentage 75"}}
```

Answer:

```json
{
  "id": "1",
  "status": "ok",
  "message": "...",
  "data": {
    "command": "r.ScreenPercentage 75",
    "output": ""
  }
}
```

### `get_cvar`

Value and help text of a console variable.

| Argument | Type | Required | Description |
|---|---|---|---|
| `name` | string | yes | CVar name. |

```json
{"id": "1", "command": "get_cvar", "args": {"name": "r.ScreenPercentage"}}
```

Answer:

```json
{
  "id": "1",
  "status": "ok",
  "message": "...",
  "data": {
    "name": "r.ScreenPercentage",
    "value": "75",
    "help": "..."
  }
}
```

### `set_cvar`

Sets a console variable. Needs `allow_project_settings`.

| Argument | Type | Required | Description |
|---|---|---|---|
| `name` | string | yes | CVar name. |
| `value` | string | yes | New value. |

```json
{"id": "1", "command": "set_cvar", "args": {"name": "r.ScreenPercentage", "value": "100"}}
```

Answer:

```json
{
  "id": "1",
  "status": "ok",
  "message": "...",
  "data": {
    "name": "r.ScreenPercentage",
    "value": "100"
  }
}
```

### `get_project_setting`

Reads a value from a project .ini file.

| Argument | Type | Required | Description |
|---|---|---|---|
| `file` | string | yes | Ini file name without extension, e.g. `DefaultEngine`. |
| `section` | string | yes | Section, e.g. `/Script/Engine.RendererSettings`. |
| `key` | string | yes | Key. |

```json
{"id": "1", "command": "get_project_setting", "args": {"file": "DefaultEngine", "section": "/Script/Engine.RendererSettings", "key": "r.Nanite"}}
```

Answer:

```json
{
  "id": "1",
  "status": "ok",
  "message": "...",
  "data": {
    "value": "True"
  }
}
```

### `set_project_setting`

Writes a value in a project .ini file. Needs `allow_project_settings`.

| Argument | Type | Required | Description |
|---|---|---|---|
| `file` | string | yes | Ini file name without extension. |
| `section` | string | yes | Section. |
| `key` | string | yes | Key. |
| `value` | string | yes | New value. |

```json
{"id": "1", "command": "set_project_setting", "args": {"file": "DefaultGame", "section": "/Script/EngineSettings.GeneralProjectSettings", "key": "ProjectVersion", "value": "1.2.0"}}
```

### `run_python`

Runs Python in the editor (Python Editor Script Plugin). Needs `allow_python_exec`.

| Argument | Type | Required | Description |
|---|---|---|---|
| `code` | string | no | Python code to run. Give `code` or `script_path`. |
| `script_path` | string | no | Path of a .py file to run. |

```json
{"id": "1", "command": "run_python", "args": {"code": "import unreal\nprint(unreal.SystemLibrary.get_engine_version())"}}
```

Answer:

```json
{
  "id": "1",
  "status": "ok",
  "message": "...",
  "data": {
    "success": true,
    "stdout": "5.7.0",
    "stderr": "",
    "errorCount": 0,
    "warningCount": 0
  }
}
```

### `create_user_enum`

Creates a UserDefinedEnum asset.

| Argument | Type | Required | Description |
|---|---|---|---|
| `path` | string | yes | Asset path, e.g. `/Game/Data/E_Weather`. |
| `entries` | array of strings | yes | Display names of the entries. |

```json
{"id": "1", "command": "create_user_enum", "args": {"path": "/Game/Data/E_Weather", "entries": ["Clear", "Rain", "Snow"]}}
```

Answer:

```json
{
  "id": "1",
  "status": "ok",
  "message": "...",
  "data": {
    "path": "/Game/Data/E_Weather",
    "internal_names": [
      "NewEnumerator0",
      "NewEnumerator1",
      "NewEnumerator2"
    ]
  }
}
```

## Import

### `import_texture`

Imports an image file as a texture.

| Argument | Type | Required | Description |
|---|---|---|---|
| `file_path` | string | yes | File on disk. |
| `dst_path` | string | yes | Content folder, e.g. `/Game/Textures`. |

```json
{"id": "1", "command": "import_texture", "args": {"file_path": "C:/Art/rock_albedo.png", "dst_path": "/Game/Textures"}}
```

Answer:

```json
{
  "id": "1",
  "status": "ok",
  "message": "...",
  "data": {
    "asset": "/Game/Textures/rock_albedo.rock_albedo",
    "class": "Texture2D"
  }
}
```

### `import_sound`

Imports a WAV or OGG file as a SoundWave (same arguments as `import_texture`).

| Argument | Type | Required | Description |
|---|---|---|---|
| `file_path` | string | yes | File on disk. |
| `dst_path` | string | yes | Content folder. |

```json
{"id": "1", "command": "import_sound", "args": {"file_path": "C:/Audio/click.wav", "dst_path": "/Game/Audio"}}
```

### `import_fbx`

Imports one or several FBX files as static meshes.

| Argument | Type | Required | Description |
|---|---|---|---|
| `file_path` | string | no | One file. Give `file_path` or `files`. |
| `files` | array of strings | no | Several files. |
| `dst_path` | string | yes | Content folder. |
| `dst_name` | string | no | Asset name. |
| `combine_meshes` | bool | no | Merge all meshes of a file into one asset. |
| `import_materials` | bool | no | Create materials. |
| `import_textures` | bool | no | Import textures. |
| `replace_existing` | bool | no | Overwrite an asset with the same name. |
| `generate_collision` | bool | no | Generate simple collision. |

```json
{"id": "1", "command": "import_fbx", "args": {"file_path": "C:/Art/crate.fbx", "dst_path": "/Game/Props", "combine_meshes": true}}
```

Answer:

```json
{
  "id": "1",
  "status": "ok",
  "message": "...",
  "data": {
    "assets": [
      "/Game/Props/crate.crate"
    ],
    "bounds_min": "X=-50 Y=-50 Z=0",
    "bounds_max": "X=50 Y=50 Z=100"
  }
}
```

### `render_thumbnail`

Renders the thumbnail of an asset to a PNG file.

| Argument | Type | Required | Description |
|---|---|---|---|
| `asset_path` | string | yes | Asset path. |
| `output_path` | string | no | PNG file. Default `Saved/EditorBridge/thumbnail.png`. |
| `size` | number | no | Size in pixels. Default 256. |

```json
{"id": "1", "command": "render_thumbnail", "args": {"asset_path": "/Game/Props/crate", "size": 512}}
```

Answer:

```json
{
  "id": "1",
  "status": "ok",
  "message": "...",
  "data": {
    "asset_path": "/Game/Props/crate",
    "output_path": "C:/MyProject/Saved/EditorBridge/thumbnail.png",
    "size": 512
  }
}
```

## Play In Editor

### `start_pie`

Starts Play In Editor. Needs `allow_pie`.

No arguments.

```json
{"id": "1", "command": "start_pie", "args": {}}
```

Answer:

```json
{
  "id": "1",
  "status": "ok",
  "message": "...",
  "data": {
    "started": true
  }
}
```

### `stop_pie`

Stops Play In Editor. Needs `allow_pie`.

No arguments.

```json
{"id": "1", "command": "stop_pie", "args": {}}
```

Answer:

```json
{
  "id": "1",
  "status": "ok",
  "message": "...",
  "data": {
    "stopped": true
  }
}
```

### `teleport_pie_pawn`

Moves the player pawn during Play In Editor.

| Argument | Type | Required | Description |
|---|---|---|---|
| `x` | number | yes | World X. |
| `y` | number | yes | World Y. |
| `z` | number | yes | World Z. |
| `pitch` | number | no | Degrees. |
| `yaw` | number | no | Degrees. |
| `roll` | number | no | Degrees. |

```json
{"id": "1", "command": "teleport_pie_pawn", "args": {"x": 0, "y": 0, "z": 500, "yaw": 90}}
```

Answer:

```json
{
  "id": "1",
  "status": "ok",
  "message": "...",
  "data": {
    "pawn": "BP_ThirdPersonCharacter_C_0",
    "x": 0,
    "y": 0,
    "z": 500
  }
}
```

### `world_to_screen`

Screen position (fractions 0..1) of a world point, seen from the Play In Editor camera.

| Argument | Type | Required | Description |
|---|---|---|---|
| `x` | number | yes | World X. |
| `y` | number | yes | World Y. |
| `z` | number | no | World Z. |

```json
{"id": "1", "command": "world_to_screen", "args": {"x": 1200, "y": -300, "z": 150}}
```

### `trace_screen`

Casts a grid of rays from the Play In Editor camera and groups the hits by mesh and material: a quick way to know what is on screen.

| Argument | Type | Required | Description |
|---|---|---|---|
| `cols` | number | no | Columns. Default 24. |
| `rows` | number | no | Rows. Default 14. |
| `x0` | number | no | Area left, fraction 0..1. Default 0. |
| `y0` | number | no | Area top. Default 0. |
| `x1` | number | no | Area right. Default 1. |
| `y1` | number | no | Area bottom. Default 1. |
| `channel` | string | no | `visibility` (default), `worldstatic` or `camera`. |

```json
{"id": "1", "command": "trace_screen", "args": {"cols": 12, "rows": 8}}
```

Answer:

```json
{
  "id": "1",
  "status": "ok",
  "message": "...",
  "data": {
    "rays": 96,
    "misses": 10,
    "hits": [
      {
        "mesh": "SM_Rock",
        "materials": [
          "MI_Rock"
        ],
        "actor": "Rock_12",
        "count": 31
      }
    ]
  }
}
```

## Mouse and keyboard (Play In Editor)

### `virtual_click`

Clicks in the game viewport through Slate, like a real mouse. Works while the editor window is in the background.

| Argument | Type | Required | Description |
|---|---|---|---|
| `x` | number | no | Horizontal position, fraction 0..1 of the viewport. Give `x`/`y` or `widget`. |
| `y` | number | no | Vertical position, fraction 0..1. |
| `widget` | string | no | Name of a UMG widget to click instead of a position. |
| `owner` | string | no | With `widget`: part of the class name of the UserWidget that contains it. |
| `button` | string | no | `left` (default), `right` or `middle`. |
| `double` | bool | no | Double click. |
| `double_gap_ms` | number | no | Pause between the two clicks. |
| `hold_ms` | number | no | How long the button stays pressed. Default 80. |
| `to_x` | number | no | Drag to this position (with `to_y`). |
| `to_y` | number | no | Drag destination. |
| `wheel` | number | no | Mouse wheel steps (negative = down). |
| `hover` | bool | no | Only move the cursor there, no click. |
| `shift` | bool | no | Hold Shift. |
| `ctrl` | bool | no | Hold Ctrl. |
| `alt` | bool | no | Hold Alt. |
| `render_tooltip` | bool | no | Give the tooltip time to appear before answering. |

```json
{"id": "1", "command": "virtual_click", "args": {"x": 0.5, "y": 0.62, "button": "left"}}
```

### `virtual_type`

Types text into the focused widget, or sends a key to the game.

| Argument | Type | Required | Description |
|---|---|---|---|
| `text` | string | no | Text to type. Give `text` or `key`. |
| `key` | string | no | Key name, e.g. `Enter`, `Escape`, `F5`, `W`. |
| `hold_ms` | number | no | How long the key stays down. |
| `to_game` | bool | no | Send to the game viewport rather than the focused widget. |
| `shift` | bool | no | Hold Shift. |
| `ctrl` | bool | no | Hold Ctrl. |
| `alt` | bool | no | Hold Alt. |

```json
{"id": "1", "command": "virtual_type", "args": {"key": "Escape", "to_game": true}}
```

## Screenshots

### `capture_viewport`

Saves a PNG of the editor or Play In Editor viewport.

| Argument | Type | Required | Description |
|---|---|---|---|
| `output_path` | string | no | PNG file. Default in `Saved/EditorBridge/`. |
| `target` | string | no | `auto` (default: PIE if running, else editor), `editor` or `pie`. |

```json
{"id": "1", "command": "capture_viewport", "args": {"output_path": "C:/Temp/view.png", "target": "editor"}}
```

Answer:

```json
{
  "id": "1",
  "status": "ok",
  "message": "...",
  "data": {
    "path": "C:/Temp/view.png",
    "width": 1920,
    "height": 1080,
    "source": "editor"
  }
}
```

### `capture_pie`

Saves a PNG of the Play In Editor viewport.

| Argument | Type | Required | Description |
|---|---|---|---|
| `output_path` | string | no | PNG file. |

```json
{"id": "1", "command": "capture_pie", "args": {"output_path": "C:/Temp/game.png"}}
```

Answer:

```json
{
  "id": "1",
  "status": "ok",
  "message": "...",
  "data": {
    "path": "C:/Temp/game.png",
    "width": 1920,
    "height": 1080
  }
}
```

### `capture_sequence`

Takes a series of screenshots at a fixed interval, with an optional contact sheet. Useful to check an animation or a transition.

| Argument | Type | Required | Description |
|---|---|---|---|
| `output_dir` | string | yes | Folder for the images. |
| `frames` | number | no | Number of images (1 to 600). Default 30. |
| `interval_ms` | number | no | Time between images. Default 100. |
| `fps` | number | no | Instead of `interval_ms`. |
| `duration_s` | number | no | Instead of `frames`. |
| `scale` | number | no | Image scale 0.1 to 1. Default 0.5. |
| `sheet` | bool | no | Also build a contact sheet. |
| `cols` | number | no | Columns of the contact sheet (1 to 12). Default 6. |
| `target` | string | no | `auto`, `editor` or `pie`. |
| `exec_each` | string | no | Console command run before each image. |

```json
{"id": "1", "command": "capture_sequence", "args": {"output_dir": "C:/Temp/seq", "fps": 10, "duration_s": 3, "sheet": true}}
```

Answer:

```json
{
  "id": "1",
  "status": "ok",
  "message": "...",
  "data": {
    "frames": 30,
    "done": true,
    "output_dir": "C:/Temp/seq"
  }
}
```

## Popups, closing and restarting (Windows)

### `get_popup_log`

Modal dialogs that appeared recently (title, text, buttons).

| Argument | Type | Required | Description |
|---|---|---|---|
| `since_seconds` | number | no | How far back. Default 60. |

```json
{"id": "1", "command": "get_popup_log", "args": {"since_seconds": 120}}
```

### `dismiss_popup`

Closes a modal dialog that blocks automation.

| Argument | Type | Required | Description |
|---|---|---|---|
| `title_substring` | string | no | Part of the window title. |
| `button` | string | no | `ok`, `cancel`, `yes` or `no`. Default: close the window. |
| `window_class` | string | no | Win32 window class. |
| `pid` | number | no | Only windows of this process. |

```json
{"id": "1", "command": "dismiss_popup", "args": {"title_substring": "Message", "button": "ok"}}
```

Answer:

```json
{
  "id": "1",
  "status": "ok",
  "message": "...",
  "data": {
    "matched": 1,
    "dismissed": 1
  }
}
```

### `inspect_writable_folders`

Explains why the editor refuses to write to a folder (allow and deny lists).

No arguments.

```json
{"id": "1", "command": "inspect_writable_folders", "args": {}}
```

### `request_close`

Closes the editor. Needs `allow_editor_close`; asks you first if `confirm_before_close` is on.

| Argument | Type | Required | Description |
|---|---|---|---|
| `reason` | string | no | Shown in the confirmation dialog. |

```json
{"id": "1", "command": "request_close", "args": {"reason": "nightly build"}}
```

### `request_restart`

Restarts the editor. Needs `allow_editor_restart`.

| Argument | Type | Required | Description |
|---|---|---|---|
| `reason` | string | no | Shown in the confirmation dialog. |

```json
{"id": "1", "command": "request_restart", "args": {}}
```
