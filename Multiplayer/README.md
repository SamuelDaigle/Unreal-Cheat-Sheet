[Home](../README.md) > [Multiplayer](README.md)  
Related Pages : [Replication Roles](Replication%20Roles.md) • [Lag Compensation](Lag%20Compensation.md) • [Network Optimization](Network%20Optimization.md) • [Replication Graph](ReplicationGraph.md) • [Anti-Cheat](Anticheat/README.md) • [Distributed Servers](Distributed%20Servers.md) • [Online & Backend](../Online/README.md)

# Multiplayer
Networking and multiplayer practices to develop bug-free server-authoritative multiplayer experiences.

### Sections
- [Replication Roles](Replication%20Roles.md)
- [Lag Compensation](Lag%20Compensation.md)
- [Network Optimization](Network%20Optimization.md)
- [Replication Graph](ReplicationGraph.md)
- [Anti-Cheat](Anticheat/README.md)
- [Distributed Servers](Distributed%20Servers.md)
- [Online & Backend](../Online/README.md)

## General Techniques
To create a multiplayer experience, you must ensure that you have :
- Good understanding on the networking architecture.
- Event-driven replication to asynchronously update game logic.
- Server-authoritative features : All player actions and game logic must run on server.
- Client prediction : Some features can be client-authoritative or run predictively on client.

Multiplayer models :
- Client-server : Centralizes game authority on the server, while clients send inputs and receive validated updates.
- Peer-to-peer : Peers communicate directly without an authoritative server, through a relay server.

## Networking architecture

### Components

| Component | Description |
| :- | :- |
| **GameMode<br />(Server)** | Server-only actor used for any backend communication and server logic (login, spawning and win/lose condition logics). Each level can only have one gamemode. You can either attach components to handle multiple gamemodes in the same level or you can use level parenting to override the gamemode. Keep it as lightweight as possible by attaching modular components or using Lyra game experience. |
| **GameState<br />(Server + Client)** | Mainly used to hold GameMode replicated variables and events that BOTH client and server has access to. Does not have any game logic. |
| **GameInstance<br/>(Server + Client)** | Non-replicated object that exists for the lifetime of the application. Used to hold values across multiple games et servers. Avoid using this for multiplayer games unless necessary, you can use seamless travel to keep data between level changes with the same server.
| **PlayerController<br />(Server + Owned Client)** | Represents a controllable player. On server, it has all bot controllers and a player controller for each connected client. On client, it only has the locally owned player controller. Takes the input data from the player and translating that into actions (movement, using items, firing weapons). It can possess a pawn for visual representation of the player. The player controller persists throughout the game, while the pawn can be destroyed. The player controller can be used to communicate between the local client and the server. You cannot and should not communicate directly with other players. |
| **Pawn<br />(Server + Client)** | Physical representation of a player within the world. Can be possessed by a player or AI controller, or it can be an unpossessed dummy. Pawns exist on all clients and on the server. Can extend inputs from the player controller to have pawn-specific inputs. It has components for running physics, movement and more. Can have data that represents the pawn, like its health. Note that the pawn can be destroyed when traveling to another world, disconnecting and reconnecting and more. These data will reset at that point. |
| **PlayerState<br />(Server + Client)** | Contains player data that persists during reconnects and world travel. Exists on all clients and server, but should not contain logic. Can be considered as an extension to the player controller to contain data (player name, score, assigned team). Depending on your game, you can store pawn data in the player state, like its health, to make it persist throughout the game. However, it is recommended to implement a PawnState within the player state to store pawn data because a player can change pawn during the game. |
| **HUD<br />(Client)** | Container for UMG widgets and used to draw low-level elements. Can be used to store data used by multiple widget instances or to allow cross-communication. To communicate to the server, you must pass through its owner, which is the local player controller. |

## Replication and Spawning network actors
Due to the nature of data replication, you need to expect some updates to happen later than desired. You cannot expect all actors to be spawned in within a few seconds and you should not run logic into Tick functions nor send data very frequently. Replication can vary severely based on the physical distance they are from the server, the internet speed they have, their computer's speed, the server's tick rate and more.

### How to test network lag
Testing multiplayer games in the editor will always have a very low ping because the server runs on the same machine as clients. You should enable Network Emulation in Editor Preferences > Level Editor > Play > Multiplayer Options > Enable Network Emulation, then set all incoming and outgoing latency to 500 for high ping. You can also set the Packet Loss Percentage to 10 to simulate real-world issues where some network packets gets lost and are never received when using unreliable RPCs.

### How to replicate data and events
There are two main ways to replicate information between clients and servers. You cannot and should not replicate from client to client.

