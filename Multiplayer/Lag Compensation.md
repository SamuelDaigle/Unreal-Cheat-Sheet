[Home](../README.md) > [Multiplayer](README.md) > [Lag Compensation](Lag%20Compensation.md)
# Lag Compensation
Lag compensation works by maintaining historical game states and "rewinding" the game to a previous state to process actions, ensuring accurate gameplay despite network delays.

The complexity of multiplayer arises when network latency (ping) is high. For example, if a client sends a server request to shoot and the response takes 1 second to reach the server, the server will spawn the projectile 1 second after the player presses the mouse button. Then, if the "spawn projectile" replication takes another second to propagate to all clients, the player will see the projectile appear only after 2 seconds.

## Client-side prediction
To make gameplay feel smoother, client prediction is used. The client performs the same validations as the server (ability delays, player state, etc), then spawn the projectile locally and send the server request within the same frame. This allows the client to see the projectile immediately, while the server processes the action and spawns a replicated projectile for all clients. For the client triggering the action, two projectiles will be spawned: one locally and one replicated.

### On Autonomous Proxy (controlled actors)
There are two common approaches to handle this: 
- **Quick and easy** : Override `AActor::IsNetRelevantFor` and return `!IsOwnedBy(RealViewer)` to not have the projectile replicate that the owned client. Other clients will have the replicated projectile that spawns X milliseconds later and the local client will have locally created and non-replicated projectile that spawned instantly. You cannot replicate important server data through the projectile actor with that approach.
- **Prediction Key** : Create prediction key when creating the locally spawned projectile and send that key to the server when requesting the action. The server will spawn a replicated projectile and store the prediction key in it. Once the client spawns the replicated projectile, you can check if an existing projectile has this prediction key and delete the locally spawned projectile to replace it with the replicated one. You can apply any modification to the replicated projectile, such as lerping from the old projectile to the new one to provide a smoother experience. It is recommended to use [Gameplay Ability System's prediction key](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Plugins/GameplayAbilities/FPredictionKey) if you are using this plugin.

### On Simulated Proxy (remote actors)
**Interpolation** smooths the transition between the client's current state and the replicated state received from the server to reduce visual jitter and create fluid motion. Interpolation is generally only done on client, but it can be used on server during rollback to achieve a sub-tick state.

**Extrapolation** can be used to predict future positions when the next server data isn't available yet. It assumes assumes that the object will continue moving in the same direction or following the same behavior. **Dead Reckoning** is an extrapolation that focuses on movement (position, velocity, direction) and **Entity Prediction** focuses on states (health, data, state machine, logic variables).

## Server-side Rollback  
Server rollback allows the server to rewind time and validate past player actions, ensuring accurate hit registration and collision detection in high-latency environments. It works by storing historical entity states at a fixed tick rate and restoring them when needed. If precise sub-tick accuracy is required, interpolation between stored states can reconstruct positions at fractional timestamps.

### Key Concepts
**Historical State Storage** : The server records entity positions, rotations, and other relevant data every tick.  
**Rewinding** : When a client action (e.g., a shot) needs validation, the server restores the game state at the exact time the action was issued. Interpolates between recorded ticks for more precise sub-tick validation.  
**Reprocessing Actions** : The server simulates the action in the past state to determine if it was valid. Detect collision overlaps, raycasts and logic states.  
**State Restoration** : After validation, the server resets entities to their latest state.

### Tick rate
Server tick rate, used for serializing historical states, can be increased or decreased depending on the game requirements. In general, you'll want to follow these guidelines.
- 30 Hz : Lower CPU and bandwidth usage, suitable for slower-paced games (RTS, turn-based).
- 60 Hz : Standard for most action games, providing smooth responsiveness without excessive CPU/network load.
- 120 Hz+ : Used in high-precision, competitive games like fighting games and FPS, reducing input latency and improving rollback accuracy.  

You can implement sub-tick interpolation reduce the need for higher tick rate. Separating tick responsibilities (replication vs state serialization) can also be made to balance optimization and gameplay.

### Plugins and references
Mover Plugin : https://portal.productboard.com/epicgames/1-unreal-engine-public-roadmap/c/1281-character-mover-2-0-experimental

Network Prediction Plugin : https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Plugins/NetworkPrediction

Client-Side Prediction and Reconciliation: https://www.gabrielgambetta.com/client-side-prediction-server-reconciliation.html

## Desync and Client-side Reconciliation
When client-side prediction are too far from replicated server values, desync happens.

**1) Desync detection** : The client checks if the server’s confirmed position matches the stored local state for the same sequence number.  
**2) Reconciliation** : Rewind to the last confirmed state from the server, then reapply prediction for all stored unconfirmed inputs.


### Unconfirmed Inputs
**Unconfirmed inputs** are inputs sent to the server but not yet acknowledged. Separate from the historical state, both client and server should store input states with sequence numbers.  They are used for prediction and reconciliation. Input states can also be tracked on server to ensure reprocessing inputs during rollbacks.

## Input Latency
**Input Latency** can be intentionally added to synchronize all players' inputs, ensuring smoother gameplay. When a player's input is received, it can be stored and executed at the same future frame for all clients, reducing desynchronization. This approach improves predictive reactions and fluid animation, but makes it feel less responsive since input presses are not instant. Using an hybrid solution combining short input latency and higher timing for rollback is a great solution for fast-paced games where every frame count for reactions.

Mortal Kombat and Injustice input latency explained : https://youtu.be/7jb0FOcImdg?t=544


© Samuel Daigle – Licensed under [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/). 