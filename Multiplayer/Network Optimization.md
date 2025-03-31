[Home](../README.md) > [Multiplayer](README.md) > [Network Optimization](Network%20Optimization.md)
# Network Optimization
Optimizing network for a multiplayer game makes it transfer less data while hosting dedicated servers, which can largely reduce costs. Also read [Lag Compensation](Lag%20Compensation.md) which includes more information like tick rate.

## Using the Network Profiler
Use the command `netprofile enable` to start and `netprofile disable` to stop recording network data. It will export into your project's `/Saved/Profiling/` folder as a .nprof file. You can load this file into the compiled Engine's `/Binaries/DotNet/NetworkProfiler.exe` were you can see how much data is sent per actor, RPC and properties.

## Sending delta
Instead of sending the whole UPROPERTY values through the network, you can send the initial value upfront, then only replicate more lightweight delta changes to end up with the same result with less bandwidth. Better yet, avoid replicating data altogether if they haven't changed.

Iris Plugin : https://dev.epicgames.com/documentation/en-us/unreal-engine/iris-replication-system-in-unreal-engine

## Data compression
Using `UE::Net::SerializeQuantizedVector`, or a quantized vector struct is essential to reduce network bandwidth by compressing data.

Use FArchive during serialization to compress and decompress various data types and messaging to prepare for replication.

## Net Relevancy
Only replicate necessary data to clients. This can also serve as an anti-cheat measure since it removes information that cheats could use. For example, we don't have to replicate player data when they are behind a wall and not potentially visible when considering network conditions.

You can use default actor settings :
| **Config**                | **Description**                                                                 | **Default Value** |
|---------------------------|---------------------------------------------------------------------------------|-------------------|
| `bOnlyRelevantToOwner`     | Actor is only relevant to its owner.                                             | `false`           |
| `bAlwaysRelevant`          | Actor is always relevant to all clients.                                         | `false`           |
| `bNetOwnerNoAuthority`     | Actor does not rely on server authority for replication (client-side authority).| `false`           |
| `bReplicates`              | Actor will replicate its properties to clients.                                  | `false`           |
| `bReplicateMovement`       | Actor's movement will be automatically replicated.                               | `false`           |
| `bNetUseOwnerRelevancy`    | Actor uses the same relevancy rules as its owner.                                | `false`           |

or override its implementation with custom logic :
```cpp
bool AMyActor::IsNetRelevantFor(const APlayerController* PC) const
{
    return ShouldReplicateToThisPlayer(PC);
}
```

## Net Frequency
Reduce the frequency of updates for less important actors. For rarely modified actors, consider setting the update frequency to 0.0f and using ForceNetUpdate to manually trigger replication when data changes.
```cpp
SetNetUpdateFrequency(0.0f);
ForceNetUpdate();
```

© Samuel Daigle – Licensed under [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/). 