# Permissions

Every command that can change your project, run code or close the editor belongs to a family that can be switched off.
All families are **on by default**, so the bridge works right after installation.

If other people or programs can write into your project folder, switch off what you do not use. Most users turn off
`allow_python_exec`, `allow_editor_close` and `allow_editor_restart`.

## Where to change them

- **Window > Dev Tools > Editor Bridge**: one checkbox per family, saved immediately. **Reset to Defaults** turns everything back on.
- `Saved/EditorBridge/permissions.json`: same settings as a file. It is read when the editor starts and rewritten by the panel.

## Families

| Key | Commands |
|---|---|
| `allow_live_coding` | `trigger_live_coding` |
| `allow_pie` | `start_pie`, `stop_pie` |
| `allow_console_commands` | `exec_console_command` |
| `allow_python_exec` | `run_python` |
| `allow_actor_modify` | `set_actor_property`, `select_actors` |
| `allow_project_settings` | `set_project_setting`, `set_cvar` |
| `allow_editor_close` | `request_close` |
| `allow_editor_restart` | `request_restart` |

## Other settings

| Key | Effect |
|---|---|
| `confirm_before_close` | asks you in a dialog before `request_close` and `request_restart` |
| `save_exclude` | package path prefixes that `save_all_dirty` leaves unsaved, e.g. `["/Game/Sandbox/"]` |
| `launch_popup_helper` | starts the small Windows helper used by `get_popup_log` and `dismiss_popup` |

## Example

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
  "save_exclude": ["/Game/Sandbox/"]
}
```

A refused command answers:

```json
{ "id": "7", "status": "permission_denied", "message": "Permission denied: 'run_python' is disabled in EditorBridge settings (category: Python Exec)" }
```
