[Home](../../README.md) > [Multiplayer](../README.md) > [Anti-Cheat](README.md) > [Prevention methods](Prevention%20methods.md)
# Anti-Cheat prevention methods

## Stripping server code from client executables
Use precompilers to remove all analytics and server-only code from client builds. WITH_SERVER_CODE serves to strip code from Client executables. It is enabled in DedicatedServer, ListenServer, Standalone and Editor. You should only use UE_SERVER if the code is only available on dedicated servers. 

|            | WITH_SERVER_CODE | UE_SERVER |
|:-----------|:-----------------|:----------|
| Server     | 1                | 1         |
| Standalone | 1                | 0         |
| Client     | 0                | 0         |


Note that WITH_SERVER_CODE and UE_SERVER are not sufficient to guide server logic, you must you use appropriate [replication roles](../README.md#server-authoritative). This only works if you do not let player host their own dedicated server.

## Obfuscation
Aims to make it difficult for people who do not have access to the source code to understand memory values. Hacks require a greater effort to can gain access to decrypted data.

There are plenty of cyphers techniques online. You need to choose an algorithm that is safe enough to prevent hacking and fast enough for the game thread.

### XOR cipher
Use a seeded key to alter values. The seed should be randomized at every game version.

|    | Encryption                             |    |    |    | Decryption                             |
|:---|:---------------------------------------|:---|:---|:---|:---------------------------------------|
|    | 0000000010001011 (Value 139)           |    |    |    | 0000000001010010 (Encrypted Value 82)  |
| ^  | 0000000011011001 (Seeded Key 217 )     |    |    | ^  | 0000000011011001 (Seeded Key 217 )     |
| =  | 0000000001010010 (Encrypted Value 82 ) |    |    | =  | 0000000010001011 (Decrypted Value 139) |

## Code protection
Code obfuscation can help to make the code less readable from an external view, but several hacking techniques ignore all efforts put into code protection. Multiple of these techniques can be automated with defines.
- Renaming function, classes and variables.
- Alter the control flow with unnecessary loops, conditionals and jumps to confuse reverse engineers.
- Split critical logic into smaller functions, use FORCEINLINE to prevent hooking into setter and getters.
- Use custom data types and structures.
- Implement checks that detect debugging tools and breakpoint.
- Use PageGuard to have one-shot alarms for memory page access.
- Compress parts of the code and decompress at runtime.

## Randomized variable order
Use defines to randomize the order of variable within the define parameters. Makes the variable address more difficult to obtain.

## Remove sensitive data
The best way to prevent cheats is to have the least amount of available data on client. For example, you can move all authoritative data to the server and strip it from client executables. You can also limit the replication only when needed, like replicating enemy positions only when they are visible.

## Inhumane mouse movement detection
Store a short history of input deltas for the mouse movement and detect if changes are too consistent or too harsh.

## Alter colors
Since some cheats use color detection on the GPU render, then we can use modify colors in runtime and see if player performance are worsened when the colors are altered.

## Analytics
Once all technical implementation of anti-cheat are in place, the best option is to send analytics from dedicated servers when they detect abnormal behaviors from players. A single analytic call cannot determine if a player is cheating, but it provides enough data to analyze risks of a player cheating. The team can decide on an algorithm or manual selection with heuristics to manually ban player with good confidence.

See [analytic examples](README.md#types-of-cheats) per types of cheat.

© Samuel Daigle – Licensed under [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/). 