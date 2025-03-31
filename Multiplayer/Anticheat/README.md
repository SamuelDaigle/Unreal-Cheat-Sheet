[Home](../../README.md) > [Multiplayer](../README.md) > [Anti-Cheat](README.md)  
Related Pages : [Prevention Methods](Prevention%20methods.md)
# Anti-Cheat

### Sections
- [Prevention Methods](Prevention%20methods.md)

## How cheats work
Hacks are built by exploiting any information given to their machine, whether it is from data, visual or sound. The goal is to send as little information as possible, make the required information more difficult to read and send analytics when detecting any potential abuse.

Review cheat forums for latest hacks :
- https://www.unknowncheats.me/forum/index.php
- https://www.mpgh.net/
- https://forum.cheatengine.org/
- https://github.com/atenfyr/UAssetGUI

### Reading memory
Some cheat engines allow to read all of the memory addresses that a game uses. Hackers first takes a capture of all current memory values, does a game action, then compares the memory values with a new capture. The software identifies which variables have changed and you can repeat this process until you pinpoint the exact memory offset used to get the memory address.

Prevention methods :
- [Obfuscation](<Prevention methods.md#obfuscation>)
- [Randomized variable order](<Prevention methods.md#randomized-variable-order>)
- [Remove sensitive data](<Prevention methods.md#remove-sensitive-data>)

### Reading GPU frame
Some hacks use the GPU output that would normally go to the screen. They read colors close to the crosshair to detect enemy colors, healthbars, nameplates or any similar visible identifier. Once they detect a potential identifier on the arduino-like peripheral, they send mouse inputs through a mouse-like peripheral which makes it almost impossible to detect as no software run on the computer.

More advanced hacks could use machine learning to detect human-like shapes and return the position and other hacks can run directly on AI monitors.

Prevention methods : 
- Inhumane mouse movement detection
- Alter colors
- Analytics

## Types of cheats

### Aimbot
Script that automatically aims at the enemy. Tweakable to make it look more legit and human-like.

Solutions :
- [Reading memory](#reading-memory)
- [Reading GPU frame](#reading-gpu-frame)
- Detect softwares running in background and hardware connected to the PC.
- Replay review system, like CSGO’s overwatch.
- Make suspected hackers play against each other through matchmaking.
- User reporting

Analytics :
- Detect delay between visibility and headshots.
- Detect headshot accuracy.
- Detect smooth aim : going in a straight, constant speed.
- Detect silent aim : low FOV instant teleport of aim.
- Detect whether grenade-like projectiles’ target destination always has an enemy present.

### ESP (Wallhack)
See actor locations through walls based on replication data (location, SFX, VFX). Can be used to detect players, items and abilities.

Solutions :  
- See [Aimbot](#aimbot) solutions.
- Replicate as little as possible to clients ([Valorant](https://technology.riotgames.com/news/demolishing-wallhacks-valorants-fog-war), [League of Legends](https://technology.riotgames.com/news/story-fog-and-war), [Valve](https://developer.valvesoftware.com/wiki/PVS)).
- Thorough automation testing that detects any data leaks.

Analytics : 
- Detect whether players frequently look directly towards enemies through walls.

### Triggerbot
Scripts that automatically do things for you, such as shoot when an enemy is in your crosshair, spam actions, automatically stop planting the bomb when an enemy is nearby, etc.

Solutions : 
- See [Aimbot](#aimbot) solutions.
- Add server-side gameplay limitations to prevent spamming.

Analytics : 
- Detect frequent failed action requests
- Detect actions when enemies start replicating, but are not yet visible.
- Detect actions made in rapid succession.

### Backtrack
Also called rewind, fake latency or interpolation abuse, it determines how far backtrack can shoot. Enabling fake latency increases in-game ping without impacting gameplay. For example, clients can look at past enemy locations and extrapolated locations to determine their position at a specific frame, then send a gunfire request with that specific frame.

This gives the illusion that you’ve been shot through a wall.

Solutions : 
- See [Aimbot](#aimbot) solutions.
- Add server-side limitations to the length and frequency of accepted backtracks.
- Limit accepted pings in matchmaking and during matches.

Analytics : 
- Detect frequent backtracks that hit enemy targets.
- Detect ping vs score abnormalities.

### Antiaim
Cheats that are designed to make enemies miss you : Spinbot, fake lags, look up to break hitboxes.

Solutions : 
- See [Aimbot](#aimbot) solutions.

Analytics : 
- Detect abnormal mouse movement and player rotation.

© Samuel Daigle – Licensed under [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/). 