| Method | Description |
| :- | :- |
| [Replicated Properties](https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-engine-uproperties)<br/>(Server&rarr;Client) | Using replicated properties is the best way to replicate server data to clients. They are always expected to be replicated to their latest value, even after joining a game or becoming net relevant. You can use `ReplicatedUsing` to receive events on client when a value has been replicated and you can add an argument to receive the previous value if needed. If you are updating a value frequently, you can alternatively use an unreliable RPC to update the value of a non-replicated property on clients. You can also send smaller data by sending delta updates, id mapping, using seeds for random generation or using smaller data like using `FVector_NetQuantize10` instead of `FVector`. |
| [RPC functions](https://dev.epicgames.com/documentation/en-us/unreal-engine/remote-procedure-calls-in-unreal-engine)<br/>(Client&rarr;Server) | Calls a function on all clients (NetMulticast), on a specific client (Client) or on the server instance (Server). This should almost only be used to send action reliable Server request from clients and to send unreliable updates to clients from the server. You cannot expect all that all clients will receive a reliable RPC when called by the server, because the actor may not be net relevant at that time.<br/>**Important** : You should avoid replicating game logic through RPCs because they will only be replicated to non-dormant and network-relevant actors on players that are already joined. Distant and late joining players will have out-of-sync states. |

### When to run client logic
You generally want to update client logic, data and widgets when receiving data through Property's `ReplicatedUsing` updates and when actors spawn. To reiterate, you should avoid sending RPCs to update client logics. For example, you can attach weapons in two ways:
- Set the weapon's owner with it's pawn when spawning it on the server. Then on client, you can attach the weapon to the pawn's skeleton mesh on BeginPlay if the pawn exists.
- Use a replicated property for the attached weapon on the player pawn, then when you receive the ReplicatedUsing function, you can attach the weapon to the pawn as you know both exists in that event.

## Server-authoritative
Server-authoritative means that the server has authority on all replicated objects and player actions. This offers the best multiplayer experience as it ensures consistent mechanics, simplifies variable replication and prevents cheating. You can use the server-authoritative approach for dedicated server, listen server and standalone games.

To build a server authoritative method, you must :
- Spawn all networked objects on the server, they will automatically spawn on net relevant clients.
- Send player action requests to the server through reliable RPCs. The server will validate if the action is valid (ability delays, player state, etc) and activate the ability server-side and replicate all necessary info to clients. You can use the RPC's validation to kick the player if invalid data gets received or to send cheat detection analytics.
- Run game logic on the gamemode. Since the gamemode is server-only, you make sure that a client can never directly modify the game logic. This way, you can have a timer and win conditions within that gamemode, then replicate any data through the game state.

Think of server as a singleplayer game that has access to everything, receives requested actions from clients and sends any necessary data to clients. On the other hand, clients have as little data as possible. They can only control their own player and ask the server to execute actions.

Since a replicated object spawned on server will also be spawned on client, unreal offers functions to make sure we run logic on the right client :

| Function | Description |
| :- | :- |
| `AActor::HasAuthority` | For executing code on Server or on client-spawned actors. See ROLE_Authority below.|
| `AActor::GetLocalRole` | For checking if local player possesses the actor (AutonomousProxy or SimulatedProxy), use HasAuthority for Authority role.<ul><li>**ROLE_Authority** : In charge of executing, validating and replicating to remote connections. This value is assigned on the server or client connection that spawned this actor. Use HasAuthority().</li><li>**ROLE_AutonomousProxy** : Replicated and possessed by client without authority. May receive inputs from a human controller and we can extrapolate based on local inputs. May want to rollback when extrapolated states become too different from replicated state</li><li>**ROLE_SimulatedProxy** : Replicated and not possessed by client. May want to extrapolate replicated data.</li><li>**ROLE_None** : Is not network relevant, never replicated.</li></ul>
| `AController::IsLocalController` | For checking if a controller is controlled locally. False on Server, except for bots and ListenServer’s controlled player controller.|
| `APawn::IsLocallyControlled` | See AController::IsLocalController. Is\[Pawn\|Bot\|Player\]Controlled serves on server to know if a pawn is controlled by anyone.|
| `AActor::IsNetMode`<br />`UWorld::IsNetMode` | For checking if we are running on dedicated server, listen server, client or standalone regardless of networking roles. Better not to use this function for most actors. Can be used in subsystems for separating client and server logic. IsNetMode is more optimized than comparing GetNetMode.<ul><li>**NM_DedicatedServer** : Server that runs headlessly. Does not render the game and starts with no local player.</li><li>**NM_ListenServer** : Client hosting the game on their machine and acting as the server, available to other players on the network.</li><li>**NM_Client** : Client connected to a remote server. Checking NetMode < NM_Client is always some variety of server. Clients do not have authority on most actor, except for locally spawned ones, but they can control some possessed actors when their local role is ROLE_AutonomousProxy.</li><li>**NM_Standalone** : Game without networking, with one or more local players. Still considered a server because it has all server functionality.</li></ul>|

See [Replication Roles](ReplicationRoles.md) to see all the values for each connection types.

## Replicating custom data
Sometimes, you need to replicate custom data that isn't possible to replicate through UPROPERTY. Using `<<` both serializes and deserializes the data. It's best use a temporary variable when modifying an array. `IsLoading()` checks if we are deserializing.
```cpp
bool NetSerialize(FArchive& Ar, UPackageMap* Map, bool& bOutSuccess)
{
    int32 BitArraySize = MyBitArray.Num();
        
    Ar << BitArraySize;
    if (Ar.IsLoading())
    {
        MyBitArray.Init(false, BitArraySize);
    }

    for (int32 Index = 0; Index < BitArraySize; ++Index)
    {
        bool BitValue = MyBitArray[Index];
        Ar << BitValue;

        if (Ar.IsLoading())
        {
            MyBitArray[Index] = BitValue;
        }
    }

    return true;
}
```

## Configuring Network Manager
```ini
[/Script/Engine.GameNetworkManager]
MaxClientRate=10000 ; Default 10000 (10KB/s). Max bandwidth before disconnecting a connection.
```

## References 
https://snapnet.dev/blog/performing-lag-compensation-in-unreal-engine-5/
https://vorixo.github.io/devtricks/network-managers/

© Samuel Daigle – Licensed under [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/). 