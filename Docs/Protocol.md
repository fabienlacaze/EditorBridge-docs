# Protocol

## Files

All files live in `<YourProject>/Saved/EditorBridge/`.

| File | Written by | Purpose |
|---|---|---|
| `editor_request.json` | your program | the command to run |
| `editor_ack.json` | editor | the request was read |
| `editor_response.json` | editor | the result |
| `permissions.json` | you or the panel | what the bridge may do |

## Exchange

1. Delete `editor_response.json` and `editor_ack.json` if they exist.
2. Write `editor_request.json`.
3. The editor checks for a request every 1.5 seconds. It reads the file, deletes it and writes `editor_ack.json`.
4. The command runs on the editor's game thread.
5. The editor writes `editor_response.json`. Long commands (Live Coding, screenshot sequences) answer when they are done.

Send one request at a time. A second request written before the first answered can be lost.

Write the request in a single operation (write to a temporary file, then rename it) if your language makes that easy. The
editor accepts UTF-8 with or without BOM.

## Request

```json
{ "id": "42", "command": "capture_viewport", "args": { "output_path": "C:/Temp/view.png" } }
```

| Field | Meaning |
|---|---|
| `id` | any string; copied into the answer |
| `command` | command name (see [Commands](Commands.md)) |
| `args` | object with the arguments; `{}` when there are none |

## Answer

```json
{ "id": "42", "status": "ok", "message": "Captured 1920x1080 viewport (source=editor) to C:/Temp/view.png", "data": { "path": "C:/Temp/view.png", "width": 1920, "height": 1080 } }
```

| `status` | Meaning |
|---|---|
| `ok` | done; `data` holds the result |
| `error` | failed; `message` says why |
| `permission_denied` | the command's family is switched off (see [Permissions](Permissions.md)) |

An unknown command answers `error` with the message `Unknown command: <name>`.
