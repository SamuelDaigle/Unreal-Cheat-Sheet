[Home](../README.md) > [Gameplay](README.md)
# Gameplay
## Gameplay Ability System
References :
- Full documentation : https://github.com/tranek/GASDocumentation
- Pros & cons : https://vorixo.github.io/devtricks/gas/
- Q&A : https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89

### Main components
| Component | Description |
| :-------- | :---------- |
| AbilitySystemComponent (ASC) | Actor component to handle all Gameplay Ability System features. You must have an ASC on the actor if you want to bind attributes, apply gameplay effects and give abilities. |
| GameplayAttributes | You can assign Attribute Sets on an ASC to have multiple Attribute (Health, Damage, Cash, Stamina) on that actor. Attributes have a BaseValue and a CurrentValue that can be modified with Gameplay Effects. |
| GameplayAbility | Represents a server-authoritative or predicted player action for an actor. Server can give and remove abilities in runtime. Generally used to apply gameplay effects. Can use AbilityTask to make abilities latent actions over multiple frames. |
| GameplayEffect | Reliably modifies an actor's attributes or gameplay tags to change an actor state. |
| GameplayCue | Unreliably executes non-gameplay effects (SFX & VFX). |
| GameplayTag | Used to describe a state of an object or an action. |

### Setup
The Gameplay Ability System plugin can be overwhelming at first, but after the initial setup is done, it gets easier to use it and keeps the project scalable and maintainable. I'd recommend downloading the [project sample](https://github.com/tranek/GASDocumentation) to view basic examples on how it is set up.

### General Tips
- Use [DataRegisteries](https://dev.epicgames.com/documentation/en-us/unreal-engine/data-registries-in-unreal-engine) to get datatable elements by gameplay tag.
- Create AbilitySystemComponent async on non-player actors. Each active ASC uses a lot of memory and CPU, so if you can spawn in ASC only when an actor first becomes damaged, whether it's for destructible environment or for numerous enemy AI.
- Use FastReplication config to send cues by index instead of by name.

© Samuel Daigle – Licensed under [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/). 