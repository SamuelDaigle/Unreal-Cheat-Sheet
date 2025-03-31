[Home](../README.md) > [General Knowledge](README.md) > [Memory Management](Memory%20Management.md)
# Memory Management
## Storing UObjects
To prevent memory leaks when storing UObject in c++, you **must** use:
| Item | Description |
|------------------|-------------------|
| Hard pointer<br/>`UPROPERTY()` | Only used when the class is the owner of the `UPROPERTY`. This allows the unreal garbage collector to clean uproperties when all of its hard-pointer owners are destroyed. Use `IsValid(Pointer)` to check if it's currently being destroyed or already destroyed, or check with `if (Pointer)` in most cases. Do not hard reference one actor multiple times in various actors, it is better to have one hard reference and other actors have weak references. |
| Weak Object Pointer<br/>`TWeakObjectPtr<UObject>` | Holds UObject without any ownership (useful in subsystems or any links to other actors). Has no link with garbage collector, Always use `Pointer.IsValid()` since it can be garbage collected from its hard pointer owner. |
| Soft Object Pointer<br/>`TSoftObjectPtr<UObject>` | Same as Weak Object Pointer, but holds the path. Only use a soft pointer when wanting to make the property usable in blueprint, otherwise use `TWeakObjectPtr` |

It is also possible to use the `FStreamableManager`, `FGCObject` or `UObjectBaseUtility::AddToRoot` to prevent accidental garbage collection, although be sure to use the more common ways before using these.

## Linking Assets
When attaching one asset to another, such as linking a skeletal mesh or a weapon blueprint to a pawn, you often create hard references that load all linked assets when the object is loaded. In most cases, you should use :

| Item | Description |
|------------------|-------------------|
| `FSoftObjectPath` | Contains a string reference to the asset that you can load async on demand. |
| `TSubclassOf<UClass>` | By referencing only the class, you do not load all of the referenced assets from that blueprint until you load it. Using classes is ideal when spawning actors or when accessing class defaults. |

If you do not use this, loading the player pawn in the main menu will load all hard referenced assets (guns, abilities, textures, sounds) and their own sub-assets. This makes loading much slower as well as increasing memory with unused objects.

## Memory Concepts
**Stack Memory**: Extremely fast (CPU cache-friendly), limited size and deallocated when the scope ends.  
**Heap Memory**: Slower than stack, large size, managed lifetime. Created with `new`, `malloc` or `NewObject`.  
Class properties can be stored on stack or heap depending on how the class is instanced.

## Garbage collection
In some rare cases, you may want to disable garbage collection to speed up important sections. For instance, you may want to limit garbage collection when enemies are nearby, so provide the smoother experience possible, or you may want to limit it while loading the level to make the loading faster. You can disable garbage collection by using `GEngine::DelayGarbageCollection` or similar methods.

## Storing large amount of data
When having to store a lot of data, consider:
- Using binary operations to store multiple information in one property. You can use properties directly or `TBitArray` if you only need a bool array as bits.
- Using disk storage instead of memory, however you may encounter difficulties when porting the game to console.
- Using a seed to generate randomized initial data and only store the deltas to modify the result.
- Using types with less precision.
  
© Samuel Daigle – Licensed under [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/). 