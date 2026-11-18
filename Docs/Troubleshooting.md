# Troubleshooting

| Symptom | What to check |
|---|---|
| No answer at all | The editor is not running, the plugin is disabled, or the request went to another project's `Saved` folder. Search the Output Log for `LogEditorBridge`. |
| `permission_denied` | Tick the family in Window > Dev Tools > Editor Bridge, or set it in `permissions.json` and restart the editor. |
| `trigger_live_coding` returns `failed` without errors | Live Coding is off in Editor Preferences, or the change needs a full build (new class members, header changes). Close the editor and build from your IDE. |
| No answer after a dialog appeared | A modal dialog blocks the editor. Turn on `launch_popup_helper`, then use `get_popup_log` and `dismiss_popup`. |
| `run_python`: "PythonScriptPlugin not available" | Enable Python Editor Script Plugin in Edit > Plugins. |
| `virtual_click` does nothing | Play In Editor must be running; `x` and `y` are fractions (0 to 1) of the game viewport, not pixels. |
| JSON parse error | Write the request in UTF-8. Check it with any JSON validator. |
| Assets changed through Python are lost | Save them (`save_all_dirty`, or `unreal.EditorAssetLibrary.save_asset` in your script) before closing the editor. |

## Questions

**Is it included in my packaged game?** No. EditorBridge is an Editor module.

**Does it open a network port?** No. It only reads and writes files in your project's `Saved/EditorBridge/` folder.

**macOS or Linux?** Not supported. EditorBridge is built and tested on Windows 64-bit.

**Support**: open an issue on the documentation repository (link on the Fab page).
