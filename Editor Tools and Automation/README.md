[Home](../README.md) > [Editor Tools and Automation](README.md)
# Editor Tools and Automation
### Scripted Action
Documentation : [Scripted Action](https://dev.epicgames.com/documentation/en-us/unreal-engine/scripted-actions-in-unreal-engine), [Editor Widget](https://dev.epicgames.com/documentation/en-us/unreal-engine/editor-utility-widgets-in-unreal-engine), [Editor Mode](https://lxjk.github.io/2019/10/01/How-to-Make-Tools-in-U-E.html#_custom_editor_mode)

Scripted action types:
- **EditorUtilityBlueprint**
  - **AssetActionUtility** : Adds a scripted action in the right-click context menu of specified assets.
  - **ActorActionUtility** : Adds a scripted action in the right-click context menu of specified actors.
- **EditorUtilityWidget** : Adds a new UI elements to the editor, accessible via the `Tools > Editor Utility Widgets` menu.
- **Custom Editor Mode** : Create a custom c++ editor mode by implementing a `SCompoundWidget` to render the widget, a `FModeToolkit` to link the editor mode to the widget and a `FEdMode` to define the editor mode. Then we must register the editor mode in the module startup.


### Designing user-friendly tools
Editor tools should be non-destructive and have a single action : 
- A single undo should revert the tool's changes. Use `FScopedTransaction` with `Object->Modify()` and `RF_Transactional` operations to affect unreal's undo system.
- The tools should not save or checkout files automatically.
- Keep files in their original state when failures happen.
- Allow users to cancel the operation at any time.

Use `FScopedSlowTask` to implement a progress bar on operations that takes more than one second.

© Samuel Daigle – Licensed under [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/). 