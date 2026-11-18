# EditorBridge Changelog

## 1.1.0 (2026-11-18)

- 46 commands (was 25): Play In Editor control (start/stop, capture, pawn teleport), virtual clicks and typing in the game viewport,
  `trace_screen`, `world_to_screen`, `capture_sequence` (bursts of screenshots and contact sheets), `run_python`, FBX/sound import,
  thumbnails, CVars, popup log and dismissal, unsaved-asset diagnostics, `watch_output_log`, `get_version`, `get_permissions`.
- Permissions panel (Window > Dev Tools > Editor Bridge) and `permissions.json`; new `save_exclude` list for `save_all_dirty`.
- Documentation split into pages: Docs/README.md (getting started, protocol, permissions, commands, troubleshooting).
- Console command `EditorBridge.Open` opens the panel.

## 1.0.0 (2026-04-12)

Initial release — 25 IPC commands covering build, editor state, viewport,
asset import, and project settings.
