# Getting Started

## Requirements

- Unreal Engine 5.7, Windows 64-bit
- Python Editor Script Plugin (ships with the engine; EditorBridge enables it)

## Install

1. Copy the `EditorBridge` folder into your project's `Plugins` folder, or install it from Fab into the engine.
2. Open the project. If the editor asks to rebuild modules, accept.
3. Check that **Edit > Plugins > Editor > EditorBridge** is enabled.

When the editor has started, the Output Log contains:

```
LogEditorBridge: EditorBridge IPC auto-poll started (1.5s interval)
```

and the folder `<YourProject>/Saved/EditorBridge/` exists.

## The panel

**Window > Dev Tools > Editor Bridge** (or the console command `EditorBridge.Open`) opens a panel with the permissions and the
main commands. You do not need it to use the bridge; it is there to switch permissions on and off.

## First request

1. Create `<YourProject>/Saved/EditorBridge/editor_request.json` with:

   ```json
   { "id": "1", "command": "get_editor_state", "args": {} }
   ```

2. Within two seconds the editor deletes that file and writes `editor_response.json`:

   ```json
   {
     "id": "1",
     "status": "ok",
     "message": "Editor state",
     "data": { "is_pie_running": false, "dirty_package_count": 0, "current_level": "MyMap", "actor_count": 163, "last_build_result": "unknown" }
   }
   ```

That is the whole mechanism. Every command works the same way.

## A small client in Python

```python
import json, os, time, uuid

BRIDGE = r"C:/MyProject/Saved/EditorBridge"

def call(command, args=None, timeout=120):
    for name in ("editor_response.json", "editor_ack.json"):
        try:
            os.remove(os.path.join(BRIDGE, name))
        except OSError:
            pass
    request = {"id": str(uuid.uuid4()), "command": command, "args": args or {}}
    with open(os.path.join(BRIDGE, "editor_request.json"), "w", encoding="utf-8") as f:
        json.dump(request, f)
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
```

## Typical sequence

```python
call("trigger_live_coding", timeout=900)          # compile
call("start_pie")                                  # play
time.sleep(10)
call("capture_pie", {"output_path": "C:/Temp/check.png"})
print(call("get_output_log", {"count": 200, "filter": "Error"}))
call("stop_pie")
```

Next: [Protocol](Protocol.md) for the details of the exchange, [Commands](Commands.md) for the full list.
