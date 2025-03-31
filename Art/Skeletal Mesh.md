[Home](../README.md) > [Art](README.md) > [Skeletal Mesh](Skeletal%20Mesh.md)
# Skeletal Mesh

### Reusing the same skeleton
You should have the least amount of skeleton possible. In general, you'll only want to use unreal's default skeleton as it is compatible with a lot of animation. You'll only want extra skeletons for non-humanoid skeletal meshes.

Shared skeleton techniques :
- Use [Skeleton retargeting](https://dev.epicgames.com/documentation/en-us/unreal-engine/animation-retargeting-in-unreal-engine#usingtheretargetmanager) to create a shared rig for different skeletons.
- Use [Animation retargeting](https://dev.epicgames.com/documentation/en-us/unreal-engine/animation-retargeting-in-unreal-engine) to reuse animation for skeletons that have a largely different proportion.
- Use [IK Retargeting](https://dev.epicgames.com/documentation/en-us/unreal-engine/retargeting-bipeds-with-ik-rig-in-unreal-engine) to retarget animation on vastly different skeletons and proportions.

### Creating a unreal compatible skeleton
There are multiple ways to create unreal compatible skeletons. The most common ways are by :
- Exporting the unreal mannequin as a FBX and modifying it.
- Using [AutoRig Pro](https://blendermarket.com/products/auto-rig-pro) automatically builds and exports compatible skeletal meshes. While it is a paid product, it is also the fastest option.
- Using [Rigify](https://docs.blender.org/manual/en/2.81/addons/rigging/rigify.html) simplifies the process of rigging skeletal meshes.

### Creating modular characters
You can parent separate body part skeletal meshes onto the main skeletal mesh, then call SetLeaderPoseComponent in the construction script.

There are [other ways](https://dev.epicgames.com/documentation/en-us/unreal-engine/working-with-modular-characters-in-unreal-engine) to achieve modular characters, like using the Copy Pose From Mesh system or the Skeletal Merging plugin.

### Attaching actors on skeletal meshes
Using sockets is the best way to attach actors on a skeleton while still support different skeleton layouts. It also allows to adjust offsets from the skeleton bones without affecting the skeleton and animations.

© Samuel Daigle – Licensed under [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/). 