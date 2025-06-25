[Home](../README.md) > [Optimization](README.md) > [Memory Optimization](MemoryOptimization.md)

# Memory Optimization
| Problem                                 | Solution                                                             |
|------------------------------------------------|----------------------------------------------------------------------|
| Large texture memory usage                     | Compress, stream, reduce resolution, remove unused mips             |
| High-poly or numerous meshes                   | Use LODs, Nanite, merge small meshes                                |
| Excessive or uncompressed animations           | Compress, reduce keyframes, share skeletons                         |
| Heavy WAV or uncompressed audio assets         | Stream audio, compress to ADPCM or Vorbis                           |
| Hard references keeping assets loaded          | Use soft references, load manually                                  |
| All sublevels loaded at once                   | Use level streaming or World Partition                              |
| Too many unique materials or heavy shaders     | Reuse instances, pack ORM textures                                  |
| Editor-only data bloating memory               | Wrap in editor-only macros, strip unused assets                     |
| Unused or accidentally retained assets         | Audit references, unload manually                                   |


© Samuel Daigle – Licensed under [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/).  