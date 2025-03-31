[Home](../README.md) > [Multiplayer](README.md) > [Replication Graph](ReplicationGraph.md)
# Replication Graph
## Replication Graph Summary

### Purpose
Efficiently manages actor replication in large multiplayer games by optimizing network performance, reducing redundant replication, and handling actors dynamically. It introduces grid spatialization, allowing actors to be replicated based on player position and movement rather than a fixed list of properties.

## Initialization

### Enable in `DefaultEngine.ini`
```ini
[/Script/OnlineSubsystemUtils.IpNetDriver]
ReplicationDriverClassName=/Script/MyGame.MyReplicationGraph
```

### Subclass `UReplicationGraph` and Override Methods
```cpp
UCLASS()
class MYGAME_API UMyReplicationGraph : public UReplicationGraph
{
    GENERATED_BODY()
public:
    UMyReplicationGraph();
    virtual void InitGlobalActorClassSettings() override;
    virtual void InitConnectionGraphNodes(UNetReplicationGraphConnection* ConnectionManager) override;
};
```

## Common Replication Nodes

### Global Nodes (Manage replication across all clients)

| Node Name | Purpose | Notes |
|-----------|---------|-------|
| `UReplicationGraphNode_ActorList` | Stores a list of always-replicated actors. | Useful for actors that should always be sent to all clients. |
| `UReplicationGraphNode_ActorListFrequencyBuckets` | Organizes actors into frequency buckets for efficient updates. | Helps optimize network performance by reducing update frequency for less important actors. |
| `UReplicationGraphNode_GridSpatialization2D` | Uses a 2D grid to replicate actors based on location. | Ideal for large open-world games. |
| `UReplicationGraphNode_GridSpatialization3D` | Uses a 3D grid for spatialized replication. | Useful for vertical-level designs like buildings or space games. |
| `UReplicationGraphNode_DynamicSpatialFrequency` | Adjusts replication frequency dynamically based on relevance. | Reduces bandwidth usage by lowering update rates for distant or less relevant actors. |

### Per-Connection Nodes (Manage replication per client connection)

| Node Name | Purpose | Notes |
|-----------|---------|-------|
| `UReplicationGraphNode_AlwaysRelevant` | Ensures specific actors are always replicated to all clients. | Used for global actors like game mode controllers. |
| `UReplicationGraphNode_AlwaysRelevant_ForConnection` | Ensures specific actors are always replicated per client. | Useful for per-player UI elements, personal game objects, etc. |
| `UReplicationGraphNode_TearOff_ForConnection` | Handles actors that stop replicating after becoming irrelevant. | Common for actors like ragdolls or destroyed objects that don’t need updates once removed. |

### Specialized Nodes (Handle specific replication scenarios)

| Node Name | Purpose | Notes |
|-----------|---------|-------|
| `UReplicationGraphNode_LegacyNodes` | Compatibility layer for older replication systems. | Helps migrate from older networking approaches. |
| `UReplicationGraphNode_ConnectionDormancyNode` | Manages actor dormancy and wakes them when needed. | Reduces replication cost by putting inactive actors to sleep. |

## Actor Registration

### Register Actors Dynamically in the Graph
Both `AddGlobalGraphNode(Node)` and `SetReplicatedActorClassSettings(AActor::StaticClass(), ActorSettings);` set a default node for all actors.
```cpp
void UMyReplicationGraph::InitGlobalActorClassSettings()
{
    // Add default settings for AActor
    FGlobalActorReplicationSettings ActorSettings;
    ActorSettings.ReplicationNode = CreateNewNode<UReplicationGraphNode_GridSpatialization2D>();
    SetReplicatedActorClassSettings(AActor::StaticClass(), ActorSettings);

    // Create combined nodes for highly dynamic actors.
    UReplicationGraphNode_GridSpatialization2D* DynamicGridNode = CreateNewNode<UReplicationGraphNode_GridSpatialization2D>();
    UReplicationGraphNode_DynamicSpatialFrequency* DynamicFrequencyNode = CreateNewNode<UReplicationGraphNode_DynamicSpatialFrequency>();
    DynamicGridNode->AddChildNode(DynamicFrequencyNode);
    UReplicationGraphNode_ConnectionDormancyNode* DynamicDormancyNode = CreateNewNode<UReplicationGraphNode_ConnectionDormancyNode>();
    DynamicFrequencyNode->AddChildNode(DynamicDormancyNode);
    
    FGlobalActorReplicationSettings DynamicSettings;
    DynamicSettings.ReplicationNode = DynamicGridNode;

    
    // Add dynamic settings for APawn and ADynamicObject
    SetReplicatedActorClassSettings(APawn::StaticClass(), DynamicSettings);
    SetReplicatedActorClassSettings(ADynamicObject::StaticClass(), DynamicSettings);
}
```

### Combining nodes
In the Replication Graph, parent nodes are executed first, followed by child nodes.
1. GridNode (Parent Node)
2. FrequencyNode (Child of GridNode)
3. DormancyNode (Child of FrequencyNode)

If you have an highly dynamic dormancy of many actors, GridNode > DormancyNode > FrequencyNode would be more optimized.

### Creating a custom node
```cpp
class UMyReplicationGraphNode_AlwaysRelevant_ForTeammate : public UReplicationGraphNode_AlwaysRelevant_ForConnection
{
    virtual void GatherActorListForConnection(const FConnectionGatherActorListParameters& Params) override
    {
        for (APlayerState* PlayerState : Params.Viewers)
        {
            if (PlayerState->IsTeammate(Params.Connection->GetPlayerState()))
            {
                Params.OutGatheredReplicationLists.Add(PlayerState->GetPawn());
            }
        }
    }
};

```

## Debugging
**Show Replication Graph**: `Net.RepGraph.PrintGraph`

**Show Actor Replication Info**: `Net.RepGraph.PrintAllActorInfo`

© Samuel Daigle – Licensed under [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/). 