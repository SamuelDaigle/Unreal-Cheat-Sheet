[Home](../README.md) > [Art](README.md) > [Material](Material.md)
# Material
## Types of maps
| Name | Uses |
| :--- | :--- |
| Albedo Map (RGB) | Defines the diffuse color |
| Normal Map (RGB) | Modified how light reflects. Greyscale bump map can be used with lesser results. |
| Height Map (R) | Store the height of the surface. Can be used for displacement, filling texture between height ranges. |
| Opacity Map (R) | Defines the transparency. Prefer modifying the mesh with extra vertices over adding transparency as it is expensive on the GPU. |
| Ambient Occlusion Map (R) | Contains lighting data to improve shadows. |
| Specular Map (R) | Contains lighting data to reflect more light. Also called reflection map. |
| 


## Techniques
https://dev.epicgames.com/documentation/en-us/unreal-engine/graphics-programming-for-unreal-engine
https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-engine-materials-tutorials

### Fresnel
Describe how the light you see reflects at different intensities based off of the angle you are viewing it from. Mainly used to have a inner outline to make a object more distinct from the background.
![image](images/Fresnel.png)

### Edge detection
Derivatives
![image](<images/EdgeDetection Simple.png>)

Sample average detection
![image](<images/EdgeDetection.png>)

### Halftone
Patterns of dots were used to create images generally used on postprocessing materials.
![image](images/Halftone.png)
Example of halftone combined with fresnel on an unlit surface material : 
![image](<images/Halftone with Fresnel.png>)

### Triplanar projection
https://www.youtube.com/watch?v=Cq5H59G-DHI
https://ronanmahon-art.gitbook.io/decal-designer/technical-reference/decal-features

### Line drawing

### Voronoid Diagram
Create random noise and diverse content :
- Terrain generation
- Natural appearance in materials : Cracks, texture variations, reflection, 
- Fraction meshes for destruction



### Height map
Use Lerp to fill textures in heightmap cracks or higher areas.


### Cel shading
https://www.youtube.com/watch?v=RkFwe7JI8R8

## Useful Functions
- WorldAlignedTexture : Repeated textures without stretching
- Saturate : Clamp from 0 to 1
- 

## Optimization

## References
["UE5 Post process materials examples: Edges, halftone, and more!" by Enrique Ventura](https://www.youtube.com/watch?v=cxcRLhoEcX0)

© Samuel Daigle – Licensed under [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/). 