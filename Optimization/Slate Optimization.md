[Home](../README.md) > [Optimization](README.md) > [Slate Optimization](Slate%20Optimization.md)
# Slate Optimization
Slate and Widget practices to optimize UI rendering and CPU time.

Make sure to test Slate performance in packaged builds as the editor can severely impact reported UI performance.

## Invalidation Panel
Use invalidation panels to prevent layouts updates from occurring at every frame. When a child layout update, the whole invalidation panel and their children will update, so separating frequently updated widgets from rarely updated ones is the best way to use invalidation panels.

Use `Slate.EnableGlobalInvalidation` for use an invalidation for the whole project's UI, but manually placing invalidation panels at strategic places can be better than global invalidation.

## Replace bound attributes
Bound attributes update their value at every frame. Replacing them with event-based updates is a good way to optimize widgets.

## High frame rate updates
Use the IsVolatile option under the performance category to prevent them from needlessly caching information, as they will repaint every frame anyway. Cache as much data as possible to reduce the amount of logic made in Tick and OnPaint functions. Consider using a separate thread to update data used by the widget, then utilize the output to render data.

## Separate and load widgets dynamically
Instead of putting all child widgets within a full UI widget, consider removing them from the widget, then load and attach them in runtime when needed. For some widgets that are frequently used or important for gameplay, you can keep them loaded and collapsed to make them open faster.


## Multithreaded updates
Look at Lyra's FLoadingScreenSyncMechanism to view how to put slate updates in a thread. Mainly useful to achieve a smoother loading animation while the game loads and processes a lot of data.

## General layout tips
- Avoid using Canvas Panels when possible as they can cause multiple draw calls. Consider using Overlays and Size Boxes with Horizontal/Vertical/Grid Boxes.
- Use Spacers instead of Size Boxes
- Avoid combining Scale Boxes and Size Boxes.
- Use standard text boxes instead of Rich Text widgets when possible.
- Use the least expensive animation method possible
  - No CPU cost : Material animation
  - Low CPU cost : Blueprint animation without Sequencer.
  - Medium CPU cost : Sequencer animations made with UMG's animation editor. Similar to blueprint animation, but has a startup cost.
  - High CPU cost : Animations that case layout changes.

## References
https://dev.epicgames.com/documentation/en-us/unreal-engine/optimization-guidelines-for-umg-in-unreal-engine
https://dev.epicgames.com/community/learning/knowledge-base/VZZD/unreal-engine-slate-general-optimization-guidelines
https://dev.epicgames.com/documentation/en-us/unreal-engine/invalidation-in-slate-and-umg-for-unreal-engine

© Samuel Daigle – Licensed under [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/).  