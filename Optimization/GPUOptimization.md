[Home](../README.md) > [Optimization](README.md) > [GPU Optimization](GPUOptimization.md)
# GPU Optimization

| Shader | Optimization Tips |
|------------------|-------------------|
| Vertex Shader | <ul><li>Limit usages of WorldPositionOffset.</li><li>Overuse of VertexColor can be costly.</li></ul> |
| Pixel Shader | <ul><li>Use texture for lookups instead of math.</li><li>Compress greyscale maps into single textures. This is called RGB masked packing and it is best to standardize this across the project, like combining AO, roughness and mask to a single texture.</li><li>Minimize layer usage</li><li>Use FeatureLevelSwitch, QualitySwitch and Switch parameters to turn off what you don't need.</li></ul> |

| Problem | Solution |
|------------------|-------------------|
| Overdraw caused by transparency | <ul><li>Have less foliage, VFX and less fog sheets. Add more details and reduce quantity.</li><li>Minimize the geometry area for overdraw by cutting the mesh around the texture (have more vertices, but less transparency).</li><li>Use particle cutouts.</li></ul> |
| Managing texture resolution | <ul><li>Always use textures of power of 2.</li><li>Use Tools > Audit > Statistics then select Texture Stats to see the lower resolution mipmap used.</li><li>Reimport textures at lower resolutions or use Oodle.</li></ul> |
| Lighting performance | <ul><li>Stationary and dynamic lights are expensive, prefer using small unshadowed lights.</li><li>Minimize meshes affected by dynamic lights.</li><li>Bake static lights with lightmap res as low as you can (use view mode for blue color).</li><li>Use mesh distance field shadows and remove dense shadow cascades.</li><li>Use shadow bias to fix artifacts.</li><li>Avoid light functions (consider IES profiles) and avoid lit translucency.</li><li>Cull shadows and dynamic lights.</li><li>Spot lights are cheaper than point lights.</li><li>Use fake shadows (capsule or blob shadows).</li></ul> |
| High materials count | Use Material Instances whenever possible. It reduces shader compile time, reuses code and slightly improves performance. Value changes to material instances doesn’t require recompile time. Have parent materials with generic values (overridable in material instances), one for each type of object or reused features. |
| High material instruction count | Keep material instruction counts low. For example, characters can have a more complex material with 350 instructions, but you could have a budget of 150 for environments. |
| High polygon count | Lower polys in a modeling software or use LODs. Do not remove faces randomly, 3d meshes can be placed with different orientations. |

© Samuel Daigle – Licensed under [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/).  