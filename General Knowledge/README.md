[Home](../README.md) > [General Knowledge](README.md)  
Related Pages : [Data Structures and Algorithms](Data%20Structures%20and%20Algorithms.md) • [Memory Management](Memory%20Management.md) • [Native Programming](Native%20Programming.md)
# General Knowledge
Contains advices on architecture and general usages of Unreal Engine

### Sections
- [Data Structures and Algorithms](Data%20Structures%20and%20Algorithms.md)
- [Design Patterns](Design%20Patterns.md)
- [Memory Management](Memory%20Management.md)
- [Native Programming](Native%20Programming.md)

## High-level decision
### Blueprint vs C++
Most teams tend to move from blueprint to c++ midway through their project. Blueprint is great for prototyping when one or two people work on a feature, but it drastically increases technical debt. It reduces performance, makes it difficult to read larger blueprints due to “spaghetti code” and more importantly, two people often cannot work on the same blueprint at once without having to merge blueprints afterwards. This makes it difficult to collaborate and to merge blueprints between source control branches.

Recommended practice : 
- **Blueprint** : Final gameplay implementation with minimal code (C++ override, independent abilities and custom features), UI and level design.
- **C++** : Everything else; All core gameplay features (parent abilities, inputs, components, etc.), all game systems and all reusable UI elements. You can use Live Coding to update .cpp changes without recompiling the project.

Alternatively, large teams can implement Python, LUA, Typescript or other languages to build quick gameplay scripts without having to recompile the game.

### Data-oriented architecture
Makes flexible code that allows designers and artists to easily create new features without needing a programmer. It generally means to combine all values at the same place for easy modification and reduces the amount of potential variable duplication. It’s easier to patch a game in live development by only updating data and it can make it easier to implement backend to datatable code later in development.

Things to change from 'regular' code-driven design :
- **Remove hardcoded values** : cpp and blueprints should never have “config” values, only runtime values. For example, an enemy’s initial health should not be in the blueprint, instead it should be in a datatable per enemy.
- **Remove hardcoded classes** : You should reduce the amount of class inheritance which “implements” an enemy. Instead, use multiple datatables to achieve this result. For example, instead of having archer.cpp and soldier.cpp, you can have two rows in a datatable where the row ‘archer’ has a specific behavior tree for AI, specific list of abilities, custom values, etc. Same for the class soldier.
- To achieve a more data-driven code, use UDataTable, UCompositeDataTable, UCurveTable and UDataAsset (or custom xml/json). Use FGameplayTag to identify rows, which the tag can be easily replicated and it’s easy to read.

### Composition vs Inheritance

| **Aspect**       | **Usability**                                                   | **Memory**                                                        | **CPU**                                                                 |
|------------------|---------------------------------------------------------------|-------------------------------------------------------------------|------------------------------------------------------------------------|
| **Inheritance**  | For fundamental systems with unique behaviors. Overrides functions directly in derived classes. | Slightly higher if deep inheritance chains exist (vtable overhead). | Deep inheritance chains may add CPU overhead due to vtable lookups and indirect calls. Can be faster than composition with direct function calls (no virtual function). |
| **Composition**  | For modular systems and reusability. Requires more setup but allows swapping or stacking behaviors dynamically with interfaces and events. | Lower per-object memory when unused, since components can be added dynamically. | More direct component interactions. Avoid `FindComponentByClass()`, blueprint interfaces or too frequent event dispatchers. |


© Samuel Daigle – Licensed under [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/). 